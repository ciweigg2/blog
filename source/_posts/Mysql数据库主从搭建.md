title: Mysql数据库主从搭建
date: 2019-07-15 22:29:59
tags: [mysql]
categories: [综合]
---
### mysql 5.7 安装

参考：https://blog.csdn.net/forezp/article/details/94173427

不用docker-compose也可以搭建的呀 这个适合生产的 还是可以的呀

<!--more-->

这是系列文章Sharding-jdbc文章的第一篇，本篇文章主要讲述如何搭建Mysql的主从。搭建环境为centos 7.5，数据库版本为5.7。需要三台虚拟机，一主两从，读者可以在自己的电脑上创建虚拟机，也可以在云服务商买三台，按小时计费，一小时几毛钱，比较实惠。Ip分配如下：

* 10.0.0.5 主
* 10.0.0.13 从
* 10.0.0.17 从

### 安装Mysql 5.7

#### 下载yum源

```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

#### 本地安装yum源

```
yum localinstall mysql57-community-release-el7-11.noarch.rpm
```

#### 检查 mysql 源是否安装成功

```
yum repolist enabled | grep "mysql.*-community.*"
```

#### 出现以下信息，说明安装成功：

```
mysql-connectors-community/x86_64    MySQL Connectors Community              108
mysql-tools-community/x86_64         MySQL Tools Community                    90
mysql57-community/x86_64             MySQL 5.7 Community Server              347
```

#### 使用 yum install 命令安装Mysql:

```
yum install -y mysql-community-server
```

#### 这个安装过程时间可能较长。安装成功后，启动mysql：

```
systemctl start mysqld
```

#### 启动完成后，查看Mysql的状态：

```
systemctl status mysqld
```

#### 上述命令显示以下信息，则安装成功：

```
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2019-06-20 10:24:39 CST; 4s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 13879 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 13799 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 13883 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─13883 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

#### 设置开机启动：

```
systemctl enable mysqld
```

#### 修改Mydql 密码

mysql 安装完成之后，生成的默认密码在 /var/log/mysqld.log 文件中。使用 grep 命令找到日志中的密码。

```
grep 'temporary password' /var/log/mysqld.log
```

#### 上面的命令显示的密码如下：

```
2019-06-20T02:24:34.544392Z 1 [Note] A temporary password is generated for root@localhost: SrwXhkgt9t+;
```

#### 首次登录mysql，修改密码：

```
mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Forezp@2019'; 
```

### 允许远程登录

修改 root 为允许远程连接，生产环境不建议：

```
mysql> use mysql;
mysql> UPDATE user SET Host='%' WHERE User='root';
mysql> flush privileges;
```

### 开启日志（非必须）

（非必须）开启sql语句的日志，生产环境不建议开启：

查看日志目录，并开启sql语句的日志：

```
mysql>  show variables like '%general_log%';
mysql>  set global general_log=on;
```

开启后，重启Mysql ，上述开启日志配置将失效。

### Mysql的主从配置

#### 机器准备

在三台机器上按照上面的步骤安装完Mysql5.7，三台机器的ip分别为:

* 10.0.0.5 主
* 10.0.0.13 从
* 10.0.0.17 从

在三台机器上分别创建2个数据库，分别为cool和cool2，字符编码为utf8:

```
CREATE DATABASE `cool` CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE DATABASE `cool2` CHARACTER SET utf8 COLLATE utf8_general_ci;
```

关闭防火墙

要保证防火墙3306端口开放，如果只是为了学习，可以直接关闭防火墙。

centos关闭防火墙方法：service iptables stop或者systemctl stop firewalld

### master 节点配置

在10.0.0.5机器上配置

vim /etc/my.inc

```
#server-id给数据库服务的唯一标识
server-id=1
#
##log-bin设置此参数表示启用binlog功能，并指定路径名称
log-bin=/var/lib/mysql/mysql-bin
sync_binlog=0
##设置日志的过期天数
expire_logs_days=7
binlog-do-db=cool
binlog-do-db=cool2
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
binlog-ignore-db=performance_schema
```

* 这里的server-id用于标识唯一的数据库，在从库必须设置为不同的值。
* binlog-ignore-db：表示同步的时候忽略的数据库
* binlog-do-db：指定需要同步的数据库

重启mysql，配置生效，执行以下的命令：systemctl restart mysqld

重启成功后，登录mysql。

赋予从库权限账号，允许用户在主库上读取日志，赋予10.0.0.13和10.0.0.17也就是Slave机器有File权限，
只赋予Slave机器有File权限还不行，还要给它REPLICATION SLAVE的权限才可以。

```
mysql> grant FILE on *.* to 'root'@'10.0.0.13' identified by 'Forezp@2019';
mysql> grant replication slave on *.* to 'root'@'10.0.0.13' identified by 'Forezp@2019';
mysql> flush privileges;

mysql> grant FILE on *.* to 'root'@'10.0.0.17' identified by 'Forezp@2019';
mysql> grant replication slave on *.* to 'root'@'10.0.0.17' identified by 'Forezp@2019';
mysql> flush privileges;
```

这里的用户是同步的时候从库使用的用户。

重启mysql(systemctl restart mysqld)，登录mysql，查看主库信息

```
mysql> show master status;
+------------------+----------+--------------+-------------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                                | Executed_Gtid_Set |
+------------------+----------+--------------+-------------------------------------------------+-------------------+
| mysql-bin.000010 |      154 | cool,cool2   | information_schema,sys,mysql,performance_schema |                   |
+------------------+----------+--------------+-------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```

File是同步会使用到的binlog文件，Position是同步的时候也要用到的。

### 配置从库

1、修改/etc/my.conf

```
log-bin=mysql-bin
server-id=3
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
replicate-do-db=cool
replicate-do-db=cool2
replicate-ignore-db=mysql
log-slave-updates
slave-skip-errors=all
slave-net-timeout=60
```

注意：两个从库的server-id不一样，需要唯一。

2.修改完/etc/my.cnf后，重启一下mysql(systemctl restart mysqld)

进入Slave的mysql控制台，执行下面操作：

```
mysql> stop slave;
mysql> change master to master_host='10.0.0.5',master_user='root',master_password='Forezp@2019',master_log_file='mysql-bin.000010', master_log_pos=154;
start slave;
```

注意：上面的master_log_file是在Master中show master status显示的File，
而master_log_pos是在Master中show master status显示的Position。

配置第二个从库的时候，需要重新从matser获取File和position。

3.然后可以通过show slave status查看配置信息。

```
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.5
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000010
          Read_Master_Log_Pos: 154
               Relay_Log_File: VM_0_13_centos-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000010
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: cool,cool2
          Replicate_Ignore_DB: mysql
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table:
```

上面的信息有Slave_IO_Running: Yes和Slave_SQL_Running: Yes，证明主从同步成功。

4.出错清理掉之前的配置，防止同步已经同步了的数据，执行以下命令：

```
mysql> stop slave;
mysql> reset slave all;
```

上述的步骤需要在2个从库中操作，操作完成后。可以在Master建表插入数据，然后再从2个库中查看，如果2个都有数据，则证明主从数据库同步成功。

### 总结

本篇文章以实战的方式讲解如何搭建Mysql数据库的主从。后面的文章将讲述如何使用Sharding-jdbc+mybatis+springboot2，来实现数据库主从的读写分离。