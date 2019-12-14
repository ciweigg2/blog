title: docker-compose安装mysql一主多从集群
date: 2019-07-15 14:27:59
tags: [docker,mysql]
categories: [综合]
---
## 介绍

随着应用业务数据不断的增大，应用的响应速度不断下降，在检测过程中我们不难发现大多数的请求都是查询操作。此时，我们可以将数据库扩展成主从复制模式，将读操作和写操作分离开来，多台数据库分摊请求，从而减少单库的访问压力，进而应用得到优化。

搭建MySQL读写分离主从集群，这里未使用`binlog`方式，使用的是`GTID`方式

master-data slave-data 里面的文件需要先删除

<!--more-->

## 主从目录结构

```bash
.
├── bin
│   ├── add-slave-account-to-master.sh
│   ├── reset-slave.sh
│   ├── slave-replias-master-start.sh
│   ├── slave2-replias-master-start.sh
│   └── stop-replicas.sh
├── config
│   ├── master.cnf
│   └── slave.cnf
│   └── slave2.cnf
├── docker-compose.yml
├── .env
├── master-data
├── show-slave-status.sh
└── slave-data
└── slave2-data
```

## 目录/文件说明：

- `bin/add-slave-account-to-master.sh` ：Master节点添加备份账户的脚本
- `config/master.cnf` : MySQL Master节点的配置文件
- `config/slave.cnf` : MySQL Slave节点的配置文件
- `config/slave2.cnf` : MySQL Slave2节点的配置文件
- `docker-compose.yml` :  构建主从节点与挂载数据目录的docker-compose配置文件
- `master-data` : 主节点数据位置，当然生产环境要挂到别的位置
- `slave-data` ：从节点数据位置，当然生产环境要挂到别的位置
- `slave2-data` ：从节点数据位置，当然生产环境要挂到别的位置
- `bin/slave-replias-master-start.sh` ：从节点添加主节点备份账号信息并开启备份的脚本
- `bin/slave2-replias-master-start.sh` ：从节点添加主节点备份账号信息并开启备份的脚本
- `bin/stop-replicas.sh` ：关闭从节点备份的脚本
- `bin/reset-slave.sh` : 重置从节点备份状态，修复由于主从集群重启后无法建立集群的问题
- `.env` : 环境变量文件
- ` bin/show-slave-status.sh `: 查看主从连接状态的脚本

## 搭建过程：

1.修改`.env`文件

```bash
# default environment arguments for docker-compose.yml
# set master data dir
MASTER_DATA=./master-data
# set slave data dir
SLAVE_DATA=./slave-data
# set slave2 data dir
SLAVE2_DATA=./slave2-data
# set master & slave root password
MASTER_PASSWD=P@ssw0rd
# set slave root passwor
SLAVE_PASSWD=P@ssw0rd
# set slave2 root passwor
SLAVE2_PASSWD=P@ssw0rd
# set replicas mysql account name
REPL_NAME=replicas
# set replicas mysql password
REPL_PASSWD=replicasPasswd
```

> - `MASTER_DATA`是Master节点的数据目录，需要修改到宿主机对应的位置，SLAVE_DATA亦然。
>
> - `MASTER_PASSWD`是主节点的root密码，bin目录下的脚本会读取这个变量的值从而进行访问数据库
> - `SLAVE_PASSWD`是从节点的root密码，脚本也会读
> - `REPL_NAME`是主节点要创建的账户名，从节点通过这个账户进行访问
> - `REPL_PASSWD`是主节点要创建的`REPL_NAME`对应的密码

2.启动两个节点，执行`docker-compose up -d`

检查已经启动

3.进入bin目录，执行脚本

```bash
cd bin
./add-slave-account-to-master.sh #读取mysql密码，为主节点添加备份账户
./slave-replias-master-start.sh #从节点使用备份账户连接主节点，开启备份
./slave2-replias-master-start.sh #从节点使用备份账户连接主节点，开启备份
```

4.查看集群状态，在bin目录下执行`./show-slave-status.sh`

到此搭建完成。

## 故障修复

1.重启MySQL集群后从节点无法正常恢复解决。

执行bin目录下的`reset-slave.sh`, 之后 连接数据库尝试，问题已经解决。

> 另提供了一个关闭备份的脚本`stop-replicas.sh`

本文demo：https://github.com/ciweigg2/mysql-cluster-docker/tree/%E4%B8%80%E4%B8%BB%E5%A4%9A%E4%BB%8E