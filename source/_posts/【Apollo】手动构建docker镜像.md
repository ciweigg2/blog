title: 【Apollo】手动构建apollo的docker镜像
date: 2019-08-11 13:12:28
tags: [apollo,docker]
categories: [综合]
---
下载apollo源码：https://github.com/ctripcorp/apollo

进入源码根目录修改数据库地址

```
vi scripts/build.sh 
```

<!--more-->

执行构建

```
scripts/build.sh
```

复制打包后的文件到docker目录

```
cp apollo-adminservice/target/apollo-adminservice-1.5.0-SNAPSHOT-github.zip apollo-adminservice/src/main/docker/
cp apollo-configservice/target/apollo-configservice-1.5.0-SNAPSHOT-github.zip apollo-configservice/src/main/docker/
cp apollo-portal/target/apollo-portal-1.5.0-SNAPSHOT-github.zip apollo-portal/src/main/docker/
```

分别进入目录执行构建

```
cd apollo-adminservice/src/main/docker/
cd apollo-configservice/src/main/docker/
cd apollo-portal/src/main/docker/
```

构建命令 如果你需要上传dockerhub ciweigg需要替换你的dockerhub用户名的额 很简单的呀

```
docker build -t ciweigg/apollo-adminservice:1.5.0 .
docker build -t ciweigg/apollo-configservice:1.5.0 .
docker build -t ciweigg/apollo-portal:1.5.0 .
```

镜像需要挂载apollo-env.properties的文件(不使用镜像可以忽略)

```
local.meta=http://localhost:8080
dev.meta=http://localhost:8080
fat.meta=http://fill-in-fat-meta-server:8080
uat.meta=http://fill-in-uat-meta-server:8080
lpt.meta=${lpt_meta}
pro.meta=http://fill-in-pro-meta-server:8080
```

具体镜像怎么启动请参考另一篇文章：【Apollo】docker安装Apollo