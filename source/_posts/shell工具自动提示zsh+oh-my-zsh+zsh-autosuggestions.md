cover: http://ciwei2.cn-sh2.ufileos.com/99.jpg
title: shell工具自动提示zsh+oh-my-zsh+zsh-autosuggestions
date: 2019-03-19 15:54:08
tags: [crt,ssh]
categories: [综合]
---
### shell工具自动提示

zsh+oh-my-zsh+zsh-autosuggestions

自动提示补全，自动关联上次输入的命令呀

<!--more-->

### 安装zsh

```java
yum install -y zsh
```

### 安装oh-my-zsh

```java
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)
```

输入命令zsh使用

修改主题

```java
vi ~/.zshrc
```

主题目录：https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

```java
ZSH_THEME="risto"
```

### 安装自动提示

```java
wget http://mimosa-pudica.net/src/incr-0.2.zsh
mkdir ~/.oh-my-zsh/plugins/incr
mv incr-0.2.zsh ~/.oh-my-zsh/plugins/incr
echo 'source ~/.oh-my-zsh/plugins/incr/incr*.zsh' >> ~/.zshrc
source ~/.zshrc
```

### 安装zsh-autosuggestions

可以超级快速的帮你补全你输入过的命令，让命令行的操作更加高效

```java
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
vi ~/.zshrc
添加：plugins=(zsh-autosuggestions)
重启打开crt生效
```

### 快捷键

ctrl+f 补全

ctrl+d 删除智能提示