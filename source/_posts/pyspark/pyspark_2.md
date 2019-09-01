---
title: PySpark(二) 在IPython Notebook上使用Spark
date: 2019-08-22 22:22:22
tags: PySpark
categories: PySpark
---

**上一篇博客: [PySpark(一): Hadoop SingleNode部署下Spark on yarn](https://star936.github.io/2019/08/21/pyspark/pyspark_1/)**

## 1. 准备
**1. 将Hadoop启动**
**2. 安装:**
* `Anaconda`
* 创建虚拟环境
```bash
conda create -n venv python=2.7
```
* 安装Ipython Notebook
```bash
conda install ipython ipython-notebook
```
* 启用虚拟环境
```bash
source active venv
```
## 2. 启动
**使用如下命令:**
```bash
PYSPARK_DRIVER_PYTHON=ipython PYSPARK_DRIVER_PYTHON_OPTS="notebook" pyspark
```

## 3. 测试
可以按照如图所示的命令测试是否成功:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822221953269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhaXlhbmdnZW5n,size_16,color_FFFFFF,t_70)