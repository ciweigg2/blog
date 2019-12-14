title: git可视化工具gitkraken
date: 2019-08-24 17:55:46
tags: [git,gitkraken]
categories: [综合]
---
### 介绍

是不是一直在找一款git可视化工具呢 有很多比如sourcetree也不错 但是gitkraken是几乎最好用的啦

<!--more-->

合并代码 解决冲突 界面都非常的人性化呀

Glo Boards集成到gitkraken client中的话 可以记录任务动态接收issues

地址：https://www.gitkraken.com/git-client

那么让我们看看怎么个牛逼呀

### clone 项目到本地

![](/images/886183-20171115205614546-519985779.gif)

### 添加 REMOTE 关联到团队远程仓库

![](/images/886183-20171115205755062-1186236945.gif)

### 切换到 dev 分支

![](/images/886183-20171115205812906-735356286.gif)

### 提交commit到自己的远程仓库

![](/images/886183-20171115205929546-940928059.gif)

### 和团队远程保存同步

![](/images/886183-20171115205943812-1402804833.gif)

### push 到自己的远程仓库 & 请求 pull request 到团队远程

![](/images/886183-20171115205953156-252441323.gif)

### 查看某次 commit 的文件改动

使用 Gitkraken 能非常方便的看到任意一次的 commit 对项目文件的改动

具体操作是：在树状分支图上单击某个 commit 记录，在右侧会显示出此次 commit 信息、文件的改动情况（分为红、绿、黄三种标志，分别代表删除文件、添加文件、修改文件）

![](/images/886183-20171116213813734-1277722761.jpg)

当单击某个改动的文件时，会展示出具体的变更情况，可选择 “Diff View” 视图查看与上一次 commit 的差别，或 “File View” 视图查看完整文件

![](/images/886183-20171116213819874-799571790.jpg)

### 修改 commit 信息

当一不小心手抖误写和提交了一个 commit 信息之后，可以在树状分支图中选中此 commit 记录（必须是最新一 次commit ，旧的 commit 信息不允许修改），点击右侧上方的 commit 信息进行修改，然后点击下方 “Update Message” 提交修改

![](/images/886183-20171116213835015-1627367694.gif)

### 合并多次 commit 信息

当从团队项目分支 Merge 到本地时， Git 会自动产生一条形如 “Merge remote-tracking branch 'upstream/dev' into dev ” 的 commit 信息，提交到团队项目上去就会有一大堆这种 merge 信息混杂在有用的 commit 记录里。这时候就可以用合并 commit 信息得以解决。

在Gitkraken中的具体操作是：在树状分支图的某个 commit 记录上右键，选择 “Reset dev to this commit” -> “Soft - keep all changes”。成功后，所有新于此条 commit 的信息都会被抹去，但文件的修改还保留着

![](/images/886183-20171116213846468-810382532.gif)

### 回到旧版本 commit 记录并 push 到远程仓库

当一不小心把某个代码改崩了且已经传到了远程仓库，想要回退到旧版本的某次 commit 记录并将此次旧记录 push 到远程时，可以做如下操作得以解决：

在树状分支图的某个 commit 记录上右键，选择 “Reset dev to this commit” -> “Hard - discard all changes”。成功后，所有新于此条 commit 的操作都会被抹去，包括对文件的修改。然后点击上方菜单栏的 push 推到远程仓库，但由于 head 指针滞后，选择 Force 强制把远程项目更为旧版本

![](/images/886183-20171116213903109-841259697.gif)

### 解决 merge 到本地时的冲突

当你在本地修改了代码文件时，队友可能修改了同一份代码，这时候从团队项目分支 merge 到本地时，就可能会产生冲突。此时当在团队远程分支右键点击 merge 时，Gitkraken会检测出 conflict ，这时候只要选择 “View conflict file” 就可以打开冲突文件的 diff 视图，通过勾选方框选择保留冲突部分的哪个版本，确定后就可以得到最下方的 Output 示意的最终合并后的文件

![](/images/886183-20171116223157843-906151243.jpg)

参考：

* https://www.cnblogs.com/thousfeet/p/7840932.html

* https://www.cnblogs.com/thousfeet/p/7846635.html