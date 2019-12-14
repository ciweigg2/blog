title: 【MySQL】MySQL审计操作记录
date: 2019-07-22 15:20:03
tags: [mysql]
categories: [mysql]
---
server_audit是一款内嵌在mariadb的审计插件，在mysql中同样适用，主要用于记录用户操作

<!--more-->

1.安装：

```
通过show variables like 'plugin_dir';查看你的插件目录，
我的是：/usr/lib64/mysql/plugin/    
把下载好的插件server_audit.so
复制到/usr/lib64/mysql/plugin/ 
注意chmod+x server_audit.so
登录mysql执行插件安装命令：
INSTALL PLUGIN server_audit SONAME 'server_audit.so';
插件安装成功后有这些全局变量：show variables like '%audit%';
```

2.配置：

```
进入mysql 执行：更改全局变量
set global server_audit_excl_users='root';
set global server_audit_events='QUERY_DDL,QUERY_DML';
set global server_audit_file_path ='/mysqllog/';
set global server_audit_file_rotate_size=1073741824;
set global server_audit_file_rotations=10;
set global server_audit_file_rotate_now=ON;
set global server_audit_logging=on;

在my.cnf 增加
#audit
server_audit_events='QUERY_DDL,QUERY_DML'
server_audit_logging=on
server_audit_file_path =/mysqllog/
server_audit_file_rotate_size=1G
server_audit_file_rotations=10
server_audit_file_rotate_now=ON
server_audit_excl_users=root
```

3.建议关闭general log

```
set global general_log=off;
在my.cnf注释
general_log_file = /mysqllog/mysql.log
general_log = 1
```

4.参数说明：

```
详细请参考：https://mariadb.com/kb/en/mariadb/server_audit-system-variables/
server_audit_output_type：指定日志输出类型，可为SYSLOG或FILE
server_audit_logging：启动或关闭审计
server_audit_events：指定记录事件的类型，可以用逗号分隔的多个值(connect,query,table)，如果开启了查询缓存(query cache)，查询直接从查询缓存返回数据，将没有table记录
server_audit_file_path：如server_audit_output_type为FILE，使用该变量设置存储日志的文件，可以指定目录，默认存放在数据目录的server_audit.log文件中
server_audit_file_rotate_size：限制日志文件的大小
server_audit_file_rotations：指定日志文件的数量，如果为0日志将从不轮转
server_audit_file_rotate_now：强制日志文件轮转
server_audit_incl_users：指定哪些用户的活动将记录，connect将不受此变量影响，该变量比server_audit_excl_users优先级高
server_audit_syslog_facility：默认为LOG_USER，指定facility
server_audit_syslog_ident：设置ident，作为每个syslog记录的一部分
server_audit_syslog_info：指定的info字符串将添加到syslog记录
server_audit_syslog_priority：定义记录日志的syslogd priority
server_audit_excl_users：该列表的用户行为将不记录，connect将不受该设置影响
server_audit_mode：标识版本，用于开发测试
```

5.卸载

```
mysql> UNINSTALL PLUGIN server_audit;
mysql> show variables like '%audit%';
Empty set (0.00 sec)
```

防止server_audit 插件被卸载，需要在配置文件中添加:
[mysqld]
server_audit=FORCE_PLUS_PERMANENT
重启MySQL生效