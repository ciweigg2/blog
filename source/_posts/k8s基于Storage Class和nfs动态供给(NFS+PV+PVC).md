title: k8s基于Storage Class和nfs动态供给(NFS+PV+PVC)
date: 2019-06-18 14:55:17
tags: [nfs,kubernetes,pv,pvc,storageclass]
categories: [综合]
---
### 介绍

Kubernetes集群管理员通过提供不同的存储类，可以满足用户不同的服务质量级别、备份策略和任意策略要求的存储需求。动态存储卷供应使用StorageClass进行实现，其允许存储卷按需被创建。如果没有动态存储供应，Kubernetes集群的管理员将不得不通过手工的方式类创建新的存储卷。通过动态存储卷，Kubernetes将能够按照用户的需要，自动创建其需要的存储

<!--more-->

基于StorageClass的动态存储供应整体过程如下图所示：

![](/images/20190322105207287.png)

主要解决kubernetes的动态挂载(helm安装grafana动态挂载遇到问题了才研究这个的)

（1）集群管理员预先创建存储类（StorageClass）

（2）用户创建使用存储类的持久化存储声明(PVC：PersistentVolumeClaim)

（3）存储持久化声明通知系统，它需要一个持久化存储(PV: PersistentVolume)

（4）系统读取存储类的信息

（5）系统基于存储类的信息，在后台自动创建PVC需要的PV

（6）用户创建一个使用PVC的Pod

（7）Pod中的应用通过PVC进行数据的持久化

（8）而PVC使用PV进行数据的最终持久化处理

创建nfs请参考博客其他文章呀

### 创建RBAC授权provisioner

vi rbac.yaml

```java
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
 
---
 
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
 
---
 
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```

### 创建nfs-client

10.2.68.77是nfs服务的监听地址，/nfs-share是nfs共享的目录

这个镜像中volume的mountPath默认为/persistentvolumes，不能修改，否则运行时会报错

vi deployment-nfs.yaml

```java
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 2
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
    #  imagePullSecrets:
    #    - name: registry-pull-secret
      serviceAccount: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          #image: quay.io/external_storage/nfs-client-provisioner:latest
          image: lizhenliang/nfs-client-provisioner:v2.0.0
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              #这个值是定义storage里面的那个值
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 10.2.68.77
            - name: NFS_PATH
              value: /nfs-share
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.2.68.77
            path: /nfs-share
```

### 创建storageclass

vi storageclass-nfs.yaml

```java

apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs
reclaimPolicy: Retain
```

* provisioner: 该字段指定使用存储卷类型，不同的存储卷提供者类型这里要修改成对应的值
* reclaimPolicy：有两种策略：Delete、Retain。默认是Delet
* 用户删除PVC释放对PV的占用后，系统根据PV的"reclaim policy"决定对PV执行何种回收操作。 目前，"reclaim policy"有三种方式：Retained、Recycled、Deleted

Retained

```java
保护被PVC释放的PV及其上数据，并将PV状态改成"released"，不将被其它PVC绑定。集群管理员手动通过如下步骤释放存储资源
手动删除PV，但与其相关的后端存储资源如(AWS EBS, GCE PD, Azure Disk, or Cinder volume)仍然存在。
手动清空后端存储volume上的数据。
手动删除后端存储volume，或者重复使用后端volume，为其创建新的PV
```

Delete

```java
删除被PVC释放的PV及其后端存储volume。对于动态PV其"reclaim policy"继承自其"storage class"，
默认是Delete。集群管理员负责将"storage class"的"reclaim policy"设置成用户期望的形式，否则需要用
户手动为创建后的动态PV编辑"reclaim policy"
```

Recycle

```java
删除被PVC释放的PV及其后端存储volume。对于动态PV其"reclaim policy"继承自其"storage class"，
默认是Delete。集群管理员负责将"storage class"的"reclaim policy"设置成用户期望的形式，否则需要用
户手动为创建后的动态PV编辑"reclaim policy"
```

Recycle

```java
保留PV，但清空其上数据，已废弃
```

fuseim.pri/ifs为上面deployment上创建的PROVISIONER_NAME(可以更改，但是这两个地方必须统一)

### StatefulSet案例

vi nginx.yaml

```java
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /persistentvolumes #推荐用这个
          #mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "managed-nfs-storage"
      resources:
        requests:
          storage: 1Gi
```

如果配置 mountPath: /usr/share/nginx/html 

但是首先容器内的权限和/nfs-share/相同

设置index.html内容呀

for i in 0 1; do kubectl exec web-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done

宿主机赋权

```java
chmod -R 755 /nfs-share/
```

容器赋权

```java
for i in 0 1; do kubectl exec web-$i -- chmod 755 /usr/share/nginx/html; done
```

可以使用写入index.html文件内容 for i in 0 1; do kubectl exec web-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done

参考：https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

创建文件：

```java
kubectl apply -f rbac.yaml
kubectl apply -f deployment-nfs.yaml
kubectl apply -f storageclass-nfs.yaml
kubectl apply -f nginx.yaml
```

注意，这里的mountPath必须指定为/persistentvolumes，否则会出现表面上PV创建成功，其实在NFS系统中找不到的问题。

当发现两个Pod都正常运行，且在NFS系统中能够找到PV，说明试验成功。

进一步的，可以用kubectl exec -it 进入Pod，并创建一个文件，看看在PV中能否发现相同的文件已生成