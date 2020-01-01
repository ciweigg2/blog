---
title: 高频使用的 Git 命令集合
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
  - springboot
  - git
categories:
  - 综合
originContent: ''
date: 2019-12-28 23:45:56
---

### 前言

汇总下我在项目中高频使用的git命令及姿势

<!--more-->

不是入门文档，官方文档肯定比我全面，这里是结合实际业务场景输出

使用的 Git版本：git version 2.24.0

### git log

查看日志，常规操作，必备

```bash
# 输出概要日志,这条命令等同于
# git log --pretty=oneline --abbrev-commit
git log --oneline

# 指定最近几个提交可以带上 - + 数字
git log --oneline -5

# 提供类似 GUI 工具的 log 展示
git log --graph --date=relative --pretty=tformat:'%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%an %ad)%Creset'
```

![image.png](/images/2019/12/28/6b96a520-2987-11ea-9c54-0994549fb268.png)

### git status

查看工作区状态的东东，不如GUI直观，但是命令行也有一些用的

```bash
# 等同 git status --long,查看当前工作区暂存区变动
git status

# 概要信息 (--short)
git status -s

# 查询工作区中是否有stash存在（暂存的东西）,有则提醒该工作区有几个 stash
git status  --show-stash
```

### git checkout

用来切换到对应记录的,可以基于分支,提交,标签。

切提交和标签一般用来热修复或者老版本需要加新特性。

```bash
# 分支切换
git checkout dev # local branch

# 切换远程分支
git checkout origin/test # remote branch

# 基于远程分支创建本地分支，并跟踪对应来自 'origin' 的远程分支
git checkout --track origin/feature-test # new local branch wih remote branch

# 基于本地分支开出新分支
git checkout -b testbranch # new local branch with current branch

# 彻底丢弃某个文件的改动
git checkout -- file

# 放弃本地所有改动
git checkout .

# 切换上一个分支
git checkout -
```

### git commit

天天打交道的命令，这里说一些很常见的姿势

```bash
# 新修改的内容,添加到上次提交中,减少提交的日志
# --no-edit:是跳过进入编辑器，直接提交
# git commit --amend这条命令等同于
# $ git reset --soft HEAD^
# $ ... do something  tree ...
# $ git commit -c ORIG_HEAD
git commit --amend --no-edit

# 跳过校验直接提交,包括任何 githooks
git commit --no-verify -m "xxx"

# 带提交概要信息
git commit -m "xxx"

# 指定目录格式提交
# -t <file>, --template=<file>
# 也可以从全局或者项目级别指定提交的模板文件
# git config [--global] commit.template xxx
# 现在一般都是 用社区的npm规范包，commitizen 和 commitlint 来规范提交
git commit -t templateFile

# 提交信息从文件读取,可以结合上面的一起用
git commit -F
```

### git reset

不得不说，代码回滚中这个命令也是用的很多，而且是 --hard

```bash
# 硬性回滚,简单粗暴，直接抛弃回滚之后改动(log 还是有保留，内容不要而已)
git reset --hard commit_sha1

# 软性回滚, 跟 rebase 常规用法差不多的效果，可以把提交的东西丢回暂存区和工作区，
# HEAD 的指向改变会对应的 commit,之后再考虑怎么 commit
git reset --soft commit_sha1

# 软回滚一个版本,可以理解为撤销最近一次的 commit
git reset --soft HEAD~1

# 清除暂存区但保留工作区变动。
git reset --mixed commit_sha1

# 保留工作区和暂存区之间的差异。
git reset --merge commit_sha1

# 保留工作区和HEAD之间的差异
git reset --keep commit_sha1
```

### git revert

一般用于master 的代码回滚，因为多人在上面协作，

revert 可以平稳的回滚代码,但却保留提交记录,不会让协作的人各种冲突！

```bash
# 回滚到某个 commit
git revert commit-sha1
```

### git rebase

变基在项目中算是很频繁的，为什么这么说。

比如你开发一个新的 feature, 遵循最小化代码提交的理念。

在整个功能开发完毕的时侯，会有非常多的 commit，用 rebase 可以让我们的commit记录很干净

```bash
# 带 -i 可以进入交互模式，效果如下
git rebase -i git-sha1|branch(HEAD)

# 若是中间毫无冲突，变基则一步到位，否则需要逐步调整。
git rebase --continue # 提交变更后继续变基下一步
git rebase --skip # 引起冲突的commits会被丢弃，continue提示没有需要改动的也可以用这个跳过
git rebase --abort # 若是变基改残废了，但是走到一半，可以彻底回滚变基之前的状态
```

![image.png](/images/2019/12/28/1e114c90-2989-11ea-9c54-0994549fb268.png)

- pick: 是保留该 commit(采用)
- edit: 一般你提交的东西多了,可以用这个把东东拿回工作区拆分更细的 commit
- reword: 这个可以重新修改你的 commit msg
- squash: 内容保留，把提交信息往上一个 commit 合并进去
- fixup: 保留变动内容，但是抛弃 commit msg
- drop: 用的比较少，无用的改动你会提交么！！！

突然发现截图还有几个新的行为，估计是新版本带来的，从字面上就可以看出来大体的意思, 就是把回滚和打标签这些放到变基中简化操作

> 温馨提示

- 本地提交之前，最好把基准点变为需要合并的分支，这样提交 PR/MR 的时侯就不会冲突(本地来解决冲突)
- 不要在公共分支上变基！！！一变其他协作者基本都一堆冲突！除非你们有很清晰的分支管理机制

### git merge

```bash
# --ff 是指fast-forward命令,当使用ff模式进行合并时，将不会创造一个新的commit节点。
# --no-ff,保留合并分支的提交记录,一般主干用的比较多.
# --ff-only 除非当前HEAD节点为最新节点或者能够用ff模式进行合并，否则拒绝合并并返回一个失败状态。
# --squash 则类似 rebase squash,可以把合并多个 commit 变成一个
git merge --no-ff branchName
```

### git pull

git pull中用的最多是带--rebase(-r)的方式(变基形式拉取合并代码),保持分支一条线。

默认的pull会走ff模式,多数情况会产生新的commit,部分参数与 merge提供一致。

### git push

当本地分支存在，远程分支不存在的时侯，可以这样推送关联的远程分支

```bash
# 这样会直接新建一个同名的远程分支
git push origin localbranch 


# 删除远程分支(--delete)
git push -d origin branchName

# 推送所有标签
git push --tags

# 推送 commit 关联的 tags
git push --follow-tags

# 强制推送(--force)
git push -f origin branchName # 一般合理的项目，主干都做了分支保护,不会允许强推行为

# 有时候真的需要强推的时侯,但可不可以柔和一点呢？
# 就是当前远程分支和你本地一致,没有别人提交的情况下可以强推
# --force-with-lease: 若是远程有人提交，此次强推失败，反之成功
git push --force-with-lease
```

### git remote

这个东西用在你需要考虑维护多个地方仓库的时侯会考虑，或者修改仓库源的时侯

```bash
# 常规关联本地 git init 到远程仓库的姿势
git remote add origin url

# 新增其他上游仓
git remote add github url

# 修改推送源
git remote set-url  origin(或者其他上游域) url
```

### git branch

该命令用的最多的就是删除本地分支，重命名分支，删除远程分支了

```bash
# 分支删除，拷贝，重命名，参数若是大写就等同多了--force，强制执行
# -c, --copy : 复制分支，
# -C：等同于 --copy --force
# -d, --delete: 删除分支
# -m, --move：移动或者重命名分支
git branch -d branchName
git branch -M oldBranch newNameBranch


# 手动指定它的当前分支的上游分支,两个写法一致的
# 有关联一般也有取消关联，--unset-upstream
git branch --set-upstream-to=origin/xxx 
git branch --set-upstream-to origin xxx
```

### git stash

暂存用的最多时侯就是你撸代码撸到一半，突然说有个紧急 BUG 要修正。

或者别人在你这里需要帮忙排查代码，你这时候也会用到。

强烈建议给每个 stash 添加描述信息！！！

```bash
# 缓存当前工作区的内容到stashName, save 这个现在不怎么推荐用，图方便也能用
# -a|--all: 除了未跟踪的文件，其他变动的文件都会保存
# -u|--include-untracked：包括没有添加到暂存区的文件
git stash save stashName
git stash -u save stashName

# 现在基本推荐用 push,因为有 pop，语义上和维护上更清晰
# 上面有的参数，它也有，还有-m 来备注这个 stash 的大概情况
git stash push -m "更改了 xx" 

# 有保存那肯定也有取用的
# pop: 取会删除对应的保存记录
# apply: 取但保留记录
# 0就是--index,这个东西哪里来的呢？
git stash apply stash@{0}
git stash pop stash@{0}

# 查看stash 保存记录
# eg: stash@{0}: On dev: 测试
git stash list


# 只想删除暂存记录怎么办：
# clear : 清空所有 stash
# drop: 清除指定的 stash
git stash clear # 慎用！
git stash drop stash@{0}

# 想看 stash 做了什么改动，类似简化版的git diff
git stash show stash@{0}
```

### git reflog

这个命令的强大之处，是记录了所有行为，包括你 rebase,merge, reset 这些

当我们不小心硬回滚的时侯,或变基错了都可以在这里找到行为之前的commit，然后回滚。

当然这个时间回溯也只在本地有用，你推送到远程分支的破坏性改动,该凉还是得凉。

```bash
# 最近五次行为,不带-n 则默认所有
git reflog -5
```

### git cherry-pick

这个东西你可以理解为你去买橘子，你会专门挑一些符合心意的橘子放到购物篮中。你可以从多个分支同时挑取部分需要的 commit 合并到同一个地方去，是不是贼骚。这货和变基有点类似，但是仅仅类似，挑过来的 commit 若是没有冲突则追加。有冲突会中断，解决后 --continue

```bash
# 在当前分支挑其他分支的 commit，把那部分的变动那过来
git cherry-pick commit-sha1

# 支持一次性拿多个
git cherry-pick master~4 master~2

# 支持区间, 区间中间是 .. 
git cherry-pick startGitSha1..endGitSha1

# --continue：继续 pick,一般有冲突解决后才需要这样
# --skip：跳过这次进入队列下一次行为 
# --abort : 完全放弃 pick，恢复 pick 之前的状态
# --quit: 未冲突的自动变更，冲突的不要，退出这次 pick
# 这几个状态跟变基差不多,解决冲突继续，跳过处理，放弃这次pick,不输出错误
```

### git rm

这个命令在旧的版本用的比较最多的姿势是为了重新索引.gitignore 的范围

```bash
# 删除某个文件的索引
# --cache 不会删除硬盘中的文件，只是 git 索引(缓存)的关系！！！
git rm --cache -- file

# 递归清除全部所有索引(也可以理解为缓存吧),这个姿势适合重新让.gitignore 新范围生效
git rm -r --cached .  
git add .
git commit -m "xxx"
```

### git rev-parse

这个估计一般人用的不是很多，可以通过这个快速获取部分git 仓库的信息

我在弄脚本的时侯就会从这里拿东西

```bash
# 获取最新有效的commit
# --short：显示七位的 sha1,不带就是全部
# --verify: 校验是否有效commit
# HEAD: 当前分支的head 指向
git rev-parse --short HEAD --verify

# 显示仓库的绝对路径
git rev-parse --show-toplevel #eg: /Users/linqunhe/Code/aozhe/thinking-ui

# 显示版本库.git 目录所在的位置
git rev-parse --git-dir

# 显示所有关联引用的 git sha1
git rev-parse --all
```

### git diff

对于这个命令，在终端比对用的不是很频繁，除了少量改动的时侯可能会用这个看看。

其他情况下我更倾向于用 GUI 工具来看，因为比对更加直观。

### 总结

git 的常用命令其实很好掌握，很多命令都有 Linux 的影子。

列出来的命令都是高频使用的，或许有一些更骚的姿势没有摸索到

一直在学习从未放弃 gl ciwei ^_^ 加油~