title: 【MySQL】Xtrabackup备份及恢复脚本
date: 2019-07-22 15:26:13
tags: [mysql]
categories: [mysql]
---
**简介：** 

此备份脚本的策略是每周日和周三进去全备 其余每天增量备份。

<!--more-->

```shell
# 备份脚本XtraBackup.sh：

#!/bin/bash
# filename      : XtraBackup.sh
# Author        : wang
day=`date +%w`
dt=`date +%Y%m%d`
lastday=`date -d '1 days ago' +%Y%m%d`
user=root
pwd='xxxxx'
log=backuplog.`date +%Y%m%d`

case $day in  
    0)  
        # Sunday Full backup
        find /backup/ -name "xtra_*" -mtime +6 -exec rm -rf {} \;
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd --no-timestamp /backup/xtra_base_$dt > /tmp/$log 2>&1
        ;;  
    1)  
        # Monday Relatively Sunday's incremental backup  
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd  --no-timestamp  --incremental  /backup/xtra_inc_$dt --incremental-basedir=/backup/xtra_base_$lastday > /tmp/$log 2>&1  
        ;;  
    2)  
        # Tuesday Compared with Monday's incremental backup  
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd  --no-timestamp  --incremental  /backup/xtra_inc_$dt --incremental-basedir=/backup/xtra_inc_$lastday > /tmp/$log 2>&1     
        ;;  
    3)  
        # Wednesday Full backup
        find /backup/ -name "xtra_*" -mtime +6 -exec rm -rf {} \;
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd --no-timestamp /backup/xtra_base_$dt > /tmp/$log 2>&1   
        ;;  
    4)  
        # Thursday  Relatively Wednesday's incremental backup  
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd  --no-timestamp  --incremental  /backup/xtra_inc_$dt --incremental-basedir=/backup/xtra_base_$lastday > /tmp/$log 2>&1    
        ;;  
    5)  
        # Friday Compared with Thursday's incremental backup  
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd  --no-timestamp  --incremental  /backup/xtra_inc_$dt --incremental-basedir=/backup/xtra_inc_$lastday > /tmp/$log 2>&1    
        ;;  
    6)  
        # Saturday Compared with Friday's incremental backup  
        innobackupex --defaults-file=/etc/my.cnf  --user=$user --password=$pwd  --no-timestamp  --incremental  /backup/xtra_inc_$dt --incremental-basedir=/backup/xtra_inc_$lastday > /tmp/$log 2>&1   
        ;;  
esac 
find /tmp -mtime +6 -type f -name 'backuplog.*' -exec rm -rf {} \;
```

```shell
# 全库恢复脚本 xtrabackup_recover.sh:

#!/bin/bash
# filename      : xtrabackup_recover.sh
# Author        : wang
day=`date +%w`
dt=`date +%Y%m%d`
lastday=`date -d '1 days ago' +%Y%m%d`
lasttwoday=`date -d '2 days ago' +%Y%m%d`
lastthreeday=`date -d '3 days ago' +%Y%m%d`
user=root
pwd='xxxxxxx'
log=recoverlog.`date +%Y%m%d`
datefile=/mysqldata

case $day in  
    0)  
        # Sunday Recover Database 
        innobackupex --apply-log  /backup/xtra_base_$dt > /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$dt >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start
        binlog=`cat /backup/xtra_base_$dt/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$dt/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    1)  
        # Monday Recover Database 
        innobackupex --apply-log --redo-only /backup/xtra_base_$lastday > /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastday/ --incremental-dir=/backup/xtra_inc_$dt/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastday >> /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$lastday >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start
        binlog=`cat /backup/xtra_base_$lastday/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$lastday/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    2)  
        # Tuesday Recover Database
        innobackupex --apply-log --redo-only /backup/xtra_base_$lasttwoday > /tmp/$log 2>&1
        innobackupex --apply-log --redo-only /backup/xtra_base_$lasttwoday/ --incremental-dir=/backup/xtra_inc_$lastday/ >> /tmp/$log 2>&innobackupex --apply-log  /backup/xtra_base_$lasttwoday/ --incremental-dir=/backup/xtra_inc_$dt/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lasttwoday >> /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-fil=/etc/my.cnf --copy-back /backup/xtra_base_$lasttwoday >> /tmp/$$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start
        binlog=`cat /backup/xtra_base_$lasttwoday/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$lasttwoday/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    3)  
        # Wednesday Recover Database 
        innobackupex --apply-log  /backup/xtra_base_$dt > /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$dt >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start 
        binlog=`cat /backup/xtra_base_$dt/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$dt/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    4)  
        # Thursday  Recover Database 
        innobackupex --apply-log --redo-only /backup/xtra_base_$lastday > /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastday/ --incremental-dir=/backup/xtra_inc_$dt/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastday >> /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$lastday >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start 
        binlog=`cat /backup/xtra_base_$lastday/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$lastday/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    5)  
        # Friday Recover Database  
        innobackupex --apply-log --redo-only /backup/xtra_base_$lasttwoday > /tmp/$log 2>&1
        innobackupex --apply-log --redo-only /backup/xtra_base_$lasttwoday/ --incremental-dir=/backup/xtra_inc_$lastday/ >> /tmp/$log 2>&innobackupex --apply-log  /backup/xtra_base_$lasttwoday/ --incremental-dir=/backup/xtra_inc_$dt/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lasttwoday >> /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$lasttwoday >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start
        binlog=`cat /backup/xtra_base_$lasttwoday/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$lasttwoday/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
    6)  
        # Saturday Recover Database  
        innobackupex --apply-log --redo-only /backup/xtra_base_$lastthreeday > /tmp/$log 2>&1
        innobackupex --apply-log --redo-only /backup/xtra_base_$lastthreeday/ --incremental-dir=/backup/xtra_inc_$lasttwoday/ >> /tmp/$log 2>&1
        innobackupex --apply-log --redo-only /backup/xtra_base_$lastthreeday/ --incremental-dir=/backup/xtra_inc_$lastday/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastthreeday/ --incremental-dir=/backup/xtra_inc_$dt/ >> /tmp/$log 2>&1
        innobackupex --apply-log  /backup/xtra_base_$lastthreeday >> /tmp/$log 2>&1
        service mysqld stop
        rm -rf $datefile/*
        innobackupex --defaults-file=/etc/my.cnf --copy-back /backup/xtra_base_$lastthreeday >> /tmp/$log 2>&1
        chown -R mysql:mysql $datefile
        service mysqld start
        binlog=`cat /backup/xtra_base_$lastthreeday/xtrabackup_binlog_info|awk '{print $1}'`
        pos=`cat /backup/xtra_base_$lastthreeday/xtrabackup_binlog_info|awk '{print $2}'`
        mysqlbinlog --no-defaults --start-position=$pos /mysqllog/$binlog | mysql -u$user -p$pwd
        ;;  
esac 
find /tmp -mtime +6 -type f -name 'recoverlog.*' -exec rm -rf {} \;
```