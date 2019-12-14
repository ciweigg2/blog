title: Jenkins Docker pipeline配置gitlab jenkins公私钥
date: 2019-08-04 12:24:54
tags: [jenkins]
categories: [综合]
---
1 生成ssh：cmd中输入 ssh-keygen，生成id_rsa.pub公钥，id_rsa 私钥

<!--more-->

2 登入gitlab对应项目，设置=》版本库，填入标题，复制公钥，勾选推送，点击增加密钥，最后会显示在当前项目启用的部署密钥 下面。

![](/images/20190804122659.png)

3 Jenkins中配置访问账号（Global credentials)

到Credentials → System → Global credentials下面， 点击左侧边栏中的 “Add Credentials", 

然后按下图选择。key里面复制入私钥

![](/images/20190804122734.png)

这样变配置好了，jenkins里面就可以选用auto了