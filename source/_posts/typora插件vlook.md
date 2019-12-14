title: typora插件vlook
date: 2019-10-31 15:03:21
tags: [typora,vlook]
categories: [综合]
---
### 下载插件

- 访问官方主页下载最新发布版本：[https://github.com/MadMaxChow/VLOOK/releases](https://github.com/MadMaxChow/VLOOK/releases)
- 可基于`VLOOK\3-demo\VLOOK-Template 文档模板.md`来创建你自己的文档，`VLOOK\3-demo`目录下也有本文档的 Markdown 源文件

<!--more-->

### 应用主题

+ 将`released\theme`下所有CSS文件复制至 Typora 的主题目录（ Typora「偏好设置」中点击「外观 - 打开主题目录」定位到该目录）；
+ 进入 Typora 的偏好设置，启用 Markdown 语法扩展下的所有选项（如：公式、上标、下标、高亮、图表等）；
+ 重启 Typora ，点击菜单`主题`，选择以`vlook-*`形式命名的主题，即可启用对应的 VLOOK 主题样式

### 应用插件

+ 在 Typora 中将 Markdown 文件导出为`HTML`文件；

+ 打开文件`released\VLOOK-TOOLBOX 插件.txt`，全选所有内容，并复制；

+ 用纯文件编辑器，如：记事本、[Visual Studio Code](https://code.visualstudio.com/)，打开该导出的 HTML 文件；

+ 搜索「**<body **」，并将复制的内容粘贴到body标签的「**>**」之后：

```html
<body ...>
← ← ← ← ← 复制的「VLOOK-TOOLBOX 插件」内容粘贴于此
...
</body>
```

### 预览

打开就能看到效果了 真的还可以

![](/images/20191031150652.png)