---
layout: post
title: git-相关命令
date: 2017-01-19 15:31:22
categories: JavaWeb
tags: git
author: MarsHu
---

* content
{:toc}

### git主要命令 ###

	从git仓库获取代码
	git clone 地址;
	
	git checkout feature/keywordsub
	
	从仓库更新代码
	git pull 仓库地址
	例:git pull origin feature/keywordsub 
	
	添加 当前库下所有修改的代码
	git add . 
	
	查看所有修改过的内容
	git status
	
	提交到本地库 的版本管理
	git commit -m 'refs 本次提交的内容' 
	
	提交到git仓库
	git push 仓库地址
	例:git push origin feature/keywordsub



  


### git其他相关命令 ###

	git reset --hard 清除修改
	
	git clean --df 清除添加
	
	git check xxx分支 选中分支
	
	
	当别人提交了代码，更新别人代码：
	1.输入 git status 查看本地的修改
	
	2.输入git stash 隐藏掉本地修改（会显示版本号）
	
	3.输入 git pull 从代码库拉取更新
	
	4.输入 git stash pop stash@{版本号}
	
	5.然后再git add git commit git push 就行了
	
	6.如果有多个相同版本号的stash时
	使用 git stash list 显示隐藏修改列表
	
	7.删除stash
	git stash drop stash@{0}
	
	
	删除git上某些没有忽略误传的文件
	首先:拉取文件
	git pull sell master
	其次:显示有哪些目录
	dir 显示有哪些目录。（这里有些带.的文件夹好像是不显示的）
	然后:删除不需要的文件
	git rm -r --cached 文件名
	git commit -m 'refs 删除.mvn文件夹'
	最后:删除后,需要提交一次!
	git push sell master

	8.当出现没有用户权限时,又不弹出输入用户窗口时
	git config --system --unset credential.helper
	然后就能重现看到了,并不适用所有状况。






