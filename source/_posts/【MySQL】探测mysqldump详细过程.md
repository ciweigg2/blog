title: 【MySQL】探测mysqldump详细过程
date: 2019-07-22 15:06:46
tags: [mysql]
categories: [mysql]
---
**前言：**相信大家对mysqldump应该不陌生，但是大家对mysqldump的原理及备份过程真的熟悉吗？今天，我们一起来深入理解一下mysqldump的备份原理及备份过程以及不同参数产生的效果。

<!--more-->

1.直接 `mysqldump -uroot -pyourpass test_db > test_db.sql` 备份 产生的general_log如下：

```mysql
2018-08-14T14:10:50.227254+08:00	   27 Connect	root@localhost on  using Socket
2018-08-14T14:10:50.227454+08:00	   27 Query	/*!40100 SET @@SQL_MODE='' */
2018-08-14T14:10:50.227640+08:00	   27 Query	/*!40103 SET TIME_ZONE='+00:00' */
2018-08-14T14:10:50.227857+08:00	   27 Query	SHOW VARIABLES LIKE 'gtid\_mode'
2018-08-14T14:10:50.232239+08:00	   27 Query	SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE, TOTAL_EXTENTS, INITIAL_SIZE ORDER BY LOGFILE_GROUP_NAME
2018-08-14T14:10:50.234201+08:00	   27 Query	SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
2018-08-14T14:10:50.235122+08:00	   27 Query	SHOW VARIABLES LIKE 'ndbinfo\_version'
2018-08-14T14:10:50.237841+08:00	   27 Init DB	test_db
2018-08-14T14:10:50.237980+08:00	   27 Query	SHOW CREATE DATABASE IF NOT EXISTS `test_db`
2018-08-14T14:10:50.238083+08:00	   27 Query	show tables
2018-08-14T14:10:50.238489+08:00	   27 Query	LOCK TABLES `act_re_model` READ /*!32311 LOCAL */,`custom_api_info` READ /*!32311 LOCAL */,`custom_application_sysem` READ /*!32311 LOCAL */,`stud` READ /*!32311 LOCALnts` READ /*!32311 LOCAL */
2018-08-14T14:10:50.238709+08:00	   9+08:00	   27 Query	show table status like 'act\_re\_model'
2018-08-14T14:10:50.238999+08:00	   27 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:10:50.239181+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.239285+08:00	   27 Query	show create table `act_re_model`
2018-08-14T14:10:50.239497+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.239606+08:00	   27 Query	show fields from `act_re_model`
2018-08-14T14:10:50.240116+08:00	   27 Query	show fields from `act_re_model`
2018-08-14T14:10:50.240487+08:00	   27 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `act_re_model`
2018-08-14T14:10:50.240693+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.240809+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.241090+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.241384+08:00	   27 Query	SHOW TRIGGERS LIKE 'act\_re\_model'
2018-08-14T14:10:50.241813+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.242010+08:00	   27 Query	show table status like 'custom\_api\_info'
2018-08-14T14:10:50.242391+08:00	   27 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:10:50.242545+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.242705+08:00	   27 Query	show create table `custom_api_info`
2018-08-14T14:10:50.242888+08:00	   27 QuerON character_set_results = 'utf8'
2018-08-14T14:10:50.243044+08:.243044+08:00	   27 Query	show fields from `custom_api_info`
2018-08-14T14:10:50.243541+08:00	   27 Query	show fields from `custom_api_info`
2018-08-14T14:10:50.244019+08:00	   27 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_api_info`
2018-08-14T14:10:50.2442782018-08-14T14:10:50.244439+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.244602+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.244765+08:00	   27 Query	SHOW TRIGGERS LIKE 'custom\_api\_info'
2018-08-14T14:10:50.245215+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14::50.245384+08:00	   27 Query	show table status like 'custom\_apppplication\_system'
2018-08-14T14:10:50.245749+08:00	   27 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:10:50.245888+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.246032+08:00	   27 Query	show create table `custom_application_system`
2018-08-14T14:10:50.246215+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.246383+08:00	   27 Query	show fields from `custom_application_system`
2018-08-14T14:10:50.246816+08:00	   27 Query	show fields from `custom_application_system`
2018-08-14T14:10:50.247256+08:00	  27 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_applicaation_system`
2018-08-14T14:10:50.247507+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T1470+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.247825+080.247825+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.247987+08:00	   27 Query	SHOW TRIGGERS LIKE 'custom\_application\_system'
2018-08-14T14:10:50.248474+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.248651+08:00	   27 Query	show table status like 'stud'
2018-08-14T14:10:50.248985+08:00	   27 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:10:50.249123+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.249291+08:00	   27 Query	show create table `stud`
2018-08-14T14:10:50.249460+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.249632+08:00	   27 Query	show fields from `stud`
2018-08-14T14:10:50.250060+08:00	   27 Query	show fields from `stud`
2018-08-14T14:10:50.250509+08:00	   27 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `stud`
2018-08-14T14:10:50.250764+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.250906+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.251064+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.251239+08:00	   27 Query	SHOW TRIGGERS LIKE 'stud'
2018-08-14T14:10:50.251662+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.251808+08:00	   27 Query	show table status like 'students'
2018-08-14T14:10:50.252120+08:00	  +08:00	   27 Query	SET SESSION character_set_result'
2018-08-14T14:10:50.252428+08:00	   27 Query	show create tablereate table `students`
2018-08-14T14:10:50.252595+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.252742+08:00	   27 Query	show fields from `students`
2018-08-14T14:10:50.253165+08:00	   27 Query	show fields from `students`
2018-08-14T14:10:50.253566+08:00	   27 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `students`
2018-08-14T14:10:50.253783+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.253894+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.254023+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.254189+08:00	   27 Query	SHOW TRIGGERS LIKE 'students'
2018-08-14T14:10:50.254589+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.254719+08:00	   27 Query	LOCK TABLES mysql.event READ
2018-08-14T14:10:50.254958+08:00	   27 Query	show events
2018-08-14T14:10:50.255413+08:00	   27 Query	UNLOCK TABLES
2018-08-14T14:10:50.255528+08:00	   27 Query	LOCK TABLES mysql.proc READ
2018-08-14T14:10:50.255705+08:00	   27 Query	use `test_db`
2018-08-14T14:10:50.255795+08:00	   27 Query	select @@collation_database
2018-08-14T14:10:50.255995+08:00	   27 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:10:50.256128+08:00	   27 Query	SHOW FUNCTION STATUS WHERE Db = 'test_db'
2018-08-14T14:10:50.258323+08:00	   27 Query	SHOW PROCEDURE STATUS WHERE Db = 'test_db'
2018-08-14T14:10:50.259955+08:00	   27 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:10:50.260080+08:00	   27 Query	UNLOCK TABLES
2018-08-14T14:10:50.260222+08:00	   27 Query	UNLOCK TABLES
2018-08-14T14:10:50.260327+08:00	   27 Quit	
```

2.增加 `single-transaction` 参数 即 `mysqldump -uroot -pyourpass --single-transaction test_db > test_db.sql` 备份 产生的general_log如下：

```mysql
2018-08-14T14:15:29.731380+08:00	   29 Connect	root@localhost on  using Socket
2018-08-14T14:15:29.731586+08:00	   29 Query	/*!40100 SET @@SQL_MODE='' */
2018-08-14T14:15:29.731755+08:00	   29 Query	/*!40103 SET TIME_ZONE='+00:00' */
2018-08-14T14:15:29.731948+08:00	   29 Query	SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2018-08-14T14:15:29.732107+08:00	   29 Query	START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
2018-08-14T14:15:29.732314+08:00	   29 Query	SHOW VARIABLES LIKE 'gtid\_mode'
2018-08-14T14:15:29.735166+08:00	   29 Query	UNLOCK TABLES
2018-08-14T14:15:29.735549+08:00	   29 Query	SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE, TOTAL_EXTENTS, INITIAL_SIZE ORDER BY LOGFILE_GROUP_NAME
2018-08-14T14:15:29.737219+08:00	   29 Query	SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
2018-08-14T14:15:29.738111+08:00	   29 Query	SHOW VARIABLES LIKE 'ndbinfo\_version'
2018-08-14T14:15:29.741207+08:00	   29 Init DB	test_db
2018-08-14T14:15:29.741344+08:00	   29 Query	SHOW CREATE DATABASE IF NOT EXISTS `test_db`
2018-08-14T14:15:29.741458+08:00	   29 Query	SAVEPOINT sp
2018-08-14T14:15:29.741544+08:00	   29 Query	show tables
2018-08-14T14:15:29.741759+08:00	   29 Query	show table status like 'act\_re\_model'
2018-08-14T14:15:29.742037+08:00	   29 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:15:29.742158+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.742283+08:00	   29 Query	show create table `act_re_model`
2018-08-14T14:15:29.742399+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.742493+08:00	   29 Query	show fields from `act_re_model`
2018-08-14T14:15:29.742944+08:00	   29 Query	show fields from `act_re_model`
2018-08-14T14:15:29.743360+08:00	   29 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `act_re_model`
2018-08-14T14:15:29.743614+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.743780+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.743941+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.744161+08:00	   29 Query	SHOW TRIGGERS LIKE 'act\_re\_model'
2018-08-14T14:15:29.744657+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.744811+08:00	   29 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:15:29.744972+08:00	   29 Query	show table status like 'custom\_api\_info'
2018-08-14T14:15:29.745351+08:00	   29 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:15:29.745503+08:00	   29 Qu5:29.745655+08:00	   29 Query	show create table `custom_api_info`
2018-08-14T14:15:29.745853+08:00	   29 Query	SET cter_set_results = 'utf8'
2018-08-14T14:15:29.746032+08:00	   2908:00	   2999 Query	show fields from `custom_api_info`
2018-08-14T14:15:29.746579+08:00	   29 Query	show fields from `custom_api_info`
2018-08-14T14:15:29.747091+08:00	   29 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_api_info`
2018-08-14T14:15:29.747367+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.747523+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.747681+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.747851+08:00	   29 Query	SHOW TRIGGERS LIKE 'custom\_api\_info'
2018-08-14T14:15:29.748329+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.748481+08:00	   29 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:15:29.748624+08:00	   29 Query	show table status like 'custom\_application\_system'
2018-08-14T14:15:29.748933+08:00	   29 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:15:29.749083+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.749244+08:00	   29 Query	show create table `custom_application_system`
2018-08-14T14:15:29.749417+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.749581+08:00	   29 Query	show fields from `custom_application_system`
2018-08-14T14:15:29.750027+08:00	   29 Query	show fields fro `custom_application_system`
2018-08-14T14:15:29.750472+08:00	    29 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_application_system`
2018-08-14T14:15:29.750689+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.750820+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.750955+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.751115+08:00	   29 Query	SHOW TRIGGERS LIKE 'custom\_application\_system'
2018-08-14T14:15:29.751558+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.751694+08:00	   29 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:15:29.751821+08:00	   29 Query	show table status like 'stud'
20:15:29.752110+08:00	   29 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018EATE=1
2018-08-14T14:15:29.752245+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.752367+08:00	   29 Query	show create table `stud`
2018-08-14T14:15:29.752509+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.752645+08:00	   29 Query	show fields from `stud`
2018-08-14T14:15:29.753040+08:00	   29 Query	show fields from `stud`
2018-08-14T14:15:29.753453+08:00	   29 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `stud`
2018-08-14T14:15:29.753687+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.753811+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.753935+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.754089+08:00	   29 Query	SHOW TRIGGERS LIKE 'stud'
2018-08-14T14:15:29.754509+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.754644+08:00	   29 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:15:29.754770+08:00	   29 Query	show table status like 'students'
2018-08-14T14:15:29.755068+08:00	   29 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:15:29.755204+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.755334+08:00	   29 Query	show create table `students`
2018-08-14T14:15:29.755482+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.755617+08:00	   29 Query	show fields from `students`
2018-08-14T14:15:29.756003+08:00	   29 Query	show fields from `students`
2018-08-14T14:15:29.756486+08:00	   29 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `students`
2018-08-14T14:15:29.756694+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.756816+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.756946+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.757097+08:00	   29 Query	SHOW TRIGGERS LIKE 'students'
2018-08-14T14:15:29.757513+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.757647+08:00	   29 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:15:29.757771+08:00	   29 Query	RELEASE SAVEPOINT sp
2018-08-14T14:15:29.757897+08:00	   29 Query	show events
2018-08-14T14:15:29.758400+08:00	   29 Query	use `test_db`
2018-08-14T14:15:29.758544+08:00	   29 Query	select @@collation_database
2018-08-14T14:15:29.758682+08:00	   29 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:15:29.758813+08:00	   29 Query	SHOW FUNCTION STATUS WHERE Db = 'test_db'
2018-08-14T14:15:29.761003+08:00	   29 Query	SHOW PROCEDURE STATUS WHERE Db = 'test_db'
2018-08-14T14:15:29.762609+08:00	   29 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:15:29.763200+08:00	   29 Quit	
```

3.使用 `--master-data=2` 及` --single-transaction` 参数 即 `mysqldump -uroot -pyourpass --master-data=2 --single-transaction test_db > test_db.sql`  备份  产生的general_log如下：

```mysql
2018-08-14T14:30:06.020562+08:00	   33 Connect	root@localhost on  using Socket
2018-08-14T14:30:06.020733+08:00	   33 Query	/*!40100 SET @@SQL_MODE='' */
2018-08-14T14:30:06.020902+08:00	   33 Query	/*!40103 SET TIME_ZONE='+00:00' */
2018-08-14T14:30:06.021082+08:00	   33 Query	FLUSH /*!40101 LOCAL */ TABLES
2018-08-14T14:30:06.023110+08:00	   33 Query	FLUSH TABLES WITH READ LOCK
2018-08-14T14:30:06.023302+08:00	   33 Query	SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2018-08-14T14:30:06.023428+08:00	   33 Query	START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
2018-08-14T14:30:06.023580+08:00	   33 Query	SHOW VARIABLES LIKE 'gtid\_mode'
2018-08-14T14:30:06.026820+08:00	   33 Query	SHOW MASTER STATUS
2018-08-14T14:30:06.026952+08:00	   33 Query	UNLOCK TABLES
2018-08-14T14:30:06.027163+08:00	   33 Query	SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE, TOTAL_EXTENTS, INITIAL_SIZE ORDER BY LOGFILE_GROUP_NAME
2018-08-14T14:30:06.029717+08:00	   33 Query	SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
2018-08-14T14:30:06.030799+08:00	   33 Query	SHOW VARIABLES LIKE 'ndbinfo\_version'
2018-08-14T14:30:06.033857+08:00	   33 Init DB	test_db
2018-08-14T14:30:06.033985+08:00	   33 Query	SHOW CREATE DATABASE IF NOT EXISTS `test_db`
2018-08-14T14:30:06.034125+08:00	   33 Query	SAVEPOINT sp
2018-08-14T14:30:06.034239+08:00	   33 Query	show tables
2018-08-14T14:30:06.034457+08:00	   33 Query	show table status like 'act\_re\_model'
2018-08-14T14:30:06.034770+08:00	   33 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:30:06.034888+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.034974+08:00	   33 Query	show create table `act_re_model`
2018-08-14T14:30:06.035075+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.035175+08:00	   33 Query	show fields from `act_re_model`
2018-08-14T14:30:06.035602+08:00	   33 Query	show fields from `act_re_model`
2018-08-14T14:30:06.035971+08:00	   33 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `act_re_model`
2018-08-14T14:30:06.036291+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.036399+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.036488+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.036582+08:00	   33 Query	SHOW TRIGGERS LIKE 'act\_re\_model'
2018-08-14T14:30:06.037013+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.037102+08:00	   33 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:30:06.037204+08:00	   33 Query	show table status like 'custom\_api\_info'
2018-08-14T14:30:06.037487+08:00	   33 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:30:06.037571+018-08-14T14:30:06.037670+08:00	   33 Query	show create table `custom_api_info`
2018-08-14T14:30:06.037793+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.0370+08:00	   33 Query	show fields from `custom_api_info`
2018-08-1114T14:30:06.038313+08:00	   33 Query	show fields from `custom_api_info`
2018-08-14T14:30:06.038688+08:00	   33 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_api_info`
2018-08-14T14:30:06.038881+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.039027+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.039204+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.039378+08:00	   33 Query	SHOW TRGGERS LIKE 'custom\_api\_info'
2018-08-14T14:30:06.039837+08:00	binary'
2018-08-14T14:30:06.039027+08:00	   33 Query	use `test_d14T14:30:06.039972+08:00	   33 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:30:06.040124+08:00	   33 Query	show table status like 'custom\_application\_system'
2018-08-14T14:30:06.040460+08:00	   33 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:30:06.04595+08:00	   33 Query	SET SESSION character_set_results = 'binarry'
2018-08-14T14:30:06.040731+08:00	   33 Query	show create table `custom_application_system`
2018-08-14T14:30:06.040905+08:00	+08:00	   33 Query	show fields from `students`
2018-08-14T14:30:4T14:30:06.041050+08:00	   33 Query	show fields from `custom_application_system`
2018-08-14T14:30:06.041477+08:00	   33 Query	show fields from `custom_application_system`
2018-08-14T14:30:06.041904+08:00	   33 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_application_system`
2018-08-14T14:30:06.042121+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.042288+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.042428+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.042579+08:00	   33 Query	SHOW TRIGGERS LIKE 'custom\_application\_system'
2018-08-14T14:30:06.043001+08:00	   33 Quer.043152+08:00	   33 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:30:06.043295+08:00	   33 Query	show table status like 'stud'
2018-08-14T14:30:06.043601+08:00	   33 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:30:06.043780+08:00	   33 Querycharacter_set_results = 'binary'
2018-08-14T14:30:06.043915+08:0043915+08:0000	   33 Query	show create table `stud`
2018-08-14T14:30:06.044070+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.044231+08:00	   33 Query	show fields from `stud`
2018-08-14T14:30:06.044630+08:00	   33 Query	show fields from `stud`
2018-08-14T14:30:06.045042+08:00	   33 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `stud`
2018-08-14T14:30:06.045306+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.045437+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.045572+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.045712+08:00	   33 Query	SHOW TRIGGERS LIKE 'stud'
2018-08-14T14:30:06.046100+08:00	   33 Query	SET SESS47622+08:00	   33 Query	show fields from `students`
2018-08-14T1:00	   33 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:30:06.046381+08:00	   33 Query	show table status like 'students'
2018-08-14T14:30:06.046668+08:00	   33 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T14:30:06.046813+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.046939+08:00	   33 Query	show create table `students`
2018-08-14T14:30:06.047086+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
201-08-14T14:30:06.047249+08:00	   33 Query	show fields from `stude
2018-08-14T14:30:06.046813+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.046939+08:00	   3CT /*!40001 SQL_NO_CACHE */ * FROM `students`
2018-08-14T14:30:06.048248+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.048379+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.048531+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.048675+08:00	   33 Query	SHOW TRIGGERS LIKE 'students'
2018-08-14T14:30:06.049066+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.049221+08:00	   33 Query	ROLLBACK TO SAVEPOINT sp
2018-08-14T14:30:06.049344+08:00	   33 Query	RELEASE SAVEPOINT sp
2018-08-14T14:30:06.049467+08:00	   33 Query	show events
2018-08-14T14:30:06.050063+08:00	   33 Query	use `test_db`
2018-08-14T14:30:06.050220+08:00	   33 Query	select @@collation_database
2018-08-14T14:30:06.050364+08:00	   33 Query	SET SESSION character_set_results = 'binary'
2018-08-14T14:30:06.050503+08:00	   33 Query	SHOW FUNCTION STATUS WHERE Db = 'test_db'
2018-08-14T14:30:06.052734+08:00	   33 Query	SHOW PROCEDURE STATUS WHERE Db = 'test_db'
2018-08-14T14:30:06.054303+08:00	   33 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T14:30:06.054808+08:00	   33 Quit
```

4.使用 `--skip-lock-tables` 参数 即 `mysqldump -uroot -pyourpass --skip-lock-tables test_db > test_db.sql` 备份 产生的general_log如下：

```mysql
2018-08-14T15:20:57.546924+08:00	   37 Connect	root@localhost on  using Socket
2018-08-14T15:20:57.547051+08:00	   37 Query	/*!40100 SET @@SQL_MODE='' */
2018-08-14T15:20:57.547310+08:00	   37 Query	/*!40103 SET TIME_ZONE='+00:00' */
2018-08-14T15:20:57.547548+08:00	   37 Query	SHOW VARIABLES LIKE 'gtid\_mode'
2018-08-14T15:20:57.550374+08:00	   37 Query	SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE, TOTAL_EXTENTS, INITIAL_SIZE ORDER BY LOGFILE_GROUP_NAME
2018-08-14T15:20:57.551896+08:00	   37 Query	SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FIE TABLE_SCHEMA IN ('test_db')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
2018-08-14T15:20:57.552833+08:00	   37 Query	SHOW VARIABLES LIKE 'ndbinfo\_version'
2018-08-14T15:20:57.555707+08:00	   37 Init DB	test_db
2018-08-14T15:20:57.555815+08:00	   37 Query	SHOW CREATE DATABASE IF NOT EXISTS `test_db`
200:57.555956+08:00	   37 Query	show tables
2018-08-14T15:20:57.556230+08:00	   37 Query	show table status like 'act\_re\_model'
2e\_model'
222018-08-14T15:20:57.556516+08:00	   37 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T15:20:57.556617+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.556713+08:00	   37 Query	show create table `act_re_model`
2018-08-14T15:20:57.556824+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.556921+08:00	   37 Query	show fields from `act_re_model`
2018-08-14T15:20:57.557334+08:00	   37 Query	show fields from `act_re_model`
2018-08-14T15:20:57.557704+08:00	   37 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `act_re_model`
2018-08-14T15:20:57.557906+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.557983+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.558076+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.558222+08:00	   37 Query	SHOW TRIGGERS LIKE 'act\_re\_model'
2018-08-14T1ts = 'utf8'
2018-08-14T15:20:57.558777+08:00	   37 Query	show table status like 'custom\_api\_info'
2018-08-14T15:20:57.559120+08:00	   37 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T15:20:57.559272+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.559389+08:00	   37 Query	show create table `custom_api_info`
2018-08-14T15:20:57.559548+08:00	   37 Q:57.559685+08:00	   37 Query	show fields from `custom_api_info`
2018-08-14T15:20:57.560092+08:00	   37 Query	show fields from `custom_api_info`
2018-08-14T15:20:57.560527+08:00	   37 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_api_info`
2018-08-14T15:20:57.560733+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.560848+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.560973+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.561104+08:0	SHOW TRIGGERS LIKE 'custom\_api\_info'
2018-08-14T15:20:57.5615:20:57.56155508+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.561642+08:00	   37 Query	show table status like 'custom\_application\_system'
2018-08-14T15:20:57.561921+08:00	   37 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T15:20:57.562033+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.562168+08:00	   37 Query	show create table `custom_application_system`
2018-08-14T15:20:57.562330+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.562457+08:00	   37 Query	show fields from `custom_application_system`
2018-08-14T15:20:57.562820+08:00	   37 Query	show fields from `custom_application_system`
2018-08-14T15:20:57.563219+08:00	   37 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_application_system`
2018-08-14T15:20:57.563416+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.563529+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.563655+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.563790+08:00	   37 Query	SHOW TRIGGERS LIKE 'custom\_application\_system'
2018-08-14T15:20:57.564170+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.564314+08:00	   37 Query	show table status like 'stud'
2018-08-14T15:20:57.564587+08:00	   37 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T15:20:57.564698+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.564815+08:00	   37 Query	show create table `stud`
2018-08-14T15:20:57.564952+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.565081+08:00	   37 Query	show fields from `stud`
2018-08-14T15:20:57.565469+08:00	   37 Query	show fields from `stud`
2018-08-14T15:20:57.565828+08:00	   37 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `stud`
2018-08-14T15:20:57.566035+08:008-14T15:20:57.566166+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.566305+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.566433+08:00	   37 Query	SHOW TRIGGERS LIKE 'stud'
2018-08-14T15:20:57.566798+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.566927+08:00	   37 Query	show table status like 'students'
2018-08-14T15:20:57.567226+08:00	   37 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-14T15:20:57.567337+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.567456+08:00	   37 Query	sw create table `students`
2018-08-14T15:20:57.567588+08:00	   3777 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.567722+08:00	   37 Query	show fields from `students`
2018-08-14T15:20:57.568071+08:00	   37 Query	show fields from `students`
2018-08-14T15:20:57.568451+08:00	   37 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `students`
2018-08-14T15:20:57.568650+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.568760+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.568879+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.569009+08:00	   37 Query	SHOW TRIGGERS LIKE 'students'
2018-08-14T15:20:57.569401+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.569533+08:00	   37 Query	show events
2018-08-14T15:20:57.569960+08:00	   37 Query	use `test_db`
2018-08-14T15:20:57.570082+08:00	   37 Query	select @@collation_database
2018-08-14T15:20:57.570248+08:00	   37 Query	SET SESSION character_set_results = 'binary'
2018-08-14T15:20:57.570394+08:00	   37 Query	SHOW FUNCTION STATUS WHERE Db = 'test_db'
2018-08-14T15:20:57.572401+08:00	   37 Query	SHOW PROCEDURE STATUS WHERE Db = 'test_db'
2018-08-14T15:20:57.573994+08:00	   37 Query	SET SESSION character_set_results = 'utf8'
2018-08-14T15:20:57.574450+08:00	   37 Quit	
```

5.使用 `--lock-all-tables` 参数 即 `mysqldump -uroot -pyourpass --lock-all-tables test_db > test_db.sql` 备份 产生的general_log如下：

```mysql
2018-08-16T15:27:44.653042+08:00	   12 Connect	root@localhost on  using Socket
2018-08-16T15:27:44.653378+08:00	   12 Query	/*!40100 SET @@SQL_MODE='' */
2018-08-16T15:27:44.653576+08:00	   12 Query	/*!40103 SET TIME_ZONE='+00:00' */
2018-08-16T15:27:44.653782+08:00	   12 Query	FLUSH TABLES
2018-08-16T15:27:44.656576+08:00	   12 Query	FLUSH TABLES WITH READ LOCK
2018-08-16T15:27:44.656779+08:00	   12 Query	SHOW VARIABLES LIKE 'gtid\_mode'
2018-08-16T15:27:44.661157+08:00	   12 Query	SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE, TOTAL_EXTENTS, INITIAL_SIZE ORDER BY LOGFILE_GROUP_NAME
2018-08-16T15:27:44.664158+08:00	   12 Query	SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('test_db')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
2018-08-16T15:27:44.665119+08:00	   12 Query	SHOW VARIABLES LIKE 'ndbinfo\_version'
2018-08-16T15:27:44.667725+08:00	   12 Init DB	test_db
2018-08-16T15:27:44.667857+08:00	   12 Query	SHOW CREATE DATABASE IF NOT EXISTS `test_db`
2018-08-16T15:27:44.668053+08:00	   12 Query	show tables
2018-08-16T15:27:44.668384+08:00	   12 Query	show table status like 'ac'
2018-08-16T15:27:44.668738+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.668888+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.669050+08:00	   12 Query	show create table `ac`
2018-08-16T15:27:44.669244+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.669408+08:00	   12 Query	show fields from `ac`
2018-08-16T15:27:44.669844+08:00	   12 Query	show fields from `ac`
2018-08-16T15:27:44.670334+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `ac`
2018-08-16T15:27:44.670605+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.670750+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.670900+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.671088+08:00	   12 Query	SHOW TRIGGERS LIKE 'ac'
2018-08-16T15:27:44.671556+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.671718+08:00	   12 Query	show table status like 'act\_re\_model'
2018-08-16T15:27:44.672046+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.672201+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.672356+08:00	   12 Query	show create table `act_re_model`
2018-08-16T15:27:44.672525+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.672680+08:00	   12 Query	show fields from `act_re_model`
2018-08-16T15:27:44.673126+08:00	   12 Query	show fields from `act_re_model`
2018-08-16T15:27:44.673618+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `act_re_model`
2018-08-16T15:27:44.673893+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.674051+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.674223+08:00	   12 Query	select @@collation_database
2018-08-1T15:27:44.674389+08:00	   12 Query	SHOW TRIGGERS LIKE 'act\_re\__model'
2018-08-16T15:27:44.674825+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.674997+08:00	   12 Query	show table status like 'custom\_api\_info'
2018-08-16T15:27:44.675333+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.675487+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.675623+08:00	   12 Query	show create table `custom_api_info`
2018-08-16T15:27:44.675812+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.675972+08:00	   12 Query	show fields from `custom_api_info`
2018-08-16T15:27:44.676422+08:00	   12 Query	show fields from `custom_api_info`
2018-08-16T15:27:44.676854+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_api_info`
2018-08-16T15:27:44.677085+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.677246+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.677390+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.677541+08:00	   12 Query	SHOW TRIGGERS LIKE 'custom\_api\_info'
2018-08-16T15:27:44.677951+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.678113+08:00	   12 Query	show table status like 'custom\_application\_system'
2018-08-16T15:27:44.678429+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.678564+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.678700+ Query	show create table `custom_application_system`
2018-08-16T2018-08-16T15:27:44.678891+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.679052+08:00	   12 Query	show fields from `custom_application_system`
2018-08-16T15:27:44.679467+08:00	   12 Query	show fields from `custom_application_system`
2018-08-16T15:27:44.679871+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `custom_application_system`
2018-08-16T15:27:44.680096+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.680257+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.680401+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.680552+08:00	   12 Query	SHOW TRIGGERS LIKE 'custom\_application\_system'
2018-08-16T15:27:44.680973+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.681121+08:00	   12 Query	show table status like 'stud'
2018-08-16T15:27:44.681440+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.681569+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.681705+08:00	   12 Query	show create table `stud`
2018-08-16T15:27:44.681864+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.682024+08:00	   12 Query	show fields from `stud`
2018-08-16T15:27:44.682431+08:00	   12 Query	show fields from `stud`
2018-08-16T15:27:44.682824+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `stud`
2018-08-16T15:27:44.683069+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.683229+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.683373+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.683522+08:00	   12 Query	SHOW TRIGGERS LIKE 'stud'
2018-08-16T15:27:44.683954+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.684121+08:00	   12 Query	show table status like 'students'
2018-08-16T15:27:44.684439+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.684568+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.684703+08:00	   12 Query	show create table `students`
2018-08-16T15:27:44.684860+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.685018+08:00	   12 Query	show fields from `students`
2018-08-16T15:27:44.685480+08:00	   12 Query	show fields from `students`
2018-08-16T15:27:44.685876+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `students`
2018-08-16T15:27:44.686108+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.686265+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.686409+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.686563+08:00	   12 Query	SHOW TRIGGERS LIKE 'students'
2018-08-16T15:27:44.686947+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.687098+08:00	   12 Query	show table status like 'upper\_table'
2018-08-16T15:27:44.687394+08:00	   12 Query	SET SQL_QUOTE_SHOW_CREATE=1
2018-08-16T15:27:44.687521+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.687648+08:00	   12 Query	show create table `upper_table`
2018-08-16T15:27:44.687796+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.687933+08:00	   12 Query	show fields from `upper_table`
2018-08-16T15:27:44.688339+08:00	   12 Query	show fields from `upper_table`
2018-08-16T15:27:44.688710+08:00	   12 Query	SELECT /*!40001 SQL_NO_CACHE */ * FROM `upper_table`
2018-08-16T15:27:44.688906+08:00	   12 Query	SET SESSION character_set_results = 'binary'
2018-08-16T15:27:44.689046+08:00	   12 Query	use `test_db`
2018-08-16T15:27:44.689229+08:00	   12 Query	select @@collation_database
2018-08-16T15:27:44.689386+08:00	   12 Query	SHOW TRIGGERS LIKE 'upper\_table'
2018-08-16T15:27:44.689890+08:00	   12 Query	SET SESSION character_set_results = 'utf8'
2018-08-16T15:27:44.691705+08:00	   12 Quit	
```

**总结：** 日常备份可使用 `--skip-lock-tables` 或 `--single-transaction` 参数；若要做从库需记录binlog位置 则推荐使用 `--master-data=2` ` --single-transaction`参数叠加使用。