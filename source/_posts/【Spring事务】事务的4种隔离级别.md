title: 【Spring事务】mysql事务的4种隔离级别
date: 2019-08-22 10:16:41
tags: [spring事务]
categories: [综合]
---
### 什么是事务

事务是应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消。也就是事务具有原子性，一个事务中的一系列的操作要么全部成功，要么一个都不做。

<!--more-->

事务的结束有两种，当事务中的所以步骤全部成功执行时，事务提交。如果其中一个步骤失败，将发生回滚操作，撤消撤消之前到事务开始时的所以操作

### 事务的 ACID

事务具有四个特征：原子性（ Atomicity ）、一致性（ Consistency ）、隔离性（ Isolation ）和持续性（ Durability ）。这四个特性简称为 ACID 特性。

* 原子性。事务是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做

* 一致性。事 务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于一致性状态。如果数据库系统 运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是 不一致的状态。

* 隔离性。一个事务的执行不能其它事务干扰。即一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。

* 持续性。也称永久性，指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执行结果有任何影响

### 事物的隔离级别

数据库事务的隔离级别有4种，由低到高分别为Read uncommitted 、Read committed 、Repeatable read 、Serializable 。而且，在事务的并发操作中可能会出现脏读，不可重复读，幻读。下面介绍一下

#### Read uncommitted

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）

#### Read committed

这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果

#### Repeatable read

这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题

#### Serializable 序列化

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争

> 这四种隔离级别采取不同的锁类型来实现，若读取的是同一个数据的话，就容易发生问题。例如：

* 脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。

* 不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

* 幻读(Phantom Read):在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就有几列数据是未查询出来的，如果此时插入和另外一个事务插入的数据，就会报错

在MySQL中，实现了这四种隔离级别，分别有可能产生问题如下所示：

![](/images/20190822173234.png)

### 测试

下面，将利用MySQL的客户端程序，我们分别来测试一下这几种隔离级别。

测试数据库为test，表为user；表结构：

```sql
CREATE TABLE `user`  (
  `id` int(11) NOT NULL COMMENT '用户id',
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NULL DEFAULT NULL COMMENT '用户名称'
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '用户表' ROW_FORMAT = Dynamic;
```

两个命令行客户端分别为A，B；不断改变A的隔离级别，在B端修改数据

#### 将A的隔离级别设置为read uncommitted(未提交读)

将A的隔离级别设置为read uncommitted(未提交读)

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

A：启动事务，此时数据为初始状态

```sql
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM `user`;
```

B：启动事务，更新数据，但不提交

```sql
START TRANSACTION;
UPDATE `user` SET `name` = '测试' WHERE id = 1;
SELECT * FROM `user`;
```

A：再次读取数据，发现数据已经被修改了，这就是所谓的"脏读"

B：回滚事务

```sql
ROLLBACK;
```

A：再次读数据，发现数据变回初始状态

经过上面的实验可以得出结论，事务B更新了一条记录，但是没有提交，此时事务A可以查询出未提交记录。造成脏读现象。未提交读是最低的隔离级别

#### 将客户端A的事务隔离级别设置为read committed(已提交读)

将客户端A的事务隔离级别设置为read committed(已提交读)

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

A：启动事务，此时数据为初始状态

```sql
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM `user`;
```

B：启动事务，更新数据，但不提交

```sql
START TRANSACTION;
UPDATE `user` SET `name` = '测试' WHERE id = 1;
SELECT * FROM `user`;
```

A：再次读数据，发现数据未被修改

B：提交事务

```sql
COMMIT;
```

A：再次读取数据，发现数据已发生变化，说明B提交的修改被事务中的A读到了，这就是所谓的"不可重复读"

经过上面的实验可以得出结论，已提交读隔离级别解决了脏读的问题，但是出现了不可重复读的问题，即事务A在两次查询的数据不一致，因为在两次查询之间事务B更新了一条数据。已提交读只允许读取已提交的记录，但不要求可重复读

#### 将A的隔离级别设置为repeatable read(可重复读)

将A的隔离级别设置为repeatable read(可重复读)

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

A：启动事务，此时数据为初始状态

```sql
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM `user`;
```

B：启动事务，更新数据，但不提交

```sql
START TRANSACTION;
UPDATE `user` SET `name` = '测试' WHERE id = 1;
SELECT * FROM `user`;
```

A：再次读取数据，发现数据未被修改

B：提交事务

```sql
COMMIT;
```

A：再次读取数据，发现数据依然未发生变化，这说明这次可以重复读了

B：插入一条新的数据，并提交

```sql
INSERT INTO `user` (`id` ,`name`) VALUES (13 ,'test');
COMMIT;
```

A：再次读取数据，发现数据依然未发生变化，虽然可以重复读了，但是却发现读的不是最新数据，这就是所谓的"幻读"

A：提交本次事务，再次读取数据，发现读取正常了

```sql
COMMIT;
```

由以上的实验可以得出结论，可重复读隔离级别只允许读取已提交记录，而且在一个事务两次读取一个记录期间，其他事务部的更新该记录。但该事务不要求与其他事务可串行化。例如，当一个事务可以找到由一个已提交事务更新的记录，但是可能产生幻读问题(注意是可能，因为数据库对隔离级别的实现有所差别)。像以上的实验，就没有出现数据幻读的问题

#### 将A的隔离级别设置为可串行化(Serializable)

将A的隔离级别设置为可串行化(Serializable)

```
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

A：启动事务，此时数据为初始状态

```sql
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM `user`;
```

B：发现B此时进入了等待状态，原因是因为A的事务尚未提交，只能等待（此时，B可能会发生等待超时）

```sql
INSERT INTO `user` (`id` ,`name`) VALUES (13 ,'test');
COMMIT;
```

A：提交事务

```sql
COMMIT;
```

B：发现插入成功

serializable完全锁定字段，若一个事务来查询同一份数据就必须等待，直到前一个事务完成并解除锁定为止。是完整的隔离级别，会锁定对应的数据表格，因而会有效率的问题

值得一提的是：大多数数据库默认的事务隔离级别是Read committed，比如Sql Server ， Oracle。Mysql的默认隔离级别是Repeatable read

如果上面测试出现update没效果是因为事务有相同的阻塞了 解决办法：

```sql
select * from information_schema.innodb_trx;
show processlist;
kill ${trx_mysql_thread_id}
```

参考：[mysql事务隔离级别](https://www.jianshu.com/p/8d735db9c2c0)