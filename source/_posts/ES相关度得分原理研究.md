---
title: ES相关度得分原理研究
date: 2022-05-17 09:20:02
categories: elasticsearch
description: "ES的得分研究"
---


## 常规查询结果

`POST /index_name/_search?explain=true`

每一查询结果就有计算的过程

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

## 相关度

![默认图片](ES相关度得分原理研究/%E9%BB%98%E8%AE%A4%E5%BE%97%E5%88%86%E7%BB%93%E6%9E%9C.png)



#### 数学公式


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


