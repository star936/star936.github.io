---
title: 使用Hexo+Github Page搭建个人博客
date: 2018-11-03 14:54:28
tags: Hexo
categories: 随笔
---
[Hexoz中文文档](https://hexo.io/zh-cn/docs/index.html)

## Quick Start

### Github
在个人`GitHub`上创建名字形式为`xxx.github.io`的repository,并clone到本地.

### 基本配置
```yml
title: 名称
subtitle: 子名称
description: 描述
keywords: 关键词
author: 作者
language: 本地化(查看themes/hexo-theme-matery/languages)
```

### 更换主题
*本博客使用的是`hexo-theme-matery`主题,在此非常感谢`blinkfox`的开源贡献.*
1. 下载主题
```bash
git clone https://github.com/blinkfox/hexo-theme-matery.git themes/hexo-theme-matery
```
2. 在根目录下修改`_config.yml`文件
```yml
theme: hexo-theme-matery
```

### 新建categories
`categories`页是用来展示所有分类的页面，如果在你的博客`source`目录下还没有`categories/index.md`文件，使用如下命令创建:

``` bash
$ hexo new page "categories"
```
编辑你刚刚新建的页面文件`/source/categories/index.md`，添加以下内容:
```yml
---
title: 分类
date: 2018-11-03 15:32:23
type: "categories"
layout: "categories"
---
```

### 新建tags
`tags`页是用来展示所有标签的页面，如果在你的博客`source`目录下还没有`tags/index.md`文件，使用如下命令创建:

``` bash
$ hexo new page "tags"
```
编辑你刚刚新建的页面文件`/source/tags/index.md`，添加以下内容:
```yml
---
title: 标签
date: 2018-11-03 14:45:54
type: "tags"
layout: "tags"
---
```

### 新建about
`about`页是用来展示关于我和我的博客信息的页面，如果在你的博客`source`目录下还没有`about/index.md`文件，使用如下命令创建:

``` bash
$ hexo new page "about"
```
编辑你刚刚新建的页面文件`/source/about/index.md`，添加以下内容:
```yml
---
title: about
date: 2018-11-03 15:48:12
type: "about"
layout: "about"
---
```

### 代码高亮
由于Hexo默认的高亮主题不好用,所以我使用了`hexo-prism-plugin`的`Hexo`插件来做代码高亮,安装命令如下:
```bash
npm install hexo-prism-plugin --save
```
然后修改根目录下的`_config.yml`文件,修改如下:
```yml
highlight:
  enable: false

prism_plugin:
  mode: 'preprocess'    # realtime/preprocess
  theme: 'tomorrow'
  line_number: false    # default false
  custom_css:
```

### 站内搜索
使用`Hexo`插件`hexo-generator-search`来做站内搜素,安装命令如下:
```bash
npm install hexo-generator-search --save
```
然后修改根目录下的`_config.yml`文件,修改如下:
```yml
search:
  path: search.json
  field: post
```

### 添加RSS订阅支持
使用`Hexo`插件`hexo-generator-feed`,安装命令如下:
```bash
npm install hexo-generator-feed --save
```
然后修改根目录下的`_config.yml`文件,修改如下:
```yml
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
  order_by: -date
```

### 修改社交链接
修改`themes/hexo-theme-matery/layout/social-link.ejs`,将社交链接改成自己的相关信息即可.

### 文章的Front-matter示例
```bash
---
title: Hexo
date: 2018-09-07 09:25:00
top: true # 如果top值为true，则会是首页推荐文章
categories: 随笔
tags:
  - Hexo 
  - Markdown
---
```

### 本地运行
**运行前先在根目录下执行命令`npm install`**
``` bash
$ hexo server | hexo s
```

### 生成静态文件

``` bash
$ hexo generate | hexo g
```

### 部署
**请先在Github配置SSH key,然后在根目录的`_config.yml`文件中进行如下的配置:**
```yml
deploy:
  type: git
  repo: 个人博客在GitHub上的repository,如 git@github.com:xxx/xxx.github.io.git 
  branch: master
```
下载`Hexo`的`git`插件,安装命令如下:
```bash
npm install --save hexo-deployer-git
```

``` bash
$ hexo deploy | hexo d
```
