title: 【MySQL】MySQL5-7应当注意的参数
date: 2019-07-22 15:33:03
tags: [mysql]
categories: [mysql]
---
**简介：** 本篇文章主要介绍 MySQL 初始化应当注意的参数，对于不同环境间实例迁移，这些参数同样应当注意。

*注：* 本文介绍的参数都是在配置文件 [mysqld] 部分。

<!--more-->

#### `server_id` 和 `log_bin` ，`binlog_format`

这几个系统变量通常成对出现，当我们想指定`log_bin` 选项时，必须也要指定`server_id` 。

`log_bin` 是全局变量 不可动态修改 默认为OFF。当我们需要开启binlog时，可将该参数设为binlog名字或绝对路径加名字。

`binlog_format` 指定binlog格式 5.7.7版本以上默认是ROW模式

建议设置：

```mysql
#server_id 各个实例建议设置不同 log_bin不指定路径时默认在数据文件目录
server_id = 213306
log_bin = mysqlbin
binlog_format = row
或者
server_id = 213306
log_bin = /data/mysql/logs/mysqlbin
binlog_format = row
```

#### `sql_mode`

该参数控制 MySQL server 在不同的SQL模式下运行，对于客户端发送的请求不同的模式会有不同的应答。

`sql_mode ` 参数分为全局和会话级别 可以动态修改 

```mysql
#sql_mode 默认为：
sql_mode = ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#关于修改和各个模式的作用 可参考官方文档：
https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html
```

该参数建议去掉ONLY_FULL_GROUP_BY，具体采用严格或非严格模式可以根据需求来修改。注意该参数在不同实例要保持一致，不然可能会出现一条sql在此环境下可以执行 在另外一个环境不能执行的情况。

#### `max_connections` 

该参数指定 MySQL 的最大连接数，是全局变量 可动态修改 默认为151。建议设置大些 防止出现连接数用满的错误。

#### `character_set_server`

该参数指定 MySQL server端字符集，分为全局和会话级别 可以动态修改 5.7版本默认值为latin1。

建议设置该参数为utf8或utf8mb4，不同实例间保持一致 特别是主从实例。

#### `lower_case_table_names`

该参数控制 MySQL 是否大小写敏感，主要影响库名及表名。

Linux下该参数默认为0 即对大小写敏感，是全局变量 不可动态修改，建议设置为1。

#### `transaction_isolation`

该参数指定 MySQL server 采用哪种事务隔离级别，默认是REPEATABLE-READ 可动态修改。

采用哪种隔离级别要根据应用要求来选择，一般可改为READ-COMMITTED，不同实例间建议保持一致。

#### `innodb_buffer_pool_size`

该参数控制InnoDB缓冲池大小，默认值为134217728字节（128MB）5.7.5版本以上可动态修改。

缓冲池是缓存数据和索引的地方，尽可能大的缓存池将确保使用内存而不是磁盘来进行大多数读取操作。典型值为5-6GB（8GB RAM），20-25GB（32GB RAM），100-120GB（128GB RAM），在一个独立使用的数据库服务器上,你可以设置这个变量到服务器物理内存大小的80%。

#### `innodb_log_file_size`

该参数定义redo日志组中每个日志文件的大小（以字节为单位）,是全局变量 不可动态修改 默认为48M。

当MySQL server 读写比较频繁时，建议增大该参数 可与 `innodb_log_files_in_group` 参数配合使用。

#### `innodb_io_capacity` 和 `innodb_io_capacity_max`

`innodb_io_capacity`参数设置InnoDB后台任务每秒执行的IO操作数的上限，默认值为200 可动态修改。

此参数应设置为系统每秒大约可执行的IO操作数 即系统的IOPS。该值取决于你的系统配置。

当MySQL server 写操作特多 刷新脏页落后时 ， `innodb_io_capacity_max` 参数是后台任务定义每秒执行的IO操作数的上限，innodb_io_capacity_max通常设置为innodb_io_capacity的2倍。

如果MySQL服务器是SSD高速磁盘，我们可以设置 innodb_io_capacity_max= 6000  和 innodb_io_capacity = 3000  （最大值的50％）。当然 运行sysbench或任何其他基准测试工具来对磁盘吞吐量进行基准测试是个好主意。

#### 其他相关参数

除了上面列举的参数 还有些其他参数需要注意 ，篇幅关系 我将其汇总如下：

```mysql
#禁用所有DNS解析 建议开启 唯一的限制是GRANT语句必须仅使用IP地址
skip_name_resolve = 1
#MySQL server关闭空闲连接等待的秒数 默认为28800
interactive_timeout = ？  
wait_timeout = ？
#日志记录时间与系统保持一致
log_timestamps = SYSTEM
#一些日志相关参数
log_error = error.log
slow_query_log = 1
slow_query_log_file = slow.log
long_query_time = 3
#binlog日志删除策略 单位为天 默认为0 及不自动清理
expire_logs_days = 30 
#允许master创建function并同步到slave，有潜在的数据安全问题
log_bin_trust_function_creators = 1
#导出文件安全目录 默认为空
secure_file_priv = /tmp
```



**总结：**

本篇文章介绍了部分MySQL初始化应当注意的参数，给出了相关参数的默认值及是否可动态修改。对于不可动态修改的参数 建议启动前设置合理，这样可以减少后面维护重启次数。

在大家修改参数之前 请记住以下几点：

> - 一次更改一个设置！这是估计变更是否有益的唯一方法。
> - 不允许在配置文件中进行重复设置。如果要跟踪更改，请使用版本控制。
> - 更改前应该在测试环境演练。
> - 确保参数位置正确，单位合理，不和其他参数冲突。
> - 不要做天真的数学运算，比如“我的新服务器有2x内存，我只需要将所有值设置为以前的2倍”。