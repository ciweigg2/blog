title: 【Apollo】docker安装Apollo
date: 2019-08-09 23:11:43
tags: [apollo]
categories: [apollo]
---
### 安装

机器配置要求:4核16G 或者 2核8G都可以

我们使用docker-compose安装 修改${数据库id:3306} 和 ${当前主机ip}

<!--more-->

vi docker-compose.yml

```
version: '3.4'
services:
 configservice:
  image: juntao2000/apollo-configservice:1.4.0
#  image: ciwei123321/apollo-configservice:1.5.0
  environment:
  - spring_datasource_url=jdbc:mysql://${数据库id:3306}/ApolloConfigDB?characterEncoding=utf8
  - spring_datasource_username=root
  - spring_datasource_password=123456
  - logging.file=/opt/logs/configservice.log
  - server.port=8080
  - eureka.instance.ip-address=${当前主机ip}
# docker多网卡解决绑定ip的问题
#  - spring.cloud.inetutils.ignoredInterfaces[0]=docker0
#  - spring.cloud.inetutils.ignoredInterfaces[1]=veth.*
  volumes:
    - /var/log/apollo:/opt/logs
  network_mode: host
  restart: always

 adminservice:
  image: juntao2000/apollo-adminservice:1.4.0
#  image: ciwei123321/apollo-adminservice:1.5.0
  environment:
  - spring_datasource_url=jdbc:mysql://${数据库id:3306}/ApolloConfigDB?characterEncoding=utf8
  - spring_datasource_username=root
  - spring_datasource_password=123456
  - logging.file=/opt/logs/adminservice.log
  - server.port=8090
  - eureka.instance.ip-address=${当前主机ip}
# docker多网卡解决绑定ip的问题
#  - spring.cloud.inetutils.ignoredInterfaces[0]docker0
#  - spring.cloud.inetutils.ignoredInterfaces[1]=veth.*

  volumes:
    - /var/log/apollo:/opt/logs
  network_mode: host
  depends_on:
   - configservice
  restart: always

 portal:
  image: juntao2000/apollo-portal:1.4.0
#  image: ciwei123321/apollo-portal:1.5.0
  environment:
  - spring_datasource_url=jdbc:mysql://${数据库id:3306}/ApolloPortalDB?characterEncoding=utf8
  - spring_datasource_username=root
  - spring_datasource_password=123456
  - server.port=8070
  - logging.file=/opt/logs/portal.log
# docker多网卡解决绑定ip的问题
#  - spring.cloud.inetutils.ignoredInterfaces[0]=docker0
#  - spring.cloud.inetutils.ignoredInterfaces[1]=veth.*
  volumes:
    - /var/log/apollo:/opt/logs
   # 挂载环境配置 多环境配置可以参考官网 我们默认使用dev
    - /root/apollo-portal/config/apollo-env.properties:/apollo-portal/config/apollo-env.properties
  network_mode: host
  depends_on:
   - adminservice
  restart: always
```

### 设置默认环境地址

在root目录创建文件夹

```
mkdir -p /root/apollo-portal/config/
cd /root/apollo-portal/config/
vi apollo-env.properties
```

添加如下内容

下面的主机ip最好使用localhost

```
local.meta=http://localhost:8080
dev.meta=http://${当前主机ip}:8080
fat.meta=http://fill-in-fat-meta-server:8080
uat.meta=http://fill-in-uat-meta-server:8080
lpt.meta=${lpt_meta}
pro.meta=http://fill-in-pro-meta-server:8080
```

### 启动

直接使用docker-compose启动呀

```
docker-compose up -d
```

### 初始化数据库

通过mysql工具分别执行下面2个脚本

https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloportaldb.sql

https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloconfigdb.sql

### 访问

http://ip:8080

http://ip:8070

启动需要耐心等待

最好等待5分钟左右

> 为什么要设置- eureka.instance.ip-address=${当前主机ip}

因为不设置的话只能内网调试 部署在外网的话 无法调试的

参考：https://github.com/ctripcorp/apollo/wiki/部署&开发遇到的常见问题#3-admin-server-或者-config-server-注册了内网ip导致portal或者client访问不了admin-server或config-server