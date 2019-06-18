---
title: 使用Travis CI自动化发布Hexo博客
date: 2019-06-18 22:00:28
tags: [Hexo, Travis]
categories: 随笔
---

## 1. Travis登陆

**打开[Travis Ci](https://www.travis-ci.org/),使用Github账号授权登录即可.**

## 2. 准备

**Hexo和Travis CI集成需要使用到GitHub的Token,处于安全考虑,我们需要对其进行加密.**

*以下操作请在个人博客项目的根目录下进行*

1. 安装Travis CI的命令行工具`travis`;
2. 创建`.travis.yml`文件,
   
   ```bash
   touch .travis.yml
   ```

3. 拷贝GitHub Token,并使用如下命令对其进行加密:
   ```bash
   travis encrypt 'REPO_TOKEN=<GitHub Token>' --add
   ```
   此时在`.travis.yml`中存在如下内容:
   
   ```yaml
    env:
      global:
        secure: <加密后的内容>
   ```

## 3. Hexo集成Travis CI

**向`.travis.yml`文件添加内容,完整内容示例如下:**

```yaml
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: required
# 指定node_js版本
node_js: stable
# 指定缓存模块，可选。缓存可加快编译速度。
cache:
  directories:
    - node_modules

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - hexo 

before_install:
  - npm install -g hexo-cli

# Start: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save

# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo generate

# 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
after_script:
  - git config user.name "<GitHub用户名>"
  - git config user.email "<GitHub邮箱>"
  # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
  - sed -i "s/gh_token/${REPO_TOKEN}/g" ./_config.yml
  - hexo deploy
# End: Build LifeCycle
env:
  global:
    secure: <加密后的内容>
```

## 4. Build

**将上述过程中产生的`.travis.yml`文件提交到代码库,即可触发Travis CI的build任务,并在成功build后将博客部署完成.**