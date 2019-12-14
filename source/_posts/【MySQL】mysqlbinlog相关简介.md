title: 【MySQL】mysqlbinlog相关简介
date: 2019-07-22 15:22:41
tags: [mysql]
categories: [mysql]
---
**binlog简介：**

二进制日志，记录对数据发生或潜在发生更改的SQL语句，并以二进制的形式保存在磁盘中。

<!--more-->

**作用：**

MySQL的作用类似于Oracle的归档日志，可以用来查看数据库的变更历史（具体的时间点所有的SQL操作）、数据库增量备份和恢复（增量备份和基于时间点的恢复）、Mysql的复制。

**开启：**

show variables like '%log_bin%'; 可查看是否开启

方法一、修改my.cnf参数文件，该方法需要重启

log-bin=mysql-bin #打开日志(主机需要打开)，这个mysql-bin也可以自定义，这里也可以加上路径，如：/home/www/mysql_bin_log/mysql-bin

关闭二进制日志的方法：log-bin = mysql-bin注释掉即可

方法二、不重启修改二进制日志配置，该方法mysql的版本需要5.6以上

SET @@global.log_bin=1|0  （1为开启，0为关闭）

**查看：**

```shell
# 可查看参数帮助
mysqlbinlog  --no-defaults --help 
# 查看最后100行
mysqlbinlog  --no-defaults --base64-output=decode-rows -vv mysql-bin.000001 |tail -100 
# 根据position查找
mysqlbinlog  --no-defaults --base64-output=decode-rows -vv mysql-bin.000001 |grep -A 20 '4939002' 
# 根据position恢复部分数据 也可根据时间点恢复
mysqlbinlog  --no-defaults --start-position=204136360 --stop-position=204136499 mysql-bin.000006 | mysql -uroot -pyourpassword test
```