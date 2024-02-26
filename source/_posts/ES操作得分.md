---
title: ES操作得分
date: 2024-01-30 09:20:02
categories: elasticsearch
description: "ES的得分研究"
---

## 可以修改的位置

有三种方式介入

similarity_boost--->functionscore --->(sort) rescore 

###  similarity_boost
similarity_boost更改索引字段匹配的算法，默认BM25 可以自定义设置。

设置setting的时候可以设置 similarity，然后么一个properties里面的字段可以设置对应的算法
，此时控制的是一个

```
{
    "settings": {
        "index": {
            "number_of_shards": 1,
            "number_of_replicas": 0,
            "max_result_window": 10000,
            "refresh_interval": "30s",
            "similarity": {
                "scripted_tfidf": {
                    "type": "scripted",
                    "script": {
                        "source": "double tf = Math.sqrt(doc.freq); double idf = Math.log((field.docCount+1.0)/(term.docFreq+1.0)) + 1.0; double norm = 1/Math.sqrt(doc.length); return query.boost * tf * idf * norm;"
                    }
                },
                "scripted_boost": {
                    "type": "scripted",
                    "script": {
                        "source": " return query.boost ;"
                    }
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                "fields": {
                    "char": {
                        "type": "text",
                        "analyzer": "ngram_analyzer",
                        "similarity": "scripted_boost"
                    },
                    "raw": {
                        "type": "keyword",
                        "ignore_above": 10000,
                        "similarity": "scripted_boost"
                    },
                    "hanLP": {
                        "type": "text",
                        "analyzer": "hanlp_index_analyzer",
                        "search_analyzer": "hanlp_standard_analyzer",
                        "similarity": "scripted_tfidf"
                    }
                }
            },
            "titleHanLPBoost": {
                "type": "text",
                "analyzer": "hanlp_index_analyzer",
                "search_analyzer": "hanlp_standard_analyzer",
                "similarity": "scripted_boost"
            },
            "createDate": {
                "type": "date",
                "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            },
            "id": {
                "type": "keyword",
                "ignore_above": 256
            }
        }
    }
}

```
此处的similarity 可以使用 doc,field,weight,query等变量写script,作用与每一个搜索词在词字段的得分，

举例 ，如果一个搜索词在此字段拆分为2个term的时候，呢么在此字段上的得分是这两个term的得分和作为此字段的得分，


[similarity](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-similarity-context.html)

此时


### functionscore

functionscore 中可以使用前一步的query中的得分，中间可以使用 params、doc,和_score 中的数据来计算functionscore。

```
{"query": {
      "function_score": {
        "query": {
          "bool": {
            "must": [
              {
                "bool": {
                  "should": [
                    {
                      "match": {
                        "title.hanLP": {
                          "query": "测试",
                          "operator": "OR",
                          "prefix_length": 0,
                          "max_expansions": 50,
                          "minimum_should_match": "1",
                          "fuzzy_transpositions": true,
                          "lenient": false,
                          "zero_terms_query": "NONE",
                          "auto_generate_synonyms_phrase_query": true,
                          "boost": 5
                        }
                      }
                    }
                  ],
                  "adjust_pure_negative": true,
                  "boost": 1
                }
              }
            ],
            "filter": [
              {
                "bool": {
                  "adjust_pure_negative": true,
                  "boost": 1
                }
              }
            ],
            "adjust_pure_negative": true,
            "boost": 1
          }
        },
        "functions": [
          {
            "filter": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "title.hanLP": {
                        "query": "测试",
                        "operator": "OR",
                        "prefix_length": 0,
                        "max_expansions": 50,
                        "minimum_should_match": "100%",
                        "fuzzy_transpositions": true,
                        "lenient": false,
                        "zero_terms_query": "NONE",
                        "auto_generate_synonyms_phrase_query": true,
                        "boost": 1
                      }
                    }
                  },
                  {
                    "range": {
                      "publishTime": {
                        "from": "2014-01-15 09:40:31",
                        "to": "2024-01-15 09:40:31",
                        "include_lower": true,
                        "include_upper": true,
                        "boost": 1
                      }
                    }
                  }
                ],
                "adjust_pure_negative": true,
                "boost": 1
              }
            },
            "script_score": {
              "script": {
                "lang": "painless",
                "source": "if(doc['title.raw']==params['factor']){return 1001;} long base = 400; if(doc['title.raw'].contains(params['factor'])){base = 500;} long day = (System.currentTimeMillis()- doc['createDate'].value.toInstant().toEpochMilli())/1000/60/60/24;  double runSourceScore = 0;boolean flag=false;  if(!flag){ if(params['types'][doc['type.keyword'].value]!=null){runSourceScore +=params['types'][doc['type.keyword'].value];}} double score = _score; if(_score<50){ score=50;} return base + runSourceScore + (-0.01*day) ;",
                "params": {
                  "types": {
                    "a": 3,
                    "b": 2,
                    "c": 41,
                    "d": 1
                  },
                  "status": {
                    "1": 40,
                    "2": 10,
                    "3": 41,
                    "4": 4,
                    "5": 1
                  },
                  "factor": "测试",
                  "base": 400
                }
              }
            }
          }
        ],
        "score_mode": "first",
        "boost_mode": "replace",
        "max_boost": 3.4028235e+38,
        "boost": 1
      }
    }
}
```
functionscore里面是一个filterScript数组 在满足filter的基础上执行script_score，作为返回值然后使用score_mode决定是用第一个，boost_mode判断和query查出来的_score怎么计算。


[Score context](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-score-context.html)
在此时返回的_score 作为es的返回相关度得分

### sort

sort 分层

返回的数据中有多个排序结果

```
  "sort": [
        1.0,
        1.0
    ],
```

如果有多个sort，则每一条数据返回的时候就有多个值，然后根据sort的顺序返回，

### rescore


如果有functionscore 则是这之后返回的_score，rescore 作用于_score 之后的，可以按照window_size大小部分调节

此处rescore 结合了functionscore，只使用一层window_size 来调整最后的得分，，实际上可以使用多层，呢样就无法准确的设置单条数据的权重，因为每一个都可能加不同得分数。

```
{
    "rescore": [
        {
            "window_size": 10000,
            "query": {
                "rescore_query": {
                    "function_score": {
                        "query": {
                            "function_score": {
                                "query": {
                                    "bool": {
                                        "adjust_pure_negative": true,
                                        "boost": 1
                                    }
                                },
                                "functions": [
                                    {
                                        "filter": {
                                            "range": {
                                                "publishTime": {
                                                    "from": "2014-01-16 18:08:58",
                                                    "to": "2024-01-16 18:08:58",
                                                    "include_lower": true,
                                                    "include_upper": true,
                                                    "boost": 1
                                                }
                                            }
                                        },
                                        "script_score": {
                                            "script": {
                                                "lang": "painless",
                                                "source": "if(doc['title.raw']==params['factor']){return 1001;} long base = 400; if(doc['title.raw'].contains(params['factor'])){base = 500;} long day = (System.currentTimeMillis()- doc['createDate'].value.toInstant().toEpochMilli())/1000/60/60/24;  double runSourceScore = 0;boolean flag=false;  if(!flag){ if(params['types'][doc['type.keyword'].value]!=null){runSourceScore +=params['types'][doc['type.keyword'].value];}} double score = _score; if(_score<50){ score=50;} return base + runSourceScore + (-0.01*day) ;",
                                                "params": {
                                                "types": {
                                                    "a": 3,
                                                    "b": 2,
                                                    "c": 41,
                                                    "d": 1
                                                },
                                                "status": {
                                                    "1": 40,
                                                    "2": 10,
                                                    "3": 41,
                                                    "4": 4,
                                                    "5": 1
                                                },
                                                "factor": "测试",
                                                "base": 400
                                                }
                                            }
                                        }
                                    },
                                    {
                                        "filter": {
                                            "match": {
                                                "title.hanLP": {
                                                    "query": "测试",
                                                    "operator": "OR",
                                                    "prefix_length": 0,
                                                    "max_expansions": 50,
                                                    "minimum_should_match": "100%",
                                                    "fuzzy_transpositions": true,
                                                    "lenient": false,
                                                    "zero_terms_query": "NONE",
                                                    "auto_generate_synonyms_phrase_query": true,
                                                    "boost": 1
                                                }
                                            }
                                        },
                                        "script_score": {
                                            "script": {
                                                "source": "return 200",
                                                "lang": "painless"
                                            }
                                        }
                                    },
                                    {
                                        "filter": {
                                            "match_phrase": {
                                                "title.char": {
                                                    "query": "测试",
                                                    "slop": 0,
                                                    "zero_terms_query": "NONE",
                                                    "boost": 1
                                                }
                                            }
                                        },
                                        "script_score": {
                                            "script": {
                                                "source": "return 100",
                                                "lang": "painless"
                                            }
                                        }
                                    },
                                    {
                                        "filter": {
                                            "term": {
                                                "title.raw": {
                                                    "value": "测试",
                                                    "boost": 1
                                                }
                                            }
                                        },
                                        "script_score": {
                                            "script": {
                                                "source": "return 300",
                                                "lang": "painless"
                                            }
                                        }
                                    }
                                ],
                                "score_mode": "sum",
                                "max_boost": 3.4028235e+38,
                                "boost": 1
                            }
                        },
                        "functions": [],
                        "score_mode": "sum",
                        "max_boost": 3.4028235e+38,
                        "boost": 1
                    }
                },
                "query_weight": 1,
                "rescore_query_weight": 1,
                "score_mode": "max"
            }
        }
    ]
}


```
