---
title: Elasticsearch(七)信息检索与结果过滤
date: 2018-11-15 23:56:28
tags: [Elasticsearch, Aggregations]
categories: 大数据
---


> Elasticsearch: 6.4.2


**聚合分类：**

*Bucketing聚合: 类似SQL中的GROUP BY；基于检索构成了逻辑文档组，满足特定规则的文档放置到一个桶里，每一个桶关联一个key; 分桶聚合可以嵌套分桶聚合。*

*Metric聚合: 基于一组文档进行聚合。所有的文档在一个检索集合里，文档被分成逻辑的分组; 对一个数据集求最大、最小、和、平均值等指标的聚合。*

*Matrix聚合： 此功能是实验性的，可在将来的版本中完全更改或删除；在多个字段上操作，并根据从请求的文档中提取的值生成矩阵结果。*

*Pipeline聚合：对聚合的结果而不是原始数据集进行操作。*

**测试数据**

```json
POST userinfo/doc/_bulk
{
  "index": {
    "_id": "1"
  }
}
{
  "username": "alfred way",
  "job": "java engineer",
  "age": 18,
  "birth": "1990-01-02",
  "isMarried": false,
  "salary": 10000
}
{
  "index": {
    "_id": "2"
  }
}
{
  "username": "tom",
  "job": "java senior engineer",
  "age": 28,
  "birth": "1980-05-07",
  "isMarried": true,
  "salary": 30000
}
{
  "index": {
    "_id": "3"
  }
}
{
  "username": "lee",
  "job": "ruby engineer",
  "age": 22,
  "birth": "1985-08-07",
  "isMarried": false,
  "salary": 15000
}
{
  "index": {
    "_id": "4"
  }
}
{
  "username": "Nick",
  "job": "web engineer",
  "age": 23,
  "birth": "1989-08-07",
  "isMarried": false,
  "salary": 8000
}
{
  "index": {
    "_id": "5"
  }
}
{
  "username": "Niko",
  "job": "web engineer",
  "age": 18,
  "birth": "1994-08-07",
  "isMarried": false,
  "salary": 5000
}
{
  "index": {
    "_id": "6"
  }
}
{
  "username": "Michell",
  "job": "ruby engineer",
  "age": 26,
  "birth": "1987-08-07",
  "isMarried": false,
  "salary": 12000
}
```

* Metric聚合

  **最值、求和、均值**

  ```bash
  POST /userinfo/doc/_search
  {
    "size": 0,
    "aggs": {
      "avg_grade": {
        "avg": {  # 可以使用max、min、sum
          "field": "salary"
        }
      }
    }
  }
  ```

  **Cardinality**

  *类似于SQL中的district count*

  ```bash
  POST /userinfo/doc/_search
  {
    "size": 0, 
    "aggs": {
      "type_count": {
        "cardinality": {
          "field": "job.keyword"
        }
      }
    }
  }
  ```

  **Stats**

  *返回一系列数值类型的统计值，包括min,max,avg,sum,count。*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "stats_age":{
              "stats":{
                  "field":"age"
              }
          }
      }
  }
  ```

  **Extended Stats**

  *对`Stats`聚合的扩展，包含更多统计数据，例如：方差，标准差等。*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "stats_age":{
              "extended_stats":{
                  "field":"age"
              }
          }
      }
  }
  ```

  **Percentiles**

  ```bash
  # 百分位数统计
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "per_age":{
              "percentiles":{
                  "field":"salary"
              }
          }
      }
  }
  # 针对特定值计算百分位数
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "per_age":{
              "percentile_ranks":{
                  "field":"salary",
                  "values":[
                      11000,
                      30000
                  ]
              }
          }
      }
  }
  # 针对特定百分位数计算对应的值 
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "per_age":{
              "percentiles":{
                  "field":"salary",
                  "percents" : [80, 95, 99, 99.9] 
              }
          }
      }
  }
  ```

  **Top Hits**

  *一般用于分桶后获取该桶内最匹配的顶部文档详情数据*

  ```bash
  # 先根据job分桶，后在桶内根据age排序
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "jobs":{
              "terms":{
                  "field":"job.keyword",
                  "size":10
              },
              "aggs":{
                  "top_employee":{
                      "top_hits":{
                          "size":10,
                          "sort":[
                          {
                              "age":{
                                  "order":"desc"
                              }
                          }
                          ]
                      }
                  }
              }
          }
      }
  }
  ```

* Bucketing聚合

  **Terms**

  *改分桶策略最简单，直接根据term分桶，如果是text类型，则按照分析器处理后的结果分桶。*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "jobs":{
              "terms":{
                  "field":"job.keyword",
                  "size":10
              }
          }
      }
  }
  ```

  **Range**

  *通过指定数值的范围来设定分桶规则*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "salary_range":{
              "range":{
                  "field":"salary",
                  "ranges":[
                  {
                      "to":10000
                  },
                  {
                      "from":10000,
                      "to":20000
                  },
                  {
                      "from":20000
                  }
                  ]
              }
          }
      }
  }
  ```

  **Date Range**

  *通过指定日期的范围来设定分桶规则*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "date_range":{
              "range":{
                  "field":"birth",
                  "format":"yyyy",
                  "ranges":[
                  {
                      "from":"1980",
                      "to":"1990"
                  },
                  {
                      "from":"1990",
                      "to":"2000"
                  },
                  {   
                      "from":"2000"
                  }
                  ]
              }
          }
      }
  }
  ```

  **Historgram**

  *以固定间隔来分割数据*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "salary_hist":{
              "histogram":{
                  "field":"salary",
                   "interval":5000,
                   "extended_bounds":{
                      "min":0,
                      "max":40000
                   }
              }
          }
      }
  }
  ```

  **Date Historgram**

  *针对日期的直方图*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "salary_hist":{
              "date_histogram":{
                  "field":"birth",
                   "interval":"year",
                   "format":"yyyy"
              }
          }
      }
  }
  ```

  **Filter**

  *将满足过滤条件的文档放入桶中*

  ```bash
  POST /userinfo/doc/_search
  {
    "size": 0, 
    "aggs": {
      "salary_aggs": {
        "filter": {
          "range": {
            "salary": {
              "gte": 10000
            }
          }
        },
        "aggs": {
          "avg_salary": {
            "avg": {
              "field": "salary"
            }
          }
        }
      }
    }
  }
  ```

  **Missing**

  *统计缺少指定字段的文档个数。*

  ```bash
  POST /userinfo/doc/_search
  {
    "size": 0, 
    "aggs": {
      "miss_aggs": {
        "missing": {
          "field": "phone"
        }
      }
    }
  }
  ```

* Pipeline聚合

  **针对聚合分析的结果再次进行聚合分析，而且支持链式调用。**

  *Pipeline的分析结果会输出到原结果中,根据输出位置的不同,分为以下两类:*

  1. Parent结果内嵌到现有的聚合分析结果中

  * Derivative

  * Moving Average

  * Cumulative Sum

  2. Sibling结果与现有聚合分析结果同级

  * Max/Min/Avg/Sum Bucket

  * Stats/Extended Stats Bucket

  * Percentitles Bucket

  **Sibing-Derivative**

  *先根据job分桶并计算各个分桶的avg值，然后输出avg最小的桶名称和值*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "jobs":{
              "terms":{
                  "field":"job.keyword",
                  "size":10
              },
              "aggs":{
                  "avg_salary":{
                      "avg":{
                          "field":"salary"
                      }
                  }
              }
          },
          "min_salary_by_job":{
              "min_bucket":{
                  "buckets_path":"jobs>avg_salary"
              }
          }
      }
  }
  ```

  *先根据job分桶并计算各个分桶的avg值，然后对所有bucket进行stats分析*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "jobs":{
              "terms":{
                  "field":"job.keyword",
                  "size":10
              },
              "aggs":{
                  "avg_salary":{
                      "avg":{
                          "field":"salary"
                      }
                  }
              }
          },
          "stats_salary_by_job":{
              "stats_bucket":{
                  "buckets_path":"jobs>avg_salary"
              }
          }
      }
  }
  ```

  **Parent-Derivative**

  *计算bucket的导数*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "birth":{
              "date_histogram":{
                  "field":"birth",
                  "interval":"year",
                  "min_doc_count":0
              },
              "aggs":{
                  "avg_salary":{
                      "avg":{
                          "field":"salary"
                      }
                  },
                  "derivative_avg_salary":{
                      "derivative":{
                          "buckets_path":"avg_salary"
                      }
                  }
              }
          }
      }
  }
  ```

* Bucket+Metric聚合分析

  *先根据job进行term分桶策略，然后再对每一个分桶进行range分桶策略*

  ```bash
  POST /userinfo/doc/_search
  {
      "size":0,
      "aggs":{
          "jobs":{
              "terms":{
                  "field":"job.keyword",
                  "size":10
              },
              "aggs":{
                  "age_range":{
                      "range":{
                          "field":"age",
                          "ranges":[
                          {"to":20},
                          {"from":20,"to":30},
                          {"from":30}
                          ]
                      }
                  }
              }
          }
      }
  }
  ```
* 搜索提示

  *`_suggest`URI已经被`_search`所替代，使用`suggest`属性。*

  ```bash
  POST /userinfo/doc/_search
  {
    "query": {
      "match": {
        "job": "web"
      }
    },
    "suggest": {
      "me_SUGGESTION": {
        "text": "web",
        "term": {
          "field": "job"
        }
      }
    }
  }
  ```