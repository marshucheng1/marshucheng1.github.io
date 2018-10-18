---
layout: post
title: linux安装nginx
date: 2017-01-16 22:41:55
categories: server
tags: nginx
author: MarsHu
---

* content
{:toc}

# centos安装nginx与配置开机启动 #
1. 安装运行库、编译环境
```
yum -y install gcc automake autoconf libtool make
yum -y install gcc gcc-c++
yum -y install openssl openssl-devel
```




2. 使用wget命令下载PCRE库
```
cd /usr/local/src/
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz
```
3. 解压并安装PCRE库
```
tar -zxvf pcre-8.41.tar.gz
cd pcre-8.41
./configure
make
make install
```
4. 下载zlib，下载最新版---老版本可能不支持下载了，官方地址 http://zlib.net/
```
cd /usr/local/src/
wget http://zlib.net/zlib-1.2.11.tar.gz
```
5. 解压并安装zlib
```
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```
6. 下载nginx，选择自己需要的版本，官方地址 http://nginx.org/download/
```
wget http://nginx.org/download/nginx-1.9.8.tar.gz
```
7. 解压并安装nginx
```
tar -zxvf nginx-1.9.8.tar.gz
cd nginx-1.9.8
./configure --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --pid-path=/usr/local/nginx/sbin/nginx.pid --with-http_ssl_module --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11
make
make install
```
8. 配置开启启动
a.在/etc/init.d下创建nginx启动脚本文件：
```
vim /etc/init.d/nginx
```
按下i键进入编辑状态，粘贴以下代码：
```
 #!/bin/sh   
 #   
 # nginx - this script starts and stops the nginx daemon   
 #   
 # chkconfig: - 85 15   
 # description: Nginx is an HTTP(S) server, HTTP(S) reverse \   
 #   proxy and IMAP/POP3 proxy server   
 # processname: nginx   
 # config: /etc/nginx/nginx.conf   
 # config: /etc/sysconfig/nginx   
 # pidfile: /var/run/nginx.pid   
 # Source function library.   
 . /etc/rc.d/init.d/functions   
 # Source networking configuration.   
 . /etc/sysconfig/network   
 # Check that networking is up.   
 [ "$NETWORKING" = "no" ] && exit 0   
     nginx="/usr/local/nginx/sbin/nginx"   
     prog=$(basename $nginx)   
     NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"   
 [ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx   
     lockfile=/var/lock/subsys/nginx    
 start() {   
     [ -x $nginx ] || exit 5   
     [ -f $NGINX_CONF_FILE ] || exit 6   
     echo -n $"Starting $prog: "   
     daemon $nginx -c $NGINX_CONF_FILE   
     retval=$?   
     echo   
 [ $retval -eq 0 ] && touch $lockfile   
     return $retval   
 }   
 stop() {   
     echo -n $"Stopping $prog: "   
     killproc $prog -QUIT   
     retval=$?   
     echo   
 [ $retval -eq 0 ] && rm -f $lockfile   
     return $retval   
     killall -9 nginx   
 }   
 restart() {   
     configtest || return $?   
     stop   
     sleep 1   
     start   
 }   
 reload() {   
     configtest || return $?   
     echo -n $"Reloading $prog: "   
     killproc $nginx -HUP   
     RETVAL=$?   
     echo   
 }   
 force_reload() {   
     restart   
 }  
 configtest() {   
     $nginx -t -c $NGINX_CONF_FILE   
 }   
 rh_status() {   
     status $prog   
 }   
 rh_status_q() {   
     rh_status >/dev/null 2>&1   
 }   
 case "$1" in   
     start)   
         rh_status_q && exit 0   
         $1   
     ;;   
     stop)   
         rh_status_q || exit 0   
         $1   
     ;;   
     restart|configtest)   
         $1   
     ;;   
     reload)   
         rh_status_q || exit 7   
         $1   
     ;;   
     force-reload)   
         force_reload   
     ;;   
     status)   
         rh_status   
     ;;   
     condrestart|try-restart)   
         rh_status_q || exit 0   
     ;;   
     *)   
         echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"   
         exit 2   
 esac 
```

	b.修改脚本权限
```
chmod 755 nginx
```

	c.将脚本文件加入到chkconfig中
```
chkconfig --add nginx
```

	d.设置nginx开机在3和5级别自动启动
```
chkconfig --level 35 nginx on
```

	e.创建软连接
```
cd /usr/bin
ln -s /etc/init.d/nginx
```

	f.命令示例：分别表示启动、停止、重启
```
nginx start
nginx stop
nginx restart
```

	h.访问nginx
	访问centos下的nginx服务器，需要在服务器设置对外开放80端口，按照如下操作：
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd  --reload
```