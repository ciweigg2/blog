title: 【MySQL】pt-online-schema-change-工具使用
date: 2019-07-22 15:24:07
tags: [mysql]
categories: [mysql]
---
 在运维mysql数据库时，我们总会对数据表进行ddl 变更，修改添加字段或者索引，对于mysql 而已，ddl 显然是一个令所有MySQL dba 诟病的一个功能，因为在MySQL中在对表进行ddl时，会锁表，当表比较小比如小于1w上时，对前端影响较小，当时遇到千万级别的表 就会影响前端应用对表的写操作。
 
<!--more-->

perconal 推出一个工具 pt-online-schema-change ，其特点是修改过程中不会造成读写阻塞。

原理：

1.建立一个与需要操作的表相同表结构的空表

2.给空表执行表结构修改

3.在原表上增加delete/update/insert的after trigger

4.copy数据到新表

5.将原表改名，并将新表改成原表名

6.删除原表

7.删除trigger

pt-osc限制条件：

1.表要有主键，否则会报错;

2.表不能有trigger;

使用方法：

1.下载

wget http://percona.com/get/percona-toolkit.tar.gz

2.安装

tar -zxvf percona-toolkit.tar.gz

cd percona-toolkit-3.0.4

perl Makefile.PL

(若执行Makefile出错 则需先执行yum install perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker)

make

make test

make install

3.使用方法

pt-online-schema-change --help 可查看参数帮助

若查看参数提示Can't locate Digest/MD5.pm in @INC错误 则需执行yum -y install perl-Digest-MD5安装相关组件

提示缺少perl-DBI模块，那么直接 yum install perl-DBI

场景1：增加列

pt-online-schema-change --host=192.168.0.0 -uroot -pyourpassword --alter "add column age int(11) default null" D=test,t='test_tb' --execute --print --statistics --no-check-alter

场景2：删除列

pt-online-schema-change --host=192.168.0.0 -uroot -pyourpassword --alter "drop column age" D=test,t='test_tb' --execute --print --statistics --no-check-alter

场景3：更改列

pt-online-schema-change --host=192.168.0.0 -uroot -pyourpassword --alter "CHANGE id id_num int(20)" D=test,t='test_tb' --execute --print --statistics --no-check-alter

场景4：创建索引

pt-online-schema-change --host=192.168.0.0 -uroot -pyourpassword --alter "add index indx_ukid(address_ukid)" D=test,t='address_tb' --execute --print --statistics --no-check-alter