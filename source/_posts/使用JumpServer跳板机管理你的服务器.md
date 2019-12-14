cover: http://ciwei2.cn-sh2.ufileos.com/137.jpg
title: 使用JumpServer跳板机管理你的服务器
date: 2019-04-19T11:39:48.360Z
tags: [jumpserver]
categories: [综合]
---
### 1.Jumpserver介绍

Jumpserver是一款开源的开源的堡垒机，如下图是官网介绍。

<!--more-->

![](/images/16a34bf373092d00.jpg)

### 2.安装

下面生成的密码一定要记住，以后会用的

mysql

```java
# mysql
$ create database jumpserver default charset 'utf8';
$ grant all on jumpserver.* to 'jumpserver'@'%' identified by 'weakPassword';
```

官方文档推荐了很多安装方式，这里由于老杨使用的是使用Docker安装，使用的自己的Redis和Mysql，首先生成随机加密秘钥(SECRET_KEY)，命令如下：

```java
if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi
```

然后生成BOOTSTRAP_TOKEN，命令如下：

```java
if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi
```

这里都是按照官方文档命令来的，然后执行Docker命令(注：需自行安装docker)，如下所示。

docker run --name jms_all -d \
    -v /opt/mysql:/var/lib/mysql \
    -v /opt/jumpserver:/opt/jumpserver/data/media \
    -p 80:80 \
    -p 2222:2222 \
    -e SECRET_KEY=$SECRET_KEY\
    -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
    -e DB_HOST=*** \
    -e DB_PORT=*** \
    -e DB_USER=*** \
    -e DB_PASSWORD=*** \
    -e DB_NAME=*** \
    -e REDIS_HOST=*** \
    -e REDIS_PORT=*** \
    -e REDIS_PASSWORD=*** \
    jumpserver/jms_all:latest

> 注意如下参数需自行设置：

* SECRET_KEY：上述步骤中生成的SECRET_KEY
* BOOTSTRAP_TOKEN：上述步骤中生成的BOOTSTRAP_TOKEN
* DB_HOST：数据库地址
* DB_PORT：数据库端口
* DB_USER：数据库用户名
* DB_PASSWORD：数据库密码
* DB_NAME：数据库名称
* REDIS_HOST：Redis地址
* REDIS_PORT：Redis端口
* REDIS_PASSWORD：Redis密码

当然，官网也声明了：不建议在生产中使用, 因为所有软件都打包到一个Docker中了, 不是Docker最佳实践

采用其他方法可以参考文档，地址：http://docs.jumpserver.org/zh/docs/step_by_step.html

这里还有一点，80端口和2222端口不要被占用了！！！

3.配置
如果安装没有问题的话，访问对应安装服务器ip地址如图所示。

![](/images/16a34ca23adeb71e.jpg)

ps:默认用户名密码都是admin，可以自行修改

登录后，如图所示，个人还是很喜欢这个设计风格的。

![](/images/16a34cb083e73004.jpg)

首先在进入资产管理-管理用户，如图。

![](/images/16a34cc33e17f5fc.jpg)

介绍一下，这个管理用的意义个人理解就是访问你服务器的账号，点击创建管理用户按钮，进入如图页面

![](/images/16a34ccef37260fb.jpg)

填写如下内容：

* 名称:这个自行设置。
* 用户名：访问服务器的用户名，比如你远程服务器的用户名root
* 密码：访问服务器的密码
* 私钥：访问服务器的私钥文件
* 备注：备注信息。
* 都填写完成后保存即可。

接下来需要创建系统用户，何为系统用户呢，官方文档给出了如下解释，

> 系统用户是 Jumpserver跳转登录资产时使用的用户，可以理解为登录资产用户，如 web, sa, dba(ssh web@some-host), 而不是使用某个用户的用户名跳转登录服务器(ssh xiaoming@some-host); 简单来说是 用户使用自己的用户名登录Jumpserver, Jumpserver使用系统用户登录资产。 系统用户创建时，如果选择了自动推送 Jumpserver会使用ansible自动推送系统用户到资产中，如果资产(交换机、windows)不支持ansible, 请手动填写账号密码。目前还不支持Windows的自动推送

填写内容这里就不介绍了，没什么特别的，根据需要自行设置即可。

接下来我们回到资产管理-资产列表，就是所有的机器列表

点击创建资产，如图所示。

![](/images/16a34d2b47b740d0.jpg)

其中填写如下必填项:

* 主机名：自定义即可
* IP：服务器IP
* 协议：根据情况选择
* 端口：根据情况设置
* 系统平台：根据情况选择
* 管理用户：选择刚刚新建的管理用户
* 配置完成后保存即可。

在下图位置可以查看硬件信息，测试连接等等。

![](/images/16a34d578f50982f.jpg)

接下来进入权限管理-资产授权 ，如图

![](/images/16a34d6177d5c6c6.jpg)

点击创建授权规则按钮，进入如图页面。

![](/images/16a34d725a328c5e.jpg)

这里需要填写如下几项信息：

* 名称：自己设置
* 用户：设置权限用户
* 用户组：设置权限用户组，这两个很好理解，就是给谁授权，或者给哪个组的用户授权
* 资产：哪些服务器
* 节点：类似资产组的概念
* 系统用户：使用刚刚设置的系统用户
* 填写完成后点击提交即可。

SSH连接跳板机2222端口，用户名和密码就是界面配置的 可以用admin/admin

连接成功后按回车显示所有可以使用的主机，按数字就能跳转了