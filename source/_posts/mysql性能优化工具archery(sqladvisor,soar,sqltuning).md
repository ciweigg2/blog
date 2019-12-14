cover: http://ciwei2.cn-sh2.ufileos.com/87.jpg
title: mysql性能优化工具archery(sqladvisor,soar,sqltuning)
date: 2018-11-05 23:19:20
tags: [sqladvisor,soar,sqltuning]
categories: [综合]
---
### 介绍

* archery集成了sqladvisor,soar,sqltuning开源mysql分析工具

* SQL审核平台，支持SQL审核、SQL优化(SQLAdvisor|SOAR|SQLTuning)、脱敏查询、慢日志管理、数据库审核、表结构同步、会话管理

* github:https://github.com/hhyo/archery

<!--more-->

### 1、下载archery：
```java
git clone https://github.com/hhyo/archery.git
```

### 2、修改mongo和mysql的用户名密码和连接地址：
```java
vi archery/archery/settings.py
```

```java
# 该项目本身的mysql数据库地址
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'archery',
        'USER': 'root',
        'PASSWORD': 'Mxc1993@!',
        'HOST': '118.184.218.184',
        'PORT': '3306',
        'OPTIONS': {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
            'charset': 'utf8mb4'
        },
        'TEST': {
            'NAME': 'test_archery',
            'CHARSET': 'utf8',
        },
    }
}

# themis审核所需mongodb数据库，账号角色必须有"anyAction" to "anyResource"权限
MONGODB_DATABASES = {
    "default": {
        "NAME": 'themis',
        "USER": 'admin',
        "PASSWORD": 'admin',
        "HOST": '118.184.218.184',
        "PORT": 27017,
    },
}
```

### 3、在archery目录下面创建docker-compose.yml：
```java
cd archery

vi docker-compose.yml

version: '3'

services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - "./mysql/datadir:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: Mxc1993@!

  mongo:
    image: mongo
    container_name: mongo
    restart: always
    volumes:
      - "./mongo/datadir:/data/db"
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin

  inception:
    image: registry.cn-hangzhou.aliyuncs.com/lihuanhuan/inception
    container_name: inception
    restart: always
    ports:
      - "6669:6669"
    volumes:
      - "./inc.cnf:/etc/inc.cnf"

  archery:
    image: registry.cn-hangzhou.aliyuncs.com/lihuanhuan/archery
    container_name: archery
    restart: always
    ports:
      - "9123:9123"
    volumes:
      - "./archery/settings.py:/opt/archery/archery/settings.py"
      - "./downloads:/opt/archery/downloads"
    command: ["bash","/opt/archery/src/docker/startup.sh"]
    environment:
      NGINX_PORT: 9123
```

### 3.1、在archery目录下面创建inc.cnf

设置接下来要创建的mysql账号密码

```java
[inception]
general_log=1
general_log_file=inception.log
port=6669
socket=/tmp/inc.socket
character-set-client-handshake=0
character-set-server=utf8
inception_remote_system_password=Mxc1993
inception_remote_system_user=ciwei
inception_remote_backup_port=3306
inception_remote_backup_host=118.184.218.184
inception_support_charset=utf8,utf8mb4
inception_enable_nullable=0
inception_check_primary_key=1
inception_check_column_comment=1
inception_check_table_comment=1
inception_osc_on=OFF
inception_osc_bin_dir=/usr/bin
inception_osc_min_table_size=1
inception_osc_chunk_time=0.1
inception_enable_blob_type=1
inception_check_column_default_value=1
```
	  
### 4、#启动
```java
docker-compose -f docker-compose.yml up -d
```

### 5、登录mysql创建数据库：
使用工具创建数据库：archery

6、#表结构初始化
```java
docker exec -ti archery /bin/bash
cd /opt/archery
source /opt/venv4archery/bin/activate
python3 manage.py makemigrations sql  
python3 manage.py migrate
```

### 7、#创建管理用户
```java
python3 manage.py createsuperuser
```

### 8、#日志查看和问题排查
```java
docker logs archery
```

### 9、配置sql分析：

文档：https://github.com/hhyo/archery/wiki/%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E#soar

* SQLAdvisor
功能说明：利用美团SQLAdvisor对收集的慢日志进行优化，一键获取优化建议
相关配置：
安装SQLAdvisor，docker镜像已包含，安装方法见项目地址
在系统管理-配置项管理中修改SQLADVISOR为程序路径，路径需要完整，docker部署的请修改为'/opt/sqladvisor'

* SQLTuning
功能说明：协助DBA高效、快速地优化语句，文章链接

* SOAR
功能说明：SOAR(SQL Optimizer And Rewriter)是一个对SQL进行优化和改写的自动化工具。 由小米人工智能与云平台的数据库团队开发与维护。项目地址
相关配置：
在系统管理-配置项管理中修改SOAR_PATH为程序路径，路径需要完整，docker部署的请修改为'/opt/soar'
修改SOAR_TEST_DSN为测试环境连接信息：ciwei:Mxc1993@118.184.218.184:3306/me2mes (me2mes是自定义的数据库)

SOAR_TEST_DSN的连接地址：mysql需要授权用户GRANT ALL

```java
GRANT ALL PRIVILEGES ON *.* TO 'ciwei'@'%' IDENTIFIED BY 'Mxc1993' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

* 配置Inception

> 里面填写inception和mysql的地址端口，mysql的用户名密码

![](/images/20181106224403.png)

### SQL上线功能使用

* 工作流

> 导入sql文件可以执行多级审批流程 最后立即执行会执行sql语句 需要配置审批流程

功能说明：项目提供简单的多级审批流配置，审批流程和资源组以及审批类型相关，不同资源组和审批类型可以配置不同的审批流程，审批流程配置的是权限组，可避免审批人单点的问题
相关配置：
在系统管理-配置项管理页面，可进行组工单审批流程的配置
对于SQL上线和SQL查询权限工单，如果用户拥有('sql_review', '审核SQL上线工单')、('sql_execute', '执行SQL上线工单')、('query_review', '审核查询权限')权限，就可以查看到当前用户所在资源组的所有工单
工单待审核时，关联当前审批权限组的所有用户，均可审核所在资源组的工单（资源组隔离）
待办列表包含所有当前用户可审核的所有工单

* SQL审核

功能说明：SQL上线和审核功能依靠Inception审核平台，建议使用前先完整阅读Inception的[项目文档](https://github.com/hhyo/inception-document)
相关配置：
在系统管理-配置项管理，有Inception配置项，需要按照配置说明进行配置

![](/images/20181106203432.png)