cover: http://ciwei2.cn-sh2.ufileos.com/64.jpg
title: Kubernetes Dashboard 及监控组件 Heapster 部署
date: 2018-09-09 16:59:33
tags: [kubernetes]
categories: [综合]
---
### UI 组件 - Dashboard 部署：
1、下载官方提供的 Dashboard 组件部署的 yaml 文件
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
<!--more-->

2、修改 yaml 文件中的镜像

![](/images/k8s.png)

> k8s.gcr.io 修改为 registry.cn-hangzhou.aliyuncs.com/google_containers，后续所有 yaml 文件中，只要涉及到 image 的，都需要做同样的修改，因为国内 k8s.gcr.io 这个地址被墙了。

3、修改 yaml 文件中的 Dashboard Service，暴露服务使外部能够访问

```java
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```

修改为

```java
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31111
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
```

4、创建能够访问 Dashboard 的用户

新建文件 account.yaml ，内容如下：

```java
# Create Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
# Create ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

5、启动 Dashboard和创建用户
进入刚才创建2个yaml的文件夹

```java
kubectl apply -f .
```

5.1、删除并停止Dashboard和用户
进入刚才创建2个yaml的文件夹

```
kubectl delete -f .
```

6、访问 Dashboard（谷歌浏览器无法访问，使用火狐可以，进去需要选择命名空间才有数据）

地址： https://:31111/
注意：必须是 https

![](/images/k8s登录界面.png)

7、获取登录 Dashboard 的令牌 （Token）

```java
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

输出如下

```java
Name:         admin-user-token-f6tct
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=81cb9047-7087-11e8-95da-00163e0c5bd1

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:  <超长字符串>
```

8、登录 Dashboard 面板如下

![](/images/k8s面板.png)

监控组件 - Heapster 部署
Heapster 用于计算并分析集群资源利用率、监控集群容器

1、下载官方提供的 Heapster 组件部署的 yaml 文件

```java
# 新建文件夹，用于存放 Heapster 部署所需的 yaml 文件
mkdir heapster
cd heapster

# 获取相关 yaml 文件
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
```

2、修改 yaml 中 image 的值

k8s.gcr.io 全部修改为 registry.cn-hangzhou.aliyuncs.com/google_containers

3、部署 Heapster

```java
kubectl create -f .
```

4、几分钟后，刷新 Dashboard 面板

![](/images/k8s监控cpu.png)

Dashboard 上多了 CPU 和 内存的信息。。。

### 可视化 - Gafana 面板：
1、修改 grafana.yaml 文件，暴露服务到外部

```java
# 前面省略，最后几行改为如下内容
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 31112
  selector:
    k8s-app: grafana
  type: NodePort
```

2、访问 Grafana

地址：http://:31112/
注意：此处是 http 不是 https

3、补充说明
此处 Grafana 服务部署时，没有指定用户登录信息，不建议暴露服务到外部，若需外部访问，建议修改 Deployment 增加用户访问的校验。

[参考](http://www.spring4all.com/article/1265)

### 备注：
修改后的kubernetes-dashboard.yaml：
 ```java
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.0
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31111
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
```
