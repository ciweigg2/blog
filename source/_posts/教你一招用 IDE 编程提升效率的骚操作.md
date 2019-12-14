cover: http://ciwei2.cn-sh2.ufileos.com/4.jpg
title: 教你一招用 IDE 编程提升效率的骚操作
date: 2019-03-12 17:44:08
tags: [idea]
categories: [综合]
---
IDEA 有个很牛逼的功能，那就是后缀补全（不是自动补全），很多人竟然不知道这个操作，还在手动敲代码。

这个功能可以使用代码补全来模板式地补全语句，如遍历循环语句（for、foreach）、使用 String.format() 包裹一个字符串、使用类型转化包裹一个表达式、根据判（非）空或者其它判别语句生成 if 语句、用 instanceOf 生成分支判断语句等。

使用的方式也很简单，就是在一个表达式后按下点号 . ，然后输入一些提示或者在列表中选择一个候选项，常见的候选项下面会给出 GIF 演示

<!--more-->

1. var 声明

![](/images/640var.gif)

2. null 判空

![](/images/640null.gif)

3. notnull 判非空

![](/images/640notnull.gif)

4. nn 判非空

![](/images/640nn.gif)

5. for 遍历

![](/images/640for.gif)

6. fori 带索引的遍历

![](/images/640fori.gif)

7. not 取反

![](/images/640not.gif)

8. if 条件判断

![](/images/640if.gif)

9. cast 强转

![](/images/640cast.gif)

10. return 返回值

![](/images/640return.gif)

看完怎么样？赶紧用起来装逼呀