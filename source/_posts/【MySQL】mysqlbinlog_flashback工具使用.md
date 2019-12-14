title: 【MySQL】mysqlbinlog_flashback工具使用
date: 2019-07-22 15:20:29
tags: [mysql]
categories: [mysql]
---
**简介：**

> mysqlbinlog_back.py 是在线读取row格式的mysqld的binlog,然后生成反向的sql语句的工具。

> 一般用于数据恢复的目的。 所谓反向的sql语句就是如果是insert，则反向的sql为delete。

> 如果delete,反向的sql是insert,如果是update, 反向的sql还是update,但是update的值是原来的值。

<!--more-->

这个项目需要安装依赖

```shell
yum install python-pip
pip install pymysql
```

官方地址：[https://github.com/58daojia-dba/mysqlbinlog_flashback](https://github.com/58daojia-dba/mysqlbinlog_flashback)

使用限制：

- 1.mysql binlog必须是row格式的。
- 2.反向生成的表必须有主键。
- 3.日志必须在主库存在
- 4.反向生成的mysql数据类型列出在下面。没有列出的类型没有经过严格的测试，也许有问题
- 5.支持的类型
  允许解析的字段类型，不在里面的会报错
  ALLOW_TYPE={ "varchar":True, "char":True, "datetime":True, "date":True, "time":True, "timestamp":True, "bigint":True, "mediumint":True, "smallint":True, "tinyint":True, "int":True, "smallint":True, "decimal":True, "float":True, "double":True, "longtext":True, "tinytext":True, "text":True, "mediumtext":True }

**工具安装：**
可在GitHub上下载安装包 
直接解压缩即可 解压后进入目录如下：

![](/images/20190722152141.png)

工具使用：

```shell
# 查看下参数使用说明
python mysqlbinlog_back.py --help  
# 回滚某个表
python mysqlbinlog_back.py --host="192.168.1.60" --port=3306 --username="root" --password="yourpassword" --schema=test --tables="test_tb" -S "mysql-bin.000009"
```

回滚完成后会在mysqlbinlog_flashback-master/log目录下生成回滚语句
之后执行以下语句在数据库中进行回滚

```shell
mysql -uroot -pyourpassword --default-character-set=utf8mb4 test < flashback_test_20170912_170610.sql
```
