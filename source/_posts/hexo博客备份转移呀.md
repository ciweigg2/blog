cover: http://ciwei2.cn-sh2.ufileos.com/45.jpg
title: hexo博客备份转移呀
date: 2019-05-11T06:54:30.492Z
tags: [hexo]
categories: [综合]
---
### 博客备份安装

服务器到期了 可能要换服务器 需要备份博客 这里就是简单的操作步骤 很方便的呀

<!--more-->

> 老服务器备份

### 备份

```java
cd /root/blog/
tar -zcvf theme.tar.gz *
```

### 下载备份

```java
sz theme.tar.gz
```

> 新服务器恢复

### 安装nodejs

```java
cd /usr/local/
wget https://nodejs.org/dist/latest-v8.x/node-v8.16.0-linux-x64.tar.gz
下载完成后解压
tar zxvf node-v8.16.0-linux-x64.tar.gz

重命名为node
mv node-v8.16.0-linux-x64 node

配置环境变量
vim /etc/profile
最后加：
#set for nodejs
export NODE_HOME=/usr/local/node
export PATH=$NODE_HOME/bin:$PATH

source /etc/profile
node -v
npm -v
```

### 安装博客：

```java
npm install -g hexo-cli
cd /root/
mkdir blog
cd blog
rz 备份
tar -zxvf 备份
hexo s
安装后台启动编辑器插件呀
npm install forever -g
forever start server.js
上传到github还需要配置ssh
ssh-keygen -t rsa -C "xxxxxx@yy.com"
测试是否配置成功了
ssh -T git@github.com
生成静态文件
hexo g
提交到github
hexo d
cd /root/blog
rm -rf .git/
备份到github
忽略提交大文件
vi .gitignore 
添加
_admin-config.yml
hexo b
```