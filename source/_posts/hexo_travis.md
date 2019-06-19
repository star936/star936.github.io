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
   travis encrypt 'GH_TOKEN=<GitHub Token>' --add
   ```
   此时在`.travis.yml`中存在如下内容:
   
   ```yaml
    env:
      global:
        secure: <加密后的内容>
   ```

## 3. Hexo集成Travis CI

**修改博客项目根目录下的`_config.yml`文件，修改内容如下：**

```yaml
deploy:
  type: git
  repo: https://github.com/star936/star936.github.io.git  # 修改为自己的
  branch: master
```

**向`.travis.yml`文件添加内容,完整内容示例如下:**

```yaml
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: required
# 指定node_js版本
node_js: stable
git:
  submodules: false
before_install:
  - npm install -g hexo-cli
install:
  - npm install
script:
  - git submodule init
  - git submodule update
  - hexo generate
after_success:
  - git config --global user.name "<GitHub用户名>"
  - git config --global user.email "<GitHub邮箱>"
  - sed -i'' "/^ *repo/s~github\.com~${GH_TOKEN}@github.com~" _config.yml
  - hexo deploy
branches:
  only:
  - hexo
env:
  global:
    secure: <加密后的内容>
```

## 4. Build

**将上述过程中产生的`.travis.yml`文件提交到代码库,即可触发Travis CI的build任务,并在成功build后将博客部署完成.**