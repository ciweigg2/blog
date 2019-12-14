title: 【MySQL】线程状态详解
date: 2019-07-22 15:08:11
tags: [mysql]
categories: [mysql]
---
**前言：**我们常用 `show processlist` 或 `show full processlist ` 查看数据库连接状态，其中比较关注的是 State 列，此列表示该连接此刻所在的状态。那么你真的了解不同 State 值所表示的状态吗？下面我们参考官方文档来一探究竟

<!--more-->

> 以MySQL 5.7版本为例
>  官方文档地址：https://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html

简单翻译下：

- `After create`

  当线程在创建表的函数末尾创建表（包括内部临时表）时，会发生这种情况。即使由于某些错误而无法创建表，也会使用此状态。

- `Analyzing`

  线程正在计算`MyISAM`表键分布（例如，for [`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html)）。

- `checking permissions`

  线程正在检查服务器是否具有执行语句所需的权限。

- `Checking table`

  该线程正在执行表检查操作。

- `cleaning up`

  该线程已经处理了一个命令，并准备释放内存并重置某些状态变量。

- `closing tables`

  该线程正在将更改的表数据刷新到磁盘并关闭已使用的表。这应该是一个快速的操作。如果没有，请验证您的磁盘空间剩余。

- `converting HEAP to ondisk`

  该线程正在将内部临时表从 `MEMORY`表转换为磁盘表。

- `copy to tmp table`

  线程正在处理一个[`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)语句。在创建具有新结构的表但在将行复制到其中之前，将发生此状态。

  对于处于此状态的线程，可以使用性能模式来获取有关复制操作的进度。

- `Copying to group table`

  如果语句具有不同的条件`ORDER BY`和 `GROUP BY`标准，则按组对行进行排序并将其复制到临时表。

- `Copying to tmp table`

  服务器正在复制到内存中的临时表。

- `altering table`

  服务器正在执行就地 LLLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)。
e.html)。

- `Copying to tmp table on disk`

  服务器正在复制到磁盘上的临时表。

- `Creating index`

  线程正在处理`ALTER TABLE ... ENABLE KEYS`一个`MyISAM`表。

- `Creating sort index`

  线程正在处理[`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)使用内部临时表解析的线程 。

- `creating table`

  线程正在创建一个表。这包括创建临时表。

- `Creating tmp table`

  该线程正在内存或磁盘上创建临时表。如果表在内存中创建但稍后转换为磁盘表，则该操作期间的状态将为`Copying to tmp table on disk`。

- `committing alter table to storage engine`

  服务器已完成就地 [`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)并提交结果。

- `deleting from main table`

  服务器正在执行多表删除的第一部分。它仅从第一个表中删除，并保存用于从其他（引用）表中删除的列和偏移量。

- `deleting from reference tables`

  服务器正在执行多表删除的第二部分，并从其他表中删除匹配的行。

- `discard_or_import_tablespace`

  线程正在处理`ALTER TABLE ... DISCARD TABLESPACE`或`ALTER TABLE ... IMPORT TABLESPACE`声明。

- `end`

  这发生在结束，但的清理之前 [`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)， [`CREATE VIEW`](https://dev.mysql.com/doc/refman/5.7/en/create-view.html)， [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)， [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)， [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)，或 [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/update.html)语句。

- `executing`

  该线程已开始执行语句。

- `Execution of init_command`

  线程正在执行`init_command`系统变量值中的语句 。

- `freeing items`

  线程执行了一个命令。在此状态期间完成的一些项目的释放涉及查询缓存。这种状态通常紧随其后`cleaning up`。

- `FULLTEXT initialization`

  服务器正准备执行自然语言全文搜索。

- `init`

  出现这种情况的初始化之前 [`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)， [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)， [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)， [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)，或 [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/update.html)语句。服务器在此状态下采取的操作包括刷新二进制日志，`InnoDB`日志和一些查询缓存清理操作。

  对于`end`状态，可能会发生以下操作：

  - 删除表中的数据后删除查询缓存条目
  - 将事件写入二进制日志
  - 释放内存缓冲区，包括blob

- `Killed`

  有人[`KILL`](https://dev.mysql.com/doc/refman/5.7/en/kill.html) 向线程发送了一个语句，它应该在下次检查kill标志时中止。在MySQL的每个主循环中检查该标志，但在某些情况下，线程可能仍然需要很短的时间才能死掉。如果线程被某个其他线程锁定，则一旦另一个线程释放其锁定，kill就会生效。

- `logging slow query`

  该线程正在向慢查询日志写一条语句。

- `login`

  连接线程的初始状态，直到客户端成功通过身份验证。

- `manage keys`

  服务器正在启用或禁用表索引。

- `NULL`

  该状态用于该[`SHOW PROCESSLIST`](https://dev.mysql.com/doc/refman/5.7/en/show-processlist.html)状态。

- `Opening tables`

  线程正在尝试打开一个表。这应该是非常快的程序，除非有什么东西阻止打开。例如，一个[`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)或一个 [`LOCK TABLE`](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)语句可以阻止在语句结束之前打开表。

- `optimizing`

  服务器正在对查询执行初始优化。

- `preparing`

  在查询优化期间发生此状态。

- `Purging old relay logs`

  该线程正在删除不需要的中继日志文件。

- `query end`

  处理查询后但在`freeing items`状态之前发生此 状态。

- `Receiving from client`

  服务器正在从客户端读取数据包。`Reading from net`在MySQL 5.7.8之前调用此状态。

- `Removing duplicates`

  该查询使用 [`SELECT DISTINCT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)的方式是MySQL无法在早期阶段优化掉不同的操作。因此，在将结果发送到客户端之前，MySQL需要额外的阶段来删除所有重复的行。

- `removing tmp table`

  该线程在处理[`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) 语句后删除内部临时表。如果未创建临时表，则不使用此状态。

- `rename`

  该线程正在重命名一个表。

- `rename result table`

  线程正在处理一个[`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)语句，创建了新表，并重命名它以替换原始表。

- `Reopen tables`

  该线程获得了表的锁定，但在获取锁定之后注意到基础表结构发生了变化。它释放了锁，关闭了桌子，并试图重新打开它。

- `Repair by sorting`

  修复代码使用排序来创建索引。

- `preparing for alter table`

  服务器正准备执行就地 [`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)。

- `Repair done`

  该线程已完成对`MyISAM`表的多线程修复 。

- `Repair with keycache`

  修复代码通过密钥缓存逐个创建密钥。这比慢得多`Repair by sorting`。

- `Rolling back`

  该线程正在回滚一个事务。

- `Saving state`

  对于`MyISAM`诸如修复或分析的表操作，线程将新表状态保存到`.MYI`文件头。状态包括诸如行数， `AUTO_INCREMENT`计数器和密钥分发之类的信息。

- `Searching rows for update`

  该线程正在进行第一阶段以在更新之前查找所有匹配的行。如果 [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/update.html)要更改用于查找所涉及行的索引，则必须执行此操作。

- `Sending data`

  线程正在读取和处理[`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)语句的行 ，并将数据发送到客户端。由于在此状态期间发生的操作往往会执行大量磁盘访问（读取），因此它通常是给定查询生命周期中运行时间最长的状态。

- `Sending to client`

  服务器正在向客户端写入数据包。`Writing to net`在MySQL 5.7.8之前调用此状态。

- `setup`

  线程正在开始一个[`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)操作。

- `Sorting for group`

  线程正在进行排序以满足 `GROUP BY`。

- `Sorting for order`

  线程正在进行排序以满足`ORDER BY`。

- `Sorting index`

  该线程正在对索引页面进行排序，以便在`MyISAM`表优化操作期间进行更有效的访

- `Sorting result`

  对于[`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)声明，这类似于`Creating sort index`非临时表。

- `statistics`

  服务器正在计算统计信息以开发查询执行计划。如果线程长时间处于此状态，则服务器可能是磁盘绑定执行其他工作。

- `System lock`

  线程已经调用 `mysql_lock_tables()` ，并且线程状态尚未更新。这是一个非常普遍的状态，可能由于多种原因而发生。

  例如，线程将请求或正在等待表的内部或外部系统锁定。[`InnoDB`](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)在执行期间等待表级锁定时会 发生这种情况[`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)。如果此状态是由外部锁的请求引起的，并且您没有使用多个访问相同 表的[**mysqld**](https://dev.mysql.com/doc/refman/5.7/en/mysqld.html)服务器，则[`MyISAM`](https://dev.mysql.com/doc/refman/5.7/en/myisam-storage-engine.html)可以使用该[`--skip-external-locking`](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_external-locking) 选项禁用外部系统锁 。但是，默认情况下禁用外部锁定，因此该选项很可能无效。对于 [`SHOW PROFILE`](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)，这个状态意味着线程正在请求锁定（不等待它）。

- `update`

  线程正准备开始更新表。

- `Updating`

  线程正在搜索要更新的行并正在更新它们。

- `updating main table`

  服务器正在执行多表更新的第一部分。它仅更新第一个表，并保存用于更新其他（引用）表的列和偏移量。

- `updating reference tables`

  服务器正在执行多表更新的第二部分，并更新其他表中的匹配行。

- `User lock`

  该线程将要求或正在等待通过[`GET_LOCK()`](https://dev.mysql.com/doc/refman/5.7/en/locking-functions.html#function_get-lock)呼叫请求的咨询锁 。对于 [`SHOW PROFILE`](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)，此状态表示线程正在请求锁定（不等待它）。

- `User sleep`

  线程已经调用了一个 [`SLEEP()`](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html#function_sleep)调用。

- `Waiting for commit lock`

  [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/5.7/en/flush.html#flush-tables-with-read-lock) 正在等待提交锁定。

- `Waiting for global read lock`

  [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/5.7/en/flush.html#flush-tables-with-read-lock) 正在等待全局读锁定或[`read_only`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_read_only)正在设置全局 系统变量。

- `Waiting for tables`

  线程得到一个通知，表明表的底层结构已经改变，它需要重新打开表以获得新结构。但是，要重新打开表，它必须等到所有其他线程关闭了相关表。

- `Waiting for table flush`

  线程正在执行[`FLUSH TABLES`](https://dev.mysql.com/doc/refman/5.7/en/flush.html#flush-tables)并且正在等待所有线程关闭它们的表，或者线程得到一个表的基础结构已经更改的通知，并且它需要重新打开表以获取新结构。但是，要重新打开表，它必须等到所有其他线程关闭了相关表。

- `Waiting for *lock_type* lock`

  服务器正在等待`THR_LOCK`从元数据锁定子系统获取 锁定或锁定，其中 *lock_type*指示锁定的类型。

  此状态表示等待 `THR_LOCK`：

  - `Waiting for table level lock`

  这些状态表示等待元数据锁定：

  - `Waiting for event metadata lock`
  - `Waiting for global read lock`
  - `Waiting for schema metadata lock`
  - `Waiting for stored function metadata lock`
  - `Waiting for stored procedure metadata lock`
  - `Waiting for table metadata lock`
  - `Waiting for trigger metadata lock`

- `Waiting on cond`

  线程正在等待条件变为真的通用状态。没有具体的州信息。

- `Writing to net`

  服务器正在将数据包写入网络。`Sending to client`从MySQL 5.7.8开始调用此状态。

关于 Command 列的含义可以参考：https://dev.mysql.com/doc/refman/5.7/en/thread-commands.html
