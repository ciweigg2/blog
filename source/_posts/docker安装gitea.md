cover: http://ciwei2.cn-sh2.ufileos.com/16.jpg
title: docker安装gitea
date: 2018-08-15 21:10:25
tags: [gitea]
categories: [综合]
---
### 简介
Gitea是一个极易安装，运行非常快速，安装和使用体验良好的自建Git服务。采用Go作为后端语言，这使得只要生成一个可执行程序即可。并且他还支持跨平台，支持Linux、macOS和Windows以及各种架构，除了x86，amd64，还包括ARM和 PowerPC。

Github地址：https://github.com/go-gitea/gitea
<!--more-->

### 功能
支持活动时间线
支持SSH以及HTTP/HTTPS协议
支持SMTP、LDAP和反向代理的用户认证
支持反向代理子路径
支持用户、组织和仓库管理系统
支持添加和删除仓库协作者
支持仓库和组织级别Web钩子（包括Slack集成）
支持仓库Git钩子和部署密钥
支持仓库工单（Issue）、合并请求（Pull Request）以及Wiki
支持迁移和镜像仓库以及它的Wiki
支持在线编辑仓库文件和Wiki
支持自定义源的Gravatar和Federated Avatar
支持邮件服务
支持后台管理面板
支持MySQL、PostgreSQL、SQLite3、MSSQL和TiDB（实验性支持）数据库
支持多语言本地化（21种语言）

vi docker-compose.yml

```
version: "2"

networks:
 gitea:
   external: false

services:
 server:
   image: gitea/gitea:latest
   environment:
    - USER_UID=1000
    - USER_GID=1000
   restart: always
   networks:
    - gitea
   volumes:
    - ./gitea:/data
   ports:
    - "3000:3000"
    - "222:22"
   depends_on:
    - db

 db:
   image: mysql:5.7
   restart: always
   environment:
    - MYSQL_ROOT_PASSWORD=gitea
    - MYSQL_USER=gitea
    - MYSQL_PASSWORD=gitea
    - MYSQL_DATABASE=gitea
   networks:
    - gitea
   volumes:
    - ./mysql:/var/lib/mysql
```

运行docker-compose.yml文件

```
docker-compose up -d
```

最后打开http://ip:3000即可

### 安装gitea
mysql地址需要使用镜像的id

![](/images/gitea1.png)

![](/images/gitea2.png)

![](/images/gitea.png)