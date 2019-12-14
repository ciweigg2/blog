title: 【MySQL】lower_case_table_names参数详解
date: 2019-07-22 15:16:24
tags: [mysql]
categories: [mysql]
---
**简介：** `lower_case_table_names` 是mysql设置大小写是否敏感的一个参数。

##### 1.参数说明：

> lower_case_table_names=0  表名存储为给定的大小和比较是区分大小写的 

>  lower_case_table_names = 1  表名存储在磁盘是小写的，但是比较的时候是不区分大小写

>   lower_case_table_names=2 表名存储为给定的大小写但是比较的时候是小写的

> **unix,linux下lower_case_table_names默认值为 0 .Windows下默认值是 1 .Mac OS X下默认值是 2**

<!--more-->

##### 2.查看方法：

```mysql
# 进入mysql命令行 执行以下任一语句查看：
show variables like 'lower_case_table_names';
select @@lower_case_table_names;
```

##### 3.更改方法：

> 更改数据库参数文件my.cnf

> 在mysqld下 添加或修改 lower_case_table_names = 1

> 之后重启数据库

##### 4.现实情况修改 注意事项：

因目前MySQL安装在Linux系统上较多 初始化时采取了默认的lower_case_table_names值 即区分大小写，后续可能会造成同一实例大小写库表都存在的情况，调用时还要注意大小写。
这时 更改步骤如下：

> 1.核实实例中是否存在大写的库及表

> 2.将大写的库名及表名改为小写

> 更改库名可参考：https://www.cnblogs.com/gomysql/p/3584881.html

> 更改表名：rename table TEST_TB to test_tb;

> 3.设置lower_case_table_names = 1

> 4.重启数据库

