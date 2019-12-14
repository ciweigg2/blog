title: 安装dapps应用商店
date: 2019-11-13 14:53:33
tags: [dapps]
categories: [应用商店]
---
## dapps是什么？

它是一个应用程序商店，包含丰富的软件，因为基于docker，使你本机电脑有云开发的效果。
一键安装程序；多版本共存，完善的使用说明，且不影响本机环境。
前端、服务端、运维、站长可以直接使用，效率提高非常多。普通用户亦可使用其中部分软件。

<!--more-->

## 目前包含的软件

- 注：每周会上架一款新应用，持续更新
- 百度网盘web版：下载速度比官方快很多 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/baidupcs-web/README.md)
- AriaNg高速下载器：2倍迅雷速度，迅雷无法下载的资源，也能下载 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/ariang/README.md)
- 百度网盘下载器(命令行) ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/baidupcs-go/README.md)
- wordpress ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/wordpress/README.md)
- py12306抢票 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/py12306/README.md)
- magnetw： 种子搜索神器 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/magnetw21/README.md)
- PhpMyAdmin：mysql管理工具 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/phpmyadmin/README.md)
- adminmongo: mongo管理工具 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/adminmongo/README.md)
- PHP： 世界上最好的语言（版本：5.6，7.1，7.2，7.3）------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/php-7-2-21/README.md)
- Mysql：数据库（版本：5.6，5.7，,8.0）------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/mysql57/README.md)
- Nginx：服务器（版本：1.16）------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/nginx116/README.md)
- redis：nosql数据库（版本：5.0）------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/redis5/README.md)
- mongo：是一个基于分布式文件存储的数据库（版本：3.4，4.0）------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/mongo-4-0-13/README.md)
- gogs版本控制 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/gogs/README.md)
- rabbitmq3.7队列服务 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/rabbitmq37/README.md)
- 2048游戏 ------ [查看效果](https://github.com/wallace5303/dapps-addons/blob/master/addons/game-2048/README.md)
- 等......

## 交流

如果你有什么软件需求，请加下面群或者留言，我们可以开发并发布到应用商店中

## 开始使用

```bash
# 下载
git clone https://gitee.com/wallace5303/dapps.git

# 安装
cd dapps
npm install --registry=https://registry.npm.taobao.org

# 启动
npm run start
```

注：# 请确保已经安装了nodejs，下载地址：http://nodejs.cn/download/

## 效果图
1. 访问： http://localhost:8000/

![](https://i.loli.net/2019/10/26/fLmqtePbAHFnksl.png)
![](https://i.loli.net/2019/10/26/jScTp4DCKRMfobk.png)
![](https://i.loli.net/2019/10/11/yWCI8TQReAMsdpB.png)

2. 一些软件效果：

百度网盘
![](https://i.loli.net/2019/11/06/MDPWZJcGaBpUznr.png)
ariang下载器
![](https://i.loli.net/2019/11/04/RoxOCNnWEdaHFLw.png)
数据库管理
![](https://i.loli.net/2019/10/08/Y2DGjzJ4opiFueM.png)

## 特性
>1.使用对象：**普通用户**，**前端**，**服务端**，**运维**

>2.支持**多版本共存**php，mysql, mongo, redis, 下载工具，管理软件，游戏等

>3.清晰的文件结构，可自定义软件

>4.商店应用持续更新

>5.操作持续优化，支持交互、无人值守安装

>6.支持系统版本：Linux、MacOs、Windows

## 软件升级（推荐时不时，升级一下^_^）

### git方式（推荐）

```bash
# 拉取新代码
cd dapps
git pull

# 重启
npm run restart
```

### 软件中升级

- 点击按钮，然后重启即可
![](https://i.loli.net/2019/10/24/b6O8aG2ZmLodFxN.png)

## 关联项目（应用程序项目）

- [github：dapps-addons](https://github.com/wallace5303/dapps-addons)
- [码云：dapps-addons](https://gitee.com/wallace5303/dapps-addons)