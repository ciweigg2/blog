title: 【MySQL】explicit_defaults_for_timestamp-参数详解
date: 2019-07-22 15:15:35
tags: [mysql]
categories: [mysql]
---
**简介：** 

`explicit_defaults_for_timestamp` 系统变量决定MySQL服务端对timestamp列中的默认值和`NULL`值的不同处理方法。此变量自MySQL 5.6.6 版本引入，分为全局级别和会话级别，可动态更新，默认值为OFF。本文主要介绍该参数打开和关闭情况下对timestamp的影响 。

<!--more-->

#### 1.explicit_defaults_for_timestamp = OFF

当该参数默认设置为OFF时，其行为如下：

- 在默认情况下，如果timestamp列没有显式的指明null属性，那么该列会被自动加上not null属性(而其他类型的列如果没有被显式的指定not null，那么是允许null值的)，如果往这个列中插入null值，会自动的设置该列的值为current timestamp值。
- 表中的第一个timestamp列，如果没有指定null属性或者没有指定默认值，也没有指定ON UPDATE语句。那么该列会自动被加上DEFAULT CURRENT_TIMESTAMP和ON UPDATE CURRENT_TIMESTAMP属性。
- 对于其它TIMESTAMP列，如果没有显示指定NULL和DEFAULT属性的话，会自动设置为NOT NULL DEFAULT '0000-00-00 00:00:00'。(当然，这个与SQL_MODE有关，如果SQL_MODE中包含'NO_ZERO_DATE'，实际上是不允许将其默认值设置为'0000-00-00 00:00:00'的。)

下面我们来测试下：(本文操作基于MySQL5.7.23 版本 SQL_MODE不包含'NO_ZERO_DATE')

```mysql
mysql> show variables like 'explicit_defaults_for_timestamp';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| explicit_defaults_for_timestamp | OFF   |
+---------------------------------+-------+

mysql> create table t1 
    -> (
    -> ts1 timestamp,
    -> ts2 timestamp,
    -> ts3 timestamp default '2010-01-01 00:00:00'
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> show create table t1\G
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `ts1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `ts2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `ts3` timestamp NOT NULL DEFAULT '2010-01-01 00:00:00'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> insert into t1 values (null,null,null);
Query OK, 1 row affected (0.00 sec)

mysql> select * from t1;
+---------------------+---------------------+---------------------+
| ts1                 | ts2                 | ts3                 |
+---------------------+---------------------+---------------------+
| 2019-04-09 15:54:56 | 2019-04-09 15:54:56 | 2019-04-09 15:54:56 |
+---------------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

从表结构来看，MySQL自动为第一个timestamp字段自动设置**NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP**属性，而后面的timestamp字段，若没有指定，则设置了 **NOT NULL DEFAULT '0000-00-00 00:00:00'**属性。如果向timestamp这个列中插入null值，系统会自动的设置该列的值为current timestamp值。即explicit_defaults_for_timestamp=OFF时，即使timestamp列设为NOT NULL也能插入NULL值，系统会自动将NULL值设为current timestamp。

#### 2.explicit_defaults_for_timestamp = ON

当该参数设置为ON时，其行为如下：

- 如果timestamp列没有显式的指定not null属性，那么默认的该列可以为null，此时向该列中插入null值时，会直接记录null，而不是current timestamp。
- 不会自动的为表中的第一个timestamp列加上DEFAULT CURRENT_TIMESTAMP 和ON UPDATE CURRENT_TIMESTAMP属性。
- 如果timestamp列被加上了not null属性，并且没有指定默认值。这时如果向表中插入记录，但是没有给该TIMESTAMP列指定值的时候，如果strict sql_mode被指定了，那么会直接报错。如果strict sql_mode没有被指定，那么会向该列中插入'0000-00-00 00:00:00'并且产生一个warning。

同样的，我们来测试下：

```mysql
mysql> show variables like 'explicit_defaults_for_timestamp';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| explicit_defaults_for_timestamp | ON    |
+---------------------------------+-------+
mysql> create table t2 
    -> (
    -> ts1 timestamp,
    -> ts2 timestamp,
    -> ts3 timestamp default '2010-01-01 00:00:00'
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> show create table t2\G
*************************** 1. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `ts1` timestamp NULL DEFAULT NULL,
  `ts2` timestamp NULL DEFAULT NULL,
  `ts3` timestamp NULL DEFAULT '2010-01-01 00:00:00'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.01 sec)

mysql> insert into t2 values (null,null,null);
Query OK, 1 row affected (0.01 sec)

mysql> select * from t2;
+------+------+------+
| ts1  | ts2  | ts3  |
+------+------+------+
| NULL | NULL | NULL |
+------+------+------+
1 row in set (0.00 sec)

-- 指定NOT NULL
mysql> create table t3 
    -> (
    -> ts1 timestamp,
    -> ts2 timestamp,
    -> ts3 timestamp not null
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> show create table t3\G
*************************** 1. row ***************************
       Table: t3
Create Table: CREATE TABLE `t3` (
  `ts1` timestamp NULL DEFAULT NULL,
  `ts2` timestamp NULL DEFAULT NULL,
  `ts3` timestamp NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.01 sec)

mysql> insert into t3 values (null,null,null);
ERROR 1048 (23000): Column 'ts3' cannot be null

mysql> insert into t3 (ts1,ts2) values (null,null);
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> show warnings;
+---------+------+------------------------------------------+
| Level   | Code | Message                                  |
+---------+------+------------------------------------------+
| Warning | 1364 | Field 'ts3' doesn't have a default value |
+---------+------+------------------------------------------+

mysql> select * from t3;
+------+------+---------------------+
| ts1  | ts2  | ts3                 |
+------+------+---------------------+
| NULL | NULL | 0000-00-00 00:00:00 |
+------+------+---------------------+
```

从表结构上看出，在参数开启的情况下MySQL默认会为timestamp列添加 **null default null**属性，而且MySQL也没有为第一个timestamp字段设置该列为current timestamp值。timestamp 字段写入null值，写入之后存储的就是null值，而不是当前的时间。当timestamp 字段指定NOT NULL时，若显式插入NULL则报错提示:该字段不能为空；若不显式插入该字段且SQL_MODE不包含'NO_ZERO_DATE'，则会向该列中插入'0000-00-00 00:00:00'并且产生一个warning。

**总结：**

实际情况下，我们经常会这样创建表：

```mysql
CREATE TABLE `table_name` (
  `increment_id` INT UNSIGNED NOT NULL auto_increment COMMENT '自增主键',
  ...
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`increment_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
```

此时timestamp字段会指定NOT NULL，这时建议`explicit_defaults_for_timestamp` 参数采用默认的OFF，这样当timestamp字段显式插入NULL值时不报错，特别是程序sql写的不规范时，可以避免程序插入报错。

在不同环境间，此参数建议统一 ，不然可能出现程序在这个环境运行正常 在另外一个环境却出现报错的情况。笔者了解到亚马逊RDS MySQL5.7实例该参数默认为ON，在环境迁移时要特别注意下该参数。

*参考：* <http://suo.im/5bDU2o>  <http://suo.im/4AJeM9>
