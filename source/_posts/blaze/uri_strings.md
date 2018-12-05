---
title: Blaze(七):URI strings
date: 2018-12-05 21:48:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---


Blaze使用strings指定数据源，使用时非常简单。

#### 1. 例子

*与一组CSV文件或一个SQL数据库交互*

```python
# coding: utf-8

from blaze import *
from blaze.utils import example

t = data(example('accounts_*.csv'))
print(t.peek())

t1 = data('sqlite:///%s::iris' % example('iris.db'))
print(t1.peek())
```

*迁移CSV文件数据到SQL数据库*

```python
from blaze.utils import example
from odo import odo

print(odo(example('iris.csv'), 'sqlite:///myfile.db::iris'))
```

**Blaze支持的URIs的种类：**

* 磁盘上文件的路径，包含如下扩展：
  * `.csv`
  * `.json`
  * `.csv.gz/json.gz`
    * `.hdf5`(使用h5py)
  * `.hdf5::/datapath`
  * `hdfstore://filename.hdf5`(使用pandas.HDFStore格式)
  * `.bcolz`
  * `.xls(x)`

* 如下的SQLAlchemy strings

  * `sqlite:////absolute/path/to/myfile.db::tablename`
  * `sqlite:////absolute/path/to/myfile.db`
  * `postgresql://username:password@hostname:port`
  * `impala://hostname`(使用impala)
  * 其余被SQLAlchemy支持的

* 如下格式的MongoDB连接strings

  `mongodb://username:password@hostname:port/database_name::collection_name`

* 如下格式的Blaze server strings

  `blaze://hostname:port`(默认端口号6363)

*如果对于传统的URI需要额外的一个位置或者一个表名，可以在URI末尾使用`::`分割并跟在其后面。*

#### 2. 如何工作的

**Blaze依赖于`odo`库来处理URI。URI是由`resource`函数管理的，而`resource`函数是基于正则表达式分发的。例如下面处理.json文件的一个简单的resource函数（尽管Blaze实际的解决方法比它更全面）：** 

```python
from blaze import resource
import json

@resource.register('.+\.json')
def resource_json(uri):
    with open(uri):
        data = json.load(uri)
    return data
```

#### 3. 可以扩展到自己的类型吗？

**当然可以。正如`2. 如何工作的`部分所展示的那样导入和扩展resource，剩下的blaze将自动接受你的更改。**