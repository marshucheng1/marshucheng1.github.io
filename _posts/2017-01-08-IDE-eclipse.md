---
layout: post
title: 修改eclipse项目栏字体大小
date: 2017-01-08 22:55:03
categories: IDE
tags: eclipse
author: MarsHu
---

* content
{:toc}

# windows下eclipse项目栏字体调整 #
eclipse中项目导航字体大小由配置文件中的设置决定。

1. 配置文件：找到eclipse安装位置（或解压路径）：
```
`[盘符]:\eclipse\plugins\org.eclipse.ui.themes_1.1.0.v20150511-0913\css`
```
注：由于eclipse版本不同上述路径中的版本数字信息可能有差别，可以模糊搜索org.eclipse.ui.themes.....




2. 找到配置文件名为e4_default_win.css
3. 添加配置设置：使用文本编辑器（例如 notepad++ 、Editplus等）打开e4_default_win.css添加如下新配置：
```
CTabFolder Tree{
    font-size: 12px;
}
```
注：系统默认字体大小为10px，根据需要进行设置，数值越大字体越大。

4. 保存后重新启动eclipse即可应用新设置的字体