---
title: termius备份本地数据
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - termius
categories:
  - 综合
date: 2019-12-29 13:40:32
---

termius更新后可能数据丢失的问题解决方案

我是通过ClanMyMac X 发现这个路径的 卸载程序-找到程序-点击显示找到Termius右击在访达中打开就知道路径了 这个是在不登录的情况下存储的路径 可以作为备份路径

<!--more-->

将下面路径中的Termius的文件夹复制出来

更新最新的termius版本后 替换Termius文件夹

termius 4.9.20路径

/Users/${用户名}/Library/Containers/com.termius.mac/Data/Library/Application Support/Termius

termius 5.2.3路径

/Users/${用户名}/Library/Application Support/Termius

command + 空格 进入目录 /Users/${用户名}/Library/Application Support 复制Termius文件夹完成备份

最好大版本对应吧 这样数据迁移比较好 复制文件夹过去的时候选择替换 重新启动termius 发现数据导入过来了