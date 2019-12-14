cover: http://ciwei2.cn-sh2.ufileos.com/49.jpg
title: IDEA插件ApiDoc自动生成Api文档
date: 2019-06-20 11:21:00
tags: [idea,apidoc]
categories: [综合]
---
参考：https://gitee.com/UnlimitedBladeWorks_123/apidoc-plugin-idea/blob/191/README.md

暂时不支持请求参数List<对象>的解析List<Model> 

但是可以对象中包含List信息Mode中包含的List<Model2>

<!--more-->

ApiDoc
---

## Install   
- Using IDE built-in system on Windowndnndows:
  - <kbd>File</kbd> > <kbd>Settings</kbd> > <kbd>Plugins</kbd> > <kbd>Browse repositories...</kbd> > <kbd>Search for "ApiDoc"</kbd> > <kbd>Install Plugin</kbd>
- Using IDE built-in plugin system on MacOs:
  - <kbd>Preferences</kbd> > <kbd>Settings</kbd> > <kbd>Plugins</kbd> > <kbd>Browse repositories...</kbd> > <kbd>Search for "ApiDoc"</kbd> > <kbd>Insttll Plugin</kbd>
- Manually:
  - From official jetbrains store Download the `latest release` and install it manually using <kbd>Preferences</kbd> > <kbd>Plugins</kbd> > <kbd>Install plugin from disk...</kbd>

## Usage
### Use IDE menu

### Use hotkey
Default **Option + Ctrl + Shift + p**(Mac), **Alt + Ctrl + Shift + p** (win)

### Examples
* operation steps

![](/images/usage.gif)

* use npm command `apidoc`, to generate html

![](/images/20190620121321.png)

### 生成apidoc文档

项目根目录添加 apidoc.json

项目根目录添加 footer.md (编写结尾部分内容)

项目根目录添加 header.md (编写头部统一请求等内容呀)

```java
{
    "name": "apidoc-example",
    "version": "1.0.0",
    "description": "apiDoc example project",
    "title": "Custom apiDoc browser title",
    "url": "https://api.ciwei.com/v1",
    "header": {
        "title": "My own header title",
        "filename": "header.md"
    },
    "footer": {
        "title": "My own footer title",
        "filename": "footer.md"
    },
    "order": [
        "GetUser",
        "PostUser"
    ],
    "template": {
        "withCompare": true,
        "withGenerator": true
    }
}
```

* name：项目名称 
* version：项目版本 
* description：项目介绍 
* title：浏览器显示的标题内容 
* url：endpoints的前缀，例如https://api.github.com/v1 
* sampleUrl：如果设置了，则在api文档中出现一个测试用的from表单 
* header 
* title：导航文字包含header.md文件 
* filename：markdown-file 文件名 
* footer 
* title：导航文字包含header.md文件 
* filename：markdown-file 文件名 
* order：用于配置输出 api-names/group-names 排序，在列表中的将按照列表中的顺序排序，不在列表中的名称将自动显示

生成apidoc文档：

结构：项目目录 /project 项目根目录/project/apidoc.json

```java
apidoc -i ./项目目录  -o ./生成的目录
```

### apigroup支持中文

```java
vi C:\Users\Ciwei\AppData\Roaming\npm\node_modules\apidoc\node_modules\apidoc-core\lib\workers\api_group.js
// group = group.replace(/[^\w]/g, '_');
```

![](/images/20190620120440.png)