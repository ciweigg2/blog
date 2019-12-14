title: 【MySQL】利用binlog回滚DML操作
date: 2019-07-22 15:05:53
tags: [mysql]
categories: [mysql]
---
**简介**：

数据库运行过程中难免会发生误操作，特别是在测试环境 开发人员或测试人员有时会误删或者更新错误某些数据。这时可以用binlog闪回DML操作。

<!--more-->

**条件：** 

- 1.mysql binlog必须存在且是row格式的
- 2.反向生成的表必须有主键
- 3.表结构不能有更改

##### 1.shell脚本闪回：

```shell
# 脚本 del_time_recovery.sh（根据起止 time恢复）用于回滚delete操作：

#!/bin/bash
# File Name   : del_time_recovery.sh
# Author      : wang
# Description : delete recover according to starttime and endtime.
Usage() {
cat << EOF
mysql_delete_recovery
OPTIONS:
   -b      binlog name
   -s      starttime
   -e      endtime
   -d      database name
   -t      table name
For secrity: This scripts check the full need arguments
EOF
}
while getopts ":b:s:e:d:t:" opt; do
  case $opt in
    b)
      logname=${OPTARG}
      ;;
    s)
      starttime=${OPTARG}
      ;;
    e)
      endtime=${OPTARG}
      ;;
    d)
      db=${OPTARG}
      ;;
    t)
      table=${OPTARG}
      ;;
    \?)
      echo "Invalid option: -$OPTARG" >&2
      exit 1
      ;;
    :)
      echo "Option -$OPTARG requires an argument." >&2
      Usage
      exit 1
      ;;
  esac
done
if [ $# != 10 ] ; then
    Usage
    exit 1;
fi
PATH=$[PATH:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin:/usr/local/mysql/bin](http://path/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin:/usr/local/mysql/bin)
export PATH

user=root
pwd='xxxxxxxx'
tmpfile=/tmp/del_recovery_$table.sql
mysqlbinlog --no-defaults -vv --base64-output=DECODE-ROWS --start-datetime="$starttime" --stop-datetime="$endtime" $logname |sed -n '/### DELETE FROM `'${db}'`.`'${table}'`/,/COMMIT/p' | \
sed -n '/###/p'    | \
sed 's/### //g;s/\/\*.*/,/g;s/DELETE FROM/INSERT INTO/g;s/WHERE/SELECT/g;'   > $tmpfile
n=0;
for i in `mysql -u$user -p$pwd --skip-column-names --silent -e "desc $db.$table" |awk '$0=$1'`;
do
        ((n++));
done
sed -i -r "s/(@$n.*),/\1;/g" $tmpfile
sed -i 's/@[1-9].*=//g' $tmpfile
sed -i 's/@[1-9][0-9]=//g' $tmpfile

# 用法：-b -s  -e -d -t 分别带别binlog名字 开始的time 结束的time 库名 表名，
# 直接使用  sh del_time_recovery.sh -b /mysqllog/mysql-bin.000005 -s "2017-11-02 19:10:00" -e "2017-11-02 19:20:00" -d test_db -t test_tb 即可调用
```

```shell
# 脚本 update_time_recovery.sh（根据起止 time恢复）用于回滚update操作：

#!/bin/bash
# File Name   : update_time_recovery.sh
# Author      : wang
# Description : update recover according to starttime and endtime.
Usage() {
cat << EOF
mysql_update_recovery
OPTIONS:
   -b      binlog name
   -s      starttime
   -e      endtime
   -d      database name
   -t      table name
For secrity: This scripts check the full need arguments
EOF
}
while getopts ":b:s:e:d:t:" opt; do
  case $opt in
    b)
      logname=${OPTARG}
      ;;
    s)
      starttime=${OPTARG}
      ;;
         
    e)
      endtime=${OPTARG}
      ;;
         
    d)
      db=${OPTARG}
      ;;
    t)
      table=${OPTARG}
      ;;
    \?)
      echo "Invalid option: -$OPTARG" >&2
      exit 1
      ;;
    :)
      echo "Option -$OPTARG requires an argument." >&2
      Usage
      exit 1
      ;;
  esac
done
if [ $# != 10 ] ; then
    Usage
    exit 1;
fi
user=root
pwd='xxxxxxx'
tmpfile=/tmp/update_recovery_$table.sql
n=0;
for i in `mysql -u$user -p$pwd --skip-column-names --silent -e "desc $db.$table" |awk '$0=$1'`;
do
        ((n++));
done
mysqlbinlog --no-defaults -vv --base64-output=DECODE-ROWS --start-datetime="$starttime" --stop-datetime="$endtime" $logname |sed -n '/### UPDATE `'${db}'`.`'${table}'`/,/COMMIT/p'          \
|       sed '/WHERE/{:a;N;/SET/!ba;s/\([^\n]*\)\n\(.*\)\n\(.*\)/\3\n\2\n\1/}'                                                   \
|       sed -r '/WHERE/{:a;N;/@'"$n"'/!ba;s/###   @2.*//g}'                                                               \          \
|       sed 's/### //g;s/\/\*.*/,/g'                                                                                            \
|       sed '/WHERE/{:a;N;/@1/!ba;s/,/;/g};s/#.*//g;s/COMMIT,//g'                                                               \
|       sed '/^$/d'     > $tmpfile
n=0;
for i in `mysql -u$user -p$pwd --skip-column-names --silent -e "desc $db.$table" |awk '$0=$1'`;
do
        ((n++));
          sed -i "s/@$n\b/$i/g" $tmpfile
done
sed -i -r "s/($i=.*),/\1/g" $tmpfile

# 用法：-b -s  -e -d -t 分别带别binlog名字 开始的time 结束的time 库名 表名，
# 直接使用  sh update_time_recovery.sh -b /mysqllog/mysql-bin.000005 -s "2017-11-03 14:30:00" -e "2017-11-03 15:00:00" -d test_db -t test_tb 即可调用
```

##### 2.利用mysqlbinlog_back.py 脚本：

*参考：*
 [【MySQL】mysqlbinlog_flashback工具使用](https://www.jianshu.com/p/e99dd813d194)

##### 3.利用MyFlash工具：

*参考：*
[【MySQL】MyFlash 回滚mysql binlog](https://www.jianshu.com/p/cf72a525157e)

网上还有很多类似的开源项目 如：binlog2sql等 都可以参考下。