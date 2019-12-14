cover: http://ciwei2.cn-sh2.ufileos.com/17.jpg
title: Docker安装Gitlab中文版本
date: 2018-08-14 16:39:44
tags: [gitlab]
categories: [综合]
---
中文镜像地址：https://hub.docker.com/r/twang2218/gitlab-ce-zh/
使用docker-compose安装Gitlab
<!--more-->
新建docker-compose.yml
```java
version: '2'
services:
    gitlab:
      image: 'twang2218/gitlab-ce-zh:11.1.4'
      restart: unless-stopped
      hostname: 'gitlab.example.com'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://gitlab.example.com'
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
          # 需要配置到 gitlab.rb 中的配置可以在这里配置，每个配置一行，注意缩进。
          # 比如下面的电子邮件的配置：
          # gitlab_rails['smtp_enable'] = true
          # gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
          # gitlab_rails['smtp_port'] = 465
          # gitlab_rails['smtp_user_name'] = "xxxx@xx.com"
          # gitlab_rails['smtp_password'] = "password"
          # gitlab_rails['smtp_authentication'] = "login"
          # gitlab_rails['smtp_enable_starttls_auto'] = true
          # gitlab_rails['smtp_tls'] = true
          # gitlab_rails['gitlab_email_from'] = 'xxxx@xx.com'
      ports:
        - '80:80'
        - '1443:443'
        - '122:22'
      volumes:
        - config:/etc/gitlab
        - data:/var/opt/gitlab
        - logs:/var/log/gitlab
volumes:
    config:
    data:
    logs:
```

如果你的服务器有域名，将上面的 gitlab.example.com 替换为实际域名或者ip地址
然后使用命令 docker-compose up -d 来启动，停止服务使用 docker-compose down
可以使用默认的 root 用户密码 5iveL!fe 登录

![](/images/gitlab2.png)