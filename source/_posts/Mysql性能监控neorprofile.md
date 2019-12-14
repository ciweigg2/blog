cover: http://ciwei2.cn-sh2.ufileos.com/88.jpg
title: Mysql性能监控neorprofile
date: 2018-09-10 15:34:21
tags: [mysql,profilesql,neorprofile]
categories: [可视化工具]
---
使用介绍地址：http://www.profilesql.com/use/
能监控mysql的所有sql和连接池等一些信息
<!--more-->

代码需要修改配置连接:

3306修改成neorprofile的数据端口

```
db_jdbc_url=jdbc:mysql://127.0.0.1:4040/tests?useUnicode=true&characterEncoding=UTF-8&useSSL=false
```

![](/images/profilesql.png)

![](/images/profilesql2.png)