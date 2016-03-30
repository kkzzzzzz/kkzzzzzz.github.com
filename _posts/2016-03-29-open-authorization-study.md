---
layout: post
title: "OAuth2.0学习笔记"
description: ""
category: [study]
tags: [Authorization]
---
{% include JB/setup %}

# OAuth2.0学习笔记

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

> The authorization code grant type is used to obtain both access
   tokens and refresh tokens and is optimized for confidential clients.

授权码模式（authorization code）可以看到是有对第三方应用做好保密的优化的，而且它是需要通过浏览器和用户进行交互，并且需要能够接收认证服务器请求。

![认证方式一](http://7xs9oq.com1.z0.glb.clouddn.com/ss385ced73a4409438415fd438640abc85.png-960.jpg)



>1. 用户访问第三方应用，第三方应用导向认证服务器，请求授权，请求包含它的标识符，重定向URL
2. 认证服务器验证用户是否有权限，并且确定用户是否同意授权（基于浏览器进行的操作）
3. 假设用户同意之后，认证服务器重定向到之前的URI，并且带有授权码
4. 第三方应用拿着授权码和获得授权码的时候的重定向URI去认证服务器请求token
5. 认证服务器验证授权码，确认重定向的URI与授权码一致，认证服务器返回token，和refresh token

第五步中（理解是URI与授权码需要对应，不是拿着一个授权码就可以请求到token）

### 隐式模式

隐式模式也被人称之为简单模式，它是**不支持**refresh token的，整个过程不存在和第三方应用的交互，因此refresh token是没有办法被使用的。

不像授权码认证那样分隔开了授权码和token的概念，隐式模式拿到的授权码实际就是token


![认证方式二](http://7xs9oq.com1.z0.glb.clouddn.com/ss65c80e415d39734f43ab9a276ab41fb6.png-960.jpg)

>1. 用户访问第三方应用，第三方应用导向认证服务器，请求授权，请求包含它的标识符，重定向URL
2. 认证服务器验证用户是否有权限，并且确定用户是否同意授权（基于浏览器进行的操作）
3. 假设用户同意之后，认证服务器重定向到之前的URI，并且在Fragment中带有token
4. 浏览器重定向到资源服务起，并且不包含Fragment
5. 资源服务器返回一段脚本
6. 浏览器执行这个脚本提取到token
7. 返回给第三方应用token

### 密码模式

这种方式是将账号密码给予了第三方应用，对第三方应用的信任度比较高，比如操作系统（device operating system），高特权应用（highly privileged application）
![认证方式三](http://7xs9oq.com1.z0.glb.clouddn.com/ssc5e5c94b00f37a71f7821009de000838.png-960.jpg)

> 1. 用户将密码给到第三方应用
> 2. 第三方应用通过密码凭证请求认证服务器
> 3. 认证服务器返回token

### 客户端模式

第三方应用可以直接通过自己的凭证请求认证服务器

![认证方式四](http://7xs9oq.com1.z0.glb.clouddn.com/ssb97f884cfe6b0dd050f3b010bdd39655.png-960.jpg)

> 1. 第三方应用通过自己的凭证，向认证服务器进行身份认证，请求token
> 2. 认证服务器确认没有问题，返回一个token

## 参考文献
* [The OAuth 2.0 Authorization Framework](http://tools.ietf.org/html/rfc6749)
* [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
