---
layout: page
title: 博客首页
tagline: 喜欢搞点事情
---
{% include JB/setup %}




{% for post in site.posts %}
<div id="article" class="well">
	<div id="article-header">
			<h3><a href="{{post.url}}">{{ post.title }}</a></h3>
		</div>
		<div id="content">
			<p class="text-indent">
				{{ post.content | strip_html | truncate : 300 }}
			</p>
		</div>
		<div id="article-footer">
			<p class="text-right">发布日期: {{ post.date | date : '%Y-%m-%d' }}</p>
		</div>
	</div>
{% endfor %}
