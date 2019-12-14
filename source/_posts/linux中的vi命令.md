cover: http://ciwei2.cn-sh2.ufileos.com/77.jpg
title: linux中的vi命令
date: 2019-06-30 11:57:13
tags: [linux]
categories: [综合]
---
一直觉得vi命令不好用介绍一下工作中常用的

```java
G 移至行行首
nG 移至第n行行首
n+ 移n行行首
n- 移n行行首
n$ 移n行(1表示本行)行尾
0 所行行首
$ 所行行尾
^ 所行首字母
h,j,k,l 左移移移右移
H 前屏幕首行行首
M 屏幕显示文件间行行首
L 前屏幕底行行首
/ 搜索内容
```

过滤显示文件的内容

```java
egrep "内容" xxx.txt
```

还有很多需要记录的