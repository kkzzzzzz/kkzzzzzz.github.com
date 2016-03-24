---
layout: post
title: "mac config"
description: ""
category:
tags: []
---


#mac配置备份

##git相关
* 多账号配置</br> 进入~/.ssh 下新建config

		cd ~/.ssh && vim config

	粘贴如下内容

		host github.com
    	HostName github.com
    	User kkzzzzzz
    	IdentityFile ~/.ssh/github_rsa

		host code.dianpingoa.com
	    HostName code.dianpingoa.com
    	User kevin.zhu
	    IdentityFile ~/.ssh/id_rsa

	公司的设置一个全局user，github设置一个loacl user

		git config --global user.name "kevin.zhu"
		git config --global user.email "kevin.zhu@dianping.com"
		git config user.name "kkzzzzz"
		git config user.email "zhukaidj@gmail.com"
