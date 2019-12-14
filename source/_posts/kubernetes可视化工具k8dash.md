title: kubernetes可视化工具k8dash
date: 2019-08-24 21:50:48
tags: [kubernetes,k8dash]
categories: [综合]
---
参考：https://github.com/herbrandson/k8dash

修改kubernetes-k8dash.yml端口类型为nodeport

<!--more-->

```
kubectl apply -f https://raw.githubusercontent.com/herbrandson/k8dash/master/kubernetes-k8dash.yaml
```

创建token

```
# Create the service account in the current namespace (we assume default)
kubectl create serviceaccount k8dash-sa

# Give that service account root on the cluster
kubectl create clusterrolebinding k8dash-sa --clusterrole=cluster-admin --serviceaccount=default:k8dash-sa

# Find the secret that was created to hold the token for the SA
kubectl get secrets

# Show the contents of the secret to extract the token
kubectl describe secret k8dash-sa-token-xxxxx
```

![](/images/20190824215417.png)