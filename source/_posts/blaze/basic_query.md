---
title: Blaze(四):基本查询
date: 2018-12-04 22:46:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---

> 使用之前x下载的iris数据集CSV文件.

*该段代码以下所有示例都会使用到*
```python
# coding: utf-8

from blaze import data
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
...
```

#### 1. 获取列数据
**两种方式获取单独列的数据:**
```python
# 第一种
print(iris.species.peek())
# 第二种
print(iris['species'].peek())
```
*输出:*
```bash
        species
0   Iris-setosa
1   Iris-setosa
2   Iris-setosa
3   Iris-setosa
4   Iris-setosa
5   Iris-setosa
...
```
**通过名称列表选择多个列数据**
```python
print(iris[['sepal_length', 'species']].peek())
```
*输出:*
```bash
    sepal_length      species
0            5.1  Iris-setosa
1            4.9  Iris-setosa
2            4.7  Iris-setosa
3            4.6  Iris-setosa
4            5.0  Iris-setosa
5            5.4  Iris-setosa
...
```
#### 2. 数学操作
**使用数学的操作和功能**
```python
from blaze import log
print(log(iris.sepal_length * 10).peek())
```
*数学功能像log应该从blaze导入,基于后端它会被转化为`np.log`、`math.log`、`sqlalchemy.sql.func.log`等.*

#### 3. Reductions
**与许多blaze操作一样,像`mean`和`sum`可以作为函数或基本功能使用.**
```python
print(iris.sepal_length.mean().peek())
# output: 5.843333333333334
print(mean(iris.sepal_length).peek())
# output: 5.843333333333334
```
#### 4. Split-Apply-Combine
*`by`操作是split-apply-combine计算,常见格式如下:*
```bash
by(table.grouping_columns, name_1=table.column.reduction(), name_2=table.column.reduction(), ...)  
```
*例如:根据species查找最短、最长和p平均的petal长度.*
```python
print(by(iris.species, shortest=iris.petal_length.min(), longest=iris.petal_length.max(),
         average=iris.petal_length.mean()).peek())
```
*输出:*
```bash
           species  average  longest  shortest
0      Iris-setosa    1.462      1.9       1.0
1  Iris-versicolor    4.260      5.1       3.0
2   Iris-virginica    5.552      6.9       4.5
```
#### 5. 添加新列
**使用`transform`方法添加新列.**
```python
from blaze import transform
print(transform(iris, sepal_ratio = iris.sepal_length / iris.sepal_width,
                petal_ratio = iris.petal_length / iris.petal_width).peek())
```
*输出:*
```bash
    sepal_length  sepal_width  petal_length  petal_width      species  petal_ratio  sepal_ratio
0            5.1          3.5           1.4          0.2  Iris-setosa     7.000000     1.457143
1            4.9          3.0           1.4          0.2  Iris-setosa     7.000000     1.633333
2            4.7          3.2           1.3          0.2  Iris-setosa     6.500000     1.468750
...
```
#### 6. 文本匹配
```python
print(iris[iris.species.like('*versicolor')].peek())
```
*输出:*
```bash
    sepal_length  sepal_width  petal_length  petal_width          species
50           7.0          3.2           4.7          1.4  Iris-versicolor
51           6.4          3.2           4.5          1.5  Iris-versicolor
...
```
#### 7. 重命名列名
```python
print(iris.relabel(petal_length='PETAL-LENGTH', petal_width='PETAL-WIDTH').peek())
```
*输出:*
```bash
    sepal_length  sepal_width  PETAL-LENGTH  PETAL-WIDTH      species
0            5.1          3.5           1.4          0.2  Iris-setosa
1            4.9          3.0           1.4          0.2  Iris-setosa
...
```

#### 8. 例子
*Blaze可以解决许多数据分析和科学计算中存在的问题,例子如下:*

* Combining separate, gzipped csv files
  ```python
  # coding: utf-8

  from blaze import odo
  from blaze.utils import example
  from pandas import DataFrame

  print(odo(example('accounts_*.csv.gz'), DataFrame))
  ```
  *输出:*
  ```bash
    id      name  amount
    0   1     Alice     100
    1   2       Bob     200
    2   3   Charlie     300
    3   4       Dan     400
    4   5     Edith     500

  ```
* Split-Apply-Combine
  ```python
  from blaze import by
  t = data('sqlite:///%s::iris' % example('iris.db'))
  print(by(t.species, max=t.petal_length.max(), min=t.petal_length.min()).peek())
  ```
  *输出:*
  ```bash
    species  max  min
    0      Iris-setosa  1.9  1.0
    1  Iris-versicolor  5.1  3.0
    2   Iris-virginica  6.9  4.5
  ```
 