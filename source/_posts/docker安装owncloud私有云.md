cover: http://ciwei2.cn-sh2.ufileos.com/19.jpg
title: docker安装owncloud私有云
tags:
  - owncloud
categories:
  - 综合
date: 2018-07-31 21:40:30
---
own是私有文件服务器，可以存储文件，和百度云这种差不多
<!--more-->

编写docker-compose.yml

```java
version: '2'
services:
  owncloud:
    image: owncloud
    links:
      - mysql:mysql
    volumes:
      - "/data/db/owncloud:/var/www/html/data"
    ports:
      - 5679:80
  mysql:
    image: mysql:5.7.19
    volumes:
      - "/data/mysql/:/var/lib/mysql"
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: ownCloud
```

然后访问http://ip:5679

安装的时候选择mysql

数据库主机填写mysql的镜像名就行了 不能填写地址 local_mysql_1