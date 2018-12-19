---
layout: post
title: Centos-Linux常用命令
date: 2016-12-08 14:22:18
categories: linux
tags: centos
author: MarsHu
---

* content
{:toc}

# 常见Linux命令汇总 #
### 相关安装源包的命令安装 ###
    安装wget命令
    yum -y install wget
    安装vim命令
    yum -y install vim

### 文件上传、下载 ###
	yum -y install 包名（支持*） ：自动选择y，全自动
	yum install 包名（支持*） ：手动选择y or n
	yum -y install lrzsz
	上传文件：
	rz + Enter键，然后选择要上传文件
	下载文件：
	sz + Enter键，然后选择保存文件的位置





### 解压缩 ###
```
tar -zxvf jdk-8u152-linux-x64.tar.gz
```

### 配置环境变量-以JAVA为例 ###
```
vim /etc/profile
```

按下`i`键，在文件尾部加入环境变量的定义：

    #set for java
    export JAVA_HOME=/usr/local/develop/jdk1.8.0_152/
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
修改完毕后，按`esc`键退出编辑状态，输入如下命令并回车：

```
:wq!
```

保存后，输入如下命令，使环境变量生效：

```
source /etc/profile
```
### 文件、文件夹相关命令 ###
    创建文件夹：mkdir 
    彻底删除：rm -rf 名称
    创建文件：touch 
    移动文件：mv zookeeper-3.4.11 ..
    重命名：mv old_name new_name

### 运行jar包 ###
    指定配置文件，springboot，指定生产环境配置文件
    nohup java -jar -Dserver.port=8088 -Dspring.profiles.active=prod yiiu.jar >yiiu.log 2>&1 &

    指定端口
    nohup java -jar -Dserver.port=8088 yiiu.jar >yiiu.log 2>&1 &

    一般运行，以配置文件端口为准
    nohup java -jar yiiu.jar >yiiu.log 2>&1 &

	后台查看进程id （假设为27660）
    ps aux|grep yiiu.jar

	结束进程
    kill -9 27660

### 防火墙 ###
	开启防火墙：
    service iptables start
	关闭防火墙：
    service iptables stop

### natapp的启动（内网穿透软件） ###
将natapp文件上传到`/usr/local/`路径下

    cd /usr/local/natapp
    chmod a+x natapp
    nohup ./natapp -authtoken=ce01630c0595f8f4 -log=stdout &（14422）
    ps -ef|grep natapp

### sh文件，以及启动jar包 ###
    待完成吧--