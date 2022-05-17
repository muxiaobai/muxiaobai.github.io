---
title: ES相关度得分原理研究
date: 2022-05-17 09:20:02
categories: elasticsearch
description: "ES的得分研究"
---

## 常规查询结果

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
默认查询，

![默认图片](ES相关度得分原理研究/%E9%BB%98%E8%AE%A4%E5%BE%97%E5%88%86%E7%BB%93%E6%9E%9C.png)


### 仅查询一个字段

默认的得分为 score(freq=1.0), computed as boost * idf * tf from

boost 为查询时候的boost*2.2，此处2.2还不清除为什么要这么设置。

idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:

n 当前文档包含的分词数，N 总共的文档数 

tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:

freq 为该文档上出现的所查词的频数。一些默认值 k1=1.2 b=0.75,
d1 当前文档长度 avgdl 平均文档长度

此处查询出来是一个字段在一个词上的得分。

然后默认时候，比如检索查询的词是 "测试文章"，会出现 两个结果 然后相加得出最终得分。



#### match

```
{
    "query": {
        "match": {
        "title.hanLP": "测试"
        }
    }
}
```

#### term

#### multi_match



###  在多个字段得分时候


需设置 type 默认情况使用 所有的字段得分加和。


### best_fields & tie_breaker

```

"type": "best_fields",
"tie_breaker": 0.3,

```

max plus 0.3 times others of: 在多个字段时候档类型为 best 的时候，在设置了 tie_breaker 的时候使用 max + others * 0.3,为该字段的最后得分。

## 调整得分结果

#### 替换 相乘 相加


```
{
  "query": {
    "match": {
      "title.hanLP": "测试"
    }
    , "function_score": {
      "query": {},
      "functions": [
        {}
      ]
    }
  }
}
```
## 添加常见的影响因子

#### 时间因子影响

#### 访问记录影响


