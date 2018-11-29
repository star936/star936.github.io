---
title: Elasticsearch(十一)利用logstash将mysql数据输出到ES
date: 2018-11-29 21:40:28
tags: [Elasticsearch, Logstash]
categories: 大数据
toc: true
---

> Elasticsearch: 6.4.2
> Logstash: 6.4.2


**利用Logstash插件logstash-input-jdbc将mysql数据库中的数据存到ES中.**

**准备工作:**
* [下载logstash](https://www.elastic.co/downloads/past-releases/logstash-6-4-2)
* [下载mysql jdbc jar包](https://dev.mysql.com/downloads/connector/j/)

*logstatsh的pipeline文件：*
```yaml
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
    stdin {
    }
    jdbc {
      jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/test?serverTimezone=Asia/Shanghai&useSSL=true&useUnicode=true&characterEncoding=UTF-8"
      jdbc_user => "root"
      jdbc_password => "123456"
      jdbc_driver_library => "../jar/mysql-connector-java-5.1.47-bin.jar"
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      statement => "select * from user"
      type => "jdbc"
    }
}
filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}
output {
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "users"
        document_id => "%{id}"
    }
    stdout {
        codec => json_lines
    }
}
```
* jdbc_driver_library：路径是相对于logstash根目录下的config目录。

*启动*
```bash
bin/logstash -f ../config/logstash-mysql.conf
```