---
title: LetsEncrypt 生成免费https证书
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - https
categories:
  - 综合
date: 2019-12-22 16:55:59
---

最近看到网上说 https 的网站 Google 会优先收录，所以就抽时间记录下配置博客的过程

<!--more-->

### ACME

使用 LetEncrypt 证书作为博客的 https 实现方式。

acme.sh 实现了 acme 协议, 可以从 letsencrypt 生成免费的证书.

github https://github.com/Neilpang/acme.sh

主要步骤:

首先需要有个域名才行

1、安装 acme.sh
2、生成证书
3、copy 证书到 nginx/apache 或者其他服务
4、更新证书
5、更新 acme.sh
6、测试 https

### 安装 acme.sh

安装很简单, 一个命令:

```bash
curl  https://get.acme.sh | sh
```

普通用户和 root 用户都可以安装使用. 安装过程进行了以下几步:

把 acme.sh 安装到你的 home 目录下:

```bash
cd ~/.acme.sh/
```

并创建 一个 bash 的 alias, 方便你的使用: aliasacme.sh=~/.acme.sh/acme.sh

2). 自动为你创建 cronjob, 每天 0:00 点自动检测所有的证书, 如果快过期了, 需要更新, 则会自动更新证书.

2. 生成证书 并且完成验证

acme.sh 实现了 acme 协议支持的所有验证协议. 一般有两种方式验证: http 和 dns 验证.

这是使用 http 验证

```bash
cd ~/.acme.sh/
yum install socat
```

如果你还没有运行任何 web 服务, 80 端口是空闲的, 那么 acme.sh 还能假装自己是一个 webserver, 临时听在 80 端口, 完成验证:

```bash
sh acme.sh  --issue -d www.ciwei.com   --standalone
```

### copy/安装 证书

前面证书生成以后, 接下来需要把证书 copy 到真正需要用它的地方.

注意, 默认生成的证书都放在安装目录下: ~/.acme.sh/, 请不要直接使用此目录下的文件, 例如: 不要直接让 nginx/apache 的配置文件使用这下面的文件. 这里面的文件都是内部使用, 而且目录结构可能会变化.

```bash
mkdir -p /certs
cd /root/.acme.sh/www.ciwei.com
cp www.ciwei.com.cer /certs
cp www.ciwei.com.key /certs
```

配置Nginx

```bash
vim /usr/local/nginx/conf/nginx.conf
```

配置证书

```bash
server 
{
    listen 443;
    ssl on;
    ssl_certificate  /certs/www.ciwei.com.cer;
    ssl_certificate_key  /certs/www.ciwei.com.key;
}
```

把 http重定向到 https

```bash
server 
{
    listen       80;
    server_name www.ciwei.com;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```

每次修改nginx配置文件后都要进行检查

```bash
/usr/local/nginx/sbin/nginx -t
```

### 更新证书

目前证书在 60 天以后会自动更新, 你无需任何操作. 今后有可能会缩短这个时间, 不过都是自动的, 你不用关心.

### 更新 acme.sh

目前由于 acme 协议和 letsencrypt CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步.

升级 acme.sh 到最新版 :

```bash
acme.sh --upgrade
```

如果你不想手动升级, 可以开启自动升级:

```bash
acme.sh  --upgrade  --auto-upgrade
```

之后, acme.sh 就会自动保持更新了

你也可以随时关闭自动更新:

```bash
acme.sh --upgrade  --auto-upgrade  0
```

### 测试 https

启动Nginx

```bash
/usr/local/nginx/sbin/nginx
```

浏览器 访问 www.ciwei.com 会自动跳转到 https://www.ciwei.com

最后说一点，由于博客使用了七牛云的 http 协议的 cdn 导致博客内的静态资源不可用,最后又把七牛云的静态资源配置了 https

配置步骤

点击菜单：融合CDN-->域名管理-->配置--> HTTPS 配置

点击 SSL证书管理 - https://portal.qiniu.com/certificate/ssl#cert，您可以在SSL证书服务页面申请或上传自有证书。

内容 ( PEM格式 ) 对应证书内容 *.cre 后缀
私钥 ( PEM格式 ) 对应证书内容 *.key 后缀
最后点击：强制 HTTPS 访问

开启后用户的 HTTP 请求会强制跳转到 HTTPS 协议进行访问