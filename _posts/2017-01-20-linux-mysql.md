---
layout: post
title: linux安装mysql5.7或5.6
date: 2017-01-20 09:55:28
categories: linux
tags: centos
author: MarsHu
---

* content
{:toc}

# centos安装mysql5.7或5.6 #
### 1.下载对应安装包 ###

5.7
```
cd /usr/local/src
wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

5.6
```
cd /usr/local/src
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm 
```





### 2.启动软件安装包 ###

5.7
```
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```

5.6
```
rpm -ivh mysql-community-release-el6-5.noarch.rpm
```

### 3.安装数据库 ###
```
yum -y install mysql-community-server
```

### 4.等待安装完mysql，相关操作命令，启动、重启、停止 ###
```
systemctl start mysqld
systemctl restart mysqld
systemctl stop mysqld
```

或
```
service mysqld start
service mysqld restart
service mysqld stop
```

### 5.将mysq数据库设置为开机启动 ###
```
systemctl enable mysqld
systemctl daemon-reload
```

或
```
chkconfig --list | grep mysqld
chkconfig mysqld on
```

### 6.修改root用户默认密码 ###

> 5.7版本的数据

5.7版本mysql安装完成之后，在`/var/log/mysqld.log`文件中给root生成了一个默认密码。

查看默认密码：`grep 'temporary password' /var/log/mysqld.log`

用默认密码登陆，输入如下命令输入默认密码登录：
```
mysql -u root -p
```

设置新密码
由于5.7密码设置有规则限制，需要更改密码设置规则，输入如下命令：

-----只基于密码长度
```
set global validate_password_policy=0;
```

----密码长度最小为6
```
set global validate_password_length=6;
```

----设置密码为123456
```
SET PASSWORD = PASSWORD('123456');
```

> 5.6版本的数据库

不进入数据库即可修改密码，输入如下命令：
```
cd /usr/bin/
mysqladmin -u root password '123456'
```

### 7.允许root远程登录 ###
`进入mysql数据库后`，输入如下命令：
```
grant all privileges on *.* to 'root'@'%'identified by '123456' with grant option;
flush privileges;
```

### 8.修改编码 ###
`进入mysql数据库后`，输入如下命令查看编码
```
show variables like 'char%';
```

如果需要修改编码，需要先停止数据库，退出数据库，输入如下命令：
```
systemctl stop mysqld
```

或
```
service mysqld stop
```

修改/etc/my.cnf配置文件，输入如下命令：
```
vim /etc/my.cnf
```

按下`i`键进入编辑状态，设置成UTF8编码请参照如下配置，其他编码类似
```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
max_allowed_packet=20M

[mysqld]
character-set-server=utf8
init-connect='SET NAMES utf8'
max_allowed_packet=20M
```

编辑完成后，按下`esc`键，输入如下命令保存：
```
:wq!
```

重新查看编码。需要重启mysql数据库，输入如下命令：
```
systemctl start mysqld
```

或
```
service mysqld start
```

进入mysql数据后输入如下命令：
```
show variables like 'char%';
```
发现编码已经变更。


### 9.Mysql5.7数据库密码忘了怎么整 ###

首先关闭`mysql`数据库

	systemctl stop mysqld

然后编辑数据库配置文件

	vi /etc/my.cnf

在配置文件`[msyqld]`下面添加一行`skip-grant-tables`。保存配置文件。

重启`mysql`数据库

	systemctl start mysqld

重新登陆`mysql`数据库，这个时候。已经不需要密码了。让我们重新修改密码。

	mysql -u root -p
	
	use mysql;

	update user set authentication_string=password('123456') where user='root' and host='localhost';

	exit;

再次关闭`mysql`数据库

	systemctl stop mysqld

然后再次编辑数据库配置文件，取消上面添加的那行。保存配置文件。

再次重启`mysql`数据库

	systemctl start mysqld

### 10.windows下修改mysql数据库密码 ###
mysqladmin -u用户名 -p旧密码 password 新密码

