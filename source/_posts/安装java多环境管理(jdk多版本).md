title: 安装java多环境管理(jdk多版本sdkman)
date: 2019-11-18 10:41:48
tags: [sdkman,java]
categories: [综合]
---
### windows安装jdk多版本

#### 安装Chocolatey

##### 点击win10桌面的开始菜单，输入cmd，右键选中命令提示符，选择以管理员身份运行

<!--more-->

![](/images/20191118104408.png)

##### 在cmd窗口中粘贴命令，按下回车键

```bash
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

##### 测试安装状态

输入命令choco，显示chocolatey版本号，说明安装成功

#### 安装unzip和zip

安装Chocolatey后，CMD（管理员权限）执行下面命令

```bash
choco install unzip
choco install zip
```

#### 安装sdkman

打开git bash 终端，CMD（管理员权限）执行命令：

```bash
curl -s "https://get.sdkman.io" | bash
```

执行成功后关闭终端，重新打开一个终端即可使用sdkman

上一步输出的类似这样的信息：

```bash
All done!


Please open a new terminal, or run the following in the existing one:

    source "/c/Users/Ciwei/.sdkman/bin/sdkman-init.sh"

Then issue the following command:

    sdk help

Enjoy!!!
```

执行：source "/c/Users/Ciwei/.sdkman/bin/sdkman-init.sh"

```bash
查看版本：
sdk version

查看java版本：
$ sdk list java
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 13.0.1.j9    | adpt    |            | 13.0.1.j9-adpt
               |     | 13.0.1.hs    | adpt    |            | 13.0.1.hs-adpt
               |     | 12.0.2.j9    | adpt    |            | 12.0.2.j9-adpt
               |     | 12.0.2.hs    | adpt    |            | 12.0.2.hs-adpt
               |     | 11.0.5.j9    | adpt    |            | 11.0.5.j9-adpt
               |     | 11.0.5.hs    | adpt    |            | 11.0.5.hs-adpt
               |     | 8.0.232.j9   | adpt    |            | 8.0.232.j9-adpt
               |     | 8.0.232.hs   | adpt    |            | 8.0.232.hs-adpt
 Amazon        |     | 11.0.5       | amzn    |            | 11.0.5-amzn
               |     | 8.0.232      | amzn    |            | 8.0.232-amzn
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu
               |     | 7.0.242      | zulu    |            | 7.0.242-zulu
               |     | 6.0.119      | zulu    |            | 6.0.119-zulu
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca
               |     | 12.0.2       | librca  |            | 12.0.2-librca
               |     | 11.0.5       | librca  |            | 11.0.5-librca
               |     | 8.0.232      | librca  |            | 8.0.232-librca
 GraalVM       |     | 19.2.1       | grl     |            | 19.2.1-grl
               |     | 19.1.0       | grl     |            | 19.1.0-grl
               |     | 19.0.2       | grl     |            | 19.0.2-grl
 Java.net      |     | 14.ea.22     | open    |            | 14.ea.22-open
               |     | 13.0.1       | open    |            | 13.0.1-open
               |     | 12.0.2       | open    |            | 12.0.2-open
               |     | 11.0.5       | open    |            | 11.0.5-open
               |     | 10.0.2       | open    |            | 10.0.2-open
               |     | 9.0.4        | open    |            | 9.0.4-open
               |     | 8.0.232      | open    |            | 8.0.232-open
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================

安装java：
sdk install java ${Identifier}
举例：sdk install java 13.0.1.j9-adpt
```

### macos和linux安装

#### 安装sdkman

```bash
curl -s "https://get.sdkman.io" | bash
```

按照相应的指令提示，完成相应的操作后继续输入:

```bash
举例输出样例：
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

到这里我们就可以验证 sdk 的安装版本了:

```bash
sdk version
```

![](/images/blog-imgXnip2019-11-13_14-45-11.jpg)

上图红色框标记显示我当前 sdkman 的版本，每次执行 sdk version 命令时，都会检查是否会有新版本，如果要更新输入 y 就可以

有些系统发行版本不包含 zip 和 unzip，如果安装时遇到相关错误，可以输入如下命令安装 zip 和 unzip

```bash
sudo apt-get install zip unzip
```

从上面的安装命令上可以看出，sdkman 默认的安装路径是在$HOME/.sdkman 下，我们也可以自定义安装路径，只需要指定 SDKMAN_DIR 变量值就好了:

```bash
export SDKMAN_DIR="/usr/local/sdkman" && curl -s "https://get.sdkman.io" | bash
```

#### sdkman使用教程

命令行下学习一个新玩意当然是查看它的 help 命令，输入:

```bash
sdkman help
```

![](/images/blog-imgXnip2019-11-13_15-35-22.jpg)

感觉上图按颜色区分内容后，sdkman 的使用说明也就结束了，我们按照上面的图来详细说明一下使用教程

##### sdk list

先来输入:

```bash
sdk list
```

![](/images/blog-imgXnip2019-11-13_16-07-35.jpg)

绿色的标记就是 sdkman 集成的所有可用的 candidate，通过按回车「enter」按键，会看到更多可用 candidate

我们指定 candidate，输入:

```bash
sdk list java
```

![](/images/blog-imgXnip2019-11-13_16-14-36.jpg)

从上图中可以看到所有 java 可用的版本 version，以及标识 indentifier，以及状态 status，我已经安装了 java 12 和 11

有了这些信息做铺垫，我们可以安装任意 sdkman 内置的软件开发包了，继续以 java 为例

##### sdk install

回看 sdkman help 命令的输出，使用 install 命令，我们再安装一个 Java 最新 13.0.1.j9 版本

![](/images/blog-imgXnip2019-11-13_16-28-17.jpg)

从上图你可以看出，绿色标记的内容是 list 命令结果中的 version 值，但是报错不可用，输入indentifier 编号才能正常下载，这里需要注意

安装完后，status 就会编程 installed 状态

##### sdk current

当安装多个版本的 java 时，我们输入下面命令获取当前正在用 candidate 的版本

```bash
sdk current java
```

##### sdk use

了解了当前使用版本，如果我们想切换到其他版本, 可以输入:

```bash
sdk use java 12.0.2.j9-adpt
```

注意⚠️: 这里同样是指定的 indentifier 的值

![](/images/blog-imgXnip2019-11-13_17-56-22.jpg)

##### sdk default

如果我们想指定某个版本为默认版本，可以输入:

```bash
sdk default java jdk1.8.0_162.jdk
```

注意⚠️: 这里同样是指定的 indentifier 的值

![](/images/blog-imgXnip2019-11-13_18-00-41.jpg)

##### sdk uninstall

当我们想卸载某个版本可以输入:

```bash
sdk uninstall java 12.0.2.j9-adpt
```

注意⚠️: 这里同样是指定的 indentifier 的值

##### sdk upgrade

如果我们想升级某个 candidate，可以输入:

```bash
sdk upgrade java
```

##### sdk flush

使用 sdkman 时间变长也会慢慢产生很多缓存内容，我们可以输入清理广播消息:

```bash
sdk flush broadcast
```

清理下载的 sdk 二进制文件(长时间使用后清理，可以节省出很多空间):

```bash
sdk flush archives
```

清理临时文件内容:

```bash
sdk flush temp
```

到这里 sdkman 的基本使用就已经介绍完了，其实这些命令都不用急，想不起来的时候执行 sdk help 来临时查看一下就好

##### sdkman 卸载

如果我们不喜欢 sdkman 了，我们也可以轻松的卸载掉它:

```bash
tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
rm -rf ~/.sdkman
```

最后打开你的 .bashrc、.bash_profile 和/或者 .profile，找到并删除下面这几行

```bash
#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```

我用的 zshrc，找到 .zshrc 文件删除掉上面内容即可

### 本地路径

sdkman下载的本地路径(可以在idea中使用当前下载的所有版本的)：

windows：C:\Users\Ciwei\.sdkman\archives

macos和linux：$HOME/.sdkman/archives