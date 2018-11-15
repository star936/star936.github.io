---
title: Elasticsearch(四)文档索引
date: 2018-11-15 20:54:28
tags: [Elasticsearch, 索引]
categories: 大数据
---

##### 1. 建立索引

```bash
PUT /users
{
  "settings": {
    "index": {
      "number_of_shards": 5,
      "number_of_replicas": 1
    }
  }
}
```

##### 2. 修改索引

**索引的主分片数在索引创建之后就不能再修改，默认是5；副本分片是可以随时修改的。**

```bash
PUT /users/_settings
{
  "index": {
    "number_of_replicas": 2
  }
}
```

* number_of_replicas: 设置索引的副本分片数
* blocks.read_only: 如为true，则索引只能读，不能写和更新
* blocks.read: 如为true，则禁止读取操作
* blocks.write: 如为true，则禁止写操作
* blocks.metadata: 如为true，则禁止对metadata操作

##### 3. 查看索引配置信息

```bash
GET /users/_settings
# 查看多个索引信息
GET /users, roles/_settings
# 使用通配符查看索引信息
GET /use*/_settings
# 查看所有索引信息
GET /_all/_settings
```

##### 4. 删除索引

```bash
DELETE /users
```

##### 5. 插入数据

*向索引名为users的索引插入一条type为user的数据*

```bash
# 由es自动生成id值
POST /users/user/
{
  "name": "Alan",
  "message": "create an example user",
  "created_at": "2018-11-05T11:00:00"
}
# 创建时指定id值
PUT /users/user/1
{
  "name": "Bourne",
  "message": "create an example user",
  "created_at": "2018-11-05T12:00:00"
}
```

##### 6. 使用映像mapping

```bash
# 通过mapping设置index中某个type下的field中的详细信息
PUT /test
{
  "mappings": {
    "example": {
      "properties": {
        "user": {
          "type": "text",
          "index": "false"
        }
      }
    }
  }
}
# 或者
PUT /test/example/_mapping
{
  "example": {
    "properties": {
      "message": {
        "type": "text",
        "index": "false"
      }
    }
  }
}
```

**注意：es6.x以后移除了string类型，另外index的值只能时boolean了。**

##### 7. 获取映像信息

```bash
GET /test/_mapping/example
# 获取特定field的信息
GET /test/_mapping/example/field/user
```

##### 8. 管理索引文件

```bash
# 打开
POST /users/_open
# 关闭
POST /users/_close
# 检测状态
HEAD /users
# 清除缓存
POST /users/_cache/clear
```

##### 9. 配置分析器

```bash
# 在索引users中基于standard分析器创建一个名为es_std的分析器
PUT /users/_settings
{
  "analysis": {
    "analyzer": {
      "es_std": {
        "type": "custom",
        "tokenizer": "standard"
      }
    }
  }
}

# 使用es_std进行分词
GET /users/_analyze
{
  "analyzer": "es_std",
  "text": "joson bourne"
}
```

##### 10. 获取文档信息

```bash
GET /users/user/1
# source过滤器
GET /users/user/1?_source=false
# 获取特定字段的值
GET /users/user/1?_source=name
```

##### 11. 删除文档信息

```bash
DELETE /users/user/1
```

##### 12. 更新文档信息

```bash
# 第一种
POST /users/user/1/_update
{
  "doc": {
   "message": "bourne" 
  }
}
# 第二种
POST /users/user/1/_update
{
  "script": "ctx._source.message = \"Bourne1\""
}
```

   **向数组添加元素**

```bash
POST /users/user/1/_update
{
  "script" : {
    "source": "ctx._source.tags.add(params.new_tag)",
    "params" : {
      "new_tag" : "world"
   }
  }
}
```

**增加新字段**

```bash
POST /users/user/1/_update
{
  "script" : {
    "source": "ctx._source.new_field=params.new_field",
    "params" : {
      "new_field" : "world"
   }
  }
}
```

**根据ID值找不到文档，则通过参数体中的upsert创建这个文档，并且加入新的字段；否则更新字段值。**

```bash
POST /users/user/2/_update
{
  "script" : {
    "source": "ctx._source.count+=params.count",
    "params" : {
      "count" : 4
   }
  },
  "upsert": {
     "counter": 1
   }
}
```

##### 13. 批量获取文档

**通过指定`_index`、`_type`、`_id`获取不同索引不同类型的文档**

```bash
POST /_mget
{
  "docs": [
    {
      "_index": "users",
      "_type": "user",
      "_id": "1"
    },
    {
      "_index": "roles",
      "_type": "role",
      "_id": "1"
    }
    ]
}
```

**获取同一索引同一类型下的文档，可以通过指定`ids`即可**

```bash
POST /users/user/_mget
{
  "ids": ["1", "2"]
}
```
