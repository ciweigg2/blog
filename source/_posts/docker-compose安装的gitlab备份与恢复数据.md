cover: http://ciwei2.cn-sh2.ufileos.com/10.jpg
title: docker-compose安装的gitlab备份与恢复数据
date: 2018-08-14 21:37:02
tags: [gitlab]
categories: [综合]
---
### docker安装的gitlab备份与恢复
<!--more-->

### 1、备份
```
#进入gitlab
docker exec -it gitlab_gitlab_1 bash
#进入备份目录
cd /var/opt/gitlab/backups
#备份gitlab
gitlab-rake gitlab:backup:create
#复制备份数据到宿主机
docker cp 5e06ad9e2498:/var/opt/gitlab/backups/1534251675_2018_08_14_11.1.4_gitlab_backup.tar /root/
```

### 2、恢复
```
#复制备份数据到容器
docker cp 1534251675_2018_08_14_11.1.4_gitlab_backup.tar 65cb60c28c67:/var/opt/gitlab/backups
#进入备份目录
cd /var/opt/gitlab/backups
#授权
chmod -R 755 1534251675_2018_08_14_11.1.4_gitlab_backup.tar
#恢复备份
gitlab-rake gitlab:backup:restore BACKUP=1534251675_2018_08_14_11.1.4
```

再次访问gitlab就能看到恢复的数据了，不需要重启