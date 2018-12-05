---
title: Blaze(五):数据的分割-应用-组合-分组
date: 2018-12-05 21:46:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---

*分组操作将一张表切分为多个块，并对每个块进行操作。*

*以species分组，并对petal求平均值*

```python
# coding: utf-8

from blaze import data, by
from blaze.utils import example

d = data('sqlite:///{}::iris'.format(example('iris.db')))
print(by(d.species, avg=d.petal_length.mean()).peek())
```

*对于描述许多有用的变换，split-apply-combine操作是一种简单而强大的方式。它被所有后端高效支持。*

#### 1. 参数

`by`方法有一个位置参数(决定如何对表进行分组)和多个关键字参数(对每一个份组执行的操作)。形式如下：

```python
by(grouper, name=reduction, name=reduction, ...)
```

*例子：以species对iris数据集分组，并对每个分组计算petal_length的最小值、最大值和最大值与最小值间的差。*

```python
print(by(d.species, min=d.petal_length.min(), max=d.petal_length.max(),
         ratio=d.petal_length.max()-d.petal_length.min()).peek())
```

*输出：*

```bash
           species  max  min  ratio
0      Iris-setosa  1.9  1.0    0.9
1  Iris-versicolor  5.1  3.0    2.1
2   Iris-virginica  6.9  4.5    2.4
```

#### 2. 限制

与内存型的dataframes例如`pandas`或`dplyr`相比，这个接口有两个方面的限制：

1. 必须同时指定分组方式和相应操作
2. “apply”阶段必须是一个reduction

这些限制让它更方便向数据集传达你的意图，并有效分发和并行计算。

#### 3. 不能做的事

**不能只指定如何分组而没有指定对每个分组的操作，例如：**

```python
groups = by(mytable.mycolumn)  # Can't do this  
```

**不能指定non-reducing的apply操作(尽管有些后端通过改变而正常工作),例如：**

```python
groups = by(d.A, result=d.B / d.B.max())  # Can't do this
```

