title: 【Yapi】部署yapi定制化版本
date: 2019-08-30 22:37:56
tags: [yapi,yapiideaupload]
categories: [综合]
---
### 介绍

yapi个性化开发版本：https://github.com/xian-crazy/yapi

<!--more-->

idea插件生成yapi文档(可以使用yapi定制化版本也可以使用官方yapi版本)：https://github.com/diwand/YapiIdeaUploadPlugin

### 安装node

```bash
curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
yum install -y nodejs
```

### 安装mongo

```bash
vi /etc/yum.repos.d/mongodb-org-4.2.repo

[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc

外网访问：

vi /etc/mongod.conf

bindIp: 172.0.0.1  改为 bindIp: 0.0.0.0

安装：
sudo yum install -y mongodb-org

启动：
systemctl start mongod.service

暂停：
systemctl stop mongod.service
```

### 安装yapi

```bash
mkdir crazy-yapi
cd crazy-yapi
git clone --depth=1  https://github.com/xian-crazy/yapi.git vendors
cp vendors/config_example.json ./config.json //复制完成后请修改相关配置（先在mongodb中创建好数据库和账户，根据实际值修改config.json ,默认mongodb不需要账户密码登录 数据库可能需要自己创建）

config.json配置：

{
  "port": "3000",
  "adminAccount": "admin@admin.com",
  "db": {
    "servername": "127.0.0.1",
    "DATABASE": "yapi",
    "port": 27017,
    "authSource": ""
  },
  "godtoken": "xxxxxxxxxxx",
  "mail": {
    "enable": true,
    "host": "smtp.163.com",
    "port": 465,
    "from": "***@163.com",
    "auth": {
      "user": "***@163.com",
      "pass": "*****"
    }
  }
}

cd vendors
npm install --registry https://registry.npm.taobao.org
npm config set registry https://registry.npm.taobao.org
npm install ykit -g
ykit pack -m   //大概60秒左右 编译过程中 如果显示 [Bundler] 1908/1912 build modules 不动了，按一下回车
npm run install-server //安装程序会初始化数据库索引和管理员账号，管理员账号名可在 config.json 配置
```

### 使用pm2管理

```bash
npm install pm2 -g
cd crazy-yapi
pm2 start "vendors/server/app.js" --name yapi
pm2 info yapi
pm2 stop yapi
pm2 restart yapi
pm2 logs -f yapi
```

### 升级yapi

```bash
分支升级说明
停止服务：pm2 stop yapi
cd xxx/yapi/vendors/
添加 分支仓库（若已经添加，无需重复添加） git remote add yehaoapi https://github.com/xian-crazy/yapi.git
拉取新代码 git pull yehaoapi master
打包 ykit pack -m
```

### 安装插件

```bash
在config.json 这层目录下运行
npm install -g yapi-cli --registry https://registry.npm.taobao.org
yapi plugin --name yapi-plugin-import-swagger-customize(地址：https://github.com/follow-my-heart/yapi-plugin-import-swagger-customize)
yapi plugin --name yapi-plugin-auto-test(地址：https://github.com/congqiu/yapi-plugin-auto-test)
yapi plugin --name yapi-plugin-api-doc(地址：https://github.com/congqiu/yapi-plugin-api-doc)
```

### 开启谷歌跨域

请参考教程开启chrome 跨域请求：http://crazy-yapi.camdy.cn/doc/documents/chromeCORS.html

Windows

1. 关闭所有的chrome浏览器
2. 新建一个chrome快捷方式，右键"属性"，"快捷方式"选项卡里选择"目标"，往后追加 --args --disable-web-security --user-data-dir=C:\MyChromeDevUserData (例子："C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --args --disable-web-security --user-data-dir=C:\MyChromeDevUserData)
3. 通过快捷方式打开谷歌浏览器(测试地址要通的不然还是会报错让你开启跨域的)

访问http://ip:3000

账号：admin@admin.com

密码：ymfe.org

### 总结

定制化版本多了许多功能 结合yapiideaupload插件可以更好的开发 我觉得这个是最好的文档了