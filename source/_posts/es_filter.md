---
title: Elasticsearch(六)过滤器
date: 2018-11-14 23:56:28
tags: [Elasticsearch, 过滤器]
categories: 大数据
---

* Bool filter

  *有`must`、`should`、`must_not`三种逻辑操作;其中当仅存在should时则必须至少满足一个条件.*

  ```bash
  GET /books/book/_search
  {
    "query": {
      "bool": {
        "must": [
          {
            "match": {
              "name": "python"
            }
          }
        ],
        "should": [
          {
            "match": {
              "title": "effective"
            }
          }
        ]
      }
    }
  }
  ```

* Exists filter

 *`exists filter`过滤搜索结果,使其必须存在某个指定的字段.*

 ```bash
# exists filter
GET /books/book/_search
{
  "query": {
    "exists": {
      "field": "name"
    }
  }
}
 ```

**Null值的讨论:**

*假如索引`test`中存在类型为`test`的如下文档:*

```bash
{"user": ""} # 1
{"user": []} # 2
{"user": null} # 3
{"user": [null]} # 4
{"user": "-"} # 5
{"user": "foo"} # 6
{"person": "bar"} # 7
```

*对于如下的查询,将会返回`1、5、6`三条数据.*

*默认情况下:`2、3、4、7`四种情况都被认为是`null`值而被过滤,最终不可被搜索.*

```bash
GET /test/test/_search
{
  "query": {
    "exists": {
      "field": "user"
    }
  }
}
```

*可以在创建索引前,通过设置`mapping`中`null_value`属性的值,从而让`3、4`这两种情况文档可被搜索.*

* Type filter

  *返回指定type的文档.*

  ```bash
  GET /books/_search
  {
    "query": {
      "type": {
        "value": "book"
      }
    }
  }
  ```

* Match_all filter

  *选中全部数据,相当于SQL语句中的*`select * from book`.

  ```bash
  GET /books/_search
  {
    "query": {
      "type": {
        "value": "book"
      }
    }
  }
  ```

* Query filter

  *由`AND`、`OR`两种逻辑*

  ```bash
  GET /books/book/_search
  {
    "query": {
      "query_string": {
        "default_field": "name",
        "query": "python OR java"  # "python AND java"
      }
    }
  }
  ```

* 结果排序

  *默认按照相关度得分`_score`倒排,还可以通过`sort`针对特定字段正排(`asc`)和倒排(`desc`);另外还可以在`sort`中指定`_last`将无值的结果放在检索集的最后.*

  ```bash
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
          "order": "desc",
          "missing": "_last"
        }
      }
    ]
  }
  ```
