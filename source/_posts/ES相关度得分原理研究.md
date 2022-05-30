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

```
{
    "query": {
        "match": {
        "title.hanLP": "测试文章"
        }
    }
}
```

```
 "_explanation" : {
          "value" : 16.725555, 两个词的和
          "description" : "sum of:",
          "details" : [
            {
              "value" : 8.399242, 其中一个词的得分 测试
              "description" : "weight(title.hanLP:测试 in 24) [PerFieldSimilarity], result of:",
              "details" : [
               ...
               ]
            },
            {
              "value" : 8.326312, 另外一个词 文章
              "description" : "weight(title.hanLP:文章 in 24) [PerFieldSimilarity], result of:",
              "details" : [
                ...
              ]
            }
          ]
        }
```
默认的是求和

### term和match 一样会有得分

### multi_match

如果多个字段的话，根据type和tie_breaker来决定，需设置 type 默认情况使用 所有的字段得分加和。best_fields & tie_breaker

```
{
  "query": {
    "multi_match": {
      "query": "测试文章",
      "fields": [
        "title",
        "content^2.0"],
        "type": "best_fields",
        "tie_breaker": 0
    }
  }
}
```



max plus 0.3 times others of: 在多个字段时候档类型为 best 的时候，在设置了 tie_breaker 的时候使用 max + others * 0.3,为该字段的最后得分。

### filter是不影响得分的

如果无需相关度，直接用filter过滤,一般为固定条件。

## 调整得分结果

### 通过方法


#### 替换 相乘 相加


```
{
  "query": {
    "function_score": {
    "query": {
      "match": {
          "content": {
                "query": "测试",
                "boost": 2  
          }
      }
    },
    "field_value_factor": {
        "field":    "createDate",
        "modifier": "log1p" ,
        "factor": 1.0
      }
    }
  }
}
```
field_value_factor 

old_score * log(1+createData)*factor = new_score,
同时可以设置 factor,和field的权重类似.
```

"value" : 221.86592,
"description" : "sum of:",
"details" : [
{
  "value" : 221.86592,
  "description" : "function score, product of:",
  "details" : [
    {
      "value" : 18.19208,
      "description" : "weight(content.hanLP:测试 in 909) [PerFieldSimilarity], result of:",
      
    },
    {
      "value" : 12.195743,
      "description" : "min of:",
      "details" : [
        {
          "value" : 12.195743,
          "description" : "field value function: log1p(doc['createDate'].value * factor=1.0)",
          "details" : [ ]
        },
        {
          "value" : 3.4028235E38,
          "description" : "maxBoost",
          "details" : [ ]
        }
      ]
    }
  ]
}
```

## 添加常见的影响因子

### 时间因子影响（日期型）

```
"gauss" :{
    "timestamp": {
      "origin": "now timestamp",
      "offset": "5d",
      "scale": "10d"
    }
  }
```

距离现在5天都为1，之后10天逐渐递减


```
这种是使用服务端直接_scripts, 推荐
POST _scripts/calculate-create-score
{
  "script":{
    "source": "if (doc['createDate'].size()==0 || doc['createDate'].size()==0) {return 0.5;} else{  long year = 1000L * 3600 * 24 * 365; double gaussScale =  1.5 * year ; long gaussOffset = year * 0; double gaussDecay = 0.5; double sigmaSquare = - (Math.pow(gaussScale, 2) / (2 * Math.log(gaussDecay)));double gaussMultiplier = Math.exp(-1 * Math.pow(Math.max(0, Math.abs(params['now'] - doc['createDate'].value.toInstant().toEpochMilli()) - gaussOffset), 2) / 2 / sigmaSquare);  return gaussMultiplier; }",
    "lang": "painless"
  }  
}

POST 
{
  "from": 0,
  "size": 4,
  "timeout": "60s",
  "query": {
    "function_score": {
      "query": {
        
        "match": {
           "content": {
                  "query": "测试",
                  "boost": 2  
                }
        }
      },
      "functions": [
        {
          "filter": {
            "match_all": {
              "boost": 1
            }
          },
          "script_score": {
            "script": {
              "id":"calculate-create-score",
              "params": {
                "now":1651728368608
              }
            }
          }
        }
      ]
    }
  },
  
  "track_total_hits": 2147483647
}



String scoreScript = "if (doc['createDate'].size()==0 || doc['createDate'].size()==0) {return 1;} else{  " +
"long year = 1000L * 3600 * 24 * 365; double gaussScale =  1.5 * year ; long gaussOffset = year * 0; " +
"double gaussDecay = 0.5; double sigmaSquare = - (Math.pow(gaussScale, 2) / (2 * Math.log(gaussDecay)" +
"));double gaussMultiplier = Math.exp(-1 * Math.pow(Math.max(0, Math.abs(params['now'] - doc['createDate'].value.toInstant().toEpochMilli()) - gaussOffset), 2) / 2 / sigmaSquare);  return gaussMultiplier; }";
Map map = new HashMap();
map.put("now", new Date().toInstant().toEpochMilli());

```
添加了一个高斯函数来进行影响 0-1之间，最近一年的相关度最高，时间越远，影响越小。和上面的类似。


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

