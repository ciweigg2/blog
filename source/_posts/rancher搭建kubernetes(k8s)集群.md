cover: http://ciwei2.cn-sh2.ufileos.com/94.jpg
title: rancher搭建kubernetes(k8s)集群
date: 2018-09-12 21:15:12
tags: [k8s,rancher,kubernetes]
categories: [综合]
---
### Install Rancher
```
$ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

### Log In
Open a web browser and enter the IP address of your host: https://<SERVER_IP>.
Replace <SERVER_IP> with your host IP address.
<!--more-->

中文官网：https://www.cnrancher.com

### Create the Cluster(创建集群)

如果都是云服务器 那必须内网是通的 所以云服务器只能从一个提供商购买

首先切换中文，右下角切换

第一台创建的是master节点

然后选择添加集群->添加主机自建Kubernetes集群->输入集群名->下一步->全部打钩（etcd，Control，Worker）->选择显示高级选项->填写公网地址ip->复制命令在所有机器上执行

node节点可以只选Worker

等待构建集群 这个过程很漫长，需要等待的

如果安装出问题 初始化节点rancher：

```
docker rm -f $(docker ps -qa)
  docker volume rm $(docker volume ls -q)
  for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
  rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico
```