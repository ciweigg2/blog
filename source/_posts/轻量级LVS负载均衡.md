title: 轻量级LVS负载均衡
date: 2019-08-17 21:24:13
tags: [lvs]
categories: [综合]
---
### 介绍

一个轻量级的负载均衡LVS 支持健康检查 虚拟ip控制

参考：https://github.com/fanux/LVScare

### 安装

下载bin

```
wget https://github.com/fanux/LVScare/releases/download/v1.0.6/lvscare
chmod -R 755 lvscare
mv lvscarc /usr/bin/
```

### 使用

环境|ip | 端口| 执行命令
-|-|-|
lvs| 10.103.97.12(虚拟ip)| 6443|enjoy it;
nginx1| 192.168.0.3| 80| docker run -p 80:80 --name nginx1 -d nginx
nginx2| 192.168.0.4| 80| docker run -p 80:80 --name nginx2 -d nginx
nginx3| 192.168.0.5| 80| docker run -p 80:80 --name nginx3 -d nginx

lvs机器的ip是192.168.0.13 虚拟ip是10.103.97.12(可以随便指定)

在lvs执行安装集群

```
nohup lvscare care --vs 10.103.97.12:6443 --rs 192.168.0.3:80 --rs 192.168.0.4:80 --rs 192.168.0.5:80 \
--health-path / --health-schem http &
```

### 测试

首先检查规则

```
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.103.97.12:6443 rr
  -> 192.168.0.3:80               Masq    1      0          0         
  -> 192.168.0.4:80               Masq    1      0          0         
  -> 192.168.0.5:80               Masq    1      0          0 
```

为了方便看清楚负载均衡的效果 在三台机器修改docker中nginx的index.html页面 分别对应ip

进入nginx容器

```
docker exec -it nginx1 /bin/bash
```

进入nginx目录

```
cd /usr/share/nginx/html/
```

修改index.html

```
cat << EOF > index.html
	  <!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx! 192.168.0.3</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
EOF
```

在lvs机器上安装nginx 把虚拟ip挂载出去 然后访问lvs的外网ip地址测试 发现每次都访问不同主机 停止nginx规则会少了一条 访问页面也会少一台机器 直到全部nginx关闭后 页面502 然后再分别启动nginx 发现规则恢复了 说明lvs成功了 为什么lvs中的虚拟ip也是一台物理机器呢 因为总要有个入口吧 还有就是lvs机器上不能参与规则的配置

lvs上的nginx配置

```
[root@JD ~]# cat /etc/nginx/conf.d/app.conf 
upstream app.com { 
#ip_hash; #如果需要用session保持一致需要用ip_hash
      server  10.103.97.12:6443; #负载均衡服务器，随机请求（server1）
} 
  
server{ 
    listen 80; 
    server_name _; #我自己注册的域名 没有域名的话用_代替。
    access_log  /var/log/nginx/melog.log  main;#自定义日志配置路径

    location / { 
        proxy_pass         http://app.com; #代理地址 代理upstream写的服务器地址
        proxy_set_header   Host             $host; 
        proxy_set_header   X-Real-IP        $remote_addr; 
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
    } 
}
```

nohup启动的程序如何关闭呢

```
查看pid
jobs -l
kill -9 pid
清理lvs规则
ipvsadm -C
查看规则
ipvsadm -Ln
```

访问：http://lvs外网ip

### 单台机器部署

当然也可以单台机器部署 但是我觉得毫无意义 这边也介绍下吧

```
单台机器必须执行
ip link add sealyun-ipvs0 type dummy
ip  addr add 10.103.97.12/32 dev sealyun-ipvs0
```

安装nginx然后启动lvs

```
docker run -p 8081:80 --name nginx1 -d nginx
docker run -p 8082:80 --name nginx2 -d nginx
docker run -p 8083:80 --name nginx3 -d nginx
nohup lvscare care --vs 10.103.97.12:6443 --rs 127.0.0.1:8081 --rs 127.0.0.1:8082 --rs 127.0.0.1:8083 \
--health-path / --health-schem http &
curl 10.103.97.12:6443 
```

清理规则

```
ip link del dev sealyun-ipvs0
ipvsadm -C
```