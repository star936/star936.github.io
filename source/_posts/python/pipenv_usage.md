---
title: Python之Pipenv使用
date: 2019-06-24 15:39:28
tags: [Python, Pipenv]
categories: python
toc: true
---


> 工欲善其事,必先利其器.

**Pipenv: Python Development Workflow for Humans**

#### 1. 安装

```bash
pip install pipenv
```

#### 2. 使用

##### 2.1 创建虚拟环境

**添加`--python`参数指定python版本号，前提条件是本地已经安装该版本的python.**

```bash
pipenv --three/two
pipenv --python 2.7
pipenv --python 3.7
```

这会在项目目录中创建两个新文件：

`Pipfile`:该文件是`TOML`格式，存放当前虚拟环境的配置信息，包括python版本，pypi源以及依赖包等，pipenv根据该文件寻找项目的根目录。

`Pipfile.lock`:该文件是对Pipfile的锁定，支持锁定项目不同版本所依赖的环境。

##### 2.2  activate与deactivate

```bash
# activate
pipenv shell
# deactivate
exit
```

##### 2.3 安装包

**pipenv支持开发环境和生产环境依赖的分离。**

`pipenv install`有多重作用：

1. *如果虚拟环境已经存在，则安装Pipfile中的依赖包;*
2. *如果虚拟环境不存在，但Pipfile存在，则根据Pipfile中python版本创建虚拟环境并安装依赖包;*
3. *如果虚拟环境和Pipfile都不存在，则根据系统默认python版本创建虚拟环境.*

```bash
pipenv install 
# 安装开发环境依赖(如py.tests,mock等)
pipenv install --dev
# 指定包名
pipenv install [package_name]
# 如果项目已经存在requirements.txt
pipenv install -r requirements.txt
```

**另外你也可以以下面格式的URL安装在git或其他版本控制系统中的包。**

```bash
<vcs_type>+<scheme>://<location>/<user_or_organization>/<repository>@<branch_or_tag>#<package_name>
```

**vcs_type有效值：**git，bzr，svn，hg

**scheme有效值：**http，https，ssh，file

**branch_or_tag：**可选参数

**强烈建议以编辑模式安装任何版本控制依赖，如下示例：**

```bash
# 安装requests
pipenv install -e git+https://github.com/requests/requests.git@v2.19#egg=requests
```



#### 3. 常用命令

**pipenv**

```bash
pipenv [OPTIONS] COMMAND [ARGS]...
```

**Options:**

 - --where

   输出项目根目录路径

- --venv

  输出虚拟环境信息

- --envs

  输出环境变量信息

- --rm

  删除当前虚拟环境

- --pypi-mirror

  指定PyPi的镜像

- --site-packages

  为虚拟环境启用site-packages

**check**：检查包的安全性

**clean**：卸载未在Pipfile.lock中指定的所有软件包

**graph**：显示当前安装的依赖关系图信息

**lock**：生成Pipfile.lock文件

**run**：在未激活虚拟环境时可以直接使用虚拟环境的Python执行命令

**sync**：安装所有在Pipfile.lock中指定的软件包

**uninstall**：卸载指定的软件包并将其从`Pipfile`中删除

**update**：更新指定包