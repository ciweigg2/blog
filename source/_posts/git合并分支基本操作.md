cover: http://ciwei2.cn-sh2.ufileos.com/39.jpg
title: git合并分支基本操作
date: 2018-09-05 10:55:07
tags: [git]
categories: [综合]
---
有时候需要在另外一个分支开发，最终合并到分支
<!--more-->

比如有2个分支 1个master 一个1.1 都修改了分支的同一行代码 需要怎么合并呢

将master的分支合并到1.1 在1.1分支的项目中

![](/images/分支2.png)

![](/images/分支3.png)

> 左边是我的版本 右边是远程的 中间是最终合并的

![](/images/git分支冲突合并例子.png)

合并后需要push

![](/images/合并后.png)

git检出分支 checkout就行了

![](/images/idea分支.png)

如果第一次没有检出过分支的话 需要New Branch 名字和远程分支一样就行了