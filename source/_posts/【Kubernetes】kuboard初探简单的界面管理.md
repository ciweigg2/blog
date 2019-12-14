title: 【Kubernetes】kuboard初探简单的界面管理
date: 2019-08-11 18:43:57
tags: [kuboard]
categories: [综合]
---
推荐一款好玩又整洁的kubernetes监控界面管理工具

首先你得搭建kubernetes单机或者高可用 里面也有安装教程的

<!--more-->

然后部署kuboard

```
kubectl apply -f https://raw.githubusercontent.com/eip-work/eip-monitor-repository/master/dashboard/kuboard.yaml
```

获取 Token

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')   
```

http://任意一个Worker节点的IP地址:32567/

> 您也可以修改 Kuboard.yaml 文件，使用自己定义的 NodePort 端口号

然后访问修改后的端口号就行了

更多教程请参考：https://kuboard.cn/overview/

![](/images/1564841972085.adf847e9.gif)