---
title: Elasticsearch(八)搜索优化
date: 2018-11-22 21:45:28
tags: [Elasticsearch]
categories: 大数据
toc: true
---

> Elasticsearch: 6.4.2
#### 1. 理解字段分析过程

*一个常被问到的问题是，为什么指定的文档没有被搜索到。很多情况下，这都归因于映射的定义和分析例程的配置存在问题。针对分析过程的调试，Elasticsearch提供了专用的REST API。*

```bash
GET /_analyze
{
  "analyzer": "standard",   # 可以替换成自定义的analyzer
  "text": "crime and publishment"
}
# 使用一个分词器和两个过滤器 
GET _analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["html_strip"],
  "text" : "this is a <b>test</b>"
}
```

#### 2. 解释查询

*给出有关文档相关度得分计算的详细信息，以解释为什么会匹配成功。*

```bash
# 针对特定文档的分析
GET /cars/doc/8/_explain?q=jeep
# 针对文本信息的分析
GET _analyze
{
  "tokenizer" : "standard",
  "filter" : ["snowball"],
  "text" : "detailed output",
  "explain" : true,
  "attributes" : ["keyword"] 
}
```

#### 3. 用加权查询影响得分

Boost(权值)：在计算得分过程中使用的附加权值，可在如下位置使用：

* 查询：告诉搜索引擎指定的查询比其他查询更重要
* 字段：指定文档中某些字段对用户很重要
* 文档：某些文档很重要

##### 3.1 在查询中使用权值

*例如：*

```bash
# 字段重要程度：from、to、subject
GET /emails/doc/_search
{
  "query": {
    "query_string": {
      "fields": ["from^5", "to^3", "subject"], 
      "query": "Bourne"
    }
  }
}
```

##### 3.2 修改得分

**constant_score**

*包装另一个查询子句，为每个文档返回得分等于boost值。*

```bash
GET /cars/doc/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "name": "bmw jeep"
        }
      },
      "boost": 1.2
    }
  }
}
```

**Boosting查询**

*Boosting查询与Bool查询中的NOT的不同：后者过滤掉满足NOT查询语句的文档，而前者仍然选择不被期望的文档，只是讲它们的得分降低。*

```bash
GET /cars/doc/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match_all": {}
      },
      "negative": {
        "term": {
          "name": {
            "value": "jeep"
          }
        }
      },
      "negative_boost": 0.2
    }
  }
}
```

**Function查询**

*boost参数设置每个文档的得分，然后对于满足functions参数中子查询条件的文档根据boost_mode、boost、weight计算新得分；*

*当存在多个function且某个文档满足多个function时该文档的得分为：先计算满足各个function后的每个得分，然后由score_mode参数计算最终等分。其中score_mode取值：multiply、sum、avg、first、max、min。*

```bash
GET /cars/doc/_search
{
  "query": {
    "function_score": {
      "boost": 5,
      "max_boost": 50,
      "score_mode": "max",
      "boost_mode": "multiply",
      "query": {
        "match_all": {}
      },
      "functions": [
        {
          "filter": {
            "match": {"name": "jeep"}
          },
          "weight": 4
        }
      ]
    }
  }
}
```

#### 4. 具有相同含义的词

##### 4.1 同义词(synonym)过滤器

*基于数组定义*

```bash
PUT /test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "synonym" : {
                        "tokenizer" : "standard",
                        "filter" : ["my_stop", "synonym"]
                    }
                },
                "filter" : {
                    "synonym" : {
                        "type" : "synonym",
                        "lenient": true,
                        "synonyms" : ["foo, bar => baz"]
                    }
                }
            }
        }
    }
}
```

*基于文件定义：文件路径是相对于Elasticsearch安装目录下config目录的。*

```bash
PUT /test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "synonym" : {
                        "tokenizer" : "whitespace",
                        "filter" : ["synonym"]
                    }
                },
                "filter" : {
                    "synonym" : {
                        "type" : "synonym",
                        "synonyms_path" : "analysis/synonym.txt"
                    }
                }
            }
        }
    }
}
```

##### 4.2 同义词定义规则

*默认使用Apache solr的同义词方案。*

**同义词显式定义**

```bash
i-pod, i pod => ipod,
sea biscuit, sea biscit => seabiscuit
```

**同义词等式定义**

```bash
ipod, i-pod, i pod
foozball , foosball
universe , cosmos
lol, laughing out loud
```

**扩展同义词**

*如果同义词过滤器中属性expand=true，则所有同义词被扩展为所有单词全部等价的形式。*

```bash
ipod, i-pod, i pod => ipod, i-pod, i pod
```

#### 5. 跨度查询

*跨度是指在一个字段中开始和结束的词条位置。*

**span_term**

```bash
GET /_search
{
    "query": {
        "span_term" : { "user" : "kimchy" }
    }
}
```

**span_multi**

*包装一个term、range、prefix、wildcard、regexp或fuzzy查询。*

```bash
GET /_search
{
    "query": {
        "span_multi":{
            "match":{
                "prefix" : { "user" :  { "value" : "ki" } }
            }
        }
    }
}
```

**span_first**

*只允许返回在字段的前几个位置上匹配查询条件的文档。*

```bash
# 查询在user字段前三个位置出现kimchy的文档
GET /_search
{
    "query": {
        "span_first" : {
            "match" : {
                "span_term" : { "user" : "kimchy" }
            },
            "end" : 3
        }
    }
}
```

**span_near**

*可以在有多个其他跨度彼此接近时对文档进行搜索，该查询也是一个能将其他跨度查询包装起来的复合查询。*

```bash
# slop：控制在跨度之间允许的词项的数量
# in_order: 限制匹配顺序；true时按照查询定义的顺序匹配文档
# 该示例查询message字段包含world everyone的文档。
GET /_search
{
    "query": {
        "span_near" : {
            "clauses" : [
                { "span_term" : { "message" : "world" } },
                { "span_term" : { "message" : "everyone" } }
            ],
            "slop" : 0,
            "in_order" : true
        }
    }
}
```

**span_or**

```bash
# 获取在message字段前两个位置处有world或者离everyone不超过一个位置处含有world的文档。
GET /_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_first": {
            "match": {
              "span_term": {
                "message": {
                  "value": "world"
                }
              }
            },
            "end": 2
          }
        },
        {
          "span_near": {
            "clauses": [
              {
                "span_term": {
                  "message": {
                    "value": "world"
                  }
                }
              },
              {
                "span_term": {
                  "message": {
                    "value": "everyone"
                  }
                }
              }
            ],
            "slop": 1,
            "in_order": true
          }
        }
      ],
      "slop": 0,
      "in_order": true
    }
  }
}
```

**span_not**

*include参数指定了哪个跨度查询应该被匹配；exclude参数指定了不与include部分重叠的跨度查询。*

```bash
# 返回在message字段中匹配了由breaks词项构造的span_term查询的所有文档，然后再定义一个匹配了world和everyone并且最大位置间距为1的跨度，当该跨度与第一个跨度查询重叠时，排除掉所有重叠部分的文档。
GET /_search
{
  "query": {
    "span_not": {
      "include": {
        "span_term": {
          "message": {
            "value": "breaks"
          }
        }
      },
      "exclude": {
        "span_near": {
          "clauses": [
            {
              "span_term": {
                "message": {
                  "value": "world"
                }
              }
            },
            {
              "span_term": {
                "message": {
                  "value": "everyone"
                }
              }
            }
          ],
          "slop": 1
        }
      }
    }
  }
}
```

**span_containing**

*查询内部有big和little两个字查询，仅返回big匹配的结果中包含little匹配结果的文档；从大到小包含*

```bash
GET /_search
{
    "query": {
        "span_containing" : {
            "little" : {
                "span_term" : { "field1" : "foo" }
            },
            "big" : {
                "span_near" : {
                    "clauses" : [
                        { "span_term" : { "field1" : "bar" } },
                        { "span_term" : { "field1" : "baz" } }
                    ],
                    "slop" : 5,
                    "in_order" : true
                }
            }
        }
    }
}
```

**span_within**

*与span_containing类似，从小到大包含。*

```bash
GET /_search
{
    "query": {
        "span_within" : {
            "little" : {
                "span_term" : { "field1" : "foo" }
            },
            "big" : {
                "span_near" : {
                    "clauses" : [
                        { "span_term" : { "field1" : "bar" } },
                        { "span_term" : { "field1" : "baz" } }
                    ],
                    "slop" : 5,
                    "in_order" : true
                }
            }
        }
    }
}
```



