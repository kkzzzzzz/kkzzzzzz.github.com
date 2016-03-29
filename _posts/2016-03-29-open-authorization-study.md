---
layout: post
title: "OAuth2.0学习笔记"
description: ""
category: [study]
tags: [Authorization]
---
{% include JB/setup %}

# OAuth2.0学习笔记 #

## OAuht简介

	所谓OAuth（即Open Authorization，开放授权），它是一种让用户允许第三方应用在某一个网站上获取自己相关信息的协议，它能在不告诉第三方应用密码的情况下，使得第三方应用可以获得相关信息，并且对第三方应用可以使用的权限进行一定的划分控制。


## 名称解释

* **Resource Owner** 资源所有者，即用户
* **Resource Server** 资源服务器，即用户存放资源的服务器
* **Authorization Server**：认证服务器，即服务提供商专门用来处理认证的服务器。可以和Resource Server在同一台服务器也可以不一致
* **clinet** 第三方应用，需要获取用户在资源服务器的相关信息
* **user-Agent** 一般指浏览器
* **token** 认证服务器授予第三方应用的认证信息


## 基础流程

![图一](http://7xs9oq.com1.z0.glb.clouddn.com/ssd6aafc922276191e63b1d655197b7a22.png-960.jpg)

1. 第三方应用请求用户，希望得到授权
2. 用户给予授权
3. 第三方应用通过授权向认证服务器获取token
4. 认证服务器确认第三方授权无误，授予token
5. 第三方应用通过token访问资源服务器
6. 资源服务器确认token无误向第三方应用开发资源

通过这样一种方式，可以在用户无须输入密码的方式下，给予第三方权限访问信息，但是每次都需要用户给予一次授权，也就是有步骤A/B两步，oauth2.0引入**Refresh Token** 在一次获取授权后，具备自动刷新token的功能
![图二](http://7xs9oq.com1.z0.glb.clouddn.com/sse2c71001d219ff9ad11f898f6c82218e.png-960.jpg)

从图中看出，请求认证服务器之后，会同时返回token和Refresh Token,在进行C/D步骤中，当token过期之后，可以滤过E/F步骤。直接进行G/H步骤，用Refresh Token获取新的token和Refresh Token，在用户无感知的情况下，进行token刷新

## 用户的授权模式
第三方应用必须得到用户的授权，才能获得令牌。OAuth 2.0定义了四种授权方式。

* 授权码模式（authorization code）
* 隐式模式（implicit）
* 密码模式（resource owner password credentials）
* 客户端模式（client credentials）

### 授权码模式

![认证方式一](http://7xs9oq.com1.z0.glb.clouddn.com/ss385ced73a4409438415fd438640abc85.png-960.jpg)

### 隐式模式
![认证方式二](http://7xs9oq.com1.z0.glb.clouddn.com/ss65c80e415d39734f43ab9a276ab41fb6.png-960.jpg)

### 密码模式
![认证方式三](http://7xs9oq.com1.z0.glb.clouddn.com/ssc5e5c94b00f37a71f7821009de000838.png-960.jpg)

### 客户端模式
![认证方式四](http://7xs9oq.com1.z0.glb.clouddn.com/ssb97f884cfe6b0dd050f3b010bdd39655.png-960.jpg)

## 参考文献
* [The OAuth 2.0 Authorization Framework](http://tools.ietf.org/html/rfc6749)
* [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
