cover: http://ciwei2.cn-sh2.ufileos.com/72.jpg
title: kubernetes存储nfs作为volume并部署tomcat上传到nginx ingress的例子
date: 2019-06-11 16:33:50
tags: [kubernetes,nfs]
categories: [综合]
---
### 介绍

kubernetes 挂载目录多节点共享问题 需要通过nfs解决

<!--more-->

举例：如果tomcat部署war需要挂载webapps目录的

### 安装nfs

使用一台服务器作为nfs服务器

```java
#通过yum目录安装nfs服务和rpcbind服务：
yum install nfs-util bind
yum install nfs-utils rpc-bind -y
yum install rpcbind -y

#查看状态
systemctl status rpcbind.service

#检查nfs服务是否正常安装 
rpcinfo -p localhost
```

#### 创建用户

```java
useradd -u nfs
mkdir -p /nfs-share
chmod a+w /nfs-share
```

#### 配置共享目录

```java
#在nfs服务器中为客户端配置共享目录，*所有地址都能访问nfs服务器
echo "/nfs-share *(rw,async,no_root_squash)" >> /etc/exports

#通过执行如下命令是配置生效：
exportfs -r
```

创建多个目录 最后需要重新执行 exportfs -r

```java
mkdir -p /nfs-share2
chmod a+w /nfs-share2
exportfs -r
```

#### 启动服务

```java
#由于必须先启动rpcbind服务，再启动nfs服务，这样才能让nfs服务在rpcbind服务上注册成功：
systemctl start rpcbind

#启动nfs服务： 
systemctl start nfs-server

#设置rpcbind和nfs-server开机启动： 
systemctl enable rpcbind
systemctl enable nfs-server
```

### kubernetes挂载目录的

nginx的/usr/share/nginx/html 目录到共享目录 /nfs-share

/nfs-share 中上传的文件会同步到 /usr/local/tomcat/webapps

需要修改nfs的外网ip地址呀 没有端口的

编辑vi tomcat-deployment.yaml

```java
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  template:
    metadata:
      labels:
        run: hello
    spec:
      containers:
       - name: hello
         image: tomcat:8 #确保node节点上有该镜像且可正常运行，注意是node节点机器上，不是master机器
         imagePullPolicy: IfNotPresent ##Always,IfNotPresent,Never
         ports:
         - name: http
           containerPort: 8080
         volumeMounts:
         - mountPath: /usr/local/tomcat/webapps
           readOnly: false
           name: tomcat-webapps
      volumes:
      - name: tomcat-webapps
        nfs:
          server: nfs的外网ip地址呀
          path: /nfs-share

---

apiVersion: v1
kind: Service
metadata:
  name: service-hello
  labels:
  name: service-hello
spec:
  type: NodePort      #这里代表是NodePort类型的,另外还有ingress,LoadBalancer
  ports:
  - port: 8080          #这里的端口和clusterIP(kubectl describe service service-hello中的IP的port)对应，即在集群中所有机器上curl 10.98.166.242:8080可访问发布的应用服务。
    targetPort: 8080  #端口一定要和container暴露出来的端口对应，nodejs暴露出来的端口是8080，所以这里也应是8080
    protocol: TCP
    nodePort: 31213   # 所有的节点都会开放此端口30000--32767，此端口供外部调用。
  selector:
    run: hello         #这里选择器一定要选择容器的标签，之前写name:kube-node是错的
```

创建

```java
kubectl create -f tomcat-deployment.yaml
```

删除

```java
kubectl delete -f tomcat-deployment.yaml
```

检查一下服务是否正在运行：

```java
kubectl get pods 
```

将war包上传到/nfs-share tomcat会自动加载新的war 不需要重启 实现了部署

访问 http://ip:31213


### 部署到nginx ingress

首先需要部署nginx ingress 其他文章中有教程的

然后部署本文上面的tomcat 将tomcat的服务部署到nginx ingress上面呀

虽然tomcat中访问的地址是 /test-walle/walle 但是下面还是需要配置/的转发规则

特别解释一下：nginx.ingress.kubernetes.io/rewrite-target: / #强制跳转到/根路径 如果项目中配置了- path: /test-walle 开启了nginx.ingress.kubernetes.io/rewrite-target: / 会无法访问带test-walle项目名称的项目，开启后必须配置- path: / 不开启的话配置对应项目名称就行了

vi ingress-tomcat.yaml

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tomcat
  namespace: default
  annotations:
    kubernets.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false" #关闭强制使用HTTPS的设置
    #nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.magedu.com
    http:
      paths:
      - path: /test-walle
        backend:
          serviceName: service-hello
          servicePort: 8080
```

发布服务

```java
kubectl apply -f ingress-tomcat.yaml
```

删除服务

```java
kubectl delete -f ingress-tomcat.yaml
```

查看nginx ingress部署的服务状态

kubectl describe ingress ingress-tomcat

```java
Name:             ingress-tomcat
Namespace:        default
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  myapp.magedu.com  
                    /test-walle   service-hello:8080 (10.244.0.43:8080,10.244.0.44:8080,10.244.0.45:8080)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"ingress.kubernetes.io/ssl-redirect":"false","kubernets.io/ingress.class":"nginx"},"name":"ingress-tomcat","namespace":"default"},"spec":{"rules":[{"host":"myapp.magedu.com","http":{"paths":[{"backend":{"serviceName":"service-hello","servicePort":8080},"path":"/test-walle"}]}}]}}

  kubernets.io/ingress.class:          nginx
  ingress.kubernetes.io/ssl-redirect:  false
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  11m   nginx-ingress-controller  Ingress default/ingress-tomcat
```

需要修改windows的hosts myapp.magedu.com指向nginx ingress服务器地址

192.168.0.13 myapp.magedu.com

访问：http://myapp.magedu.com:32080/test-walle/walle 就可以啦

发现每次访问ip都不一样 因为nginx ingress默认的规则是轮询的呀

怎么看nginx ingress 发布的服务呢

![](/images/20190611224836.png)