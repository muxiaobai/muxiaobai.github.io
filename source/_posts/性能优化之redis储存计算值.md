---
title: 性能优化之redis储存计算值
date: 2019-02-16 04:17:43
tags: redis
categories: 性能优化
description: 'redis存储值解决性能问题'
---

## 问题

两张表，一个主表，一个树形结构表，

每一条树形结构数据都有一个状态，要统计出所有叶子节点的状态，展示在主表的列上，前期使用了oracle的`start connect`统计根节点，在前期主表中的数据量很小的时候，列表价在正常，后期数据量增大，列表响应速度异常。后经过分析，得出每一条根节点循环的时候需要600ms左右，然后数据增大直接是主表的600ms*n。

树形结构超过三级，前两级结构超过50条记录，

## 优化路思路：

redis优化查询结果，因为是计算结果，这种数据，在数据库上执行第一次,随后再次查询列表的时候从缓存中获取，减少计算次数。

## 解决方法

#### 查询

查询的时候先从redis中查询，如果有直接返回；如果没有，再查数据库，


```
//查询状态,有num直接返回:

if(Redis.isCached(key, FORP.SPRING_CONTEXT.getBean("applicationPool", JedisPool.class))){
	return Integer.parseInt(Redis.getString(key, FORP.SPRING_CONTEXT.getBean("applicationPool", JedisPool.class)));
}

//查询出的结果
String sql =  " select count(*) as num "
			+ " from pre_dutysubitem t1 ,pre_dutyitem t2 "
			+ " where t2.fk_dutyid = ?  "
			+ " and (select  count(*) from pre_dutysubitem t3 "
			+ " start with t3.id = t1.id "
			+ " connect by prior t3.id =t3.fk_parentid ) = 1 "
			+ " and t2.id=t1.fk_dutyitemid and t1.currstatus is not null ";
SqlRowSet rs = null;

//如果currStatus没值查统计全部子项数量，否则按状态统计
if(StringUtils.isNotBlank(currStatus)){
	sql+=" and t1.currstatus = ?  ";
	rs = jdbc.queryForRowSet(sql, dutyId, currStatus);
}else{
	rs = jdbc.queryForRowSet(sql, dutyId);
}
int num = 0;
while(rs.next()){
	num =rs.getInt("num");
}
//缓存查询结果
Redis.cacheString(key, String.valueOf(num), FORP.SPRING_CONTEXT.getBean("applicationPool", JedisPool.class));

return num;
```

#### 修改状态

修改状态的时候，先删除redis缓存，再更改数据库状态，可能会出现刚删除，数据库还没来得及更改，又有用户查询，导致redis缓存脏数据。

```
//更新数据状态
//删除缓存状态：

Redis.delete(key, FORP.SPRING_CONTEXT.getBean("applicationPool", JedisPool.class));
```
