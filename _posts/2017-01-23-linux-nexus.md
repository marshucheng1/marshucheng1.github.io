---
layout: post
title: Nexus搭建Maven私服
date: 2017-01-23 11:11:11
categories: linux
tags: nexus
author: MarsHu
---

* content
{:toc}

# centos使用Nexus搭建Maven私服 #
### 1.下载nexus-2.11.2-06-bundle.tar.gz ###
```
cd /usr/local/nexus
wget https://sonatype-download.global.ssl.fastly.net/nexus/oss/nexus-2.11.2-06-bundle.tar.gz
```
### 2.解压nexus-2.11.2-06-bundle.tar.gz ###
```
tar zxvf nexus-2.11.2-06-bundle.tar.gz
```




### 3.修改配置文件 ###
```
cd nexus-2.11.2-06/conf
vim nexus.properties
```

按下`i`键，按下如下配置进行修改，以下部分，代表启动端口号和包下载位置
```
 #Jetty section
 application-port=8081      ##修改Jetty端口号
 # nexus section
 nexus-work=${bundleBasedir}/../sonatype-work/nexus
```
按需修改，一般最多改端口号，正常采用默认即可。

如果不修改，按`esc`键退出编辑状态，输入如下命令并回车：
```
:q!
```

### 5.配置运行用户 ###
输入如下命令进入运行用户配置：
``
vim /usr/local/nexus/nexus-2.11.2-06/bin/nexus
``

进入修改界面，按照如下代码修改
```
 #RUN_AS_USER=
 RUN_AS_USER=root
```

修改完毕后，按`esc`键退出编辑状态，输入如下命令并回车：
```
:wq!
```

### 6.启动nexus ###
依次输入如下命令启动nexus
```
cd /usr/local/nexus/nexus-2.11.2-06/bin/
./nexus start
```

### 7.访问nexus ###

在浏览器地址栏输入`http://所在服务器ip:8081/nexus`访问。

默认用户名为admin，密码admin123

### 8.Maven添加本地jar包到本地仓库 ###

	mvn install:install-file -Dfile=XXX-3.6.7.jar -DgroupId=com.spiredoc -DartifactId=XXX-XXX-XXX -Dversion=3.6.7 -Dpackaging=jar