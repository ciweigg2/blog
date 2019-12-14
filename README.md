# 博客采用matery主题

参考：https://github.com/L1cardo/hexo-theme-matery

安装本博客 直接clone下来 npm i安装依赖后hexo clean hexo g hexo s 就行啦

# 博客地址Ciwei

<p align="center"><a href="https://ciweigg2.github.io" target="_blank" rel="noopener noreferrer"><img width="100" src="https://ciwei3.cn-sh2.ufileos.com/233.jpg" alt="Blog logo"></a></p>

<p align="center">
  <a href="https://github.com/shw2018/hexo-blog-fly/network"><img src="https://img.shields.io/github/forks/ciweigg2/blog.svg" alt="GitHub forks"></a>
  <a href="https://github.com/shw2018/hexo-blog-fly/stargazers"><img src="https://img.shields.io/github/stars/ciweigg2/blog.svg" alt="GitHub stars"></a>
  <br>

# 添加代码块阴影效果

修改模块中的源码

```
vi /root/hexo-blog-ciwei/node_modules/prismjs/plugins/line-numbers/prism-line-numbers.css
```

添加box-shadow代码

```
pre[class*="language-"].line-numbers {
        position: relative;
        padding-left: 3.8em;
        counter-reset: linenumber;
        /* 添加阴影效果 */
        box-shadow:18px 18px 15px 0px rgba(0,0,0,.4);
}
```

# 使用客户端

我们使用hexo-client来发布文章

首先在电脑上安装hexo

```bash
npm install hexo-cli -g
```

下载博客备份

```bash
git clone https://github.com/ciweigg2/blog.git
```

安装博客

```bash
cd blog
npm i
```

配置和github关联的秘钥 生成个ssh 在github配置下就行啦

安装hexo-client

https://github.com/gaoyoubo/hexo-client/releases

然后配置博客目录也就是上面clone的目录

主要讲一下Front-matter

这个配置文件会读取刚才下载的博客目录blog/scaffolds/post.md

我的博客主题使用这个 每个主题根据主题配置修改 我的是matery主题呀

```bash
---
title: {{ title }}
date: {{ date }}
author: Ciwei
img: ""
coverImg: ""
top: false
cover: false
toc: true
mathjax: false
password: ""
summary: ""
tags:
categories:
---
```
