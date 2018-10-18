---
layout: post
title: centos-java环境安装
date: 2017-01-07 20:09:45
categories: java
tags: javabasic
author: MarsHu
---

* content
{:toc}

# centos-jdk #
1. 本地下载好jdk，上传到centos，以 jdk-8u152-linux-x64.tar.gz 为例
在centos中安装上传和下载命令
```
yum -y install lrzsz
```
进入如下目录等待上传jdk包
```
cd /usr/local/java
```
使用rz 空格 enter键选择jdk包所在位置，等待上传完毕
```
rz
```




2. 解压压缩包
```
tar zxvf jdk-8u152-linux-x64.tar.gz
```
3. 配置java环境变量
```
vim /etc/profile
```
按下i键，进入编辑状态，在`export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL`
一行的上面添加如下内容:
```
#set for java
export JAVA_HOME=/usr/local/java/jdk1.8.0_152/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
按下esc键，输入如下命令保存并退出：
```
:wq!
```
4. 编译/etc/profile 使配置生效
```
source /etc/profile
```
5. 验证是否配置完成
```
java -version
```
显示java信息，则表示配置成功