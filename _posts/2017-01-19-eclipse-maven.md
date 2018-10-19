---
layout: post
title: eclipse-阿里云maven本地索引
date: 2017-01-17 13:28:14
categories: IDE
tags: eclipse
author: MarsHu
---

* content
{:toc}

# eclipse-为阿里云maven仓库建立本地索引 #
## 1.手动下载索引文件 ##
在浏览器地址栏访问如下地址:
`http://repo1.maven.org/maven2/.index/`

打开网页后，将滚动条拉到网页最底部，分别下载

`nexus-maven-repository-index.gz`和`nexus-maven-repository-index.properties`两个文件

## 2.将索引文件放入tomcat ##
将下载后的索引文件放入如下tomcat目录

（没有对应文件夹，请创建，tomcat目录请参考使用者本地路径）
```
D:\javawork\apache-tomcat-7.0.70\webapps\ROOT\nexus\content\groups\public\.index
```
提示：在windows电脑中，不能直接新建.index文件夹，
在所选目录下（即最后一个public目录）用cmd命令。

在最后一个public目录下，按住shift，鼠标右键，选择`在此处打开Powershell窗口`

输入如下命令
```
mkdir .index
```
## 3.修改本机hosts地址映射 ##
进入如下目录
```
C:\Windows\System32\drivers\etc
```
选择记事本或其他工具打开hosts文件，在hosts文件的空白处添加如下配置并保存
```
127.0.0.1 maven.aliyun.com
```
如果不能保存请自行修改hosts文件的修改权限。
## 4.在eclipse中建立索引目录 ##
`修改tomcat启动端口为80，并启动tomcat`。打开eclipse，按如下操作配置。
![eclipse1.png](http://marshucheng1.github.io/assets/eclipse1.png)
![eclipse2.png](http://marshucheng1.github.io/eclipse2.png)
## 5.手动更新下载maven索引 ##
当进行完第4步时，默认已经可以自动更新索引了（eclipse右下角出现运行标志）。

如果没有自动更新，请按如下步骤操作。
![eclipse3.png](http://marshucheng1.github.io/assets/eclipse3.png)
![eclipse4.png](http://marshucheng1.github.io/assets/eclipse4.png)
![eclipse5.png](http://marshucheng1.github.io/assets/eclipse5.png)

在更新索引时，可以看到eclipse窗口右下角在运行，别管它，经过漫长等待之后，会自动消失，就表示索引完毕了！可以关闭tomcat了。
## 6.将本机hosts地址映射复原(`一定要做`) ##
打开hosts文件，将之前第3步添加到hosts文件中的映射地址删除。

