---
title: Blaze(一):前言
date: 2018-12-04 20:55:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---


*Blaze生态系统为python用户对大数据提供了高效计算的高层接口。主要由Anaconda赞助。*

**应用领域**

*Blaze整合了包括Python的Pandas、NumPy及SQL、Mongo、Spark在内的多种技术，使用Blaze能够非常容易地与一个新技术进行交互。*

*Blaze目前主要用于数据库和数组技术的分析查询。并且它在不断地整合和提供基于其它计算系统的应用接口。Blaze主要通过为数据科学家提供直观的各种工具访问来展现性能。*

**Blaze生态系统：**

* Blaze项目：忽略用户使用的各种不同的计算方案和不同类型的数据库，为用户提供了统一、友好、熟悉的界面。它帮助我们更好地与文档、数据结构以及数据库进行交互，根据需要适当优化和转换用户查询语句以提供一个流畅和交互式会话。Blaze项目允许数据科学家和分析人员以统一的方式编写他们的查询语句，不必因为数据存储格式或存储介质的不同而进行转换。它还提供了一个服务器组件，允许使用URIs来方便地提供数据视图，并在本地脚本、查询和程序中远程引用数据。

  将NumPy/Pandas式的句法规则转换为数据计算系统（例如数据库、内存、分布式计算）。这为Python用户查询存储于其他数据存储系统中的数据提供了便捷和熟悉的界面。一条Blaze查询命令能够完成从CSV文件到分布式数据库的各种数据处理。

* DataShape：数据类型系统

  结合了NumPy中的dtype和数据形状，并且将内容扩展到了缺失值、可变长度字符串、不规则数组以及多任意嵌套方面。它包含了从数据库到HDF5文件再到JSON片段数据类型的统一描述。

* Odo：实现不同格式数据的转换

  利用一个复杂可扩展的转换网络，通过一个简单的界面有效稳定地实现了不同类型（CSV、JSON、数据库）和不同位置（本地、远程、HDFS）数据的传输和转换。

* DyND：内存中的动态数组

  DyND是一个类似于NumPy一样的动态ND数组库，是一个实现数据格式类型处理的系统。它支持可变长度的字符串、不规则数据以及GPU。它是一个与Python绑定的独立的C++代码库。通常DyND比NumPy更具扩展性，但不如NumPy成熟。

* Dask.array：多核/基于磁盘存储的Numpy数组

  Dask.dataframe：多核/基于磁盘的Pandas数据帧

  Dask.arrays在NumPy之上提供了阻塞算法，并利用多核处理大于内存的数组。它们是NumPy算法常用集的简单替换。

  Dask.dataframes在Pandas之上提供了阻塞算法，并利用多核处理大于内存的数据帧（dataframe），它们是Pandas用例算法集的简单替换。

  除了多核和多线程调度器外，Dask还利用简单装饰器和新生分布式调度器为用户提供了一个“Bag”类型数据和一种构建“任务图（task graphs）”的方法。

> 以上部分来自于博客[http://hao.jobbole.com/blaze/](http://hao.jobbole.com/blaze/)



**例子**

*Blaze将要执行的计算从数据的描述中分离*

```python
# coding: utf-8

from blaze import *


accounts = symbol('accounts', 'var * {id: int, name: string, amount: int}')

deadbeats = accounts[accounts.amount < 0].name

L = [[1, 'Alice',   100],
     [2, 'Bob',    -200],
     [3, 'Charlie', 300],
     [4, 'Denis',   400],
     [5, 'Edith',  -500]]

print(list(compute(deadbeats, L)))

df = DataFrame(L, columns=['id', 'name', 'amount'])
print(compute(deadbeats, df))

```

*输出：*

```bash
['Bob', 'Edith']
1      Bob
4    Edith
Name: name, dtype: object
```

*Blaze不计算这些结果，它聪明地驱动其他项目来计算。这些项目从简单的纯python迭代器到高性能、分布式的spark集群。*