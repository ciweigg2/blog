title: 【MySQL】MySQL 快速创建千万级测试数据之存储过程
date: 2019-07-23 11:25:12
tags: [mysql]
categories: [mysql]
---
### 创建基础表结构

不管用何种方式，我要插在那张表总要创建的吧

<!--more-->

```sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `c_user_id` varchar(36) NOT NULL DEFAULT '',
  `c_name` varchar(22) NOT NULL DEFAULT '',
  `c_province_id` int(11) NOT NULL,
  `c_city_id` int(11) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`c_user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 采用存储过程和内存表

创建内存表

利用 MySQL 内存表插入速度快的特点，我们先利用函数和存储过程在内存表中生成数据，然后再从内存表插入普通表中

```sql
CREATE TABLE `t_user_memory` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `c_user_id` varchar(36) NOT NULL DEFAULT '',
  `c_name` varchar(22) NOT NULL DEFAULT '',
  `c_province_id` int(11) NOT NULL,
  `c_city_id` int(11) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`c_user_id`)
) ENGINE=MEMORY DEFAULT CHARSET=utf8mb4;
```

创建函数和存储过程

```sql
# 创建随机字符串和随机时间的函数
delimiter $$
CREATE DEFINER=`root`@`%` FUNCTION `randStr`(n INT) RETURNS varchar(255) CHARSET utf8mb4
         DETERMINISTIC
     BEGIN
         DECLARE chars_str varchar(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
         DECLARE return_str varchar(255) DEFAULT '' ;
         DECLARE i INT DEFAULT 0;
         WHILE i < n DO
             SET return_str = concat(return_str, substring(chars_str, FLOOR(1 + RAND() * 62), 1));
             SET i = i + 1;
         END WHILE;
         RETURN return_str;
     END$$

CREATE DEFINER=`root`@`%` FUNCTION `randDataTime`(sd DATETIME,ed DATETIME) RETURNS datetime
         DETERMINISTIC
     BEGIN
         DECLARE sub INT DEFAULT 0;
         DECLARE ret DATETIME;
         SET sub = ABS(UNIX_TIMESTAMP(ed)-UNIX_TIMESTAMP(sd));
         SET ret = DATE_ADD(sd,INTERVAL FLOOR(1+RAND()*(sub-1)) SECOND);
         RETURN ret;
     END $$

# 创建插入数据存储过程
CREATE DEFINER=`root`@`%` PROCEDURE `add_t_user_memory`(IN n int)
     BEGIN
         DECLARE i INT DEFAULT 1;
         WHILE (i <= n) DO
             INSERT INTO t_user_memory (c_user_id, c_name, c_province_id,c_city_id, create_time) VALUES (uuid(), randStr(20), FLOOR(RAND() * 1000), FLOOR(RAND() * 100), NOW());
             SET i = i + 1;
         END WHILE;
     END
     $$
```

设置内存表最大内存和表最大行数

```sql
-- 查看当前内存表所能使用的最大容量
show variables like '%heap%';
-- 312m
set max_heap_table_size=327772160;
-- 100w数据
alter table t_user_memory max_rows=1000000;
```

调用存储过程

```sql
CALL add_t_user_memory(1000000);
ERROR 1114 (HY000): The table 't_user_memory' is full
```

出现内存已满时，修改 max_heap_table_size参数的大小，我使用64M内存，插入了22W数据，看情况改，不过这个值不要太大，默认32M或者64M就好，生产环境不要乱尝试

### 从内存表插入普通表

```sql
INSERT INTO t_user SELECT * FROM t_user_memory;
Query OK, 218953 rows affected (1.70 sec)
Records: 218953  Duplicates: 0  Warnings: 0
```