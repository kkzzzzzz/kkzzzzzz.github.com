---
layout: post
title: "基于alfred的自动化图片处理实践总结"
description: ""
category: tools
tags: [markdown,workflow]
---
{% include JB/setup %}

在日常工作中，需要有一个比较好的图床，把自己的图片直接变成URL，这个时候结合alferd和[七牛云](https://portal.qiniu.com/)可以完美的达成目的!

***
操作步骤主要参考 [博文](https://www.zybuluo.com/fyywy520/note/317999)

其中有几点是自己在配置中遇到的问题

* QRSync的使用 配置conf.json 替换成自己的AccessKey 和 SecretKey 时 ，不要带上尖括号<>,否则你会报错如下

~~~json
{
  "error":"bad token",
  "reqid":"MBoAABzL4reGaj8U",
  "details":["UP:1/401"],
  "code":401
}
~~~

* 使用七牛云的时候，需要把你这个bucket设置为公开的，不然你自己的url，是无法访问的

* 在更改markdown image中的脚本的时候，看清楚路径，在注释"--这里换成你自己放置图片的路径"一行，需要改成路径为
```
    /Users/Kevin/qiniupic/Data/
```  而不是
```
    /Users/Kevin/qiniupic/Data
```
注意路径的地址
