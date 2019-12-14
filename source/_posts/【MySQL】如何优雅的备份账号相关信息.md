title: 【MySQL】如何优雅的备份账号相关信息
date: 2019-07-22 15:31:16
tags: [mysql]
categories: [mysql]
---
**前言：**

最近遇到实例迁移的问题，数据迁完后还需要将数据库用户及权限迁移过去。进行逻辑备份时，我一般习惯将MySQL系统库排除掉，这样备份里面就不包含数据库用户相关信息了。这时候如果想迁移用户相关信息 可以采用以下三种方案，类似的 我们也可以采用以下三种方案来备份数据库账号相关信息。（本文方案针对MySQL5.7版本，其他版本稍有不同）

<!--more-->

#### 1.mysqldump逻辑导出用户相关信息

我们知道，数据库用户密码及权限相关信息保存在系统库`mysql ` 里面。采用mysqldump可以将相关表数据导出来 如果有迁移用户的需求 我们可以按照需求在另外的实例中插入这些数据。下面我们来演示下：

```shell
#只导出mysql库中的user，db，tables_priv表数据 
#如果你有针队column的赋权 可以再导出columns_priv表数据
#若数据库开启了GTID 导出时最好加上 --set-gtid-purged=OFF
mysqldump -uroot -proot mysql user db tables_priv -t --skip-extended-insert > /tmp/user_info.sql

#导出的具体信息
--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES ('%','root','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','',_binary '',_binary '',_binary '',0,0,0,0,'mysql_native_password','*
81F5E21E35407D884A6CD4A731AEBFB6AF209E1B','N','2019-03-06 03:03:15',NULL,'N');
INSERT INTO `user` VALUES ('localhost','mysql.session','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','Y','N','N','N','N','N','N','N','N','N','N','N','N','N','',_binary '',_binary '',_binary '',0,0,0,0,'mysql_na
tive_password','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE','N','2019-03-06 02:57:40',NULL,'Y');
INSERT INTO `user` VALUES ('localhost','mysql.sys','N',N','N','N','N','N','N','N','N','N','',_binary '',_binary '',_binary '',_binnnary '',0,0,0,0,'mysql_native
_password','*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE','N','2019-03-06 02:57:40',NULL,'Y');
INSERT INTO `user` VALUES ('%','test','N','N','N','N','N','N','N','N','N','N','N','N','N','',_binary '',_binary '',_binarymysql_native_password','*
94BDCEBE19083CE2A1F959FD02F964C7AF4CFC964C7AF4CFCCC29','N','2019-04-19 06:24:54',NULL,'N');
INSERT INTO `user` VALUES ('%','read','Y','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','',_binary '',_binary '',_binary '',0,0,0,0,'mysql_native_password','*
2158DEFBE7B6FC24585930DF63794A2A44F22736','N','2019-04-19 06:27:45',NULL,'N');
INSERT INTO `user` VALUES ('%','test_user','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','',_binary '',_binary '',_binary '',0,0,0,0,'mysql_native_passwor
d','*8A447777509932F0ED07ADB033562027D95A0F17','N','2019-04-19 06:29:38',NULL,'N');
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Dumping data for table `db`
--

LOCK TABLES `db` WRITE;
/*!40000 ALTER TABLE `db` DISABLE KEYS */;
INSERT INTO`db` VALUES ('localhost','performance_schema','mysql.session','YY','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N');
INSERT INTO `db` VALUES ('localhost','sys','mysql.sys','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','Y');
INSERT INTO `db` VALUES ('%','test_db','test','Y','Y','Y','Y','Y','Y','N','N','N','Y','N','N','Y','Y','N','N','Y','N','N');
/*!40000 ALTER TABLE `db` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Dumping data for table `tables_priv`
--

LOCK TABLES `tables_priv` WRITE;
/*!40000 ALTER TABLE `tables_priv` DISABLE KEYS */;
INSERT INTO `tables_priv` VALUES ('localhost','mysql','mysql.session','user','boot@connecting host','0000-00-00 00:00:00','Select','');
INSERT INTO `tables_priv` VALUES ('localhost','sys','mysql.sys','sys_config','root@localhost','2019-03-06 02:57:40','Select','');
INSERT INTO `tables_priv` VALUES ('%','test_db','test_user','t1','root@localhost','0000-00-00 00:00:00','Select,Insert,Update,Delete','');
/*!40000 ALTER TABLE `tables_priv` ENABLE KEYS */;
UNLOCK TABLES;

#在新的实例插入所需数据 就可以创建出相同的用户及权限了 

```



#### 2.自定义脚本导出

首先拼接出创建用户的语句：

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
		'mysql.session',
		'mysql.sys'
	);
	
#结果 在新实例执行后可以创建出相同密码的用户
mysql> SELECT
    -> CONCAT(
    -> 'create user \'',
    ->     user,
    ->     '\'@\'',
    ->     Host,
    ->     '\''
    ->     ' IDENTIFIED BY PASSWORD \'',
    ->     authentication_string,
    -> '\';'
    -> ) AS CreateUserQuery
    -> FROM
    -> mysql.`user`
    -> WHERE
    -> `User` NOT IN (
    -> 'mysql.session',
    -> 'mysql.sys'
    -> );
+---------------------------------+
| CreateUserQuery                                                                                 |
---------------------+
| create user 'root'@'%' IDENTIFIED BY PAIFIED BY PAAASSWORD '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B';      |
| create user 'test'@'%' IDENTIFIED BY PASSWORD '*94BDCEBE19083CE2A959FD02F964C7AF4CFC29';      |
| create user 'read'@'%' IDENTIFIIIED BY PASSWORD '*2158DEFBE7B6FC24585930DF63794A2A44F22736';      |
| create user 'test_user'@'%' IDENTIFIED BY PASSWORD '*8A447777509932F0ED07ADB033562027D95A0F17'; |
+-------------------------------------------------------------------------------------------------+
4 rows in set (0.00 sec)

```



然后通过脚本导出用户权限：

```shell
#导出权限脚本
#!/bin/bash  
#Function export user privileges  
 
pwd=root  
expgrants()  
{  
  mysql -B -u'root' -p${pwd} -N $@ -e "SELECT CONCAT(  'SHOW GRANTS FOR ''', user, '''@''', host, ''';' ) AS query FROM mysql.user" | \
  mysql -u'root' -p${pwd} $@ | \
  sed 's/\(GRANT .*\)/\1;/;s/^\(Grants for .*\)/-- \1 /;/--/{x;p;x;}'  
}  
 
expgrants > /tmp/grants.sql
echo "flush privileges;" >> /tmp/grants.sql

#执行脚本后结果
-- Grants for read@% 
GRANT SELECT ON *.* TO 'read'@'%';

-- Grants for root@% 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

-- Grants for test@% 
GRANT USAGE ON *.* TO 'test'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, EXECUTE, CREATE VIEW, SHOW VIEW ON `test_db`.* TO 'test'@'%';

-- Grants for test_user@% 
GRANT USAGE ON *.* TO 'test_user'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON `test_db`.`t1` TO 'test_user'@'%';

-- Grants for mysql.session@localhost 
GRANT SUPER ON *.* TO 'mysql.session'@'localhost';
GRANT SELECT ON `performance_schema`.* TO 'mysql.session'@'localhost';
GRANT SELECT ON `mysql`.`user` TO 'mysql.session'@'localhost';

-- Grants for mysql.sys@localhost 
GRANT USAGE ON *.* TO 'mysql.sys'@'localhost';
GRANT TRIGGER ON `sys`.* TO 'mysql.sys'@'localhost';
GRANT SELECT ON `sys`.`sys_config` TO 'mysql.sys'@'localhost';
```



#### 3.mysqlpump直接导出用户

mysqlpump是mysqldump的一个衍生，也是MySQL逻辑备份的工具。mysqlpump可用的选项更多，可以直接导出创建用户的语句及赋权的语句。下面我们来演示下：

```shell
#exclude-databases排除数据库 --users指定导出用户 exclude-users排除哪些用户 
#还可以增加 --add-drop-user 参数 生成drop user语句
#若数据库开启了GTID 导出时必须加上 --set-gtid-purged=OFF
mysqlpump -uroot -proot --exclude-databases=% --users  --exclude-users=mysql.session,mysql.sys > /tmp/user.sql

#导出的结果
-- Dump created by MySQL pump utility, version: 5.7.23, linux-glibc2.12 (x86_64)
-- Dump start time: Fri Apr 19 15:03:02 2019
-- Server version: 5.7.23

SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE;
SET SQL_MODE="NO_AUTO_VALUE_ON_ZERO";
SET @@SESSION.SQL_LOG_BIN= 0;
SET @OLD_TIME_ZONE=@@TIME_ZONE;
SET TIME_ZONE='+00:00';
SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT;
SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS;
SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION;
SET NAMES utf8mb4;
CREATE USER 'read'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*2158DEFBE7B6FC24585930DF63794A2A44F22736' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK;
GRANT SELECT ON *.* TO 'read'@'%';
CREATE USER 'root'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
CREATE USER 'test'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK;
GRANT USAGE ON *.* TO 'test'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, EXECUTE, CREATE VIEW, SHOW VIEW ON `test_db`.* TO 'test'@'%';
CREATE USER 'test_user'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*8A447777509932F0ED07ADB033562027D95A0F17' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK;
GRANT USAGE ON *.* TO 'test_user'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON `test_db`.`t1` TO 'test_user'@'%';
SET TIME_ZONE=@OLD_TIME_ZONE;
SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT;
SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS;
SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;
SET SQL_MODE=@OLD_SQL_MODE;
-- Dump end time: Fri Apr 19 15:03:02 2019

#可以看出 导出结果只包含创建用户及赋权的语句 十分好用
#mysqlpump详细用法可参考：
http://www.cnblogs.com/zhoujinyi/p/5684903.html
https://dev.mysql.com/doc/refman/5.7/en/mysqlpump.html

```



**总结：**

本篇文章介绍了三种导出数据库用户信息的方案，每种方案都给出了脚本并进行演示。同时 这三种方案稍加以封装都可以作为备份数据库用户权限的脚本。可能你还有其他方案，如：pt-show-grants等，欢迎分享出来哦，也欢迎大家收藏或者改造成更适合自己的脚本，说不定什么时候就会用到哦 特别是一个实例有好多用户时，你会发现脚本更好用哈。
