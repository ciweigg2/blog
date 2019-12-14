title: 安装docker和docker-compose
date: 2019-09-08 22:45:37
tags: [docker]
categories: [综合]
---
### 安装docker

安装依赖包

```
yum install -y yum-utils \
                  device-mapper-persistent-data \
                  lvm2
```

<!--more-->

设置stable repository

```
yum-config-manager \
  --add-repo \
  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

查看可用版本的docker-ce
yum list docker-ce --showduplicates | sort -r

例如：yum install docker-ce-18.03.0.ce 或者呀 yum install docker-ce-18.09.0

```
systemctl enable docker.service
systemctl start docker
```

### 升级docker-ce

直接安装指定版本的docker-ce，即可完成版本升级，但是要执行上面的操作

```
yum install docker-ce-18.03.0.ce
```

阿里云加速（自己的这是地址：https://cr.console.aliyun.com）

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dytr18st.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 安装docker-compose

安装docker-compose：

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

赋权：

```
sudo chmod +x /usr/local/bin/docker-compose
```

查看：

```
docker-compose --version
```