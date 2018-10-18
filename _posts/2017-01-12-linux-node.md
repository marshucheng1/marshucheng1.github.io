---
layout: post
title: linux安装Node.js
date: 2017-01-12 00:12:36
categories: linux
tags: node.js
author: MarsHu
---

* content
{:toc}

# centos安装Node.js #
本文中以8.12.0版本为例，最好安装新版本，因为新版Node.js集成了npm，无需额外安装。

1.下载Node.js，在服务器上选择位置，使用wget命令下载。具体地址 https://nodejs.org/en/download/
```
cd /usr/local/src/
wget https://nodejs.org/download/release/v8.12.0/node-v8.12.0.tar.gz
```




或
```
wget https://nodejs.org/dist/v8.12.0/node-v8.12.0.tar.gz
```
如果提示没有wget命令，需要安装wget命令，再下载安装包
```
yum -y install wget
```
2.解压源码
```
tar zxvf node-v8.12.0.tar.gz
```
3.指定路径编译安装
```
cd node-v8.12.0
./configure --prefix=/usr/local/node/8.12.0
make
make install
```

在编译过程中可能会提示安装gcc库，需要按照如下命令安装gcc库，然后再编译安装
```
yum -y install gcc-c++
```
4.进入profile配置环境变量
```
vim /etc/profile
```

如果提示找不到vim命令，需要先安装vim命令
```
yum -y install vim
```

按下i键进入编辑模式，在
`export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL`
一行的上面添加如下内容
```
#set for nodejs
export NODE_HOME=/usr/local/node/8.12.0
export PATH=$NODE_HOME/bin:$PATH
```

按下esc键，输入如下命令
```
:wq!
```
按下回车键保存并退出
5.编译/etc/profile 使配置生效
```
source /etc/profile
```
6.验证是否安装成功
```
node -v
```
如果有版本信息显示，则表示安装成功。
