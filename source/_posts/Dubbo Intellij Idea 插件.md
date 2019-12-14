cover: http://ciwei2.cn-sh2.ufileos.com/29.jpg
title: Dubbo Intellij Idea 插件
date: 2018-10-20 17:37:17
tags: [dubbo,idea]
categories: [dubbo]
---
# Dubbo Intellij Idea 插件

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Jetbrains Plugins](https://img.shields.io/jetbrains/plugin/v/10946-a8translate.svg)][plugin]
[![Downloads](https://img.shields.io/jetbrains/plugin/d/10946.svg?style=flat-square)][plugin]

## 功能

- 快速构建一个dubbo+springboot的项目 包含ApiModule 和 providerModule
- 可选其他依赖 如mybatis pageHelper hikari 并且配置好 生成代码例子和testcase 可立即运行 

<!--more-->

## 目前支持的依赖
| 依赖名            |  开始支持的版本        |
|----------------------|-------------------  |
| MYBAITS   |   v1.0              |
| MYSQL       |   v1.0            |
|LOMBOK      |   v1.0             |
| REDIS    |   v1.0           |
| FASTJSON   |   v1.0            |
| RABBIT_MQ       |   v1.0          |
| COMMON_LANGS_3     |  v1.0          |
| SPRING_RETRY |   v1.0            |
| ZOOKEEPER      |   v1.0             |
|WEB_TOMCAT| v1.0 |
|PAGE_HELPER | v1.0 |
|HIKARI | v1.0 |


## 截图

- 第一步操作  
![](/images/start.png)

- 第二步操作  
![](/images/second.png)

- 第三步操作  
![](/images/third.png)

- 生成项目结构  
![](/images/projectStructure.png)

- 生成的主类  
![](/images/demoApplication.png)

- 生成的dubboProvider类  
![](/images/generateProvider.png)

 
- 生成的mybatis接口例子  
![](/images/mybatisExample.png)


- 生成的service例子 此处使用了pageHelper  
![](/images/generateService.png)

- 生成的testcase  
![](/images/generateTestCase.png)

## 安装 

插件支持Intellij 2018.1版本及以上版本

**使用 IDE 内置插件系统:**
- <kbd>Preferences(Settings)</kbd> > <kbd>Plugins</kbd> > <kbd>Browse repositories...</kbd> > <kbd>搜索并找到"Dubbo"</kbd> > <kbd>Install Plugin</kbd>


**直接下载**
- download[`lastest plugin zip`](http://ogyxv3y5w.bkt.clouddn.com/dubboPlugin-1.0.zip) -> <kbd>Preferences(Settings)</kbd> > <kbd>Plugins</kbd> > <kbd>Install plugin from disk...</kbd>


## 使用说明

- 如果使用了zookeeper 请搭建好zookeeper服务器
- 启动前 先修改生成项目 application.properties中的数据库连接配置 改成自己的数据库配置
- 如果要跑生成例子中的testcase 请将项目下的schema.sql 在数据库中执行下

[plugin]: https://plugins.jetbrains.com/plugin/10946





