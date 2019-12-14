cover: http://ciwei2.cn-sh2.ufileos.com/93.jpg
title: rancher安装kubernetes集群
date: 2019-06-22 21:32:33
tags: [rancher,kubernetes]
categories: [综合]
---
### 安装docker

docker最新版本为18.09.6

<!--more-->

安装过程不快的哟

```java
添加yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

保存到路径：repo saved to /etc/yum.repos.d/docker-ce.repo

更新yum索引
yum makecache fast

安装 docker-ce
yum install docker-ce
```

### 安装rancher

详细数据库安装参考：https://www.cnrancher.com/quick-start

```java
sudo docker run -d -v <主机路径>:/var/lib/rancher/ --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:stable
```

访问：https://ip 右下角可以切换中文的哟

### 安装kubernetes

点击集群 添加集群 勾选所有角色作为master管理节点呀

![](/images/20190623110503.jpg)

复制命令在管理节点执行

后面需要添加worker节点查看加入节点的命令 点击编辑集群选择worker就行啦 一定要内网是通的哦服务器一定要在一个内网

![](/images/20190623104737.png)

![](/images/20190623104635.png)

等待一段时间就可以部署完成了

### 安装kubernetes-dashboard

因为他是安装在kube-system空间的 所以需要切换到相应的空间才能安装呀

![](/images/20190623102454.png)

勾选跳过登录 我找不到令牌- -哎

![](/images/20190623102331.png)

最后切换到应用商店点击index.html跳转到界面呀

![](/images/20190623110956.png)

![](/images/20190623102250.png)

可以通过rancher界面管理kubectl命令哟

![](/images/20190623124819.png)