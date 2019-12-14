cover: http://ciwei2.cn-sh2.ufileos.com/21.jpg
title: Docker安装Sentry日志框架
date: 2018-07-25 22:07:55
tags: [sentry]
categories: [日志框架]
---
Sentry 是一款基于 Django实现的错误日志收集和聚合的平台，它是 Python 实现的，但是其日志监控功能却不局限于python，对诸如 Node.js, php,ruby, C#,java 等语言的项目都可以做到无缝集成，甚至可以用来对iOS, Android 移动客户端以及 Web前端异常进行跟踪。我们可以在程序中捕获异常，并发送到 Sentry服务端进行聚合统计、展示和报警。
<!--more-->
```java
docker拉取redis postsql 和sentry 
docker pull redis 
docker pull postgres 
docker pull sentry

启动redis和sentry 
docker run -d --name sentry-redis redis 
docker run -d --name sentry-postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=sentry postgres 
docker run --rm sentry config generate-secret-key

启动sentry(上一行得到secret-key，然后把key复制到下面四行的单引号中) 
docker run -it --rm -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade（这一步会提示输入邮箱和密码） 
docker run -d -p 9000:9000 --name my-sentry -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-redis:redis --link sentry-postgres:postgres sentry 
docker run -d --name sentry-cron -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run cron 
docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run worker
```