---
title: Elasticsearch(十) Logstash入门
date: 2018-11-28 21:20:28
tags: [Elasticsearch、Logstash]
categories: 大数据
toc: true
---

> Logstash: 6.4.2

### 1. 配置语法

#### 1.1 语法

**区段**
Logstash使用`{}`来定义区段,区域内可以包含插件区域定义,可以在一个区域内定义多个插件;插件区域内可以定义键值设置,例如:
```bash
input {
    stdin{}
}
```

**数据类型**
* bool
* string
* number
* array
* hash

**字段引用**
*字段是`Logstash::Event`对象的属性,将字段的名称写在`[]`里就可以在Logstash配置里使用该字段的值.*
*对于嵌套字段,可以使用如下格式:*
```bash
[字段名][字段名或数组下标值]
# 例如:
[user][firstname]
[users][0]
```
> Logstash里数组支持倒序下标,例如:[-1]表示数组的最后一个元素.
*Logstash还支持变量内插,在字符串里使用字段引用的方法如下:*
```bash
"the first name is %{[user][firstname]}"
```
**条件判断**
* ==(等于)、!=(不等于)、<(小于)、<=(小于等于)、 >(大于)、>=(大于等于)
* =~(正则匹配)、!~(不正则匹配)
* in(包含)、not in(不包含)
* and(与)、or(或)、nand(非与)、xor(非或)
* ()(复合表达式)、!()(对复合表达式结果取反)

#### 1.2 命令行参数解析
* -e
  意即执行,默认值为:
  ```ruby
  input {
      stdin{ }
  }
  output {
      stdout{ }
  }
  ```

* --config或-f
  指定配置文件.
* --configtest或-t
  用来测试Logstash读取到的配置文件语法是否能正常解析.
* --log或-l
  指定日志输出位置,默认输出日志到标准错误.
* --pipeline-workers或-w
  指定运行filter和output的pipeline线程数量,默认为CPU核数.
* --pipeline-batch-size或-b
  指定每个pipeline线程在打包批量日志的时候最多等待几秒,默认为5ms.
* --pluginpath或-P
  指定自定义插件的路径
* --verbose
  输出一定的调试日志
* --debug
  输出更多的调试日志

#### 1.3 配置文件
*针对命令行参数,我们可以在`$LS_HOME/config/logstash.yml`文件中进行设置,例如:*
```yaml
pipeline:
    workers: 24
    batch:
        size: 125
        delay: 5
```

### 2. 执行过程

**Logstash事件进程管道有三个阶段：input->filter->output，其中input和output是必须的，filter是可选的；input默认为stdin，output默认为stdout。**

* inputs：常用的有file、syslog、redis、jdbc、beats等。
* filters：常用的有：
  * grok：解析和结构化任意文本。常用来解析非结构化的log；内置120种解析模式。
  * mutate：对事件类型执行转换，可以rename、remove、replace、modify。
  * drop：完全丢掉一种事件。
  * geoip：添加IP地址的地理位置信息。

* outputs：常用的有elasticsearch、file、graphite、statsd。

**Codecs是基本的对input/output的流式过滤器，常见的有json、multiline、plain。**

> 执行过程：logstash管道中每个input阶段都运行在独立的线程中。Inputs将事件写到一个存在于内存或硬盘的集中式queue中；每个管道worker从queue取一批事件（数量可配置）并按序执行配置的filters，最后将结果写入到outputs。



### 3. 简单实例

下载filebeat、logstash二进制包并解压。

下载[apache日志](https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz)。

**filebeat读取apache日志并发送到logstash，logstash将其输出到stdout。**

*filebeat.yml*

```yaml
filebeat.prospectors:
- type: log
  paths:
    - /path/to/logstash-tutorial.log 
output.logstash:
  hosts: ["localhost:5044"]
```

*启动filebeat*

```bash
./filebeat -e -c filebeat.yml -d "publish"
```

*logstash-sample.conf*

```conf
input {
  beats {
        port => "5044"
    }
}

output {
  stdout { codec => rubydebug }
}
```

*启动logstash*

```bash
bin/logstash -f config/logstash-sample.conf --config.reload.automatic
```

* --config.reload.automatic：自动重加载配置文件，当你修改配置文件后不再需要stop并restart logstash了。