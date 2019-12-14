title: 【Apollo】docker安装apollo官方
date: 2019-08-16 21:09:35
tags: [apollo]
categories: [综合]
---
### 介绍

官方版安装apollo

地址：https://github.com/idoop/docker-apollo

<!--more-->

### 安装

暂时不介绍分布式部署 这边单机部署 并且让本地可以调试 -e PRO_LB (可以让本地调试部署在云主机需要写云主机的ip) -e JAVA_OPTS (可以配置一些springboot配置文件参数)

```
-e JAVA_OPTS='-Deureka.service.url=http://localhost:8080/eureka/'
-e JAVA_OPTS='-Ddev_meta=http://localhost:8080/'
-e JAVA_OPTS='-Dpro_meta=http://localhost:8083/'
```

创建数据库ApolloPortalDB ApolloConfigDBDev ApolloConfigDBPro configdb导入的时候因为默认选择的数据库的是ApolloConfigDB 所以导入的时候修改下create的数据库和use的数据库

[portaldb数据库](https://github.com/ctripcorp/apollo/blob/master/scripts/db/migration/portaldb/V1.0.0__initialization.sql)

[configdb数据库](https://github.com/ctripcorp/apollo/blob/master/scripts/db/migration/configdb/V1.0.0__initialization.sql)

数据库修改ApolloConfigDBDev ApolloConfigDBPro 数据库对应的 ServerConfig 表中的字段为 eureka.service.url 改成当前主机外网ip:端口 如果不修改的话 数据会混乱的

dev环境：http://外网ip:8080/eureka/

pro环境：http://外网ip:8083/eureka/

```
docker run --net='host' --name apollo -d \
 -e PORTAL_DB='jdbc:mysql://192.168.0.5:3309/ApolloPortalDB?characterEncoding=utf8' \
 -e PORTAL_DB_USER='root' \
 -e PORTAL_DB_PWD='123456' \
 -e DEV_DB='jdbc:mysql://192.168.0.5:3309/ApolloConfigDBDev?characterEncoding=utf8' \
 -e DEV_DB_USER='root' \
 -e DEV_DB_PWD='123456' \
 -e DEV_LB='192.168.0.6' \
 -e PRO_DB='jdbc:mysql://192.168.0.5:3309/ApolloConfigDBPro?characterEncoding=utf8' \
 -e PRO_DB_USER='root' \
 -e PRO_DB_PWD='123456' \
 -e PRO_LB='192.168.0.6' \
 -v /opt/logs:/opt/logs \
 -e JAVA_OPTS='-Ddev_meta=http://localhost:8080/'
 -e JAVA_OPTS='-Dpro_meta=http://localhost:8083/'
 --name apollo idoop/docker-apollo:latest
```

暂时使用dev和pro环境 以下是环境对应的默认端口 需要更改的话请配置 DEV_ADMIN_PORT DEV_CONFIG_PORT DEV改成环境就是多环境的对应端口配置

```
开启dev环境, 默认端口: config 8080, admin 8090
开启fat环境, 默认端口: config 8081, admin 8091
开启uat环境, 默认端口: config 8082, admin 8092
开启pro环境, 默认端口: config 8083, admin 8093
```

### 调整ApolloPortalDB配置

配置项统一存储在ApolloPortalDB.ServerConfig表中，也可以通过管理员工具 - 系统参数页面进行配置，无特殊说明则修改完一分钟实时生效。

1.apollo.portal.envs - 可支持的环境列表

默认值是dev，如果portal需要管理多个环境的话，以逗号分隔即可（大小写不敏感），如：

```
#使用几个环境开启几个不用的别开
DEV,FAT,UAT,PRO
```

2.organizations - 部门列表

Portal中新建的App都需要选择部门，所以需要在这里配置可选的部门信息，样例如下：

```
[{"orgId":"TEST1","orgName":"样例部门1"},{"orgId":"TEST2","orgName":"样例部门2"}]
```

修改完需要重启生效

更多配置参考：https://github.com/ctripcorp/apollo/wiki/分布式部署指南#213-调整服务端配置