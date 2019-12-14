title: Spring Boot整合Sharding-JDBC实现分库分表+读写分离
date: 2019-07-15 21:59:37
tags: [springboot,shardingsphere,shardingjdbc]
categories: [综合]
---
### 架构回顾

根据不同的数据量选择合适的方案呀

<!--more-->

在数据量不是很多的情况下，我们可以将数据库进行读写分离，以应对高并发的需求，通过水平扩展从库，来缓解查询的压力。如下：

![](/images/20190715214601.png)

在数据量达到500万的时候，这时数据量预估千万级别，我们可以将数据进行分表存储。

![](/images/20190715215230.png)

在数据量继续扩大，这时可以考虑分库分表，将数据存储在不同数据库的不同表中，如下：

![](/images/20190715220138.png)

### 案例详解

测试就懒得搭建2台不同端口的主从啦

数据库类型 | 数据库	 |  ip:端口
-|-|-
主 | cool | 127.0.0.1:3340
从 | cool | 127.0.0.1:3341
从 | cool | 127.0.0.1:3342
主 | cool2 | 127.0.0.1:3340
从 | cool2 | 127.0.0.1:3341
从 | cool2 | 127.0.0.1:3342

在主库主机的Mysql执行以下脚本，分别为数据库cool和cool2创建5个表，这5个表分别为user_0、user_1、user_2、user_3、user_4。
执行的脚本如下：

```
CREATE DATABASE /*!32312 IF NOT EXISTS*/`cool` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `cool`;



/*Table structure for table `user_0` */

DROP TABLE IF EXISTS `user_0`;

CREATE TABLE `user_0` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=149 DEFAULT CHARSET=utf8;

/*Table structure for table `user_1` */

DROP TABLE IF EXISTS `user_1`;

CREATE TABLE `user_1` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=150 DEFAULT CHARSET=utf8;

/*Table structure for table `user_2` */

DROP TABLE IF EXISTS `user_2`;

CREATE TABLE `user_2` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=147 DEFAULT CHARSET=utf8;

/*Table structure for table `user_3` */

DROP TABLE IF EXISTS `user_3`;

CREATE TABLE `user_3` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=148 DEFAULT CHARSET=utf8;



DROP TABLE IF EXISTS `user_4`;

CREATE TABLE `user_4` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=148 DEFAULT CHARSET=utf8;


/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;


/*
SQLyog Community v13.1.2 (64 bit)
MySQL - 5.7.26-log : Database - cool
*********************************************************************
*/

/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`cool2` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `cool2`;



/*Table structure for table `user_0` */

DROP TABLE IF EXISTS `user_0`;

CREATE TABLE `user_0` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=149 DEFAULT CHARSET=utf8;

/*Table structure for table `user_1` */

DROP TABLE IF EXISTS `user_1`;

CREATE TABLE `user_1` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=150 DEFAULT CHARSET=utf8;

/*Table structure for table `user_2` */

DROP TABLE IF EXISTS `user_2`;

CREATE TABLE `user_2` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=147 DEFAULT CHARSET=utf8;

/*Table structure for table `user_3` */

DROP TABLE IF EXISTS `user_3`;

CREATE TABLE `user_3` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=148 DEFAULT CHARSET=utf8;


DROP TABLE IF EXISTS `user_4`;

CREATE TABLE `user_4` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=148 DEFAULT CHARSET=utf8;
```

案例的工程是在上一篇文章的工程基础上进行改造，其中pom文件的依赖包不变，详情可见源码。

在工程的application中做sharding-jdbc的分库分表配置，代码如下：

```
master-host=127.0.0.1
slave1-host=127.0.0.1
slave2-host=127.0.0.1

spring.jpa.properties.hibernate.hbm2ddl.auto=create
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.jpa.properties.hibernate.show_sql=true
sharding.jdbc.datasource.names=ds-master-0,ds-master-1,ds-master-0-slave-0,ds-master-0-slave-1,ds-master-1-slave-0,ds-master-1-slave-1

sharding.jdbc.datasource.ds-master-0.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-0.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-0.url=jdbc:mysql://${master-host}:3340/cool?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-0.username=root
sharding.jdbc.datasource.ds-master-0.password=

sharding.jdbc.datasource.ds-master-0-slave-0.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-0-slave-0.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-0-slave-0.url=jdbc:mysql://${slave1-host}:3341/cool?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-0-slave-0.username=root
sharding.jdbc.datasource.ds-master-0-slave-0.password=
sharding.jdbc.datasource.ds-master-0-slave-1.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-0-slave-1.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-0-slave-1.url=jdbc:mysql://${slave2-host}:3342/cool?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-0-slave-1.username=root
sharding.jdbc.datasource.ds-master-0-slave-1.password=

sharding.jdbc.datasource.ds-master-1.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-1.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-1.url=jdbc:mysql://${master-host}:3340/cool2?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-1.username=root
sharding.jdbc.datasource.ds-master-1.password=

sharding.jdbc.datasource.ds-master-1-slave-0.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-1-slave-0.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-1-slave-0.url=jdbc:mysql://${slave1-host}:3341/cool2?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-1-slave-0.username=root
sharding.jdbc.datasource.ds-master-1-slave-0.password=
sharding.jdbc.datasource.ds-master-1-slave-1.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds-master-1-slave-1.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds-master-1-slave-1.url=jdbc:mysql://${slave2-host}:3342/cool2?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
sharding.jdbc.datasource.ds-master-1-slave-1.username=root
sharding.jdbc.datasource.ds-master-1-slave-1.password=

#数据库分片规则
#sharding.jdbc.config.sharding.default-database-strategy.inline.sharding-column=id
#sharding.jdbc.config.sharding.default-database-strategy.inline.algorithm-expression=ds_$->{id % 2}
#数据库分片自定义规则
sharding.jdbc.config.sharding.default-database-strategy.standard.sharding-column=id
sharding.jdbc.config.sharding.default-database-strategy.standard.precise-algorithm-class-name=com.forezp.shardingjdbcdbmstbl.MyDatasourcesPreciseShardingAlgorithm

sharding.jdbc.config.sharding.tables.user.actual-data-nodes=ds_$->{0..1}.user_$->{0..4}
sharding.jdbc.config.sharding.tables.user.table-strategy.inline.sharding-column=id
sharding.jdbc.config.sharding.tables.user.table-strategy.inline.algorithm-expression=user_$->{id % 5}
sharding.jdbc.config.sharding.tables.user.key-generator-column-name=id
#sharding.jdbc.config.sharding.tables.t_order_item.actual-data-nodes=ds_$->{0..1}.t_order_item_$->{0..1}
#sharding.jdbc.config.sharding.tables.t_order_item.table-strategy.inline.sharding-column=order_id
#sharding.jdbc.config.sharding.tables.t_order_item.table-strategy.inline.algorithm-expression=t_order_item_$->{order_id % 2}
#sharding.jdbc.config.sharding.tables.t_order_item.key-generator-column-name=order_item_id

sharding.jdbc.config.sharding.master-slave-rules.ds_0.master-data-source-name=ds-master-0
sharding.jdbc.config.sharding.master-slave-rules.ds_0.slave-data-source-names=ds-master-0-slave-0, ds-master-0-slave-1
sharding.jdbc.config.sharding.master-slave-rules.ds_1.master-data-source-name=ds-master-1
sharding.jdbc.config.sharding.master-slave-rules.ds_1.slave-data-source-names=ds-master-1-slave-0, ds-master-1-slave-1

sharding.jdbc.config.props.sql.show=true

mybatis.config-location=classpath:META-INF/mybatis-config.xml
server.port=8085
```

* 在上面的配置中，其中sharding.jdbc.datasource部分是配置数据库的信息，配置了6个数据库。
* sharding.jdbc.config.sharding.master-slave-rules.ds_0.master-data-source-name配置的是ds_0区的的主库名称，同理ds_1。
* sharding.jdbc.config.sharding.master-slave-rules.ds_0.slave-data-source-names配置的是ds_0区的的从库名称，同理ds_1。
* sharding.jdbc.config.sharding.default-database-strategy.inline.sharding-column配置的分库的字段，本案例是根据id进行分。
* sharding.jdbc.config.sharding.default-database-strategy.inline.algorithm-expression配置的分库的逻辑，根据id%2进行分。
* sharding.jdbc.config.sharding.tables.user.actual-data-nodes配置的是user表在真实数据库中的位置，ds_KaTeX parse error: Expected group after '_' at position 14: ->{0..1}.user_̲->{0…4}表示
数据在ds_0和ds_1中的user_0、user_1、user_2、user_3、user_4中。
* sharding.jdbc.config.sharding.tables.user.table-strategy.inline.sharding-column，配置user表数据切分的字段
* sharding.jdbc.config.sharding.tables.user.table-strategy.inline.algorithm-expression=user_$->{id % 5}，配置user表数据切分的策略。
* sharding.jdbc.config.sharding.tables.user.key-generator-column-name=id 自动生成id。

然后在Spring Boot启动类的注解@SpringBootApplication，加上exclude={DataSourceAutoConfiguration.class}，代码如下：

```java
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
@EnableConfigurationProperties
public class ShardingJdbcDbMsTblApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShardingJdbcDbMsTblApplication.class, args);
    }

}
```

### 测试

测试同上一篇文章，不想多说啦

demo：https://github.com/ciweigg2/sharding-jdbc-example/tree/master/sharding-jdbc-db-ms-tbl