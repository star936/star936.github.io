---
title: Elasticsearch(五)信息检索
date: 2018-11-15 20:56:28
tags: [Elasticsearch, 检索]
categories: 大数据
---

##### 1. 简单检索

**由于自Elasticsearch6开始不再支持一个索引下存在多个类型的操作，因此也就没有了查询同一索引下多个类型的信息操作。**

*from、size分别指定了从哪个结果开始返回、查询的结果集包含的最大文档数(默认是10)*

* 查询指定索引指定类型下的信息

  ```bash
  GET /users/user/_search?q=name:bourne
  ```

* 查询多个或者所有索引，指定类型或多个类型下的信息

  ```bash
  GET /users,test/user/_search?q=name:bourne
  GET /users,test/user, example/_search?q=name:bourne
  GET /*/user/_search?q=name:bourne
  ```

* 查询所有索引所有类型下的信息

  ```bash
  GET /_search?q=name:bourne
  ```

##### 2. 基本检索

* 设置不同字段的排序权重

  *会影响文档的相关性得分，从而影响排序结果。*

  ```bash
  GET /users/user/_search
  {
    "query": {
      "multi_match": {
        "query": "bourne",
        "fields": ["name^2", "message"]
      }
    }
  }
  ```

* 指定返回的字段子集

  ```bash
  GET /users/user/_search
  {
    "query": {
      "match_all": {}
    },
    "_source": ["name", "message"]
  }
  ```

* term查询、terms查询、wildcard查询

  *term查询: 仅匹配在给定字段有某个此项的文档，查询中的词项不再被解析属于精确查询;另外可以增加boost属性提升改term查询的重要性.*

  *wildcard查询: 允许在要查询的内容中使用通配符(*****和**？**)`*

  ```bash
  # term查询
  GET /users/user/_search
  {
    "query": {
      "term": {
        "name": {
          "value": "bourne"
        }
      }
    }
  }
  # terms查询
  GET /users/user/_search
  {
    "query": {
      "terms": {
        "name": [
          "anna",
          "bourne"
        ]
      }
    }
  }
  # wildcard查询
  GET /users/user/_search
  {
    "query": {
      "wildcard": {
        "name": {
          "value": "b*"
        }
      }
    }
  }
  ```

* match查询、match_all查询、match_phrase查询

  *match_phrase查询: 短语查询，可以通过指定slop参数定义在查询文本的词项之间应间隔多少个未知单词才视为成功,slop默认为0.*

  ```bash
  # match查询
  GET /users/user/_search
  {
    "query": {
      "match": {
        "name": "bourne"
      }
    },
    "sort": [
      {
        "created_at": {
          "order": "desc"
        }
      }
    ],
    "size": 20
  }
  # match_all查询
  GET /users/user/_search
  {
    "query": {
      "match_all":{}
    },
    "sort": [
      {
        "created_at": {
          "order": "desc"
        }
      }
    ],
    "size": 20
  }
  # match_phrase：对于查询文本the wife of Bourne;
  # 如果slop不大于0，那么不会匹配成功.
  GET /users/user/_search
  {
    "query": {
      "match_phrase": {
        "message": {
          "query": "wife bourne",
          "slop": 1
        }
      }
    }
  }
  ```

* Prefix 、Range查询

  *prefix查询：查找某个字段以给定前缀开头的文档，支持boost属性影响排序结果*

  ```bash
  # prefix查询
  GET /users/user/_search
  {
    "query": {
      "prefix": {
        "name": {
          "value": "j"
        }
      }
    }
  }
  # range查询：10天前至当前时间的文档
  GET /users/user/_search
  {
    "query": {
      "range": {
        "created_at": {
          "gte": "now-10d",
          "lte": "now"
        }
      }
    }
  }
  ```

* 跨字段查询multi_match

  ```bash
  GET /users/user/_search
  {
    "query": {
      "multi_match": {
        "query": "baby user",
        "fields": ["name", "message"]
      }
    }
  }
  ```