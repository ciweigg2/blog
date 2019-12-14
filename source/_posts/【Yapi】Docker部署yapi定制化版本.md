title: 【Yapi】Docker部署yapi定制化版本
date: 2019-12-01 21:21:33
tags: [yapi]
categories: [综合]
---
Dokcer安装yapi定制版配合idea插件完成文档生成的

<!--more-->

```bash
定制化yapi的源码地址：https://github.com/xian-crazy/yapi

git clone https://github.com/ciweigg2/yapi-docker.git

Docker中的2019.11.15.tar.gz替换成最新版本

项目文件中的2019.11.15.tar.gz也需要替换成最新的

构建版本镜像 20191115已经推送到dockerhub可以直接下载 可以省略构建

docker build -t ciwei123321/yapi:20191115 .

docker-compose up -d

如果mongodb中已经初始化过数据了 那么需要清理mongodb的数据 否则yapi起不来的(可以偷懒清理mongodb挂在到数据的./data/db下面的数据)

访问http://ip:3000

账号：admin@admin.com

密码：ymfe.org
```