title: 【MySQL】mysqldump备份与恢复
date: 2019-07-22 15:23:28
tags: [mysql]
categories: [mysql]
---
**简介：**

mysqldump常用于MySQL数据库逻辑备份。

<!--more-->

**备份操作：**

- 1.备份所有库：

```shell
mysqldump -uroot -pyourpass --all-databases > /tmp/all.dump
```

- 2.备份单个库：

```shell
mysqldump -uroot -pyourpass test > test.dump
```

- 3.备份单张表：

```shell
mysqldump -uroot -pyourpass test test_tb > test_tb.sql
```

- 4.备份存储过程：

```shell
mysqldump -uroot -pyourpass -n -d -t -R test > test_procedure.sql
```

**恢复操作：**

恢复前确保要恢复的数据库存在

- 恢复单个库：

```shell
mysql -uroot -pyourpass test < test.dump
```

或者进入数据库 执行 source  test.dump;

- 从整库备份中恢复单张表：

```shell
cat test.dump | sed -e'/./{H;$!d;}' -e 'x;/CREATE TABLE `test_tb`/!d;q' > /tmp/test_tb.sql  --筛选出建表语句
cat test.dump | grep --ignore-case  'insert into `test_tb`' > /tmp/insert_test_tb.sql  --筛选插入数据的insert语句
```

然后根据需要在数据库中执行相应的sql即可恢复单张表。