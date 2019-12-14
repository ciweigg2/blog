title: 【MySQL】监控组复制
date: 2019-07-22 15:05:00
tags: [mysql]
categories: [mysql]
---
> 原文：https://dev.mysql.com/doc/refman/8.0/en/group-replication-monitoring.html

> 译者：kun

> 最近在翻译MySQL8.0官方文档 本文是第18.3“监控组复制”部分。

<!--more-->

# 18.3 监控组复制

假设MySQL已经在启用了性能模式的情况下编译，使用Perfomance Schema表监控组复制。组复制添加以下表：

- `performance_schema.replication_group_member_stats`
- `performance_schema.replication_group_members`

这些现有的Perfomance Schema复制表也显示有关组复制的信息：

- `performance_schema.replication_connection_status` 显示有关组复制的信息，例如，已从组接收并在应用程序队列中排队的事务（中继日志）。
- `performance_schema.replication_applier_status` 显示与组复制相关的通道和线程的状态，如果有许多不同的工作线程应用事务，那么这个表也可用于监视每个工作线程正在执行的操作。

Group Replication插件创建的复制通道命名为：

- `group_replication_recovery` - 此通道用于与分布式恢复阶段相关的复制更改。
- `group_replication_applier` - 此通道用于来自组的传入更改。并且应用直接来自组的事务的通道。



以下部分描述了每个表中可用的信息。

### 18.3.1 组成员实例状态

组中的server实例可以处于多种状态。如果server都正常通信，则所有server都报告相同的状态。但是，如果存在网络分隔，或者组成员离开组，则可能报告不同的信息，这取决于查询了哪个server。要注意的是，如果某个组成员已经离开组，那么显然它不能报告关于其他server状态的最新信息。如果发生网络分隔，如果超出仲裁数量的server都断开了，那么server之间将不能相互协作。因此，他们无法得知不同server成员的状态。因此，他们会报告一些server不可访问，而不是猜测他们的状态。



**表18.1 Server State** 

| Field         | 描述                                                         | 组同步 |
| ------------- | ------------------------------------------------------------ | ------ |
| `ONLINE`      | 该成员可以作为一个具有所有功能的组成员，这意味着客户端可以连接并开始执行事务。 | Yes    |
| `RECOVERING`  | 该成员正在成为该组的有效成员，并且正处于恢复过程中，从数据源节点（数据源节点）接收状态信息。 | No     |
| `OFFLINE`     | 插件已加载，但成员不属于任何组。                             | No     |
| `ERROR`       | 本地成员的状态。 只要恢复阶段或应用更改时出现错误，server就会进入此状态。 | No     |
| `UNREACHABLE` | 每当本地故障检测器怀疑某个给定的server可能由于已经崩溃或被意外地断开而不可访问时，server的状态显示为“UNREACHABLE” | No     |



> **Important**
>
> 一旦实例进入`ERROR`状态后，该 `super_read_only`选项将设置为`ON`。要离开`ERROR` 状态，您必须手动配置实例`super_read_only=OFF`



需要注意的是，组复制不是同步复制，但最终是同步的。更确切地说，事务以相同的顺序传递给所有组成员，但是它们的执行不同步，这意味着在接受事务被提交之后，每个成员以其自己的速度提交。

### 18.3.2 replication_group_members表

该 `performance_schema.replication_group_members` 表用于监视作为组成员的不同server实例的状态。每当视图更改时，表replication_group_members就会更新，例如，当组的配置动态更改时。在此基础上，server成员之间交换他们的一些元数据以保持同步并继续协作。信息在组复制成员之间共享，因此可以从任何成员查询有关所有组成员的信息。此表可用于获取复制组状态的高级视图，例如通过发出：

```
SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+--------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST  | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+------------------+--------------+-------------+----------------+
| groooup_replication_applier | 041f26d8-f3f3-11e8-adff-080027337932 0027337932 | example1     |      3306   | ONLINE       | SECONDARY   | 8.0.13         |
| group_replication_applier | f60a3e10-f3f2-11e8-8258-080027337932 | example2     |      3306   | ONLINE       | PRIMARY     | 8.0.13         |
| group_replication_applier | fc890014-f3f2-11e8-a9fd-080027337932 | example3     |      3306   | ONLINE       | SECONDARY   | 8.0.13         |
+---------------------------+--------------------------------------+--------------+-------------+--------------+-------------+----------------+
```



根据这个结果，我们可以看到该组由三个成员组成，每个成员的主机和端口号，客户端用来连接成员，以及成员的 `server_uuid`。该`MEMBER_STATE`列显示了 第18.3.1节“组成员实例状态”之一，在该情况下，它显示该组中的所有三个成员都是 `ONLINE`，并且该`MEMBER_ROLE` 列显示有两个从节点和一个主节点。因此，该组必须是以单主模式运行的。`MEMBER_VERSION`当您升级组并且组合中正在运行不同MySQL版本的成员时，该列可能很有用。请参见 第18.3.1节“组成员实例状态” 获得更多信息。



有关`Member_host` 值及其对分布式恢复过程的影响的更多信息，请参见 第18.2.1.3节“用户凭据”。

### 18.3.3 Replication_group_member_stats

复制组中的每个成员都会验证并应用该组提交的事务。有关验证和应用程序的统计信息对于了解申请队列增长情况、触发了多少冲突、检查了多少事务、哪些事务已被所有成员提交等等非常有用。



该 `performance_schema.replication_group_member_stats` 表提供与认证过程相关的组级信息，以及由复制组的每个成员接收和发起的事务的统计信息。信息在组成员实例之间共享，因此可以从任何成员查询有关所有组成员的信息。请注意，刷新远程成员的统计信息由`group_replication_flow_control_period` 选项中指定的消息周期控制 ，因此这些信息可能与进行查询的成员的本地收集的统计信息略有不同。



**表18.2 replication_group_member_stats**

| Field                                      | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| CHANNEL_NAME                               | 组复制通道的名称。                                           |
| VIEW_ID                                    | 此组的当前视图标识符。                                       |
| Member_id                                  | 此值为我们当前连接到的server成员的UUID。组中的每个成员具有不同的值。因为它对每个成员是唯一的，所以它也成为了一个关键字。 |
| Count_transactions_in_queue                | 队列中等待冲突检测检查的事务数。冲突检查通过后，他们排队等待应用。 |
| Count_transactions_checked                 | 表示已进行过冲突检查的事务数。                               |
| Count_conflicts_detected                   | 表示未通过冲突检测检查的事务数。                             |
| Count_transactions_rows_validating         | 表示冲突检测数据库的当前大小（每个事务经过验证的数据库）。   |
| Transactions_committed_all_members         | 表示已在当前视图的所有成员上成功提交的事务。 此值以固定的时间间隔更新。 |
| Last_conflict_free_transaction             | 显示最后一个经检查无冲突的事务标识符。                       |
| Count_transactions_remote_in_applier_queue | 此成员从复制组收到的等待应用的事务数。                       |
| Count_transactions_remote_applied          | 此成员从已应用的复制组收到的事务数。                         |
| Count_transactions_local_proposed          | 此成员发起并发送到复制组以进行协调的事务数。                 |
| Count_transactions_local_rollback          | 此成员发起的事务在发送到复制组后的回滚数。                   |



这些字段对于监控组中的成员的性能很重要。例如，假设组的成员之一出现延迟，并且不能与该组的其他成员同步。在这种情况下，您可能会在队列中看到大量的事务。基于此信息，您可以决定从组中删除成员或延迟组中其他成员的事务处理，从而减少排队的事务的数量。此信息还可以帮助您决定如何调整组复制插件的流控制。
