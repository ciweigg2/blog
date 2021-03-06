---
title: 基于Docker安装破解版Jira
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - jira
categories:
  - 综合
date: 2019-08-19 21:17:28
---

### 介绍

插件基本都收费 所以决定重建Jira软件系统，并采用Docker来实现破解版的Jira安装

参考：https://www.clxz.top/2019/05/30/114432/

<!--more-->

### 步骤

1、下载镜像

一定要安装jira-software(不是jira),否则没有agile

```
docker pull dchevell/jira-software:7.13.6
```

2、运行容器

```
docker run -d -it -p  8080:8080  --privileged  -m 4096M -v /data/jira-data:/var/atlassian/application-data/jira  -v /etc/localtime:/etc/localtime --name jira dchevell/jira-software:7.13.6
```

3、拷贝文件到容器内

```
# 下载3个文件(mysql-connector-java-5.1.25-bin.jar,atlassian-universal-plugin-manager-plugin-2.22.4.jar,atlassian-extras-3.2.jar)
git clone https://github.com/hlwojiv/some-software.git
cd some-software-master/Jira/
# 拷贝mysql-connector
docker cp mysql-connector-java-5.1.25-bin.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
# 进入容器修改mysql-connector的权限
docker exec -it jira bash
chmod 755 /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/mysql-connector-java-5.1.25-bin.jar
# 重启容器
docker restart jira
```

3、Web设置

浏览器访问JiraWeb，语言可以设为中文，选择「我将设置它自己」——「下一步」

数据库设置，数据库类型选择「MySQL」，接着填入你的MySQL连接信息（需要你在你的MySQL数据库中创建数据库，数据库的字符类型必须是utf8，暂时只支持5.7的mysql版本），测试可以连接之后点击「下一步」

设置应用程序的属性——「下一步」

申请许可证关键字，点击「生成Jira试用许可证」

需要注册账号，注册完之后重新回到这个页面，选择Jira Software (Server)，点击「Generate License」

点击「Yes」

页面就会带着你的许可证关键字回到Jira的设置页面，接着点击「下一步」

等待一会就进入设置管理员页面，填入一些信息即可，接着「下一步」

点击「完成」即完成设置

4、拷贝文件到容器内

```
# 拷贝atlassian-extras到容器内
docker cp atlassian-extras-3.2.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
# 进入容器设置atlassian-extras的权限
docker exec -it jira bash
chmod 755 /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-3.2.jar
# 重启容器
docker restart jira
# 拷贝插件到容器内
docker cp atlassian-universal-plugin-manager-plugin-2.22.4.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/
# 进入容器内修改插件的属性
docker exec -it jira bash
chmod 755 /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.4.jar
# 删除另一个插件
rm -rf /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.9.jar
```

破解结束，进入Jira下载任意插件，申请试用，自动破解

5、下面来下载一个收费软件试试

安装完了点击「获取许可证」

接着在「管理应用」中可以看到，该插件已经破解了

推荐几款不错的插件

gears-desk for jira

Mbile for JIRAa(web安装后ios下载app使用呀)

BigPicture 非常高级的任务管理