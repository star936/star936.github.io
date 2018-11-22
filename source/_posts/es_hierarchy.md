---
title: Elasticsearch(九)数据关联
date: 2018-11-22 21:45:28
tags: [Elasticsearch]
categories: 大数据
toc: true
---

> Elasticsearch: 6.4.2 


#### 1. 索引树形结构

*创建简单映射*

```bash
PUT /categories
{
  "settings": {
    "analysis": {
      "analyzer": {
        "path_analyzer": {
          "tokenizer": "path_hierarchy"
        }
      }
    }
  },
  "mappings": {
    "category": {
      "properties": {
        "content": {
          "type": "text",
          "fields": {
            "path": {
              "analyzer": "path_analyzer",
              "type": "text",
              "store": true
            }
          }
        }
      }
    }
  }
}
```

*测试*

```bash
GET /categories/_analyze
{
  "field": "content.path",
  "text": "/home/ubuntu/work"
}
# 输出：/home、/home/ubuntu、/home/ubuntu/work
```

#### 2. 嵌套对象

*创建映射*

```bash
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "user": {
          "type": "nested" 
        }
      }
    }
  }
}
```

*索引数据*

```bash
PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

*搜索*

```bash
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "white" }} 
          ]
        }
      }
    }
  }
}
```

#### 3. 父子关系

* **6.0之后父子结构使用join类型来定义，关系的映射使用relations字段来指定。**
* **一个索引中只能有一个join类型字段。**
* **建好索引之后, join字段中的relations集合，可以增加映射、或者给原有的映射添加child，但是不能删除原有的映射。**
* **父文档和子文档存在同一个分片上，因此查询、更新、删除子文档时应该使用routing参数。**

*创建映射*

```bash
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "my_join_field": { 
          "type": "join",
          "relations": {
            "question": "answer" 
          }
        }
      }
    }
  }
}
```

*创建父文档*

```bash
PUT my_index/_doc/1?refresh
{
  "text": "This is a question",
  "my_join_field": "question" 
}
```

*创建子文档*

```bash
PUT my_index/_doc/3?routing=1&refresh 
{
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer", 
    "parent": "1" 
  }
}
```

*根据parent_id查询属于某一特定父文档的子文档*

```bash
GET /my_index/_search
{
  "query": {
    "parent_id": {
      "type": "answer",
      "id": "1"
    }
  }
}
```

*基于父文档查找子文档*

```bash
GET my_index/_search
{
    "query": {
        "has_parent" : {
            "parent_type" : "question",
            "query" : {
                "match" : {
                    "text" : "This is"
                }
            }
        }
    }
}
```

*基于子文档查找父文档*

```bash
GET my_index/_search
{
"query": {
        "has_child" : {
            "type" : "answer",
            "query" : {
                "match" : {
                    "text" : "This is question"
                }
            }
        }
    }
}
```

*Parent-join查询和聚合*

* 查询父文档id=1，子文档类型为answer的子文档；
* 基于父文档类型question进行聚合；
* 基于指定的field处理。

*如果一个文档是子文档，则`my_join_field#question`包含的是该子文档指向的父文档的id值；如果是父文档，则是该文档的id值。*

```bash
GET my_index/_search
{
  "query": {
    "parent_id": { 
      "type": "answer",
      "id": "1"
    }
  },
  "aggs": {
    "parents": {
      "terms": {
        "field": "my_join_field#question", 
        "size": 10
      }
    }
  },
  "script_fields": {
    "parent": {
      "script": {
         "source": "doc['my_join_field#question']" 
      }
    }
  }
}
```

