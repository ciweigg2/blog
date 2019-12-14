cover: http://ciwei2.cn-sh2.ufileos.com/149.jpg
title: 安装nfs服务器
date: 2019-06-18 14:50:29
tags: [nfs]
categories: [综合]
---
### 介绍

kubernetes 挂载目录多节点共享问题 需要通过nfs解决

<!--more-->

举例：如果tomcat部署war需要挂载webapps目录的

### 安装nfs

使用一台服务器作为nfs服务器

```java
#通过yum目录安装nfs服务和rpcbind服务：
yum install nfs-util bind
yum install nfs-utils rpc-bind -y
yum install rpcbind -y

#查看状态
systemctl status rpcbind.service

#检查nfs服务是否正常安装 
rpcinfo -p localhost
```

#### 创建用户

```java
useradd -u nfs
mkdir -p /nfs-share
chmod a+w /nfs-share
```

#### 配置共享目录

```java
#在nfs服务器中为客户端配置共享目录，*所有地址都能访问nfs服务器
echo "/nfs-share *(rw,async,no_root_squash)" >> /etc/exports

#通过执行如下命令是配置生效：
exportfs -r
```

创建多个目录 最后需要重新执行 exportfs -r

```java
mkdir -p /nfs-share2
chmod a+w /nfs-share2
exportfs -r
```

#### 启动服务

```java
#由于必须先启动rpcbind服务，再启动nfs服务，这样才能让nfs服务在rpcbind服务上注册成功：
systemctl start rpcbind

#启动nfs服务： 
systemctl start nfs-server

#设置rpcbind和nfs-server开机启动： 
systemctl enable rpcbind
systemctl enable nfs-server
```