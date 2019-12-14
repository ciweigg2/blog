title: Docker搭建Oracle
date: 2019-11-07 16:44:01
tags: [docker,oracle]
categories: [综合]
---
### 前言

这两天需要搭建Oracle19c版本进行测试，发现去网上找，或者是去docker hub上搜索，基本上都是sath89/oracle12c这个或者是bfom/oracle-12c这些个镜像，但是就没有人制作了Oracle19c的镜像，我想，这个私人没有制作，那么Oracle自己肯定来了个offical(官方镜像)，然后我就去找啊找，但是结果：没有

<!--more-->

后面搜索到一个信息，大致意思是:Oracle官方不允许私人拿他的数据库进行制作一个即pull即run的docker image了，如果在docker hub上有的，那么估计被甲骨文律师事务所给警告删除了.不过，官方还是给了个方案来做Docker的Oracle image

### Oracle官方制作docker image过程

#### 先下载Oracle写的一个dockfile的东西

```bash
git clone https://github.com/oracle/docker-images.git
这里面包含很多版本的镜像制作脚本工具
我的是clone到了：/media/liuxu/data/leonard/tools/oracle19c
```

#### 下载Oracle对应版本的压缩包(zip文件，如果有多个那就下载多个)

```bash
https://www.oracle.com/database/technologies/oracle-database-software-downloads.html
这个地址包含了11.2-19c的版本的，如果想找之前，请拉到最后：Previous Database Release Software，进行查找
```

#### 将下载好的ZIP文件copy到dockfile目录

```bash
cp ~/Downloads/LINUX.X64_193000_db_home.zip /media/liuxu/data/leonard/tools/oracle19c/docker-images/OracleDatabase/SingleInstance/dockerfiles/19.3.0/
注意：看你的数据库版本，copy到对应的版本文件夹下面去
```

#### 运行命令，制作docker image

```bash
$ cd /media/liuxu/data/leonard/tools/oracle19c/docker-images/OracleDatabase/SingleInstance/dockerfiles 
$ ./buildDockerImage.sh -v 19.3.0 -e
注意：这一个步骤是漫长的等待。也就半个小时而已.输出信息太多，我就不贴了
```

#### 最后的输出日志：

```bash
Changing groupname of /opt/oracle/oraInventory to dba.
The execution of the script is complete.
Check /opt/oracle/product/19c/dbhome_1/install/root_b26c57265eb5_2019-11-06_05-08-00-978749653.log for the output of root script
Removing intermediate container b26c57265eb5
 ---> 05ead40e7f4e
Step 18/23 : USER oracle
 ---> Running in 6d577e76fba0
Removing intermediate container 6d577e76fba0
 ---> a51d59624aaf
Step 19/23 : WORKDIR /home/oracle
 ---> Running in 5114f387b868
Removing intermediate container 5114f387b868
 ---> 15f8e332761c
Step 20/23 : VOLUME ["$ORACLE_BASE/oradata"]
 ---> Running in bacecd10e611
Removing intermediate container bacecd10e611
 ---> ddbbf7bd1071
Step 21/23 : EXPOSE 1521 5500
 ---> Running in cf58f4f7a5ef
Removing intermediate container cf58f4f7a5ef
 ---> 4ea9a2715652
Step 22/23 : HEALTHCHECK --interval=1m --start-period=5m    CMD "$ORACLE_BASE/$CHECK_DB_FILE" >/dev/null || exit 1
 ---> Running in a63ca1e440ee
Removing intermediate container a63ca1e440ee
 ---> 6dfd61e0006a
Step 23/23 : CMD exec $ORACLE_BASE/$RUN_FILE
 ---> Running in 45122cd289f3
Removing intermediate container 45122cd289f3
 ---> c592b9cb2a9c
Successfully built c592b9cb2a9c
Successfully tagged oracle/database:19.3.0-ee


  Oracle Database Docker Image for 'ee' version 19.3.0 is ready to be extended: 
    
    --> oracle/database:19.3.0-ee

  Build completed in 919 seconds.
```

详细的可以参考这个:docker-images/OracleDatabase/SingleInstance/README.md 这个里面讲的很仔细(或者这个：https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance)

#### 创建目录

我们创建一个目录，以存储Oracle数据文件

```bash
mkdir -p /media/liuxu/data/leonard/tools/oracle19c/oracle19c-data
sudo chmod 777 -R /media/liuxu/data/leonard/tools/oracle19c/oracle19c-data
```

#### 启动

在第一次运行容器的时候，会自动创建新的数据库，使用-v参数，将之前新创建的目录映射到容器内的/opt/oracle/oradata目录中，这样就完成了将数据文件存储在本机文件系统中，而非docker容器内

第一次启动会比较慢 查看日志 等个30分钟左右吧

```bash
docker run --name oracle19c \
-p 1521:1521 \
-p 5500:5500 \
-v /media/liuxu/data/leonard/tools/oracle19c/oracle19c-data:/opt/oracle/oradata \
-d oracle/database:19.3.0-ee
```

注意第一行日志:ORACLE PASSWORD FOR SYS, SYSTEM AND PDBADMIN: iV86pmS7ZxI=1
后面:iV86pmS7ZxI=1 都是密码

如果需要修改数据库用户密码，可以在容器运行之后，通过以下命令修改

```bash
docker exec <container name> ./setPassword.sh <your password> 
```

比如下面的例子，将数据库sys用户密码设置为简单的oracle（当然这不推荐）

```bash
$ docker exec oracle19c ./setPassword.sh oracle
The Oracle base remains unchanged with value /opt/oracle

SQL*Plus: Release 19.0.0.0.0 - Production on Tue May 21 15:30:50 2019
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL>
User altered.

SQL>
User altered.

SQL>
Session altered.

SQL>
User altered.

SQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0
```

一直到出现以下字样，表示数据库已经正常创建并且可以使用了

```bash
######################### DATABASE IS READY TO USE! #########################
```

默认创建的数据库SID是ORCLCDB，创建的PDB是ORCLPDB1，也可以在第一次docker run的时候，用-e参数来指定SID和PDB的名字。比如：

```bash
docker run --name new-oracle19c \ 
-p 1521:1521 -p 5500:5500 \ 
-e ORACLE_SID=ORCL \ 
-e ORACLE_PDB=MYPDB1 \ 
-v /Users/Kamus/oracle/oradata/oracle19c:/opt/oracle/oradata \
-d oracle/database:19.3.0-ee
```

所有参数设置：

```bash
Parameters:
   --name:        The name of the container (default: auto generated)
   -p:            The port mapping of the host port to the container port. 
                  Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express)
   -e ORACLE_SID: The Oracle Database SID that should be used (default: ORCLCDB)
   -e ORACLE_PDB: The Oracle Database PDB name that should be used (default: ORCLPDB1)
   -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDB_ADMIN password (default: auto generated)
   -e ORACLE_CHARACTERSET:
                  The character set to use when creating the database (default: AL32UTF8)
   -v /opt/oracle/oradata
                  The data volume to use for the database
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container!
                  If omitted the database will not be persisted over container recreation.
   -v /opt/oracle/scripts/startup | /docker-entrypoint-initdb.d/startup
                  Optional: A volume with custom scripts to be run after database startup.
                  For further details see the "Running scripts after setup and on startup" section below
   -v /opt/oracle/scripts/setup | /docker-entrypoint-initdb.d/setup
                  Optional: A volume with custom scripts to be run after database setup.
                  For further details see the "Running scripts after setup and on startup" section below
```

#### 之后自己就可以使用软件登录Oracle了

参考链接：https://www.dbform.com/2019/05/06/how-to-build-and-run-oracle-database-19c-on-docker/

也可以在浏览器中登录Oracle 19c内置的EM Express来通过图形界面访问和监控

https://localhost:5500/em

![](/images/20191107170742.png)

### 问题

启动容器的时候碰到了这个问题:

```bash
Cannot create directory "/opt/oracle/oradata/ORCLCDB"
这个问题：是因为，你挂载出来了，然后Docker肯定是写到你挂载的路径下面，然后，没有写入权限。
解决方案就是：给个写的权限就行，我使用：sudo chmod 777 -R /media/liuxu/data/leonard/tools/oracle19c/oracle19c-data (有点暴力)
```