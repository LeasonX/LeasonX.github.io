---
layout: post
title: Run Jekyll At Localhost
date:   2018-02-11
categories: "readme"
tags: [jekyll, ruby]
author: Leason
comment: false
---

如何在本地使用jekyll预览本项目.


## Steps

---

- OS : `Ubuntu Desktop 16.04`
- 安装 `curl` : ```sudo apt-get install curl(安装过可忽略) ```
- 安装 `gnupg2` : ```sudo apt-get install gnupg2```
- 安装 `rvm` : ```sudo curl -L https://get.rvm.io | bash -s stable```
- 安装 `ruby` : ```rvm install ruby``` (可能会出现[404](https://github.com/rvm/rvm/issues/3411)类似的错误,这时需要你修改`/etc/apt/sources.list`文件去掉报404的组件源，或者更简单的去更新源设置界面去掉404的组件源)
- 运行 ```/bin/bash --login```
- 进入博客git目录
- 运行 ```gem install bundler```
- 运行 ```bundle install```(我在安装过程中json包安装一直报错,然后我自己运行```gem install json```,因为安装之后的json版本为最新版本,接着需要更改Gemfile.lock文件中的json版本为安装时的版本,如果你有类似问题同理)
- 最后如果成功就可以直接运行`bundle exec jekyll server`,根据控制台信息打开对应的链接了(默认为[http://127.0.0.1:4000](http://127.0.0.1:4000))。


## Last
如果你有更多关于这个方面的问题,可以联系我[15736871271@163.com]

