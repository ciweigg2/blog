cover: http://ciwei2.cn-sh2.ufileos.com/44.jpg
title: hexo主题matery更新
date: 2018-10-04 12:47:58
tags: [hexo主题]
categories: [综合]
---
### 修改社交链接
在主题文件的layout/_partial/social-link.ejs文件中，你可以找到social-link的内容，可以在其中添加你需要的链接地址，增加内容如：
```java
<a href="https://github.com/ciweigg2" class="tooltipped" target="_blank" data-tooltip="访问我的GitHub" data-position="top" data-delay="50">
    <i class="fa fa-github"></i>
</a>
<a href="#" class="tooltipped" target="_blank" data-tooltip="邮件联系我" data-position="top" data-delay="50">
    <i class="fa fa-envelope-open"></i>
</a>
<a href="#!" class="tooltipped" data-tooltip="QQ联系我:"#" data-position="top" data-delay="50">
    <i class="fa fa-qq"></i>
</a>
<% if (config.feed && config.feed.path == 'atom.xml') { %>
<a href="<%- url_for('/atom.xml') %>" class="tooltipped" target="_blank" data-tooltip="RSS 订阅" data-position="top" data-delay="50">
    <i class="fa fa-rss"></i>
</a>
<% } %>
```

<!--more-->

### 修改主题颜色
在主题文件的/source/css/matery.css文件中，搜索.bg-color来修改背景颜色：
```java
修改第一个背景颜色
body {
    background-color: #ffffff;
    margin: 0;
    color: #34495e;
}

修改第二个标题栏颜色 .bg-color
.bg-color {
    background-image: linear-gradient(to right, #304fbf 0%, #a0c3b2 100%);
}

注释掉下面2个颜色切换的
/*
@-webkit-keyframes rainbow {
   /* 动态切换背景颜色. */
}

@keyframes rainbow {
    /* 动态切换背景颜色. */
}
*/
```

### 修改文章背景图上不显示标题 移到背景图下面
```java
vi /root/blog/themes/hexo-theme-matery/layout/index.ejs
```

![](/images/20181108154646.png)

注释掉上面的 添加下面两行
```java
<span class="card-title"><%- post.title %></span>
<HR style= " FILTER: alpha (opacity = 100, finishopacity =0 , style= 3 )" width ="80%" color =#987 cb 9 SIZE=3>
```

### 修改banner图和文章特色图
```java
/layout/_partial/bg-cover.ejs
修改背景图
$('.bg-cover').css('background-image', 'url(http://ciwei2.cn-sh2.ufileos.com/' + new Date().getDay() + '.jpg)');
```

### 主题_config.yml中的gitalk配置

```java
gitalk:
  enable: true
  owner: ciweigg2
  repo: ciweigg2.github.io
  oauth:
    clientId: 4e9a7bfed30119302290
    clientSecret: 28f6acd9ebbeff567a56b52da427a6622f861607
  admin:
    - ciweigg2
```

### 修改文章随机图片和内容
```java
_config.yml

添加菜单呀
menu:
  首页:
    url: /
    icon: fa-home
  博客:
    url: /Home
    icon: fa-home
  前端:
    url: /GuideV2/FrontEndGuide
    icon: fa-archive
  生活:
    url: /garlly
    icon: fa-bicycle
  标签:
    url: /tags
    icon: fa-tags
  分类:
    url: /categories
    icon: fa-bookmark
  归档:
    url: /archives
    icon: fa-archive
  关于:
    url: /about
    icon: fa-user-circle-o    

修改内容呀
dream:
  enable: true
  text: 要用多少个晴天 交换多少张相片 还记得锁在抽屉里面的滴滴点点 小而温馨的空间 因为有你在身边
```
修改文章的随机图片
```java
featureImages: 
- http://ciwei2.cn-sh2.ufileos.com/0.jpg
- http://ciwei2.cn-sh2.ufileos.com/1.jpg
- http://ciwei2.cn-sh2.ufileos.com/3.jpg
- http://ciwei2.cn-sh2.ufileos.com/4.jpg
- http://ciwei2.cn-sh2.ufileos.com/5.jpg
- http://ciwei2.cn-sh2.ufileos.com/6.jpg
- http://ciwei2.cn-sh2.ufileos.com/7.jpg
- http://ciwei2.cn-sh2.ufileos.com/8.jpg
- http://ciwei2.cn-sh2.ufileos.com/9.jpg
- http://ciwei2.cn-sh2.ufileos.com/10.jpg
- http://ciwei2.cn-sh2.ufileos.com/11.jpg
- http://ciwei2.cn-sh2.ufileos.com/12.jpg
- http://ciwei2.cn-sh2.ufileos.com/13.jpg
- http://ciwei2.cn-sh2.ufileos.com/14.jpg
- http://ciwei2.cn-sh2.ufileos.com/15.jpg
- http://ciwei2.cn-sh2.ufileos.com/16.jpg
- http://ciwei2.cn-sh2.ufileos.com/17.jpg
- http://ciwei2.cn-sh2.ufileos.com/18.jpg
- http://ciwei2.cn-sh2.ufileos.com/19.jpg
- http://ciwei2.cn-sh2.ufileos.com/20.jpg
- http://ciwei2.cn-sh2.ufileos.com/21.jpg
- http://ciwei2.cn-sh2.ufileos.com/22.jpg
- http://ciwei2.cn-sh2.ufileos.com/23.jpg
- http://ciwei2.cn-sh2.ufileos.com/24.jpg
- http://ciwei2.cn-sh2.ufileos.com/25.jpg
- http://ciwei2.cn-sh2.ufileos.com/26.jpg
- http://ciwei2.cn-sh2.ufileos.com/27.jpg
- http://ciwei2.cn-sh2.ufileos.com/28.jpg
- http://ciwei2.cn-sh2.ufileos.com/29.jpg
- http://ciwei2.cn-sh2.ufileos.com/30.jpg
- http://ciwei2.cn-sh2.ufileos.com/31.jpg
- http://ciwei2.cn-sh2.ufileos.com/32.jpg
- http://ciwei2.cn-sh2.ufileos.com/33.jpg
- http://ciwei2.cn-sh2.ufileos.com/34.jpg
- http://ciwei2.cn-sh2.ufileos.com/35.jpg
- http://ciwei2.cn-sh2.ufileos.com/36.jpg
- http://ciwei2.cn-sh2.ufileos.com/37.jpg
- http://ciwei2.cn-sh2.ufileos.com/38.jpg
- http://ciwei2.cn-sh2.ufileos.com/39.jpg
- http://ciwei2.cn-sh2.ufileos.com/40.jpg
- http://ciwei2.cn-sh2.ufileos.com/41.jpg
- http://ciwei2.cn-sh2.ufileos.com/42.jpg
- http://ciwei2.cn-sh2.ufileos.com/45.jpg
- http://ciwei2.cn-sh2.ufileos.com/46.jpg
- http://ciwei2.cn-sh2.ufileos.com/47.jpg
- http://ciwei2.cn-sh2.ufileos.com/48.jpg
- http://ciwei2.cn-sh2.ufileos.com/49.jpg
- http://ciwei2.cn-sh2.ufileos.com/50.jpg
- http://ciwei2.cn-sh2.ufileos.com/51.jpg
- http://ciwei2.cn-sh2.ufileos.com/52.jpg
- http://ciwei2.cn-sh2.ufileos.com/53.jpg
- http://ciwei2.cn-sh2.ufileos.com/54.jpg
- http://ciwei2.cn-sh2.ufileos.com/55.jpg
- http://ciwei2.cn-sh2.ufileos.com/56.jpg
- http://ciwei2.cn-sh2.ufileos.com/57.jpg
- http://ciwei2.cn-sh2.ufileos.com/58.jpg
- http://ciwei2.cn-sh2.ufileos.com/59.jpg
- http://ciwei2.cn-sh2.ufileos.com/60.jpg
- http://ciwei2.cn-sh2.ufileos.com/61.jpg
- http://ciwei2.cn-sh2.ufileos.com/62.jpg
- http://ciwei2.cn-sh2.ufileos.com/63.jpg
- http://ciwei2.cn-sh2.ufileos.com/64.jpg
- http://ciwei2.cn-sh2.ufileos.com/65.jpg
- http://ciwei2.cn-sh2.ufileos.com/66.jpg
- http://ciwei2.cn-sh2.ufileos.com/67.jpg
- http://ciwei2.cn-sh2.ufileos.com/68.jpg
- http://ciwei2.cn-sh2.ufileos.com/69.jpg
- http://ciwei2.cn-sh2.ufileos.com/70.jpg
- http://ciwei2.cn-sh2.ufileos.com/71.jpg
- http://ciwei2.cn-sh2.ufileos.com/72.jpg
- http://ciwei2.cn-sh2.ufileos.com/73.jpg
- http://ciwei2.cn-sh2.ufileos.com/74.jpg
- http://ciwei2.cn-sh2.ufileos.com/75.jpg
- http://ciwei2.cn-sh2.ufileos.com/76.jpg
- http://ciwei2.cn-sh2.ufileos.com/77.jpg
- http://ciwei2.cn-sh2.ufileos.com/78.jpg
- http://ciwei2.cn-sh2.ufileos.com/79.jpg
- http://ciwei2.cn-sh2.ufileos.com/80.jpg
- http://ciwei2.cn-sh2.ufileos.com/81.jpg
- http://ciwei2.cn-sh2.ufileos.com/82.jpg
- http://ciwei2.cn-sh2.ufileos.com/83.jpg
- http://ciwei2.cn-sh2.ufileos.com/84.jpg
- http://ciwei2.cn-sh2.ufileos.com/85.jpg
- http://ciwei2.cn-sh2.ufileos.com/86.jpg
- http://ciwei2.cn-sh2.ufileos.com/87.jpg
- http://ciwei2.cn-sh2.ufileos.com/88.jpg
- http://ciwei2.cn-sh2.ufileos.com/89.jpg
- http://ciwei2.cn-sh2.ufileos.com/90.jpg
- http://ciwei2.cn-sh2.ufileos.com/91.jpg
- http://ciwei2.cn-sh2.ufileos.com/92.jpg
- http://ciwei2.cn-sh2.ufileos.com/93.jpg
- http://ciwei2.cn-sh2.ufileos.com/94.jpg
- http://ciwei2.cn-sh2.ufileos.com/95.jpg
- http://ciwei2.cn-sh2.ufileos.com/96.jpg
- http://ciwei2.cn-sh2.ufileos.com/97.jpg
- http://ciwei2.cn-sh2.ufileos.com/98.jpg
- http://ciwei2.cn-sh2.ufileos.com/99.jpg
- http://ciwei2.cn-sh2.ufileos.com/100.jpg
- http://ciwei2.cn-sh2.ufileos.com/101.jpg
- http://ciwei2.cn-sh2.ufileos.com/102.jpg
- http://ciwei2.cn-sh2.ufileos.com/103.jpg
- http://ciwei2.cn-sh2.ufileos.com/104.jpg
- http://ciwei2.cn-sh2.ufileos.com/105.jpg
- http://ciwei2.cn-sh2.ufileos.com/106.jpg
- http://ciwei2.cn-sh2.ufileos.com/107.jpg
- http://ciwei2.cn-sh2.ufileos.com/108.jpg
- http://ciwei2.cn-sh2.ufileos.com/109.jpg
- http://ciwei2.cn-sh2.ufileos.com/110.jpg
- http://ciwei2.cn-sh2.ufileos.com/111.jpg
- http://ciwei2.cn-sh2.ufileos.com/112.jpg
- http://ciwei2.cn-sh2.ufileos.com/113.jpg
- http://ciwei2.cn-sh2.ufileos.com/114.jpg
- http://ciwei2.cn-sh2.ufileos.com/115.jpg
- http://ciwei2.cn-sh2.ufileos.com/116.jpg
- http://ciwei2.cn-sh2.ufileos.com/117.jpg
- http://ciwei2.cn-sh2.ufileos.com/118.jpg
- http://ciwei2.cn-sh2.ufileos.com/119.jpg
- http://ciwei2.cn-sh2.ufileos.com/120.jpg
- http://ciwei2.cn-sh2.ufileos.com/121.jpg
- http://ciwei2.cn-sh2.ufileos.com/122.jpg
- http://ciwei2.cn-sh2.ufileos.com/123.jpg
- http://ciwei2.cn-sh2.ufileos.com/124.jpg
- http://ciwei2.cn-sh2.ufileos.com/125.jpg
- http://ciwei2.cn-sh2.ufileos.com/126.jpg
- http://ciwei2.cn-sh2.ufileos.com/127.jpg
- http://ciwei2.cn-sh2.ufileos.com/128.jpg
- http://ciwei2.cn-sh2.ufileos.com/129.jpg
- http://ciwei2.cn-sh2.ufileos.com/130.jpg
- http://ciwei2.cn-sh2.ufileos.com/131.jpg
- http://ciwei2.cn-sh2.ufileos.com/132.jpg
- http://ciwei2.cn-sh2.ufileos.com/133.jpg
- http://ciwei2.cn-sh2.ufileos.com/134.jpg
- http://ciwei2.cn-sh2.ufileos.com/135.jpg
- http://ciwei2.cn-sh2.ufileos.com/136.jpg
- http://ciwei2.cn-sh2.ufileos.com/137.jpg
- http://ciwei2.cn-sh2.ufileos.com/138.jpg
- http://ciwei2.cn-sh2.ufileos.com/139.jpg
- http://ciwei2.cn-sh2.ufileos.com/140.jpg
- http://ciwei2.cn-sh2.ufileos.com/140.jpg
- http://ciwei2.cn-sh2.ufileos.com/142.jpg
- http://ciwei2.cn-sh2.ufileos.com/143.jpg
- http://ciwei2.cn-sh2.ufileos.com/144.jpg
- http://ciwei2.cn-sh2.ufileos.com/145.jpg
- http://ciwei2.cn-sh2.ufileos.com/146.jpg
- http://ciwei2.cn-sh2.ufileos.com/147.jpg
- http://ciwei2.cn-sh2.ufileos.com/148.jpg
- http://ciwei2.cn-sh2.ufileos.com/149.jpg
- http://ciwei2.cn-sh2.ufileos.com/150.jpg
```