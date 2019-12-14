title: 内网穿透Serveo
date: 2019-09-17 17:44:47
tags: [内网穿透,serveo]
categories: [综合]
---
项目地址：https://serveo.net

<!--more-->

基本的知识呀

---

将一个本地应用的 8080 端口映射到公网中

```
# 如果要转发其它端口，只需替换端口为其它就可以了
$ ssh -R 80:localhost:8080 serveo.net
Hi there
Forwarding HTTP traffic from https://heryum.serveo.net
Press g to start a GUI session and ctrl-c to quit.
```

SSH 初次和一个新服务器建立连接时会有提示，直接选择 yes 即可

如果你不想使用随机域名，想指定一个固定的二级域名也是可以的

```
# 这里指定为 ywzm.serveo.net，可以根据自身情况进行替换
$ ssh -R ywzm:80:localhost:8080 serveo.net
Hi there
Forwarding HTTP traffic from https://ywzm.serveo.net
Press g to start a GUI session and ctrl-c to quit.
...

# 上面的域名是简写的，你也可以写出完整的域名。
$ ssh -R ywzm.serveo.net:80:localhost:8080 serveo.net
```

SSH 连接成功后，此时就可以在公网上使用 ywzm.serveo.net 访问到你本地计算机 8080 端口的服务了

上面的例子中，我们转发的是 HTTP 服务。如果你需要转发的是 TCP 服务，又应该怎么做呢？其实方法也很简单，同样只需设置公网的转发端口和本地端口就可以了。例如：我们需要将本地 3306 端口转发到公网中，使用下面命令即可

```

# 可以自行设置公网端口，这里设置为 1492
$ ssh -R 1492:localhost:3306 serveo.net

# 如果公网端口设置为 0，就会采用一个随机端口进行转发
$ ssh -R 0:localhost:3306 serveo.net
```

将本地 SSH 重定向到公网

在很多场景下，我们需要远程访问到本地计算机的 SSH 服务。要实现这样的需求也很简单，只需要使用下面的命令

```

# 名称为自定义的，这里设置为 myhost
$ ssh -R myhost:22:localhost:22 serveo.net
Hi there
Forwarding SSH traffic from alias "myhost"
Press g to start a GUI session and ctrl-c to quit.
...
```

连接成功后，接下来你就可以从公网上对这个内网计算机的 SSH 进行访问了

```

$ ssh -J serveo.net myuser@myhost
Hi there
myuser@myhost's password:
Last login: Mon Dec 24 21:00:32 2019 from 127.0.0.1
...
```

-J 选项是在 OpenSSH 7.3 版本才引入的，如果你使用的 SSH 客户端版本较旧，则可以使用 ProxyCommand 选项来替代

```
$ ssh -o ProxyCommand="ssh -W myhost:22 serveo.net" user@myhost
```

一些其它技巧

---

保持 SSH 连接不超时

众所周知，SSH 连接一旦超时就会自动断开，这样就很容易造成服务中断。这里我们只需给 SSH 连接增加一个保活参数 -o ServerAliveInterval=60 就可以了。

# 每隔 60 秒做一次连接保活

```
$ ssh -o ServerAliveInterval=60 -R 80:localhost:8080 serveo.net
```

对 SSH 连接进行守护

上面的方法虽然可以解决超时的问题，但进程始终是在前台运行的。为了彻底解决这个问题，官方推荐使用 AutoSSH 来进行进程守护。

AutoSSH 是一个用来对 SSH 连接进行监控的程序，可在遇到程序问题或者是网络问题时自动进行重连，以达到长期保持 SSH 稳定连接的目的 。

安装 AutoSSH

```
# Debian / Ubuntu 系统
$ apt install autossh -y

# CentOS / RHEL 系统
$ yum install autossh -y
```

将 AutoSSH 加入到系统服务

这里以加入到 Systemd 系统服务为例，此方法适用于 CentOS 7、Debian 8、Ubuntu 16 及以上系统版本。首先，我们创建一个 AutoSSH 的 Systemd 服务

```
$ cat > /etc/systemd/system/autossh.service <<EOF
[Unit]
Description=autossh
After=network.target

[Service]
Type=simple
Environment="AUTOSSH_GATETIME=0"
ExecStart=$(command -v autossh) -M 0 -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -R 80:localhost:8080 serveo.net
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF
```

AutoSSH 的 -M 参数主要用于指定一个监听端口来监视 SSH 连接状态，这里指定为 0 的主要目的是禁用 AutoSSH 的监控端口。保活依然使用 SSH 自己的 ServerAliveInterval 和 ServerAliveCountMax 选项来完成

其次，Systemd 系统服务创建完成后，我们启动这个 AutoSSH 的服务并设置为开机自启

```
$ systemctl start autossh
$ systemctl enable autossh
```

如果你无法通过 22 端口连接到 Serveo，官方还预留了 443 端口给你使用

```
$ ssh -p 443 -R 80:localhost:8080 serveo.net
```

使用自定义的域名 / 子域名

默认情况下，我们都是使用的 Serveo 生成的二级域名进行连接的。如果你想使用自己的域名也是可以的，方法非常简单。只需要在你的域名所在 DNS 中添加一条 A 记录和一条 TXT 记录就可实现。

4.1 添加一条 A 记录

```
A | serveo | 159.89.214.31
```

4.2 添加一条 TXT 记录

```
TXT | serveo | authkeyfp=SHA256:pmc7ZRv7ymCmghUwHoJWEm5ToSTd33ryeDeps5RnfRY
```

authkeyfp 后面跟的那一串字符是 RSA 密钥指纹，你可以使用 ssh-keygen -l 命令进行查看。

DNS 解析记录增加好后，你就可以使用自定义域名进行连接了

```
$ ssh -R serveo.ywzm.org:80:localhost:3000 serveo.net
```

至此，Serveo 的基本用法就介绍完了。如果你对它有更多的兴趣，欢迎去官网进行探索。

参考的资料呀

---
https://www.google.com

https://www.moerats.com/archives/990/

https://blog.rxliuli.com/p/5ad7fa84/

https://www.jianshu.com/p/d0b3991a9ce1

https://blog.csdn.net/kongxx/article/details/86178364

https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/