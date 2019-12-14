title: 【MySQL】主从异步复制配置
date: 2019-07-22 15:09:17
tags: [mysql]
categories: [mysql]
---
**简介：**

> Mysql的 主从同步 是一个异步的复制过程，从一个 Master复制到另一个 Slave上。在 Master 与 Slave 之间的实现整个复制过程主要由三个线程来完成，其中两个线程(Sql线程和IO线程)在 Slave 端，另外一个线程(IO线程)在 Master 端。　
>  要实现 MySQL 的 主从同步 ，首先必须打开 Master 端的BinaryLog(mysql-bin)功能，否则无法实现。因为整个复制过程实际上就是Slave从Master端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。打开 MySQL 的 Binary Log 可以通过在启动 MySQL Server 的过程中使用 “—log-bin” 参数选项，或者在 my.cnf 配置文件中的 mysqld 参数组([mysqld]标识后的参数部分)增加 “log-bin” 参数项。

<!--more-->

**原理：**

> (1)master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events）；
> (2) slave将master的binary log events拷贝到它的中继日志(relay log)；
> (3) slave重做中继日志中的事件，将改变反映它自己的数据。

下图描述了复制的过程：

![](/images/20190722151045.png)

**具体配置过程：**

##### 1.主库配置：

用vi /etc/my.cnf打开文件，对文件进行修改，在[mysqld]下面进行添加修改：

```shell
server-id = 1  # 这是数据库ID,此ID是唯一的，主库默认为1，其他从库以此ID进行递增，ID值不能重复，否则会同步出错；
log-bin = mysql-bin  # 二进制日志文件，此项为必填项，否则不能同步数据；
binlog_format=row # bilog设置为row模式 防止复制出错
```

##### 2.从库配置：

用vi /etc/my.cnf打开文件，对文件进行修改，在[mysqld]下面进行添加修改：

```shell
server_id = 2
log-bin=mysql-bin
relay_log=mysql-relay-bin
# 不指定以下参数则全库同步
#replicate-do-table=test.test_tb 同步某张表
#binlog-do-db = testcreate  需要同步的数据库，如果需要同步多个数据库；则继续添加此项。
#binlog-ignore-db = mysql 不需要同步的数据库；
```

##### 3.配置完需要重启主从库

##### 4.主库创建同步账号：

```mysql
create user 'replica'@'%' identified by '123456';
grant replication slave,replication client,reload,super on *.* to 'replica'@'%' identified by '123456';
```

##### 5.进入从库开启同步

同步开启前需要保持主从要同步的数据库数据一致。

```mysql
# 从库启动slave：
# (MASTER_LOG_FILE与MASTER_LOG_POS在主库运行SHOW MASTER STATUS;取得)
CHANGE MASTER TO MASTER_HOST='192.168.1.60',
    MASTER_USER='replica',
    MASTER_PASSWORD='123456',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=875;
    
start slave;
show slave status \G; --查看slave状态 确保Slave_IO_Running: Yes Slave_SQL_Running: Yes
```