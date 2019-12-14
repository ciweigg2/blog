title: 【MySQL】Xtrabackup备份工具使用
date: 2019-07-22 15:25:36
tags: [mysql]
categories: [mysql]
---
**简介：**

> Xtrabackup是由percona提供的mysql数据库备份工具，支持在线热备份(备份时不影响数据读写).

> Xtrabackup有两个主要的工具：xtrabackup、innobackupex

<!--more-->

> 1、xtrabackup只能备份InnoDB和XtraDB两种数据表，而不能备份MyISAM数据表

> 2、innobackupex是参考了InnoDB Hotbackup的innoback脚本修改而来的.innobackupex是一个perl脚本封装，封装了xtrabackup。主要是为了方便的 同时备份InnoDB和MyISAM引擎的表，但在处理myisam时需要加一个读锁。并且加入了一些使用的选项。如slave-info可以记录备份恢 复后，作为slave需要的一些信息，根据这些信息，可以很方便的利用备份来重做slave。

本文将介绍如何用innobackupex工具做全量和增量备份。
**安装：**

```shell
yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
yum install percona-xtrabackup-24

# 可使用innobackupex -verion检查版本 若出现版本号则安装成功
# innobackupex --help 可查看参数帮助
```

**备份前准备：**
修改InnoDB为独立表空间模式，即在my.cnf中[mysqld]下设置innodb_file_per_table=1
建议创建一个单独的备份目录 例：/xbackup
**全备：**

```shell
innobackupex --defaults-file=/etc/my.cnf  --user=root --password='yourpassword'  /xbackup
# 出现completed OK!则代表备份成功，文件会保存至一个以时间戳命名的目录内。
```

**恢复：**
若全部恢复，则需要先停止mysql服务，还需确保mysqldata目录下无文件

```shell
service mysqld stop
innobackupex  --apply-log  /xbackup/2017-09-07_09-50-11/
# apply-log称作准备阶段，是为了保持数据一致性，回滚备份过程中未提交的事务，提交已提交的事务
innobackupex --defaults-file=/etc/my.cnf  --copy-back /xbackup/2017-09-07_09-50-11/
chown -R mysql:mysql /mysqldata
service mysqld start
```

**单表恢复**

```shell
innobackupex  --apply-log --export /xbackup/2017-09-07_15-53-53/
# 若t1表数据误删 确保表结构存在
ALTER TABLE t1 DISCARD TABLESPACE;
cp /xbackup/2017-09-07_15-53-53/test/t1.{ibd,exp,cfg}  /mysqldata/test/
chown -R mysql:mysql /mysqldata
ALTER TABLE t1 IMPORT TABLESPACE;
```

**增量备份与恢复：**

```shell
innobackupex --defaults-file=/etc/my.cnf  --user=root --password='xxxxxx'  --no-timestamp  --incremental  /xbackup/inc1 --incremental-basedir=/xbackup/2017-09-07_09-50-11
# 恢复
service mysqld stop
innobackupex --apply-log /xbackup/2017-09-07_09-50-11/  --incremental-dir=/xbackup/inc1/
innobackupex  --copy-back /xbackup/2017-09-07_09-50-11/
chown -R mysql:mysql /mysqldata
service mysqld start
```