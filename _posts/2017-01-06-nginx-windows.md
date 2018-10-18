---
layout: post
title: windows-nginx常用命令
date: 2017-01-06 15:08:11
categories: server
tags: nginx
author: MarsHu
---

* content
{:toc}

# windows-nginx #
    start nginx   # 启动Nginx
    nginx -s stop   # 快速停止Nginx，可能并不保存相关信息
    nginx -s quit# 完整有序的停止Nginx，并保存相关信息
    nginx -s reload# 重新载入Nginx，当配置信息修改，需要重新载入这些配置时使用此命令。
    nginx -s reopen   # 重新打开日志文件
    nginx -v  # 查看Nginx版本
    
    netstat -aon|findstr "443"  # 检查443端口是否被占用