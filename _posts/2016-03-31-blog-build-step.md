---
layout: post
title: "基于github的博客搭建过程"
description: ""
category:
tags: []
---
{% include JB/setup %}



本来打算这做第一篇博文的，但是之前有碰到一些继续分享记录的东西，这个就拖了拖~下面进入正题吧

------

一直想要搭建一个博客，尝试过hexo，台湾人编写的，基于nodejs的静态博客，也很方便，但是需要自己的云主机，维护起来比较麻烦。后面发现全球最大的同性交友网站也支持博客搭建

1. 基于git的操作符合一般工作流
2. 支持markdown
3. 部署在github上，宕机可能性小

基于以上考虑，选择了github，下面开始详细步骤

## 1、环境准备
本文是基于mac的教程，如果你是win，出门左拐，默认大家已经做好了以下准备

* ruby (ruby -v检测，一般mac自带)
* github账号
* jekyll（gem install jekyll）

## 2、创建博客repository
在gitbub上创建自己的repositoy，提供企业和个人的两种


| 博客种类 | 默认域名 | 发布的分支 |
| ------------ | ------------- | ------------ |
| 用户个人博客 | username.github.io | master |
| 企业博客 | orgname.github.io  | master |
| 用户的某个项目的博客 | username.github.io/projectname  | 	gh-pages |
| 企业的某个项目博客 | orgname.github.io/projectname  | 	gh-pages |

一般我们开发自己博客，选第一种就行，例如
![](http://7xs9oq.com1.z0.glb.clouddn.com/ssc3d89561c1fda55d54e9c6bda868727f.png-960.jpg)
(之后默认以第一种做说明)

在项目的setting中找到GitHub Pages 按照步骤一步一步 就可以把你的网站建立起来了，到了这一步 访问你的网站应该是ok的了（username.github.io），但是这里用到的主题或者样式，是没有什么意义的，之后我们追求自己的美观，主题会全部换掉。

## 3、搭建本地预览模式
在本地写好文档之后，需要预览一下的，这个时候我们需要保证自己的环境和github的发布环境一致，参照官网，我们做如下配置

* gem install bundler
* add Gemfile  in root
Gemfile 内容如下

 >source 'https://rubygems.org'
>
>gem 'github-pages'

* bundle exec jekyll build —safe(可能缺少 github-pages执行 gem install github-pages)
* 配置_config.yml 文件 其中highlighter: rouge为比较重要的配置项

到此步基本就配置好了本地开发环境。执行如下命令

~~~
bundle exec jekyll serve
~~~  

然后访问

	http://localhost:4000

就可以在本地看到你的博客了



## 4、穿个衣服（找个自己喜欢的主题）

你已经可以用命令行跑一个

~~~shell
jekyll new YOUR-JEKYLL-SITE
~~~
然后直接就可以看到一个大概的目录结构了，其中_posts是之后我们文章主要存放的地方， index.md则是我们的首页了

这个时候你有两种选择，一种是继续用这样的主题，一种是自己选个，博主是用Jekyll-Bootstrap,把它代码统统弄到我们自己的项目中

~~~
 git clone https://github.com/plusjade/jekyll-bootstrap.git codingtiger.github.com
        cd codingtiger.github.com
        git remote set-url origin git@github.com:codingtiger/codingtiger.github.com.git
        git push origin master
~~~

然后你也可以安装一些自己的喜欢的主题
参考[主题安装与切换](http://jekyllbootstrap.com/usage/jekyll-theming.html)


安装rank命令可以快速进行写作，如

~~~
rake post title="Hello World"
~~~

注意title不支持中文，所以给你的博客写一个因为名字吧。然后在你的文章中改title
![](http://7xs9oq.com1.z0.glb.clouddn.com/ss1f119b885b5c1e1c9ee9ad521983c08d.png-960.jpg)
这样你的文章就是显示中文名字了。
