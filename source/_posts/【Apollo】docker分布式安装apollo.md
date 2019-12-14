title: 【Apollo】docker分布式安装apollo
date: 2019-08-17 09:37:33
tags: [apollo]
categories: [综合]
---
### 介绍

多台机器分布式部署apollo多环境

地址：https://github.com/idoop/docker-apollo

<!--more-->

环境| 地址 | 端口
-|-|-|
portal| 10.123.33.33 | 8070
pro | 10.123.33.31 | 8083
dev | 10.123.33.32 | 8080

### 安装

`在10.123.33.31机器上安装pro环境`

vi docker-compose-pro.yml

```yaml
version: '2'
services:
  apollo-pro:
    container_name: apollo-pro  
    image: idoop/docker-apollo:latest
    restart: always
    network_mode: host
    ports:
      - "8083:8083"
      - "8093:8093"
    environment:
      PRO_DB: jdbc:mysql://${mysqlIp}:3306/ApolloConfigDBPro?characterEncoding=utf8
      PRO_DB_USER: root
      PRO_DB_PWD: 123456
      # 此处LB设置必须为弹性IP,且同数据库中的eureka.service.url地址一样.
      PRO_LB: 10.123.33.31
    volumes:
      - /apollo/logs:/opt/logs
```

数据库修改ApolloConfigDBPro 数据库对应的 ServerConfig 表中的字段为 eureka.service.url 改成当前主机外网ip:端口 http://10.123.33.31:8083/eureka/ 如果还有一台机器也启动了pro环境那么就是高可用需要在eureka.service.url配置多个eureka地址http://pro1:8083/eureka/,http://pro2:8083/eureka/

`在10.123.33.32机器上安装dev环境`

vi docker-compose-dev.yml

```yaml
version: '2'
services:
  apollo-dev:
    container_name: apollo-dev  
    image: idoop/docker-apollo:latest
    restart: always
    network_mode: host
    ports:
      - "8080:8080"
      - "8090:8090"
    environment:
      DEV_DB: jdbc:mysql://${mysqlIp}:3306/ApolloConfigDBDev?characterEncoding=utf8
      DEV_DB_USER: root
      DEV_DB_PWD: 123456
      # 此处LB设置必须为弹性IP,且同数据库中的eureka.service.url地址一样.
      DEV_LB: 10.123.33.32
    volumes:
      - /apollo/logs:/opt/logs
```

数据库修改ApolloConfigDBDev 数据库对应的 ServerConfig 表中的字段为 eureka.service.url 改成当前主机外网ip:端口 http://10.123.33.32:8080/eureka/ 如果还有一台机器也启动了dev环境那么就是高可用需要在eureka.service.url配置多个eureka地址http://dev1:8080/eureka/,http://dev2:8080/eureka/

`在10.123.33.33机器上安装Portal`

vi docker-compose-portal

```yaml
version: '2'
services:
  apollo-portal:
    container_name: apollo-portal  
    image: idoop/docker-apollo:latest
    restart: always
    network_mode: host
    environment:
      PORTAL_DB: jdbc:mysql://${mysqlIp}:3306/ApolloPortalDB?characterEncoding=utf8
      PORTAL_DB_USER: root
      PORTAL_DB_PWD: 123456
      # Pro环境地址 多个Pro环境使用逗号分割 不过生产环境还是建议使用域名(走slb) ip变化可控
      PRO_URL: http://10.123.33.31:8083
      # Dev环境地址 多个Dev环境使用逗号分割 不过生产环境还是建议使用域名(走slb) ip变化可控
      DEV_URL: http://10.123.33.32:8080
    volumes:
      - /apollo/logs:/opt/logs
```

### 部署

在对应机器启动对应服务

```
docker-compose -f docker-compose-pro.yml up -d
docker-compose -f docker-compose-dev.yml up -d
docker-compose -f docker-compose-portal up -d
```

pro环境地址是10.123.33.32:8083

dev环境eureka地址是10.123.33.32:8080

更多细节参考官网或者上一篇文章：【Apollo】docker安装apollo官方

访问：http://10.123.33.33:8070 然后根据下面的调整环境和部门列表

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

### 总结

基本这样配置就可以用了 如果需要更多参数 请参考[docker-apollo](https://github.com/idoop/docker-apollo)