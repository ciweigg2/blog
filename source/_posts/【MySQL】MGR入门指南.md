title: 【MySQL】MGR入门指南
date: 2019-07-22 15:18:55
tags: [mysql]
categories: [mysql]
---
> 原文：[https://dev.mysql.com/doc/refman/8.0/en/group-replication-getting-started.html](https://dev.mysql.com/doc/refman/8.0/en/group-replication-getting-started.html)

<!--more-->

> 译者：kun

> 最近在翻译MySQL8.0官方文档 本文是第18.2入门指南部分。

MySQL组复制是MySQL server的插件，组中的每个server都需要配置和安装该插件。本节提供了一个详细的教程，其中包含创建至少三台server的复制组所需的步骤。

### 18.2.1在单主模式下部署组复制

组中的每个server实例可以在独立的物理机器上运行，也可以在同一台机器上运行。本节介绍如何在一台物理机上创建具有三个MySQL Server实例的复制组。这意味着需要三个数据目录，每个server实例一个，每个实例都需要单独配置。

**图18.4组架构**

![](/images/20190722151843.png)

本教程介绍如何使用组复制插件获取和部署MySQL Server，如何在创建组之前配置每个server实例以及如何使用Performance Schema来验证一切是否正常。



#### 18.2.1.1 部署组复制实例

第一步是部署MySQL服务器的三个实例。组复制是MySQL Server 8.0提供的内置MySQL插件。有关MySQL插件的更多背景信息，请参见第5.6节“MySQL服务器插件”。下载MySQL服务器包后，需要解压缩并安装二进制文件。此过程假定MySQL服务器已下载并解压缩到当前目录，该目录需要在mysql-8.0的目录下。由于本教程使用一个物理机，每个MySQL实例都需要一个特定的数据目录，用于存储实例的数据。在名为`data` 的目录中创建数据目录，并初始化每个目录。

```bash
mkdir data
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s1
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s2
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s3
```

里面`data/s1`，`data/s2`， `data/s3`是一个初始化的数据目录，包含了MySQL系统数据库和相关表等。要了解有关初始化过程的更多信息，请参见 第2.10.1节“初始化数据目录”。



> **Warning**
> 不要在生产环境中使用--initialize-insecure，它只用于简化教程。有关安全设置的更多信息，请参见第18.5节“组复制安全性”。



#### 18.2.1.2 配置组复制实例

本节介绍要用于组复制的MySQL Server实例所需的配置设置。有关背景信息，请参见 第18.8.2节“组复制限制”。

##### 组复制 Server设置

要安装和使用组复制插件，必须正确配置MySQL server实例。建议将配置存储在my.cnf文件中。有关详细信息，请参见第4.2.7节“文件的使用”。如没有特殊说明，以下是组中第一个实例的配置，在此节中称为s1。以下部分展示server的示例配置。

```bash
[mysqld]

# server configuration
datadir=<full_path_to_data>/data/s1
basedir=<full_path_to_bin>/mysql-8.0/

port=24801
socket=<full_path_to_sock_dir>/s1.sock
```

这些设置对MySQL server进行了配置，使其使用先前已创建的数据目录，并配置server应打开哪个端口，并开始侦听传入连接。

> **Note**
> 在此使用非默认端口24801，因为在本教程中，三个服务器实例使用相同的主机名。在具有三个不同机器的环境中，这种设置不是必需的。



组复制需要成员之间网络互通，这意味着每个成员必须能够解析所有其他成员的网络地址。例如，在本教程中，所有的三个实例都在一台机器上运行，因此为了确保成员可以相互联系，您还可以在配置文件中添加一行，例如 `report_host=127.0.0.1`。



##### 复制框架

以下设置根据MySQL组复制要求配置复制。

```bash
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
```

这些设置将server配置为使用唯一标识号1，以启用全局事务标识符，以允许仅执行基于GTID安全记录的语句并禁用二进制日志事件校验和。

如果您使用的是低于8.0.3的MySQL版本，其中默认值已针对复制进行了改进，则需要将这些行添加到成员的配置文件中。

```bash
log_bin=binlog
log_slave_updates=ON
binlog_format=ROW
master_info_repository=TABLE
relay_log_info_repository=TABLE
```

这些设置指定server打开二进制日志记录，使用基于行的格式，将复制元数据存储在系统表而不是文件中，并禁用二进制日志事件校验和。有关更多详细信息，请参见 第18.8.1节“组复制要求”。

##### 组复制设置

确保server中my.cnf文件此时已按要求配置，且server按配置实例化复制基础结构。以下部分是server组复制的配置。

```bash
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24901"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group=off
```

- 配置 `transaction_write_set_extraction` 指示服务器对于每个事务，它必须收集写集并使用XXHASH64哈希算法将其编码为散列。从MySQL 8.0.2开始，此设置是默认设置，因此可以省略此行。
- 配置 `group_replication_group_name` 告诉插件它正在加入或创建的组被命名为“aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa”<br />值 `group_replication_group_name` 必须是有效的UUID。在二进制日志中为组复制事件设置GTID时，将在内部使用此UUID。可使用`SELECT UUID()`生成一个UUID。
- 配置 `group_replication_start_on_boot` 指示插件在server启动时不自动启动组复制。这在设置组复制时很重要，因为它确保您可以在手动启动插件之前配置server。配置成员后，您可以设置 `group_replication_start_on_boot` 为on，以便在server重启时自动启动Group Replication。
- 配置 `group_replication_local_address` 告诉插件使用IP地址127.0.0.1和端口24901与组中的其他成员进行内部通信。

> **Important**
> server在此端口上侦听组成员之间的连接。此端口不能用于用户应用程序，它必须保留，用于在运行组复制时组的不同成员之间的通信。

 由 `group_replication_local_address` 配置的本地地址必须可供所有组成员访问。例如，如果每个server实例位于不同的计算机上，则可以使用计算机的IP地址，例如10.0.0.1。如果使用主机名，则必须使用完全限定名称，并确保它可以通过DNS解析，并且配置正确的 `/etc/hosts`文件或其他域名解析过程。从MySQL 8.0.14开始，可以使用IPv6地址（或可以解析到它的主机名）以及IPv4地址。一个组可以包含使用IPv6的成员和使用IPv4的成员的混合。有关IPv6网络以及混合IPv4和IPv6组的组复制支持的更多信息，请参见 第18.4.5节“支持IPv6以及混合IPv6和IPv4组”。

建议的端口 `group_replication_local_address` 是33061.在本教程中，我们使用在一台计算机上运行的三个server实例，因此使用端口24901到24903用于内部通信。 `group_replication_local_address` Group Replication使用它作为复制组中组成员的唯一标识符。只要主机名或IP地址都不同，您就可以为组复制的所有成员使用相同的端口，并且如本教程所示，只要具有相同的主机名或IP地址，就可以使用相同的主机名或IP地址。只是端口都不一样。

- 配置 `group_replication_group_seeds` 设置组成员的主机名和端口，新成员使用它们建立与组的连接。这些成员称为种子成员。建立连接后，组成员身份信息在 `performance_schema.replication_group_members`表中。通常， `group_replication_group_seeds`列表包含`hostname:port`每个组成员的列表 `group_replication_local_address`，但这不是强制性的，可以选择组成员的子集作为种子。

> **Important**
> 该`hostname:port`列在 `group_replication_group_seeds` 是种子成员的内部IP地址，基于配置`group_replication_local_address` ，而不是SQL `hostname:port`用于客户端连接，并且显示在`performance_schema.replication_group_members` 表中。

启动组的server不使用此选项，因为它是初始server，因此它负责引导组。第二个加入的server向组中的唯一成员申请加入，然后组得以扩容。第三个加入的server可以向这两个server中的任意一个申请加入，然后组再次扩容。后续server在加入时重复此过程。

> **Warning**
> 当多个server同时加入时，请确保它们已在组中的种子成员。不要使用同时正在申请加入组的种子成员，因为他们可能在访问时尚未加入群组。
> 首先启动引导成员并让它创建组， 然后使其成为正在加入的其余成员的种子成员。这确保了当加入其余成员时已有一个组。
> 不支持创建组并同时加入多个成员。在操作竞争时，这种情况可能会发生，但是加入组的行为最终会出现错误或超时。



要加入的成员必须使用种子成员在`group_replication_group_seeds` 选项中通告的相同协议（IPv4或IPv6）与种子成员通信 。出于组复制的IP地址白名单的目的，种子成员上的白名单必须包含种子成员提供的协议的加入成员的IP地址，或者解析为该协议的地址的主机名。除了加入成员之外，还必须设置此地址或主机名并列入白名单 `group_replication_local_address` 如果该地址的协议与种子成员的通告协议不匹配。如果加入成员没有适当协议的白名单地址，则拒绝其连接尝试。有关更多信息，请参见 第18.5.1节“IP地址白名单”。

- 配置 `group_replication_bootstrap_group` 指示插件是否是引导组。

> **Important**
> 此选项在任何时候只能在一个server实例上使用，通常是首次引导组时（或在整个组被崩溃然后恢复的情况下）。如果您多次引导组，例如，当多个server实例设置了此选项，则它们可能会人为地造成脑裂的情况，其中存在两个具有相同名称的不同组。在第一个server实例加入组后禁用此选项。

组中的所有server成员的配置都非常相似。您需要更改每个server的详细信息（例如`server_id`， `datadir`，`group_replication_local_address`）。这将在本教程的后面进行介绍。

#### 18.2.1.3 用户凭据

组复制使用异步复制协议来实现分布式恢复，在将组成员加入组之前将其同步。分布式恢复过程依赖于group_replication_recovery复制通道，该复制通道用于在组成员之间传输事务。因此，您需要设置具有正确权限的复制用户，以便组复制可以直接建立成员到成员的恢复复制通道。

使用配置文件启动服务器：

```bash
mysql-8.0/bin/mysqld --defaults-file=data/s1/s1.cnf
```

创建具有REPLICATION-SLAVE权限的MySQL用户。此操作不应记录到二进制日志中，以避免将更改传递到其他server实例。连接到server s1并执行以下语句：

```bash
mysql> SET SQL_LOG_BIN=0;
```

以下示例演示了创建用户rpl_user的过程，密码为rpl_pass。配置服务器时需要使用正确的用户名和密码。

```bash
mysql> CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
mysql> FLUSH PRIVILEGES;
```

如果禁用了二进制日志记录，则在创建用户后再次启用它。

```bash
mysql> SET SQL_LOG_BIN=1;
```

用户进行上述配置后，需要使用CHANGE MASTER TO语句将server配置为，在下次需要从其他成员恢复其状态时，使用group_replication_recovery复制通道的给定凭据。执行以下命令，将rpl_user和rpl_pass替换为创建用户时使用的值。

```bash
mysql> CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' \\
		      FOR CHANNEL 'group_replication_recovery';
```

分布式恢复是加入组的server执行的第一步。如果未正确设置这些凭据，server将无法执行恢复过程并获得与其他组成员同步，因此最终将无法加入组。

类似地，如果成员无法通过server的主机名正确识别其他成员，则恢复过程可能会失败。建议运行MySQL的操作系统都正确地配置唯一主机名，使用DNS或本地设置。可以在performance_schema.replication_group_members表的Member_host列中验证此主机名。如果多个组成员使用操作系统设置的默认主机名，则会出现有成员无法解析到正确的成员地址且无法加入组的情况。在这种情况下，可以使用report_host来配置每个server唯一主机名。

##### 使用组复制和Caching SHA-2用户凭据插件

默认情况下，在MySQL 8中创建的用户使用 第6.5.1.3节“缓存SHA-2插件身份验证”。如果您配置的用于分布式恢复的用户_`rpl_user`_使用缓存SHA-2认证插件并没有使用 第18.5.2，“安全套接字层支持（SSL）” 的`group_replication_recovery` 复制通道，RSA密钥用于密码交换，看第6.4.3节“创建SSL和RSA证书和密钥”。您可以将`rpl_user`应该从组中恢复其状态的成员的公钥复制到该组，也可以将捐赠者配置为在请求时提供公钥。

更安全的方法是将公钥复制 `rpl_user`到应该从捐赠者恢复组状态的成员。然后，您需要`group_replication_recovery_public_key_path` 在加入组的成员上配置 系统变量，并为其提供公钥的路径`rpl_user`。

可选地，不太安全的方法是设置 `group_replication_recovery_get_public_key=ON` 捐赠者，以便他们`rpl_user`在加入组时提供成员的公钥 。无法验证服务器的身份，因此只有`group_replication_recovery_get_public_key=ON` 在您确定没有服务器身份被泄露的风险时才会设置 ，例如通过中间人攻击。

#### 18.2.1.4 启动组复制

配置并启动server s1后，安装组复制插件。然后连接到server并执行以下命令：

```bash
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```

> **Important**
> 在加载组复制之前 ，mysql.session用户必须存在。 mysql.session用户在MySQL 8.0.2版中添加了。如果使用早期版本需要初始化数据字典，则必须执行MySQL升级过程（请参见第2.11节“升级MySQL”）。如果未运行升级，则组复制启动时会报错如下：There was an error when trying to access the server with user: mysql.session@localhost. Make sure the user is present in the server and that mysql_upgrade was ran after a server update..

要检查插件是否已成功安装，请执行SHOW PLUGINS并检查输出。它应该显示如下：

```bash
mysql> SHOW PLUGINS;
+----------------------------+----------+--------------------+----------------------+-------------+
| Name                       | Status   | Type               | Library              | License     |
+---------------------------+----------+--------------------+----------------------+--------------+
| binlog                     | ACTIVE   | STORAGE ENGINE     | NULL                 | PROPRIETARY |
(...)
| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | PROPRIETARY |
+----------------------------+--------------------+----------------------+-------------+
---------+
```

要启动组，请指示server s1引导组，然后启动组复制程序。此引导应仅由单个sever独立完成，该server启动组并且只启动一次。这就是为什么引导配置选项的值不保存在配置文件中的原因。如果将其保存在配置文件中，则在重新启动时，server会自动引导具有相同名称的第二个组。这将导致两个不同的组具有相同的名称。同样的道理适用于停止和重新启动插件，并且此选项设置为ON。

```bash
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;
```

START GROUP_REPLICATION语句返回后，组就已启动了。您可以检查该组现在是否已创建并且其中已经有一个成员：

```bash
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+----------------------------------   |
+---------------------------+------------------------------ation_applier | ce9be252-2b71-11e6-b8f4-00212844f856 | myhost      |       24801 | ONLINE        |
+---------------------------+---------------------------+-------------+-------------+----------+---------------+
```

此表中的信息确认了组中有一个成员并且具有唯一的标识符ce9be252-2b71-11e6-b8f4-00212844f856，它是在线的并且在myhost侦听端口24801上的客户端连接。

为了演示server确实在一个组中，并且能够处理加载，创建一个表并向其中添加一些内容。

```bash
mysql> CREATE DATABASE test;
mysql> USE test;
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL);
mysql> INSERT INTO t1 VALUES (1, 'Luis');
```

检查表t1和二进制日志的内容。

```bash
mysql> SELECT * FROM t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+
mysql> SHOW BINLOG EVENTS;
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |   4 | Format_desc    |         1 |         123 | Server ver: 8.0.2-gr080-log, Binlog ver: 4                        |
| binlog.000001 | 123 | Previous_gtids |         1 |         150 |                                                         |
| binlo  |
| binlog.000001 | 150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 | 211 | Query          |         1 |     |
| binlog.000001 | 270 | View_change    |         1 |        1 |           369 | view_id=14724817264259180:1                                        |
| binlog.000001 | 369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 | 434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 | 495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 | 585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 | 646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 | 770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 | 831 | Query          |         1 |         899      |
| binlog.000001 | 899 | Table_map      |         1 |        942 | table_id: 108 (test.t1)                                             |
| binlog.000001 | 942 | Write_rows     |         1 |         984 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 | 984 | Xid            |         1 |        1011 | COMMIT /* xid=38 */                                                |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
```

如上所示，创建了数据库和表对象，并且将其对应的DDL语句写入二进制日志。此外，数据被插入到表中并被写入二进制日志。当组成员增长并且由于新成员尝试加入并变为在线而执行分布式恢复时，二进制日志条目的重要性将在以下部分中说明。

#### 18.2.1.5 向组中添加实例

此时，组中有一个成员，已经有一些数据存在的server s1。此时就可以通过添加先前配置的其他两个server来扩展组了。

##### 18.2.1.5.1 添加第二个实例

为了添加第二个实例server s2，首先为它创建配置文件。该配置类似于用于server s1的配置，除了诸如数据目录的位置，s2将要监听的端口或其server_id之类的事情之外。这些不同的行已在下面的列表中突出显示。

```bash
[mysqld]
# server configuration
datadir=<full_path_to_data>/data/s2
basedir=<full_path_to_bin>/mysql-8.0/
port=24802
socket=<full_path_to_sock_dir>/s2.sock
#
# Replication configuration parameters
#
server_id=2
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
#
# Group Replication configuration
#
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24902"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group= off
```

server s1的过程类似，在配置文件就位的情况下启动server。

```bash
mysql-8.0/bin/mysqld --defaults-file=data/s2/s2.cnf
```

然后按如下所示配置恢复凭据。由于用户在组中共享，该命令与设置server s1时使用的命令相同。在s2上执行以下语句。

```bash
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' \\
	FOR CHANNEL 'group_replication_recovery';
```

> **Tip**
> 如果您使用的是缓存SHA-2身份验证插件（MySQL 8中的默认设置），请参阅 使用组复制和缓存SHA-2用户凭据插件。

安装组复制插件，并启动将server加入组的程序。以下示例使用与部署server s1时相同的方式安装插件。

```bash
mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```

将server s2添加到组。

```bash
mysql> START GROUP_REPLICATION;
```

与之前的步骤不同，这里与s1上执行的那些步骤有一个区别，就是不执行SET GLOBAL group_replication_bootstrap_group = ON的操作; 在启动组复制之前，因为该组已由server s1创建和引导。此时，server s2只需要添加到已经存在的组中。

> **Tip**
> 当组复制成功启动并且服务器加入组时，它会检查 `super_read_only`变量。通过`super_read_only` 在成员的配置文件中设置为ON，可以确保因任何原因启动组复制时出现故障的服务器不接受事务。如果服务器应将该组作为读写实例加入，例如作为单主组中的主要组或多主组的成员，则当该 `super_read_only`变量设置为ON时，则在加入时会将其设置为OFF。

再次检查performance_schema.replication_group_members表，可以看出组中现在有两个ONLINE的server。

```bash
mysql> SELECT * FROM performance_schema.replication_group_members;
+----------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 | myhost      |       24801 | ONLINE        |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 | myhost      |       24802 | ONLINE        |
+---------------+---------------+
```

在server s2或server s1上发出此相同的查询会产生相同的结果。----+-------------+---------------+
```

由于server s2也被标记为ONLINE，它必须已经与server s1同步。按照如下方式验证它确实与server s1同步。

```bash
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+
mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+
mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         2 |         123 | Server ver: 8.0.3-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         2 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.00001 |  369 | Query          |         1 |         434 | COMMIT                                                              |
| binlog.000001 |  434 | Gtid           |         1 |         495 |2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |       1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa---aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=30 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT  ---------------------------------------------------------+
```------+
```

如上所示，第二个server已添加到组中，并且已自动从server s1复制了更改。按照分布式恢复过程，这意味着加入组之后并且在即将被声明在线之前，server s2自动地连接到server s1并且从其获取丢失的数据。换句话说，它从s1的二进制日志中复制了它在加入组的时间节点之前缺少的事务。

##### 18.2.1.5.2 添加其他实例

向组添加其er基本上是相同的步骤，除了配置必须更改为对应的server。总结所需的命令如下：

_1.创建配置文件_

```basic
[mysqld]
# server configuration
datadir=<full_path_to_data>/data/s3
basedir=<full_path_to_bin>/mysql-8.0/
port=24803
socket=<fulsocket=<fullll_path_to_sock_dir>/s3.sock
#
# Replication configuration parameters
#
server_id=3
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
#
# Group Replication configuration
#
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24903"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group= off
```

_2.启动server_

```bash
mysql-8.0/bin/mysqld --defaults-file=data/s3/s3.cnf
```

_3.配置group_replication_recovery通道的恢复凭据。_

```bash
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password'  \\
FOR CHANNEL 'group_replication_recovery';
```

_4.安装Group Replication插件并启动它。_

```bash
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
START GROUP_REPLICATION;
```

此时，server s3被引导并且正在运行，并且已经加入组且已与组中的其他server成员同步。访问performance_schema.replication_group_members表再次确认真实情况确实如此。

```bash
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 | myhost      |       24801 | ONLINE        |
| group_replication_applier | 7eb217ff-6df3-11e6-966c-00212844f856 | myhost      |       24803 | ONLINE        |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 | myhost      |       2480       |
+---------------------------+--------------------------
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 | myhost      |       24801 | ONLINE        |
| group_replic3已经同步：

```bash
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+
mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+
mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         3 |         123 | Server ver: 8.0.3-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         3 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.000001 |  369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 |  434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=29 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT                                                             |
| binlog.000001 | 1326 | Gtid           |         1 |        1387 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:6'  |
| binlog.000001 | 1387 | Query          |         1 |        1446 | BEGIN                                                              |
| binlog.000001 | 1446 | View_change    |         1 |        1585 | view_id=14724832985483517:3                                        |
| binlog.000001 | 1585 | Query          |         1 |        1650 | COMMIT                                                             |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```
