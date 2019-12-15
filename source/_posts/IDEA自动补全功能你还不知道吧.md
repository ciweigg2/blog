---
title: IDEA自动补全功能你还不知道吧
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
  - idea
categories:
  - 综合
date: 2019-12-15 22:06:43
---

IDEA 有个很牛逼的功能，那就是后缀补全（不是自动补全），很多人竟然不知道这个操作，还在手动敲代码

<!--more-->

这个功能可以使用代码补全来模板式地补全语句，如遍历循环语句（for、foreach）、使用 String.format() 包裹一个字符串、使用类型转化包裹一个表达式、根据判（非）空或者其它判别语句生成 if 语句、用 instanceOf 生成分支判断语句等

使用的方式也很简单，就是在一个表达式后按下点号 . ，然后输入一些提示或者在列表中选择一个候选项，常见的候选项下面会给出 GIF 演示

1. var 声明

![var.gif](/images/2019/12/15/6adb3710-1f44-11ea-907f-019ab0265256.gif)

2. null 判空

![null.gif](/images/2019/12/15/79f5cad0-1f44-11ea-907f-019ab0265256.gif)

3. notnull 判非空

![notnull.gif](/images/2019/12/15/81024bf0-1f44-11ea-907f-019ab0265256.gif)

4. nn 判非空

![nn.gif](/images/2019/12/15/862d3bd0-1f44-11ea-907f-019ab0265256.gif)

5. for 遍历

![for.gif](/images/2019/12/15/8ff07880-1f44-11ea-907f-019ab0265256.gif)

6. fori 带索引的遍历

![fori.gif](/images/2019/12/15/953001d0-1f44-11ea-907f-019ab0265256.gif)

7. not 取反

![not.gif](/images/2019/12/15/9a3d7ea0-1f44-11ea-907f-019ab0265256.gif)

8. if 条件判断

![if.gif](/images/2019/12/15/9e873640-1f44-11ea-907f-019ab0265256.gif)

9. cast 强转

![cast.gif](/images/2019/12/15/a1f078f0-1f44-11ea-907f-019ab0265256.gif)

10. return 返回值

![return.gif](/images/2019/12/15/a5d3f4b0-1f44-11ea-907f-019ab0265256.gif)