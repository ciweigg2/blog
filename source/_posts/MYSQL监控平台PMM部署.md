cover: http://ciwei2.cn-sh2.ufileos.com/89.jpg
title: MYSQL监控平台PMM部署
date: 2019-06-21 11:11:54
tags: [mysql,pmm]
categories: [综合]
---
### PMM简介

> Percona Monitoring and Management (PMM)是一款开源的用于管理和监控MySQL和MongoDB性能的开源平台，通过PMM客户端收集到的DB监控数据用第三方软件Grafana画图展示出来。在这个产品之前，Percona提供了Zabbix和Cacti的图形模板，也许是考虑到了用户部署起来繁琐等问题，Percona发布了PMM Docker镜像，用户只需要下载镜像运行就全部搞定，开箱即用相当easy

<!--more-->

### PMM官方文档资源获取

> https://learn.percona.com/hubfs/Manuals/Percona_Monitoring_And_Management_PMM/PMM%201.17/Percona-Monitoring-And-Management-1.17.0.pdf

### PMM Server端安装部署

* 我们的部署是基于docker容器的方式实现，简单高效

注意，需要开启linux内核的ip_forward转发功能

获取PMM Server安装脚本并安装

```java
没有设置用户名和密码的
#curl -fsSL https://raw.githubusercontent.com/percona/pmm/master/get-pmm.sh -o get-pmm.sh
#sh get-pmm.sh
```

* 根据需要,调整脚本参数，比如监听服务的监听端口以及开启Grafana的登录验证功能等，脚本参考如下： 

可以修改端口和添加用户名密码

```java
    run_docker run -d \
        -p 80:80 \
        --volumes-from pmm-data \
        --name pmm-server \
        --restart always \
        percona/pmm-server:latest
    -e SERVER_USER=ciwei
    -e SERVER_PASSWORD=password
```

访问其前端Grafana页面访问是否正常

http://$PMM-SERVER-IP:PORT

* 登录的需要的用户和密码就是你脚本里面设置的两个参数值
用户名：ciwei
密码：password

### PMM Client端安装部署

* 获取PMM Client客户端软件包，并安装

```java
wget https://www.percona.com/downloads/pmm/1.17.0/binary/redhat/7/x86_64/pmm-client-1.17.0-1.el7.x86_64.rpm

rpm -ivh pmm-client-1.17.0-1.el7.x86_64.rpm
```

* 连接PMM Client到PMM Server 

```java
未置用户名和密码的连接方式
pmm-admin config --server $PMM-SERVER-IP:PORT
设置用户名和密码的连接方式
pmm-admin config --server $PMM-SERVER-IP:PORT --server-user abc --server-password password
```

* 添加mysql监控

```java
pmm-admin add mysql --user root --password 123456 --host $IP --port $PORT
```

其实就是添加监控实例，我们需要监控的是mysql服务，因此我们添加每个节点的mysql实例，这里的账号和密码，是你在数据库实例中建立的账号

* 验证PMM Client连接PMM Server的信息命名汇总 

```java
pmm-admin ping
pmm-admin info
pmm-admin list
pmm-admin check-network
```

![](/images/20190621111110.png)