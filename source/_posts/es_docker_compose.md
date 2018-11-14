---
title: 使用dokcer-compose编排Elasticsearch服务
date: 2018-11-05 14:54:28
tags: Elasticsearch
categories: 大数据
---

> Elasticsearch: 6.4.2
> Kibana: 6.4.2

*在一台主机上使用docker-compose编排es的单主机多容器集群和kibana服务是非常有必要的,可以集中管理这些服务了.*

`docker-compose.yml`
```yaml
version: '3'
services:
  node1:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    container_name: node1
    environment:
      - node.name=es01
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      nproc: 65535
      memlock:
        soft: -1
        hard: -1
    cap_add:
      - ALL
    privileged: true
    deploy:
      mode: global
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    volumes:
      - ./node1/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
  node2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    container_name: node2
    environment:
      - node.name=es02
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=node1"
    ulimits:
      nproc: 65535
      memlock:
        soft: -1
        hard: -1
    cap_add:
      - ALL
    privileged: true
    deploy:
      mode: global
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    volumes:
      - ./node2/data:/usr/share/elasticsearch/data
    ports:
      - 9201:9200
      - 9301:9300
  kibana:
    image: docker.elastic.co/kibana/kibana:6.4.2
    container_name: kibana
    environment:
      SERVER_NAME: localhost
      ELASTICSEARCH_URL: http://node1:9200
    ports:
      - 5601:5601
    ulimits:
      nproc: 65535
      memlock:
        soft: -1
        hard: -1
    cap_add:
      - ALL
    deploy:
      mode: global
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```
**使用如下命令启动:**
```bash
docker-compose up -d
```
**浏览器打开http://localhost:5601即可监控集群状态.**