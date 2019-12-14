cover: http://ciwei2.cn-sh2.ufileos.com/76.jpg
title: kubernetes根据nginx ingress实现负载均衡
date: 2019-06-06 22:50:33
tags: [nginx,ingress]
categories: [综合]
---
### 部署nginx-ingress-controller

Pod的IP以及service IP只能在集群内访问，如果想在集群外访问kubernetes提供的服务，可以使用nodeport、proxy、loadbalacer以及ingress等方式，由于service的IP集群外不能访问，就是使用ingress方式再代理一次，即ingress代理service，service代理pod

<!--more-->

介绍一下原理呀：

![](/images/20190106214440123.png)

### 下载nginx-ingress-controller配置文件

```java
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml
```

### 修改镜像路径

简单的说就是把镜像改成如下的

```java
#image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
image: willdockerhub/nginx-ingress-controller:0.21.0
```

```java
sed -i 's#quay.io/kubernetes-ingress-controller/nginx-ingress-controller#willdockerhub/nginx-ingress-controller#g' mandatory.yaml
```

### 执行yaml文件部署

```java
kubectl apply -f mandatory.yaml 
```

### nodeport方式对外提供服务：

通过ingress-controller对外提供服务，现在还需要手动给ingress-controller建立一个servcie，接收集群外部流量。
service-nodeport配置文件：

```java
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/provider/baremetal/service-nodeport.yaml
```

### 执行yaml

```java
修改端口 添加type: NodePort nodePort: 32080  #http nodePort: 32443  #https 暴露给外面呀：
vi service-nodeport.yaml
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 32080  #http
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 32443  #https
      
kubectl apply -f service-nodeport.yaml
```

### 查看ingress-nginx组件状态

```java
[centos@k8s-master ~]$ kubectl get pod -n ingress-nginx 
NAME                                        READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-6bdcbbdfdc-wd2bn   1/1     Running   0          24s
[centos@k8s-master ~]$ kubectl get svc -n ingress-nginx 
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.104.138.113   <none>        80:30737/TCP,443:31952/TCP   13s
[centos@k8s-master ~]$ 
```

### 查看创建的ingress service暴露的端口：

```java
[centos@k8s-master ~]$ kubectl get svc -n ingress-nginx 
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.102.214.165   <none>        80:32080/TCP,443:32443/TCP   13m
```

### 创建ingress-nginx后端服务

这里自己有个demo打成镜像了 测试端口8080

```java
[centos@k8s-master ~]$ vim deploy-demon.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
  - name: http
    port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: demo:1.0
        ports:
        - name: httpd
          containerPort: 8080
```

### 创建相关服务及检查状态是否就绪

```java
[centos@k8s-master ~]$ kubectl apply -f deploy-demon.yaml 
service/myapp unchanged
deployment.apps/myapp-deploy configured
[centos@k8s-master ~]$
[centos@k8s-master ~]$ kubectl get pods                   
NAME                             READY   STATUS    RESTARTS   AGE
myapp-deploy-5cc79fc966-2228d    1/1     Running   0          62s
myapp-deploy-5cc79fc966-42w2d    1/1     Running   0          62s
```

### 创建myapp的ingress规则

因为demo:1.0的镜像默认是根路径 所以设置/ 否则都是404 如果springboot设置了路径那这边也可以需要设置对应路径

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
  annotations:
    kubernets.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false" ##关闭强制使用HTTPS的设置
spec:
  rules:
  ## 根据域名实现转发
  - host: myapp.magedu.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp
          servicePort: 8080
  - host: myapp.magedu2.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp2
          servicePort: 8080
```

下面再提供个根据url路径转发的例子，域名为 *

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
  annotations:
    kubernets.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false" ##关闭强制使用HTTPS的设置
spec:
  rules:
  ## 根据域名实现转发
  - http:
      paths:
      - path: /demo
        backend:
          serviceName: mydemo
          servicePort: 8080
  - http:
      paths:
      - path: /test
        backend:
          serviceName: mytest
          servicePort: 8080
```

### 查看创建的ingress规则

```java
[centos@k8s-master ~]$ kubectl apply -f ingress-myapp.yaml  
ingress.extensions/ingress-myapp created
[centos@k8s-master ~]$ kubectl get ingress
NAME            HOSTS              ADDRESS   PORTS   AGE
ingress-myapp   myapp.magedu.com             80      11s
[centos@k8s-master ~]$ 
```

### 配置集群外域名解析，当前测试环境我们使用windows hosts文件进行解析

```java
192.168.15.59 myapp.magedu.com
```

windows访问：http://myapp.magedu.com:30766/walle

```java
build walle success2 http://10.244.0.160:8080
```

想要部署多套项目 只需要创建deploy-demon.yaml 然后配置 ingress-myapp.yaml 规则 可以对应这2个根据项目创建新的文件来实现的

成功啦 负载均衡完美的实现了呀

更多例子参考：https://kubernetes.io/docs/concepts/services-networking/ingress/

### 再创建一个Service及后端Deployment(以tomcat为例)

```java
[root@k8s-master ingress-nginx]# cat tomcat-deploy.yaml 
apiVersion: v1
kind: Service
metadata:
  name: tomcat
  namespace: default
spec:
  selector:
    app: tomcat
    release: canary
  ports:
    port: 8009
    targetPort: 8009

---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: tomcat-deploy
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: tomcat
      release: canary
  template:
    metadata:
      labels:
        app: tomcat
        release: canary
    spec:
      containers:
      - name: tomcat
        image: tomcat:8
        ports:
        - name: httpd
          containerPort: 8080
        - name: ajp
          containerPort: 8009 
[root@k8s-master ingress-nginx]# kubectl apply -f tomcat-deploy.yaml 
service "tomcat" created
deployment.apps "tomcat-deploy" created
[root@k8s-master ingress-nginx]# kubectl get pod #等待pod状态就绪
```

### 将tomcat添加至ingress-nginx中

```java
[root@k8s-master ingress-nginx]# cat ingress-tomcat.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tomcat
  namespace: default
  annotations: 
    kubernets.io/ingress.class: "nginx"
spec:
  rules:
  - host: tomcat.magedu.com
    http:
      paths:
      - path: 
        backend:
          serviceName: tomcat
          servicePort: 8080
[root@k8s-master ingress-nginx]# kubectl apply -f ingress-tomcat.yaml 
ingress.extensions "ingress-tomcat" created
```

### 添加域名解析及访问服务

192.168.15.59 tomcat.magedu.com

http://tomcat.magedu.com:32080

### 下面我们对tomcat服务添加httpds服务

创建私有证书及secret

```java
[root@k8s-master ingress-nginx]# openssl genrsa -out tls.key 2048
Generating RSA private key, 2048 bit long modulus
.......+++
..............................+++
e is 65537 (0x10001)
[root@k8s-master ingress-nginx]# openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=tomcat.magedu.com #注意域名要和服务的域名一致 
[root@k8s-master ingress-nginx]# kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key #创建secret
secret "tomcat-ingress-secret" created
[root@k8s-master ingress-nginx]# kubectl get secret
NAME                    TYPE                                  DATA      AGE
default-token-bf52l     kubernetes.io/service-account-token   3         9d
tomcat-ingress-secret   kubernetes.io/tls                     2         7s
[root@k8s-master ingress-nginx]# kubectl describe secret tomcat-ingress-secret
Name:         tomcat-ingress-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1294 bytes  #base64加密
tls.key:  1679 bytes
```

### 将证书应用至tomcat服务中

```java
[root@k8s-master01 ingress]# cat ingress-tomcat-tls.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tomcat-tls
  namespace: default
  annotations: 
    kubernets.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - tomcat.magedu.com        #与secret证书的域名需要保持一致
    secretName: tomcat-ingress-secret   #secret证书的名称
  rules:
  - host: tomcat.magedu.com
    http:
      paths:
      - path: 
        backend:
          serviceName: tomcat
          servicePort: 8080
```

使用https访问服务就行啦

如果需要部署war应该用挂载的方式的

要部署war应该用挂载的方式的要部署war应该用挂载的方式的 要部署war应该用挂载的方式的 Dockerfile使用tomcat的时候就挂载webapps目录就行了

另一篇文章通过nfs挂载webapps目录实现的
