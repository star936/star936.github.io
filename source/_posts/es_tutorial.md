---
title: Elasticsearch(三)入门使用-索引与映射
date: 2018-11-14 21:11:28
tags: Elasticsearch
categories: 大数据
---

> Elasticsearch: 6.4.2

#### 1. 索引

*索引是Elasticsearch中存储数据的一种逻辑结构。与关系型数据库对比：索引index类似于表，文档document类似于表中的行，字段field类似于表中的列，映射mapping类似于schema。索引是有分片组成的，可以分散到集群的多个节点中；分片实际上是Lucene的索引。*

*在Elasticsearch6.x中一个索引只能包含一种类型的文档。*

##### 1.1 索引操作

创建Index

```bash
PUT /twitter
```

获取Index

```bash
GET /twitter
```

删除Index

```bash
DELETE /twitter
```

Index是否存在

```bash
HEAD /twitter
```

打开/关闭Index

```bash
POST /twitter/_close
POST /twitter/_open
```

##### 1.2 索引别名

*索引别名是一个或多个缩影的另外一个名字。每个别名可以对应多个索引，每个索引也可以有多个别名。例如：我们将每天的日志创建一个索引，所有这些日志可以有有一个统一的索引名`log`；另外我们还可以将每年的日志的索引名再统一到一起起个别名`log2018`。*

**不能使用已存在的索引名字作为别名。**

创建别名

```bash
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
```

将索引从一个别名中删除

```bash
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
```

获取所有别名

```bash
GET /_aliases
```

过滤别名

*Elasticsearch支持以类似SQL数据中使用视图的方式使用别名。用户可以使用完整的查询DSL，并将查询应用在基于查询的统计、搜索、删除等操作上。*

*确保在映射中已存在该字段。*

```bash
# 创建能够为客户12345返回数据的别名，则当使用该别名时，所有操作返回文档的clientId字段的值都是12345
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "data",
                 "alias" : "client",
                 "filter" : { "term" : { "clientId" : "12345" } }
            }
        }
    ]
}
```

别名和路由选择

*对于命名为client的索引别名，使用12345、12346、12347作为路由值用于索引，并仅将12345用于查询*

```bash
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "data",
                 "alias" : "client",
                 "index_routing" : "12345,12346,12347",
                 "search_routing": "12345"
            }
        }
    ]
}
```

*当使用client别名对数据进行索引时，index_routing属性指定的值将被使用；当进行查询时，search_routing属性指定的值被使用*

```bash
# search_routing和查询中routing参数的共同值12345作为路由值
GET /client/_search?q=test&routing=99999,12345
```



#### 2.  映射

##### 2.1 映射操作

*映射Mapping用于定义索引Index的结构。*

> 6.0版本之后_all可能不再支持，因此最好使用copy_to



**copy_to**: 允许创建定制的_all字段，将多个字段的值拷贝到查询时被当作单一字段使用的一组字段中。

**_source**: 包含文本被索引阶段的原始JSON文档内容，它不能被索引，因此也不能被搜索，只在fetch请求时返回数据。

创建或更新Mapping

```bash
PUT /twitter/_mapping/_doc 
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```

获取Mapping

```bash
GET /twitter/_mapping/_doc
# 类型为_doc的所有索引的映射
GET /_mapping/_doc
GET /_all/_mapping/_doc
# 所有索引所有类型的映射
GET /_all/_mapping
GET /_mapping
```

获取某字段的Mapping

```bash
GET /twitter/_mapping/_doc/field/email
```

##### 2.2 分析器

**通常情况下索引和搜索时应该使用相同的分析器，针对使用分析器搜索特定字段，查找顺序如下：**

* 当前查询语句中指定的分析器
* 映射中`search_analyzer`参数定义的分析器
* 映射中`analyzer`参数定义的分析器
* 在索引settings部分`default_search`参数定义的分析器
* 在索引settings部分`default`参数定义的分析器
* 标准分析器

*自定义分析器时需要在映射中添加一个settings部分，它保存创建索引时所需的信息。需要：*

* 零个或多个字符过滤器
* 一个分词器
* 零个或多个词过滤器

```bash
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type":      "custom", 
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}

```

##### 2.3 动态映射和模版

*Elasticsearch最重要的特性之一就是动态映射，你不要先创建索引、定义映射，可以直接索引一个文档，此时elasticsearch自动帮你做这些工作。*

*每一个模版定义了一个模式，用来与新建索引的名称进行比较；如果匹配则模版中定义的值被复制到索引结构的定义中。如果有多个模版匹配上了新建索引名称，所有模版都会被应用，并且后应用的模版会覆盖先应用的模版。其中的order参数可以控制模版的预期使用顺序。*

```bash
PUT my_index
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        {
          "longs_as_strings": {
            "match_mapping_type": "string",
            "match":   "long_*",
            "unmatch": "*_text",
            "mapping": {
              "type": "long"
            }
          }
        }
      ]
    }
  }
}
```

##### 2.4 路由

**怎么索引和搜索**

*默认情况下，elasticsearch计算文档ID的hash值及`hash值%主分片数`的值,基于后者的值将文档放到某个可用的主分片中，然后复制到副分片。*

*搜索时将请求分发到索引的所有分片中，然后在每一个分片中根据条件查找文档并传递给某一个节点，该节点进行全局的过滤和排序等，并将最终结果返回给客户端。*

路由选择可以控制文档和查询被转发到某一个特定的分片上。既可以在索引和搜索请求时使用routing参数，也可以在类型定义时使用_routing参数。

```bash
PUT /twitter/_doc/1?routing=kimchy
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

PUT my_index2
{
  "mappings": {
    "_doc": {
      "_routing": {
        "required": true,
        "path": "id"
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
  "properties": {
    "interests": {
      "type": "text",
      "fielddata": true
    }
  }
}
```

