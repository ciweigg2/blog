title: SpringBoot 在IDEA中实现热部署
date: 2019-11-13 14:24:21
tags: [springboot,idea]
categories: [综合]
---
### 一、开启IDEA的自动编译（静态）

需要使用 Ctrl + Shift + F9 手动去编译

具体步骤：打开顶部工具栏 File -> Settings -> Default Settings -> Build -> Compiler 然后勾选 Build project automatically

<!--more-->

![](/images/8069210-135f80127f474608.png)

### 二、开启IDEA的自动编译（动态）

动态会自动更新不需要手动更新了 我建议使用静态 想编译的时候编译比较好呀

具体步骤：同时按住 Ctrl + Shift + Alt + / 然后进入Registry ，勾选自动编译并调整延时参数

* compiler.automake.allow.when.app.running -> 自动编译
* compile.document.save.trigger.delay -> 自动更新文件

PS：网上极少有人提到compile.document.save.trigger.delay 它主要是针对静态文件如JS CSS的更新，将延迟时间减少后，直接按F5刷新页面就能看到效果

![](/images/8069210-8a46a17cf996c87d.png)

### 三、开启IDEA的热部署策略（非常重要）

具体步骤：顶部菜单- >Edit Configurations->SpringBoot插件->目标项目->勾选热更新

![](/images/8069210-ea0039f62fe4efe9.png)