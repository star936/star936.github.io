---
title: Blaze(三):快速入门
date: 2018-12-04 22:46:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---


*该篇文章通过展示创建和操作Blaze Symbols快速入门。*

#### 1. 与数据的交互

*通过嵌套的list/tuple创建简单的Blaze表达式。Blaze将推导出要使用的维度和数据类型。*

```python
# coding: utf-8

from blaze import *

t = data([(1, 'Alice', 100),
          (2, 'Bob', -200),
          (3, 'Charlie', 300),
          (4, 'Denis', 400),
          (5, 'Edith', -500)],
         fields=['id', 'name', 'balance'])

print(t.peek())
```

*输出：*

```bash
   id     name  balance
0   1    Alice      100
1   2      Bob     -200
2   3  Charlie      300
3   4    Denis      400
4   5    Edith     -500
```
>  *可能报错：* AttributeError: 'Graph' object has no attribute 'edges_iter'
>
> 解决方法：安装networkx的1.9版本。

#### 2. 简单计算

*与Pandas相似的列选择和过滤语法*

```python
# coding: utf-8
from blaze import *

t = data([(1, 'Alice', 100),
          (2, 'Bob', -200),
          (3, 'Charlie', 300),
          (4, 'Denis', 400),
          (5, 'Edith', -500)],
         fields=['id', 'name', 'balance'])

print(t[t.balance < 0].peek())
print('-' * 10)
print(t[t.balance < 0].peek().name)
```
> 在0.11版本中Blaze表达式repr不再隐式地计算,需要使用peek()函数去看改表达式结果的预览.

*输出:*
```bash
   id   name  balance
0   2    Bob     -200
1   5  Edith     -500
----------
0      Bob
1    Edith

```

#### 3. 已保存的数据
*操作[iris](https://raw.githubusercontent.com/blaze/blaze/master/blaze/examples/data/iris.csv)数据集的CSV文件*
```python
# coding: utf-8

from blaze import *
from blaze.utils import example

iris = data(example('iris.csv'))
print(iris.peek())
```
*输出:*
```bash
    sepal_length  sepal_width  petal_length  petal_width      species
0            5.1          3.5           1.4          0.2  Iris-setosa
1            4.9          3.0           1.4          0.2  Iris-setosa
2            4.7          3.2           1.3          0.2  Iris-setosa
3            4.6          3.1           1.5          0.2  Iris-setosa
4            5.0          3.6           1.4          0.2  Iris-setosa
5            5.4          3.9           1.7          0.4  Iris-setosa
6            4.6          3.4           1.4          0.3  Iris-setosa
7            5.0          3.4           1.5          0.2  Iris-setosa
8            4.4          2.9           1.4          0.2  Iris-setosa
9            4.9          3.1           1.5          0.1  Iris-setosa
10           5.4          3.7           1.5          0.2  Iris-setosa
```
*用相同的方式使用远程数据,例如:SQL数据库或者spark分布式数据结构.例子: 操作sqlite中的数据库.*
```python
    sepal_length  sepal_width  petal_length  petal_width      species
0            5.1          3.5           1.4          0.2  Iris-setosa
1            4.9          3.0           1.4          0.2  Iris-setosa
2            4.7          3.2           1.3          0.2  Iris-setosa
3            4.6          3.1           1.5          0.2  Iris-setosa
4            5.0          3.6           1.4          0.2  Iris-setosa
5            5.4          3.9           1.7          0.4  Iris-setosa
6            4.6          3.4           1.4          0.3  Iris-setosa
7            5.0          3.4           1.5          0.2  Iris-setosa
8            4.4          2.9           1.4          0.2  Iris-setosa
9            4.9          3.1           1.5          0.1  Iris-setosa
10           5.4          3.7           1.5          0.2  Iris-setosa
```
#### 4. 更多计算
*常用操作例如Joins和split-apply-combine对任何种类的数据都可用.*
```python
# coding: utf-8

from blaze import *
from blaze.utils import example

iris = data('sqlite:///{}::iris'.format(example('iris.db')))
print(by(iris.species, min=iris.petal_width.min(), max=iris.petal_width.max()).peek())
```
*输出:*
```bash
           species  max  min
0      Iris-setosa  0.6  0.1
1  Iris-versicolor  1.8  1.0
2   Iris-virginica  2.5  1.4
```
#### 5. 结束
*使用`odo`操作将结果输出到一个合适的容器类型中.*
```python
# coding: utf-8

from blaze import *
from blaze.utils import example

iris = data('sqlite:///{}::iris'.format(example('iris.db')))
result = by(iris.species, avg=iris.petal_width.mean())
result_list = odo(result, list)
print(odo(result, DataFrame))
```
*输出:*
```bash
           species    avg
0      Iris-setosa  0.246
1  Iris-versicolor  1.326
2   Iris-virginica  2.026
```