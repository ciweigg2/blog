---
title: mac安装brew
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
  - brew
categories:
  - 综合
date: 2020-01-11 22:12:52
---

### 安装brew

方式一（网速快的推荐）：

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

<!--more-->

方式二（网速慢的推荐）：

```bash
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> brew_install

vi brew_install

找到如下代码(新版本HomeBrew可能没有CORE_TAP_REPO这句代码，如果没有不用新增)：
BREW_REPO = "https://github.com/Homebrew/brew".freeze
CORE_TAP_REPO = "https://github.com/Homebrew/homebrew-core".freeze
复制代码更改为：
BREW_REPO = "https://mirrors.ustc.edu.cn/brew.git".freeze
CORE_TAP_REPO = "https://mirrors.ustc.edu.cn/homebrew-core.git".freeze

执行脚本：
/usr/bin/ruby brew_install

此时脚本应该停在
==> Tapping homebrew/core
Cloning into ‘/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core’…

解决方法，手动执行下面这句命令，更换为中科院的镜像：
git clone git://mirrors.ustc.edu.cn/homebrew-core.git/ /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core --depth=1

复制代码然后把homebrew-core的镜像地址也设为中科院的国内镜像
cd $(brew --repo)
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

执行更新：
brew update

检查错误：
brew doctor
```

### 卸载brew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```