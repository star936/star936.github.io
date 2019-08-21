---
title: Hadoop SingleNode部署下Spark on yarn
date: 2019-08-21 23:24:28
tags: PySpark
categories: PySpark
---

> 环境: MacOX系统
>  	1. Java: 8
>  	2. Scala: 2.12.4
>  	3. Hadoop: 2.7.7
>  	4. Spark: 2.4.0


## 1. 准备工作
1. 安装Java, Scala, 并下载Spark及其相应版本的Hadoop;
2. 编辑`~/.zshrc`
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
export HADOOP_HOME=/Users/dang/work/hadoop/hadoop-2.7.7
export SCALA_HOME=/Users/dang/work/hadoop/scala-2.12.4
export SPARK_HOME=/Users/dang/work/hadoop/spark-2.4.0
export PATH=$PATH:$SCALA_HOME/bin:$SPARK_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
```

## 2. 配置并启动Hadoop
### 1. 配置

1. 编辑`$HADOOP_HOME/etc/hadoop/core-site.xml`,并添加如下内容:
```xml
<configuration>
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://localhost:9000</value>
</property>
</configuration>
```
2. 编辑`$HADOOP_HOME/etc/hadoop/hdfs-site.xml`,并添加如下内容:
```xml
<configuration>
<property>
  <name>dfs.replication</name>
  <value>1</value>
</property>
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file:/Users/dang/work/hadoop/hadoop_data/hdfs/namenode</value>
</property>
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file:/Users/dang/work/hadoop/hadoop_data/hdfs/datanode</value>
</property>
</configuration>
```
**注意: `dfs.namenode.name.dir`和`dfs.datanode.data.dir`目录在Hadoop启动前必须已存在**

3. 编辑`$HADOOP_HOME/etc/hadoop/mapred-site.xml``,并添加如下内容:
**首先执行`cp mapred-site.xml.template mapred-site.xml`**
```xml
<configuration>
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
</configuration>
```
4. 编辑`$HADOOP_HOME/etc/hadoop/yarn-site.xml`,并添加如下内容:
```xml
<configuration>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
  <name>yarn.nodemanager.pmem-check-enabled</name>
  <value>false</value>
</property>

<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
</property>
<property>
  <description>Whether to enable log aggregation</description>
  <name>yarn.log-aggregation-enable</name>
  <value>true</value>
</property>
</configuration>
```
5. 编辑`$HADOOP_HOME/etc/hadoop/hadoop-env.sh`,并添加如下内容:
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
```

### 3. 初始化HDFS
执行如下命令:
```bash
hadoop namenode -format
```
### 3. 启动/停止
执行如下命令:
```bash
# 启动
start-all.sh
## 或者
start-dfs.sh # 先执行
start-yarn.sh # 后执行

# 停止
stop-all.sh
## 或者
stop-yarn.sh # 先执行
stop-dfs.sh # 后执行
```
## 2. Spark配置并启动
### 1. 配置
1. 编辑`$SPARK_HOME/conf/spark-env.sh`,并添加如下内容:
**首先执行 `cp spark-env.template spark-env.sh`**
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
export HADOOP_HOME=/Users/dang/work/hadoop/hadoop-2.7.7
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_HOME=/Users/dang/work/hadoop/spark-2.4.0
export SPARK_LOCAL_IP="127.0.0.1"
export SPARK_LIBARY_PATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$HADOOP_HOME/lib/native
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native/:$LD_LIBRARY_PATH
```
2. 上传Spark Jar包到Hadoop HDFS
```bash
hadoop fs -put -p jars/ /hadoop/spark_jars
```

3. 编辑`$SPARK_HOME/conf/spark-defaults.conf`,并添加如下内容:
**首先执行 `cp spark-defaults.conf.templatespark-defaults.conf`**
```bash
spark.yarn.jars hdfs://localhost:9000/hadoop/spark_jars/*
```

### 2.启动
执行如下命令:
```bash
pyspark --master yarn --deploy-mode client
```

## 4. 遇到的问题:
```bash
Exception from container-launch.
Container id: container_e64_1481762217559_27152_01_000002
Exit code: 127
```
**解决方法:**
* 设置`hadoop-env.sh`中的`JAVA_HOME`
* 设置`spark-env.sh`
* Hadoop启动之后等几步分钟,让程序完全启动后,再去启动Spark