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


### 仅查询一个字段

默认的得分为 score(freq=1.0), computed as boost * idf * tf from

boost 为查询时候的boost*2.2，此处2.2还不清楚为什么要这么设置。

> idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:

n 当前文档包含的分词数，N 总共的文档数 

> tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:

freq 为该文档上出现的所查词的频数。一些默认值 k1=1.2 b=0.75,
d1 当前文档长度 avgdl 平均文档长度

此处查询出来是一个字段在一个词上的得分。

然后默认时候，比如检索查询的词是 "测试文章"，会出现 两个结果 然后相加得出最终得分。


### 如果match了两个分词的时候会在这个字段上相加两个词的和


### term和match 一样会有得分

### multi_match


max plus 0.3 times others of: 在多个字段时候档类型为 best 的时候，在设置了 tie_breaker 的时候使用 max + others * 0.3,为该字段的最后得分。

### filter是不影响得分的

如果无需相关度，直接用filter过滤,一般为固定条件。

## 调整得分结果

### 通过方法


#### 替换 相乘 相加



### 访问记录影响（数字型）

使用log1p函数
```
      "field_value_factor": {
        "field":    "votes",
        "modifier": "log1p"
      }
```
将field_value_factor设为对数计算，计算公式：new_score = old_score * log(1 + number_of_votes)。

### （坐标型）

```
{
  "gauss": {
    "lat_lon": { 
      "origin": { "lat": 51.5, "lon": 0.12 },
      "offset": "2km",
      "scale":  "3km"
    }
  }
},
```
距离中心点( "lat": 51.5, "lon": 0.12),offset 2km内的都是1.0，之后每相差scale 3km得分逐渐减小。

