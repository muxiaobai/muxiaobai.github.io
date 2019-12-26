---
title: mongodb常用命令
date: 2019-01-11 03:07:27
tags: [mongo,常用命令]
categories: 数据库
description: "mongo 的常用数据库命令总结"
---

常用的mongo语句，汇总，以便之后查阅。
<!--more-->


## 全局、数据库级别

默认根目录：/db/mongodb-3.4.7/
### 导入导出
--导入
/db/mongodb-3.4.7/bin/mongorestore --host=127.0.0.1 -u test_user -p test_pwd --port 27017 --db jy_db /db/mongo_bak/test_db

--强制删除
--drop：先删除所有的记录，然后恢复.
/db/mongodb-3.4.7/bin/mongorestore --host=127.0.0.1 -u test_user -p test_pwd --port 27017 --db test_db --drop /db/mongo_bak/test_db

--导出
/db/mongodb-3.4.7/bin/mongodump --host 127.0.0.1 --port 9430 -d test_db -u test_user -p 123456 -o /db/mongo_bak/test_db

/db/mongodb-3.4.7/bin/mongodump --host 127.0.0.1 --port 9430 -d test_db  -o /db/mongo_bak/test_db

--打包
tar -czvf  /db/mongo_bak/jy_db20181207.tar.gz /db/mongo_bak/test_db

--导入
/db/mongodb-3.4.7/bin/mongodump --host 127.0.0.1 --port 9430 -d test_db -u test_use -p test_pwd -o /db/mongo_bak/

### 启动 登录，创建用户

修改配置文件
`vi /db/mongodb-3.4.7/mongodb.conf `
常用配置：

```

后台启动
`/db/mongodb-3.4.7/bin/mongod -f /db/mongodb-3.4.7/mongodb.conf `
登录：
`/db/mongodb-3.4.7/bin/mongo  --host 127.0.0.1 --port 9430 -u root -p FpTH2a`

show dbs;//展示所有数据库

use test_db;//创建数据库.切换到本数据库

/db/mongodb-3.4.7/bin/mongod -f /db/mongodb-3.4.7/mongodb.conf  --shutdown

db.createUser({user:'name',pwd:'123456',roles: [{ role:"readWrite",db:"name"}]});
# 写密码校验方式
db.createUser({user:"name",pwd:"123456",roles:[{role:"readWrite",db:"name"}],mechanisms : ["SCRAM-SHA-1"]})

db.getCollection('system.users').find({})

db.getCollection('system.users').remove({user:"name"});

```

查询用户：

### 数据库

use test_db;//

db.repairDatabase();//释放空间

db.runCommand({"distinct":"fs.files", "key":"metadata.type"});//所有的集合

## 集合级别 collection

- 所有  show collections;
- 创建  db.createCollection("test");
- 添加  db.getCollection("test").insert({"name":"test"});
- 清空  db.getCollection("test").remove({});
- 删除  db.getCollection("test").drop();
- 查询  db.getCollection("test").find({"name":"test"});
