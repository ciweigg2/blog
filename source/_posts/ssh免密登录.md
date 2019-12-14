cover: http://ciwei2.cn-sh2.ufileos.com/123.jpg
title: ssh免密登录
date: 2018-09-16 10:32:54
tags: [ssh免密登录]
categories: [综合]
---
跳板机就是一台机器作为ssh登录的机器 可以使用crt登录 其他都是内网登录

<!--more-->

修改hosts：

```
10.254.4.1 mysql
10.254.4.2 redis
10.254.4.3 mongodb
```

比如我们现在有一台机器 10.254.4.0 作为跳板机

生成秘钥：

```
ssh-keygen -t rsa
```

复制秘钥给其他内网主机：

```
ssh-copy-id -i .ssh/id_rsa.pub root@10.254.4.1

ssh-copy-id -i .ssh/id_rsa.pub root@10.254.4.2

ssh-copy-id -i .ssh/id_rsa.pub root@10.254.4.3
```

成功后 在10.254.4.0机器上就能使用免密登录其他主机了

```
ssh mysql
ssh redis
ssh mongodb
```