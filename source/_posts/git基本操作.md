cover: http://ciwei2.cn-sh2.ufileos.com/40.jpg
title: git基本操作
date: 2018-09-12 10:24:49
tags: [git]
categories: [综合]
---
## git基本操作：
1. git怎么新建分支并且提交代码 创建还原点
2. git怎么切换远程新建的分支(刷新的操作)

<!--more-->

### git新建分支并提交代码(还原点恢复以前代码)

![](/images/idea的新建分支.png)

然后修改个代码提交上去 新的分支就建立了

![](/images/git分支.png)

如果提交代码的时候 本地有修改过很多代码 新分支需要这些修改 老的分支不需要 那么我们需要打补丁创建还原点

master修改了代码 这些代码要提交到test分支 master稍后会还原 怎么操作呢

![](/images/master分支修改代码.png)

![](/images/新建和还原.png)

![](/images/创建补丁.png)

切换到test分支然后提交代码(提交代码省略)

![](/images/idea中切换分支.png)

切换master分支 还原补丁 点击ok

![](/images/idea的还原代码.png)

![](/images/导入成功后的还原点.png)

接下来该还原该提交都可以正常操作了

### git切换远程新建的分支(刷新的操作)

![](/images/idea的刷新分支.png)

接下来本地就有这个分支了 可以直接切换了
