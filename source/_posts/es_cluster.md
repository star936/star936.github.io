---
title: Elasticsearch 6 Docker 集群部署—单机多容器实例
date: 2018-11-05 13:54:28
tags: Elasticsearch
categories: 大数据
---

> Elasticsearch: 6.4.2
>
> 环境：在Mac上搭建的单机多容器实例：1个master节点，一个slave节点



#### 1. 以Docker形式安装Elasticsearch

拉去镜像：

```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```

#### 2. 启动

启动master节点：

```bash
docker run -d -p 9200:9200 -p 9300:9300 --name elas1 -h elas1  -e cluster.name=lookout-es -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e xpack.security.enabled=false docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```

启动slave节点：

```bash
docker run -d -p 9211:9200 -p 9311:9300 --link elas1  --name elas2 -e cluster.name=lookout-es -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e xpack.security.enabled=false -e discovery.zen.ping.unicast.hosts=elas1 docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```



| 参数   | 作用                               |
| ------ | ---------------------------------- |
| -p     | 端口映射(主机:)                    |
| --name | 启动容器的名字                     |
| -h     | 容器主机名                         |
| -e     | 指定docker内环境变量参数           |
| -d     | 程序后台运行                       |
| --link | 相当于在容器中配置了link对象的host |



#### 3. 验证

##### 3.1、检查节点状态

```bash
http://localhost:9211/
```

##### 3.2、检查集群状态

```bash
http://localhost:9200/_cat/health?v
```

#### 4. 监控

**使用官方提供的Kibana，以docker方式安装:**

```bash
docker pull docker.elastic.co/kibana/kibana:6.4.2
```

**运行Kibana**

```bash
docker run -d --name kibana  --link elas1 -e ELASTICSEARCH_URL=http://elas1:9200 -p 5601:5601 docker.elastic.co/kibana/kibana:6.4.2
```

**通过http://localhost:5601进行访问.**

 