title: 【MySQL】常用拼接语句
date: 2019-07-22 15:04:05
tags: [mysql]
categories: [mysql]
---
**前言：**在MySQL中 CONCAT ()函数用于将多个字符串连接成一个字符串，利用此函数我们可以将原来一步无法得到的sql拼接出来，在工作中也许会方便很多，下面主要介绍下几个常用的场景。

*注：*适用于5.7版本 低版本可能稍许不同。

<!--more-->

##### 1.拼接查询所有用户

```mysql
SELECT DISTINCT
	CONCAT(
		'User: \'',
		USER,
		'\'@\'',
		HOST,
		'\';'
	) AS QUERY
FROM
	mysql.USER;
# 当拼接字符串中出现'时 需使用\转义符
```

##### 2.拼接DROP table

```mysql
SELECT
	CONCAT(
		'DROP table ',
		TABLE_NAME,
		';'
	)
FROM
	information_schema. TABLES
WHERE
	TABLE_SCHEMA = 'test';
```

##### 3.拼接kill连接

```mysql
SELECT
	concat('KILL ', id, ';')
FROM
	information_schema. PROCESSLIST
WHERE
	STATE LIKE 'Creating sort index';
```

##### 4.拼接创建数据库语句

```mysql
SELECT
	CONCAT(
		'create database ',
		'`',
    SCHEMA_NAME,
    '`',
    ' DEFAULT CHARACTER SET ',
    DEFAULT_CHARACTER_SET_NAME,
		';'
	) AS CreateDatabaseQuery
FROM
	information_schema.SCHEMATA
WHERE
	SCHEMA_NAME NOT IN (
		'information_schema',
		'performance_schema',
		'mysql',
		'sys'
	);
```

##### 5.拼接创建用户的语句

```mysql
SELECT
	CONCAT(
		'create user \'',
    user,
    '\'@\'',
    Host,
    '\''
    ' IDENTIFIED BY PASSWORD \'',
    authentication_string,
		'\';'
	) AS CreateUserQuery
FROM
	mysql.`user`
WHERE
	`User` NOT IN (
		'root',
		'mysql.session',
		'mysql.sys'
	);
#有密码字符串哦 在其他实例执行 可直接创建出与本实例相同密码的用户
```

##### 6.导出权限脚本 这个shell脚本也用到了拼接

```shell
#!/bin/bash  
#Function export user privileges  

pwd=yourpass  
expgrants()  
{  
  mysql -B -u'root' -p${pwd} -N $@ -e "SELECT CONCAT(  'SHOW GRANTS FOR ''', user, '''@''', host, ''';' ) AS query FROM mysql.user" | \
  mysql -u'root' -p${pwd} $@ | \
  sed 's/\(GRANT .*\)/\1;/;s/^\(Grants for .*\)/-- \1 /;/--/{x;p;x;}'  
}  
 
expgrants > /tmp/grants.sql
echo "flush privileges;" >> /tmp/grants.sql
```

##### 7.查找表碎片

```mysql
SELECT t.TABLE_SCHEMA,
       t.TABLE_NAME,
       t.TABLE_ROWS,
	   concat(round(t.DATA_LENGTH / 1024 / 1024, 2), 'M') AS size,
       t.INDEX_LENGTH,
       concat(round(t.DATA_FREE / 1024 / 1024, 2), 'M') AS datafree
FROM information_schema.tables t
WHERE t.TABLE_SCHEMA = 'test' order by DATA_LENGTH desc;
```

##### 8.查找无主键表 这个没用到拼接 也分享出来吧

```mysql
#查找某一个库无主键表
SELECT
table_schema,
table_name
FROM
    information_schema.TABLES
WHERE
    table_schema = 'test'
AND TABLE_NAME NOT IN (
    SELECT
        table_name
    FROM
        information_schema.table_constraints t
    JOIN information_schema.key_column_usage k USING (
        constraint_name,
        table_schema,
        table_name
    )
    WHERE
        t.constraint_type = 'PRIMARY KEY'
    AND t.table_schema = 'test'
);

#查找除系统库外 无主键表
SELECT
	t1.table_schema,
	t1.table_name
FROM
	information_schema. TABLES t1
LEFT OUTER JOIN information_schema.TABLE_CONSTRAINTS t2 ON t1.table_schema = t2.TABLE_SCHEMA
AND t1.table_name = t2.TABLE_NAME
AND t2.CONSTRAINT_NAME IN ('PRIMARY')
WHERE
	t2.table_name IS NULL
AND t1.TABLE_SCHEMA NOT IN (
	'information_schema',
	'performance_schema',
	'mysql',
	'sys'
) ;
```
