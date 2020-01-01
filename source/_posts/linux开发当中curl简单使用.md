---
title: linux开发当中curl简单使用
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
  - linux
categories:
  - 综合
originContent: ''
date: 2020-01-01 18:50:18
---

> curl是linux上可以发送http请求的命令。当然Postman是一个很好的接口调用管理工具，但在验证一个linux服务器调用另外一个linux服务器API是否可用的场景下，非curl命令莫属

<!--more-->

### 打开一个网站

```bash
curl www.mynamecoder.com
```

###发送POST 请求

```bash
curl -d "userName=xiaoming&password=coder1024" http://www.mynamecoder.com/login
```

参数说明

-d：指定传入的参数

使用-d参数以后，HTTP 请求会自动加上请求头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法。

### 发送GET请求

```bash
curl -d "title=curl" http://www.mynamecoder.com/search
```

参数说明

-G：表示get请求，缺省为post请求

### 发送JSON格式的POST请求

```bash
$ curl -d '{"userName": "xiaoming", "password": "123456"}' -H 'Content-Type: application/json' https://www.mynamecoder.com/login
```

参数说明

-H：指明 HTTP 请求头

### 上传文件

```bash
curl -F 'file=@head.png' https://mynamecoder.com/upload
```

参数说明

上面命令会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。

-F参数可以指定 MIME 类型。

curl -F 'file=@photo.png;type=image/png' https://mynamecoder.com/upload
上面命令指定 MIME 类型为image/png，否则 curl 会把 MIME 类型默认设为application/octet-stream。

-F参数也可以指定文件名。

$ curl -F 'file=@photo.png;filename=me.png' https://mynamecoder.com/upload
上面命令中，原始文件名为photo.png，但是服务器接收到的文件名为me.png