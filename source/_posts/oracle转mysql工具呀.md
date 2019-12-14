title: oracle转mysql工具呀
date: 2019-08-05 22:49:49
tags: [工具]
categories: [综合]
---
oracle转mysql的工具

官网：http://www.convert-in.com/oracle-to-mysql.htm#buy

<!--more-->

下载好的在网盘oracletomysql 因为是免费版 每张表只能导入50条数据

oracle的sid在databasename中填写的呀

mysql8.0比较特殊需要做下面的处理

```
SET GLOBAL max_allowed_packet=16M;

ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

FLUSH PRIVILEGES;
```

非常好的呀