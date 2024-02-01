---
title: ES性能问题
date: 2024-01-30 09:20:02
categories: elasticsearch
description: "ES研究"
---


## 线程池 

es 改 内存大小 8g  内核 24核 search process数(3*24/2+1) 37 队列数 2000

429 rejected TOO_MANY_REQUESTS   主要就是线程池和队列 无法响应

GET /_cat/thread_pool?v=true&h=id,node_name,name,active,queue,rejected,completed


[ES 线程池配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)

## 优化逻辑-方向

主要就是在

## 分片

索引 一个分片 ,一个segement
PUT index/_settings
{
  "index.blocks.write":"false"
}
POST index/_shrink/index_shrink 

不要副本
PUT /index_shrink/_settings
{
  "settings":{
    "number_of_replicas":0
  }
}

#### 变为一个段

POST index_shrink/_forcemerge?max_num_segments=1

GET /_cat/segments/index_shrink?v
GET index_shrink/_segments


#### 减少字段 减少IO

POST /_reindex
{
  "source": {
    "index": "index",
    "_source": ["title","createDate","id"],
    "query": {
      "exists": {"field": "createDate"}
    }
  },
  "dest": {
    "index": "index_similarity_boost"
  },
  "script": {
    "source": " ctx._source.title = ctx._source.title;"
  }, 
  "size":100000000
}



- [Elasticsearch慢查询故障诊断](https://www.cnblogs.com/zhq1007/p/11744102.html)
- [Elasticsearch 磁盘使用率超过警戒水位线](https://www.cnblogs.com/hahaha111122222/p/16048771.html)
- [让你的ES查询性能起飞：Elasticsearch 查询优化攻略“一网打尽”](https://zhuanlan.zhihu.com/p/647279604?utm_id=0)

