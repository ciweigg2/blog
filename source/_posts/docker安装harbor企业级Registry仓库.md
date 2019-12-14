cover: http://ciwei2.cn-sh2.ufileos.com/18.jpg
title: docker安装harbor企业级Registry仓库
date: 2018-08-02 17:48:56
tags: [harbor]
categories: [综合]
---
Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，通过添加一些企业必需的功能特性，例如安全、标识和管理等，扩展了开源Docker Distribution。作为一个企业级私有Registry服务器，Harbor提供了更好的性能和安全。提升用户使用Registry构建和运行环境传输镜像的效率。Harbor支持安装在多个Registry节点的镜像资源复制，镜像全部保存在私有Registry中， 确保数据和知识产权在公司内部网络中管控。另外，Harbor也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等
<!--more-->

### 安装harbor
```java
下载harbor：
wget https://github.com/vmware/harbor/releases/download/0.5.0/harbor-offline-installer-0.5.0.tgz
#我下载的是offline离线包，这样在后续的部署及安装都会比较快，总共有300M左右的大小！
tar zxvf harbor-offline-installer-0.5.0.tgz
cd harbor/
#vim harbor.cfg
hostname = 192.168.6.113(主机ip或者域名)
#这里只是简单的测试，所以只编辑这一行，其他的默认不做修改；当然也可以根据你自己的实际情况做修改！
```

配置docker：
* 因为docker默认使用的是https连接，而harbor默认使用http连接，所以需要修改docker配置标志insecure registry不安全仓库的主机！
* vim /usr/lib/systemd/system/docker.service
* ExecStart=/usr/bin/dockerd --insecure-registry=192.168.6.113
* 只加上--insecure-registry这个参数即可。

#重启docker：
```java
#systemctl daemon-reload
#systemctl restart docker.service
```

#会拉取好几个镜像下来，及检查环境：

安装：
```java
#./instsll.sh
Note: docker version: 1.12.5
Note: docker-compose version: 1.9.0

[Step 0]: checking installation environment ...
....

[Step 1]: loading Harbor images ...
....

[Step 2]: preparing environment ...
....

[Step 3]: checking existing instance of Harbor ...
....

[Step 4]: starting Harbor ...
....

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.6.113. 
For more details, please visit https://github.com/vmware/harbor .
```

Harbor容器的stop与start：
进入Harbor目录执行如下命令即可：
```java
docker-compose stop/start 
```
到此便安装完成了，直接打开浏览器登陆即可：
默认用户密码是：admin/Harbor12345

访问 http://ip

### 推送镜像库例子：
先登录：docker login 192.168.1.132  输入harbor账号密码
接下来向Harbor推一个镜像：
1、首先在Harbor上创建一个项目”common”
2、查看本地的镜像：
```java
root@docker:~# docker images 
REPOSITORY TAG IMAGE ID CREATED SIZE 
centos latest 98d35105a391 2 weeks ago 192.5 MB 
ubuntu latest 0ef2e08ed3fa 4 weeks ago 130 MB
```
3、给”ubuntu”这个镜像打上tag： 
```java
docker tag ubuntu 192.168.1.132/common/ubuntu
```
4、推送至Harbor：
```java
root@docker:~# docker push 192.168.1.132/common/ubuntu 
The push refers to a repository [192.168.1.132/common/ubuntu] 
56827159aa8b: Pushed 
440e02c3dcde: Pushed 
29660d0e5bb2: Pushed 
85782553e37a: Pushed 
745f5be9952c: Pushed 
latest: digest: sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535 size: 1357
```
5.在Harbor上common项目下可以看到这个镜像

![paste image](http://oisa91ton.bkt.clouddn.com/1533203533588lt2bthuy.png?imageslim)