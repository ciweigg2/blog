---
title: Linux上，最常用的一批命令解析
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
  - linux
categories:
  - 综合
originContent: ''
date: 2020-01-05 20:03:04
---

Linux这么多命令，通常会让初学者望而生畏。下面是我结合日常工作，以及在公司的内部培训中，针对对Linux不是很熟悉的同学，精选的一批必须要搞懂的命令集合

<!--more-->

任何一个命令其实都是可以深入的，比如tail -f和tail -F的区别。我们不去关心，只使用最常见的示例来说明。本文不会教你具体的用法，那是抢man命令的饭碗。这只是个引导篇，力求简洁。

**学习方式：多敲多打，用条件反射替代大脑记忆–如果你将来或者现在要用它来吃饭的话。** 其中，也有一些难啃的骨头，关注小姐姐味道微信公众号，我们一起用锋利的牙齿，来把它嚼碎。
内容：
✔ 目录操作
✔ 文本处理
✔ 压缩
✔ 日常运维
✔ 系统状态概览
✔ 工作常用

## 目录操作

工作中，最常打交道的就是对目录和文件的操作。linux提供了相应的命令去操作他，并将这些命令抽象、缩写。

### 基本操作

可能是这些命令太常用了，多打一个字符都是罪过。所以它们都很短，不用阿拉伯数字，一个剪刀手就能数过来

![image.png](/images/2020/01/05/fae72d40-2fb2-11ea-a9cb-bb9279b30331.png)

看命令
mkdir 创建目录 make dir cp 拷贝文件 copy mv 移动文件 move rm 删除文件 remove
例子：

```bash
# 创建目录和父目录a,b,c,d
mkdir -p a/b/c/d

# 拷贝文件夹a到/tmp目录
cp -rvf a/ /tmp/

# 移动文件a到/tmp目录，并重命名为b
mv -vf a /tmp/b

# 删除tmp目录的所有文件
rm -rvf /tmp/
```

### 漫游

linux上是黑漆漆的命令行，依然要面临人生三问：我是谁？我在哪？我要去何方？
ls 命令能够看到当前目录的所有内容。ls -l能够看到更多信息，判断你是谁。
pwd 命令能够看到当前终端所在的目录。告诉你你在哪。
cd 假如你去错了地方，cd命令能够切换到对的目录。
find find命令通过筛选一些条件，能够找到已经被遗忘的文件。
至于要去何方，可能就是主宰者的意志了。

### 文本处理

这是是非常非常加分的技能。get到之后，也能节省更多时间来研究面向对象

![image.png](/images/2020/01/05/13ae1a50-2fb3-11ea-a9cb-bb9279b30331.png)

### 查看文件

**cat**
最常用的就是`cat`命令了，注意，如果文件很大的话，cat命令的输出结果会疯狂在终端上输出，可以多次按`ctrl+c`终止。

```bash
# 查看文件大小
du -h file

# 查看文件内容
cat file
```

**less**
既然cat有这个问题，针对比较大的文件，我们就可以使用less命令打开某个文件。 类似vim，less可以在输入/后进入查找模式，然后按n(N)向下(上)查找。
有许多操作，都和vim类似，你可以类比看下。

**tail**
大多数做服务端开发的同学，都了解这么命令。比如，查看nginx的滚动日志。

```bash
tail -f access.log
```

tail命令可以静态的查看某个文件的最后n行，与之对应的，head命令查看文件头n行。但head没有滚动功能，就像尾巴是往外长的，不会反着往里长。

```bash
tail -n100 access.log
head -n100 access.log
```

### 统计

sort和uniq经常配对使用。 sort可以使用-t指定分隔符，使用-k指定要排序的列。

下面这个命令输出nginx日志的ip和每个ip的pv，pv最高的前10

```bash
# 2019-06-26T10:01:57+08:00|nginx001.server.ops.pro.dc|100.116.222.80|10.31.150.232:41021|0.014|0.011|0.000|200|200|273|-|/visit|sign=91CD1988CE8B313B8A0454A4BBE930DF|-|-|http|POST|112.4.238.213

awk -F"|" '{print $3}' access.log | sort | uniq -c | sort -nk1 -r | head -n10
```

### 其他

**grep**
grep用来对内容进行过滤，带上–color参数，可以在支持的终端可以打印彩色，参数n则输出具体的行数，用来快速定位。
比如：查看nginx日志中的POST请求。

```bash
grep -rn --color POST access.log
```

推荐每次都使用这样的参数。

如果我想要看某个异常前后相关的内容，就可以使用ABC参数。它们是几个单词的缩写，经常被使用。 A after 内容后n行 B before 内容前n行 C count? 内容前后n行
就像是这样：

```bash
grep -rn --color Exception -A10 -B2   error.log
```

**diff**
diff命令用来比较两个文件是否的差异。当然，在ide中都提供了这个功能，diff只是命令行下的原始折衷。对了，diff和patch还是一些平台源码的打补丁方式，你要是不用，就pass吧。

## 压缩

为了减小传输文件的大小，一般都开启压缩。linux下常见的压缩文件有tar、bzip2、zip、rar等，7z这种用的相对较少。
**.tar** 使用tar命令压缩或解压
**.bz2** 使用bzip2命令操作
**.gz** 使用gzip命令操作
**.zip** 使用unzip命令解压
**.rar** 使用unrar命令解压
最常用的就是`.tar.gz`文件格式了。其实是经过了tar打包后，再使用gzip压缩。
**创建压缩文件**

```bash
tar cvfz  archive.tar.gz dir/
```

**解压**

```bash
tar xvfz. archive.tar.gz
```

快去弄清楚它们的关系吧。

## 日常运维

开机是按一下启动按钮，关机总不至于是长按启动按钮吧。对了，是shutdown命令，不过一般也没权限-.-!。passwd命令可以用来修改密码，这个权限还是可以有的

![image.png](/images/2020/01/05/2b3e0e00-2fb3-11ea-a9cb-bb9279b30331.png)

**mount**
mount命令可以挂在一些外接设备，比如u盘，比如iso，比如刚申请的ssd。可以放心的看小电影了。

```bash
mount /dev/sdb1 /xiaodianying
```

**chown**
`chown` 用来改变文件的所属用户和所属组。
`chmod` 用来改变文件的访问权限。

这两个命令，都和linux的文件权限777有关。
示例：

```bash
# 毁灭性的命令
chmod 000 -R /

# 修改a目录的用户和组为 xjj
chown -R xjj:xjj a

# 给a.sh文件增加执行权限（这个太常用了)
chmod a+x a.sh
```

**yum**
假定你用的是centos，则包管理工具就是yum。如果你的系统没有wget命令，就可以使用如下命令进行安装。

```bash
yum install wget -y
```

**systemctl**
当然，centos管理后台服务也有一些套路。service命令就是。systemctl兼容了service命令，我们看一下怎么重启mysql服务。 推荐用下面这个。

```bash
service mysql restart
systemctl restart  mysqld
```

对于普通的进程，就要使用kill命令进行更加详细的控制了。kill命令有很多信号，如果你在用`kill -9`，你一定想要了解`kill -15`以及`kill -3`的区别和用途。

**su**
su用来切换用户。比如你现在是root，想要用xjj用户做一些勾当，就可以使用su切换。

```bash
su xjj
su - xjj
```

`-`可以让你干净纯洁的降临另一个账号，不出意外，推荐。

## 系统状态概览

登陆一台linux机器，有些命令能够帮助你快速找到问题。这些命令涵盖内存、cpu、网络、io、磁盘等

![image.png](/images/2020/01/05/384467c0-2fb3-11ea-a9cb-bb9279b30331.png)

**uname**
`uname`命令可以输出当前的内核信息，让你了解到用的是什么机器。

```bash
uname -a
```

**ps**
ps命令能够看到进程/线程状态。和top有些内容重叠，常用。

```bash
# 找到java进程
ps -ef | grep java
```

**top**
系统状态一览，主要查看。cpu load负载、cpu占用率。使用内存或者cpu最高的一些进程。下面这个命令可以查看某个进程中的线程状态。

```bash
top -H -p pid
```

**free**
top也能看内存，但不友好，`free`是专门用来查看内存的。包括物理内存和虚拟内存swap。

**df**
`df`命令用来查看系统中磁盘的使用量，用来查看磁盘是否已经到达上限。参数h可以以友好的方式进行展示。

```bash
df -h
```

**ifconfig**
查看ip地址，不啰嗦，替代品是ip addr命令。

**ping**
至于网络通不通，可以使用ping来探测。（不包括那些禁ping的网站）

**netstat**
虽然ss命令可以替代netstat了，但现实中netstat仍然用的更广泛一些。比如，查看当前的所有tcp连接。

```bash
netstat -ant
```

此命令，在找一些`本地起了什么端口`之类的问题上，作用很大。

## 工作日常

还有一些在工作中经常会用到的命令，它们的出现频率是非常高的 ，都是些熟面孔

![image.png](/images/2020/01/05/447e70d0-2fb3-11ea-a9cb-bb9279b30331.png)

**export**
很多安装了jdk的同学找不到java命令，`export`就可以帮你办到它。`export`用来设定一些环境变量，env命令能看到当前系统中所有的环境变量。比如，下面设置的就是jdk的。

```bash
export PATH=$PATH:/home/xjj/jdk/bin
```

有时候，你想要知道所执行命令的具体路径。那么就可以使用whereis命令，我是假定了你装了多个版本的jdk。

**crontab**
这就是linux本地的job工具。不是分布式的，你要不是运维，就不要用了。比如，每10分钟提醒喝茶上厕所。

```bash
*/10 * * * * /home/xjj/wc10min
```

**date**
`date`命令用来输出当前的系统时间，可以使用-s参数指定输出格式。但设置时间涉及到设置硬件，所以有另外一个命令叫做`hwclock`。

**xargs**
`xargs`读取输入源，然后逐行处理。这个命令非常有用。举个栗子，删除目录中的所有class文件。

```bash
find . | grep .class$ | xargs rm -rvf

#把所有的rmvb文件拷贝到目录
ls *.rmvb | xargs -n1 -i cp {} /mount/xiaodianying
```

### 网络

linux是一个多作业的网络操作系统，所以网络命令有很多很多。工作中，最常和这些打交道。
**ssh**
这个，就不啰嗦了。你一定希望了解`ssh隧道`是什么。你要是想要详细的输出过程，记得加参数-v。
**scp**
`scp`用来进行文件传输。也可以用来传输目录。也有更高级的sftp命令。

```bash
scp a.txt 192.168.0.12:/tmp/a.txt
scp -r a_dir 192.168.0.12:/tmp/
```

**wget**
你想要在服务器上安装jdk，不会先在本地下载下来，然后使用scp传到服务器上吧（有时候不得不这样）。wget命令可以让你直接使用命令行下载文件，并支持断点续传。

```java
wget -c http://oracle.fuck/jdk2019.bin
```

**mysql**
`mysql`应用广泛，并不是每个人都有条件用上`navicat`的。你需要了解mysql的连接方式和基本的操作，在异常情况下才能游刃有余。

```bash
mysql -u root -p -h 192.168.1.2
```

## End

不要觉得复杂，命令是有限的，但激情无限；都会也不要骄傲，一个vim就够折腾一辈子。捷径就是总结，深入只有探索。白马过隙，终会行云流水，手到擒来。
物是人非，年华易老。唯有时光，不会辜负