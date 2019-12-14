cover: http://ciwei2.cn-sh2.ufileos.com/135.jpg
title: 使用anaconda安装python3
date: 2018-11-08 11:40:31
tags: [python,anaconda]
categories: [综合]
---
```java
wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh

sh  Anaconda3-5.2.0-Linux-x86_64.sh  选择默认，输入yes，最后一步不用安装VC的包。

cd /root/anaconda3/bin

查看版本 ./conda --version

查看安装情况 ./conda info -e

4、 创建删除环境

# 创建新的环境

--name 后面跟环境名

./conda create --name python36 python=3.6

# 激活环境

source activate python36 #linux&Mac

# 查看当前python版本

python --version

# 设置版本(我安装的目录是/root/anaconda3)
cd /root
vi .bashrc 
export PATH=/root/anaconda3/bin:$PATH
source   ~/.bashrc
退出crt后重新登陆
python -V

# 返回默认的root环境

source deactivate python36 #linux&Mac

# 删除环境

conda remove --name python36 --all
```