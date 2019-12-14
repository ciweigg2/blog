cover: http://ciwei2.cn-sh2.ufileos.com/8.jpg
title: Docker Swarm 进阶：NFS 共享数据卷
date: 2018-09-16 10:44:52
tags: [docker,swarm]
categories: [综合]
---
### 启动 NFS 服务（CentOS 7）

首先，安装 rpcbind 和 nfs-utils

```
yum install -y rpcbind nfs-utils
```

<!--more-->

然后，编辑 /etc/exports 配置 NFS 共享目录，示例：

```
[root@VM_0_5_centos ~]# cat /etc/exports
/root/share *(rw,sync,all_squash,anonuid=0,anongid=0)
```

> 更多详细配置参考 https://www.centos.bz/2017/07/centos7-1-install-nfs/

启动 rpcbind 和 nfs

```
sudo service rpcbind start

sudo service nfs start
```

查看共享的目录

```
exportfs
```

修改配置文件后，重新加载配置文件

```
exportfs -vr
```

> 视频演示：https://www.youtube.com/watch?v=_4XudYZ6M_k

### 创建 NFS 数据卷

![](/images/3424642-969494530d4af137.png)

创建一个 volume name 为 share的目录

```
docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=<NFS-Server>,rw \
    --opt device=:<Shared-Path> \
    share
```

### 创建多副本服务

将宿主机的目录挂载到容器的目录

```
docker service create \
  --mount type=volume,source=<Volume-Name>,destination=<Container-Path> \
  --replicas 2 \
  <Image>
```

Container-Path 是容器的目录

Volume-Name 是宿主机的目录，已经创建了

Image 镜像
