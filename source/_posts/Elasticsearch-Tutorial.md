---
title: Elasticsearch基础入门
date: 2018-11-05 14:54:28
tags: Elasticsearch
categories: 大数据
---

> 本文参照[Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

##### 1. 基本概念

索引(名词)：一个`索引`类似于关系型数据库中的的一个数据库，是一个存储关系型文档的地方。它的复数词为`indices`或`indexes`.

索引(动词)：`索引`一个文档就是存储一个文档到一个`索引(名词)`中以便它可以被检索和查询到。

倒排索引：关系型数据库通过增加一个 *索引* 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 *倒排索引* 的结构来达到相同的目的。

##### 2. 基础操作

###### 2.1、索引文档

**文档类型为`employee`类型，该类型位于索引`megacorp`内。**

```bash
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

*结果：*

```json
{
    "_index": "megacorp",
    "_type": "employee",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 2,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



###### 2.2、检索文档

**执行`GET`请求并指定文档的地址——索引库、类型和ID。**

```bash
GET /megacorp/employee/1
```

*结果：*

```json
{
    "_index": "megacorp",
    "_type": "employee",
    "_id": "1",
    "_version": 1,
    "found": true,
    "_source": {
        "first_name": "John",
        "last_name": "Smith",
        "age": 25,
        "about": "I love to go rock climbing",
        "interests": [
            "sports",
            "music"
        ]
    }
}
```

###### 2.3、轻量搜索

**搜索所有的雇员信息**

```bash
GET /megacorp/employee/_search
```

**另外，可以使用一个高亮搜索，该方法涉及到一个查询字符串搜索，例如：**

查询last_name为Smith的雇员

```bash
GET /megacorp/employee/_search?q=last_name:Smith
```

###### 2.4、使用查询表达式搜索

*查询last_name为Smith的雇员*

```bash
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

*更加复杂的搜索：搜索last_name为Smith，同时年龄大于30岁的雇员。使用到filter过滤器*

```bash
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```

###### 2.5、全文搜索

*搜索所有喜欢rock climbing的雇员*

```bash
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

*结果：*

**Elasticsearch默认按照相关性得分排序，即每个文档跟查询的匹配度.**

```json
{
    "took": 19,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 0.5753642,
        "hits": [
            {
                "_index": "megacorp",
                "_type": "employee",
                "_id": "1",
                "_score": 0.5753642,
                "_source": {
                    "first_name": "John",
                    "last_name": "Smith",
                    "age": 25,
                    "about": "I love to go rock climbing",
                    "interests": [
                        "sports",
                        "music"
                    ]
                }
            },
            {
                "_index": "megacorp",
                "_type": "employee",
                "_id": "2",
                "_score": 0.2876821,
                "_source": {
                    "first_name": "Jane",
                    "last_name": "Smith",
                    "age": 32,
                    "about": "I like to collect rock albums",
                    "interests": [
                        "music"
                    ]
                }
            }
        ]
    }
}
```

###### 2.6、短语搜索

*搜索同时匹配`rock`和`climbing`并且二者以短语`rock climbing`形式紧挨着的雇员记录*

```bash
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

###### 2.7、高亮搜索

*在每个搜索结果中高亮部分文本片段，以便让用户知道为何该文档符合查询条件.*

```bash
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

###### 2.8、分析---聚合

*最受欢迎的兴趣爱好：*

```bash
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

*聚合还支持分级汇总。比如：查询特定兴趣爱好员工的平均年龄：*

```bash
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

**可能发生的错误：`Fielddata is disabled on text fields by default`**.

解决方法：

*开启fileddata*

```bash
# 格式：/{index}/_mapping/{type};其中index：指存储文档的index； type：指文档的类型。
PUT /megacorp/_mapping/employee
{
    "interests": {
        "type": "text",
        "fielddata": true
    }
}
```

