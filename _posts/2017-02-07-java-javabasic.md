---
layout: post
title: jdk的下载及java环境变量的配置
date: 2017-02-07 15:22:43
categories: java
tags: javabasic
author: MarsHu
---

* content
{:toc}

# 下载及安装jdk #
1.百度搜索`jdk`，点击如下链接：

![jdk.png](http://localhost:4000/assets/jdk.png)





2.找到对应的`jdk`版本，选择下载，这里以1.8版本为例：

![jdk1.8.png](http://localhost:4000/assets/jdk1.8.png)

3.进入如下图所示界面。

![jdk64.png](http://localhost:4000/assets/jdk64.png)

①点击`Accept License Agreement`，表示接受相关许可。

②选择对应各自`windows`系统的`jdk`，其中`x86`表示`32位`操作系统。
`x64`表示`64位`操作系统。

③点击对应版本进行下载。

4.等待浏览器下载完毕，点击安装程序进行安装，进入安装界面。

![jdk1.png](http://localhost:4000/assets/jdk1.png)

点击下一步。进入下图界面。

![jdk2.png](http://localhost:4000/assets/jdk2.png)

首先点`公共JRE`、然后再点`此功能将不可用`。因为`jdk`已经自带`JRE`了。

所以不需要安装公共的`JRE`。

然后安装界面变成如下所示：

![jdk3.png](http://localhost:4000/assets/jdk3.png)

点击下一步。进入安装界面。

![jdk4.png](http://localhost:4000/assets/jdk4.png)

点击关闭完成安装。

# 环境变量配置，以win10为例 #
1.选择我的电脑，鼠标右键，选择属性。如下图所示

![jdk5.png](http://localhost:4000/assets/jdk5.png)

2.进入如下所示界面，选择高级设置。如下图所示

![jdk6.png](http://localhost:4000/assets/jdk6.png)

3.进入如下所示界面，选择高级，再点环境变量。如下图所示

![jdk7.png](http://localhost:4000/assets/jdk7.png)

4.进入如下所示界面，在系统变量中找到`Path`变量。如下图所示

![jdk8.png](http://localhost:4000/assets/jdk8.png)

5.双击`Path`变量，进入如下图所示界面。

![jdk9.png](http://localhost:4000/assets/jdk9.png)

6.点击新建，在空白处输入类似如下内容，然后点击确定。如下图所示

![jdk10.png](http://localhost:4000/assets/jdk10.png)

输入的内容实际上就是`jdk`的安装目录下的`bin`文件路径。

7.配置`JAVA_HOME`。按下图操作顺序，内容参考图中所示：

![jdk11.png](http://localhost:4000/assets/jdk11.png)

①点击新建，弹出新建系统变量窗口

②在窗口中参考图示输入内容

③点击确定

④点击确定

因为有很多其他工具，会依赖java环境。所以建议配置`JAVA_HOME`。

8.至此！java环境变量配置完成。至于还有一个`CLASSPATH`的环境变量。

那是`java1.5`之前的版本才需要配置的。`1.5之后版本`可以不配置。

如果需要配置可以参考`JAVA_HOME`环境变量的配置。内容参考下图：

![jdk12.png](http://localhost:4000/assets/jdk12.png)

其中变量值为：

`.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`