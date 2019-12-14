title: 使用pm2管理sh脚本
date: 2019-09-08 16:15:25
tags: [pm2,sh]
categories: [综合]
---
sh脚本启动一般要nohup java -jar test.jar &

<!--more-->

使用pm2管理可以比较方便

安装pm2

```
npm install pm2 -g
pm2 start <sh文件路径>.sh --interpreter bash --name seata
pm2 start seata
查看日志^ ^
pm2 logs -f seata
```