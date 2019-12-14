cover: http://ciwei2.cn-sh2.ufileos.com/69.jpg
title: kubernetes上用helm部署loki prometheus grafana
date: 2019-06-15 14:21:12
tags: [kubernetes,loki,prometheus,grafana]
categories: [综合]
---
### 提前在loki所有节点拉取所需要的镜像

> loki_k8s安装呀

<!--more-->

```java
docker pull grafana/loki:v0.1.0
docker pull grafana/promtail:v0.1.0
```

> 部署

```java
helm repo add loki https://grafana.github.io/loki/charts
helm repo update
helm upgrade --install loki loki/loki-stack --namespace monitoring
```

### prometheus_k8s

修改apiserver

使apiserver开放所端口1-65535，在配置文件中添加一行信息即可

vim /etc/kubernetes/manifests/kube-apiserver.yaml

- --service-node-port-range=1-65535

```java
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.22.45
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --service-node-port-range=1-65535
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.14.2
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.22.45
        path: /healthz
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-apiserver
    resources:
      requests:
        cpu: 250m
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
status: {}
```

### 获取helm安装prometheus使需要用的详细配置文件

下载所需要的

```java
git clone https://github.com/gjmzj/kubeasz/
cd kubeasz/manifests/prometheus/
```

提前在所有节点拉取prometheus所需要的镜像

```java
docker pull prom/node-exporter:v0.15.2
docker pull mirrorgooglecontainers/kube-state-metrics:v1.4.0
docker pull jimmidyson/configmap-reload:v0.2.2
docker pull prom/prometheus:v2.4.3
```

### helm安装promethheus

```java
helm install \
        --name monitor \
        --namespace monitoring \
        -f prom-settings.yaml \
        -f prom-alertsmanager.yaml \
        -f prom-alertrules.yaml \
        prometheus
```

### 验证

```java
##访问prometheus的web界面：
http://$NodeIP:39000

##访问alertmanager的web界面：
http://$NodeIP:39001
```

### 修改配置文件的persistence段落

注意：storageClassName的部署需要参考：k8s基于Storage Class和nfs动态供给(NFS+PV+PVC)

【想使用动态存储卷配置此项】

cd kubeasz/manifests/prometheus/

vim grafana/values.yaml

```java
persistence:
  enabled: ture
  storageClassName: "glusterfs-storage"
  accessModes:
    - ReadWriteOnce
  size: 10Gi
  annotations: {}
  subPath: ""
  existingClaim:
```

### 提前在所有节点拉取grafana所需镜像

```java
docker pull grafana/grafana:6.1.6

将grafana版本改成6.1.6

vi kubeasz/manifests/prometheus/grafana/values.yaml

从git拉取的默认配置为grafana/grafana:5.2.4，此版本过旧不能展示loki,所以选择pull镜像6.1.6版本

所以需要去修改kubeasz/manifests/prometheus/grafana/values.yaml中的tag和kubeasz/manifests/prometheus/grafana/Chart.yaml中的appVersion，使其为6.1.6
```

### 使用helm安装grafana

```java
helm install \
  --name grafana \
  --namespace monitoring \
  -f grafana-settings.yaml \
  grafana
```

### 安装成功的NOTES提示

```java
NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   grafana.monitoring.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:
export NODE_PORT=$(kubectl get --namespace monitoring -o jsonpath="{.spec.ports[0].nodePort}" services grafana)
     export NODE_IP=$(kubectl get nodes --namespace monitoring -o jsonpath="{.items[0].status.addresses[0].address}")
     echo http://$NODE_IP:$NODE_PORT


3. Login with the password from step 1 and the username: admin
```

### 验证

```java
##访问grafana的web界面:
http://$NodeIP:39002
```

最后添加面板 监控prometheus 8856这个面板不错呀