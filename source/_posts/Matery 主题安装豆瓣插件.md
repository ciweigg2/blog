title: Matery主题安装豆瓣插件
date: 2019-11-02 15:54:20
tags: [hexo,matery]
categories: [综合]
---
### 介绍

前几天发现一个很好用的插件，可以将自己在豆瓣上面的电影、书籍、游戏清单显示在 Hexo 搭建的博客页面上。由于 Matery 也是由 Hexo 驱动的，所以就想的安装来看看。

安装完，配置完，效果是惨不忍睹，可能由于没有适配，画面太美不能想象……

于是我就自己摸索，看能不能将这个主题适配到 Matery 主题上面，经过一个医学生一下午的坚持不懈，终于将插件进行了适配，准确的来说，是将 Matery 对插件进行了适配，因为插件是别人写的，我改不了😂😂

<!--more-->

最终的显示效果就像下面显示的一样，我觉得还是可以的：

### 安装

在博客所在文件夹下执行以下命令行：

```bash
npm install hexo-douban --save
```

### 配置

#### 配置博客 config 文件

将下面的配置写入博客的 _config.yml 文件里（⚠️ 注意：不是主题的配置文件）

```yml
douban:
  user: Licardo
  builtin: false
  book:
    title: 'This is my book title'
    quote: 'This is my book quote'
  movie:
    title: 'This is my movie title'
    quote: 'This is my movie quote'
  game:
    title: 'This is my game title'
    quote: 'This is my game quote'
  timeout: 10000
```

* user: 你的豆瓣ID。打开豆瓣，登入账户，然后在右上角点击 ”个人主页“，这时候地址栏的URL大概是这样：https://www.douban.com/people/xxxxxx/ ，其中的”xxxxxx”就是你的个人ID了。
* builtin: 是否将生成页面的功能嵌入 hexo s 和 hexo g 中，默认是 false ，另一可选项为 true 。
* title: 该页面的标题。
* quote: 写在页面开头的一段话,支持html语法。
* timeout: 爬取数据的超时时间，默认是 10000ms，如果在使用时发现报了超时的错(ETIMEOUT)可以把这个数据设置的大一点。

如果只想显示某一个页面(比如movie)，那就把其他的配置项注释掉即可。

#### 配置 config 文件

在你的主题的 _config.yml 文件中配置以下内容，也就是你存放生成的页面的位置，如下：

```yml
menu:
  Home: /
  Archives: /archives
  Books: /books     # 这是你的书单页面
  Movies: /movies   # 这是你的电影页面
  Games: /games     # 这是你的游戏页面
```

### 适配 Matery

#### 主题端适配

在你的主题的 /themes/hexo-theme-matery/layout 文件夹下面创建一个名为 douban.ejs 的文件，并将下面的内容复制进去：

```javascript
<%- partial('_partial/post-cover') %>

<style>
    .hexo-douban-picture img {
        width: 100%;
    }
</style>

<main class="content">
    <div id="contact" class="container chip-container">
        <div class="card">
            <div class="card-content" style="padding: 30px">
                <h1 style="margin: 10px 0 10px 0px;"><%= page.title %></h1>
                <%- page.content %>
            </div>
        </div>
        <div class="card">
            <div class="card-content" style="text-align: center">
                <h3 style="margin: 5px 0 5px 5px;">如果你有好的内容推荐，欢迎在下面留言！</h3>
            </div>
         </div>
        <div class="card">
            <% if (theme.gitalk && theme.gitalk.enable) { %>
            <%- partial('_partial/gitalk') %>
            <% } %>

            <% if (theme.gitment.enable) { %>
            <%- partial('_partial/gitment') %>
            <% } %>

            <% if (theme.disqus.enable) { %>
            <%- partial('_partial/disqus') %>
            <% } %>

            <% if (theme.livere && theme.livere.enable) { %>
            <%- partial('_partial/livere') %>
            <% } %>

            <% if (theme.valine && theme.valine.enable) { %>
            <%- partial('_partial/valine') %>
            <% } %>
        </div>
    </div>
</main>
```

#### 插件适配

在你的博客文件夹内找到这个文件夹 /node_modules/hexo-douban/lib ，这个文件夹内找到以下三个文件： books-generator.js 、games-generator.js 、movies-generator.js

将每个文件内最下面的：

```javascript
layout: ['page', 'post']
```

改为：

```javascript
layout: ['page', 'douban']
```

上面提到的三个文件都要改！要是以后插件更新了，你的页面出问题了，那么这个地方就要重新改。

### 使用

所有的事情都准备好了，现在你可以生成你的页面，然后部署到相应的平台

```bash
hexo douban
```

就是这一行命令就行了，就可以生成相应的豆瓣页面了。

当然了，你也可以使用简化命令：

```bash
hexo d
```

需要注意的是，通常大家都喜欢用 hexo d 来作为 hexo deploy 命令的简化，但是当安装了 hexo douban 之后，就不能用 hexo d 了，因为 hexo douban 跟 hexo deploy 的前缀都是 hexo d ，你以后执行的 hexo d 将不再是 Hexo 页面的生成，而是豆瓣页面的生成

```bash
-h, --help    # 帮助页面
-b, --books   # 只生成书单页面
-g, --games   # 只生成游戏页面
-m, --movies  # 只生成电影页面
```

如果在配置文件中配置了 builtin 参数为 true ，那么除了可以使用 hexo douban 命令之外，hexo g 或 hexo s 也内嵌了生成页面的功能