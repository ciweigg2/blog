cover: http://ciwei2.cn-sh2.ufileos.com/41.jpg
title: git提交commit后未push怎么还原
date: 2018-08-13 10:57:32
tags: [git]
categories: [综合]
---
还原Git上commit，但是没有push代码
<!--more-->

![](/images/git.png)

在To Commit里面填写：HEAD^，表示将commit的信息还原为上一次的

HEAD 最近一个提交
HEAD^ 上一次

![](/images/git2.png)