---
title: ES操作得分
date: 2024-01-30 09:20:02
categories: elasticsearch
description: "ES的得分研究"
---

## 可以修改的

`POST /index_name/_search?explain=true` 每一查询结果就有计算的过程，_explanation节点为计算过程

## 相关度基础查询

```
{
    "query": {
        "match": {
        "title.hanLP": "测试文章"
        }
    }
}
```
默认查询形式

![默认图片](ES相关度得分原理研究/%E9%BB%98%E8%AE%A4%E5%BE%97%E5%88%86%E7%BB%93%E6%9E%9C.png)


