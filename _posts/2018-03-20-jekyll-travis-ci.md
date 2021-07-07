---
layout:     post
title:      "jekyll-travis-ci"
subtitle:   " \"利用travis-ci自动构建博客\""
date:       2018-03-20 18:01:25
author:     "zhouhaoo"
header-img: "img/travis/post-bg-travis.png"
tags:
    - 博客
    - travis-ci
    - jekyll
---
## jekyll
> [jekyll](https://www.jekyll.com.cn/)是将纯文本转化为静态网站和博客。

本博客使用[huxpro.github.io](https://github.com/huxpro/huxpro.github.io/)编写的主题，感谢大神。
详细使用教程见[Hux Blog](https://github.com/Huxpro/huxpro.github.io/blob/master/README.md)。本文主要介绍travis-ci的集成。
## travis-ci
> [**travis-ci**](https://travis-ci.org)是一个自动构建的工具,分布式的持续集成服务，用来构建及测试在GitHub托管的代码。

注册可以使用github授权，注册 ，登录成功之后，添加仓库。
### 构建方案
1. 新建一篇markdown文章。
2. 编写完成后，push和文章相关的文件（md，img下图片）到gh-pages分支。
3. travis-ci监听到分支数据变化时，按照.travis.yml开始构建。
4. 构建完成后，travis-ci将生成的_site文件夹下的文件push到master的分支。

### 脚本

1. 新建.travis.yml文件

```
language: ruby
rvm:
  - 2.3.3
#cache: bundler
before_install:
 
script:
  - bundle install
  - bundle exec jekyll build
after_success:
  - mv CNAME README.md -t _site/
  - cd _site
  - git init
  - git config user.name "zhouhaoo"
  - git config user.email "zhouhao7@icloud.com"
  - git add --all .
  - git commit -m "Travis-CI auto builder"
  - git remote add origin https://$DEPLOY_TOKEN@github.com/zhouhaoo/zhouhaoo.github.io.git
  - git remote show origin
  - git push --force origin master
#  - git push origin master
branches:
  only:
  - gh-pages
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true

```

DEPLOY_TOKEN 变量需要在github->settings->Developer settings->Personal access tokens 里面新建一个token，
添加到travis中。

github新建access tokens。
 ![](/img/travis/travis-github-token.png)
travis-ci添加，如图。
 ![](/img/travis/travis-vaiables.png)


 2. 拷贝Gemfile 文件到根目录。

### 总结 
主要在脚本文件和拷贝Gemfile，脚本照着改成自己github名字就可以。