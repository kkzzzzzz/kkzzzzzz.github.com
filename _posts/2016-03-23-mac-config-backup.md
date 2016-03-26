---
layout: post
title: "mac一些配置的备份"
description: ""
category:
tags: [mac]
---





* 公司开发会用到一个gitlab账号，github也会用到一个账号，这个时候需要用到多账号配置，
进入~/.ssh 下新建config

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
		git config user.name "kkzzzzzz"
		git config user.email "zhukaidj@gmail.com"
