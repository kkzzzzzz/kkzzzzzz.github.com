---
layout: post
title: "七周七并发之函数式编程"
description: ""
category:
tags: [并发,读书]
---
{% include JB/setup %}



## 概述

函数式编程有别于面向对象编程，它具有如下特性

* 函数是一等公民，可以把它当作参数传递给另外一个函数，也可以把函数作为返回值
* 函数式语言里面的数据是不可修改的
* 状态被维护在函数的参数上，而参数放在栈(stack)上面，不会被维护在全局的堆（heap）上面

其中特性2、3 能够很好的支持我们在函数级别的并行，而不带来副在用。


## Clojure Start
书中用Clojure这样一个基于JVM的语言举例，Clojure的名字由来也比较有意思
> "我想把这就几个元素包含在里面： C (C#), L (Lisp) and J (Java). 所以我想到了 Clojure, 而且从这个名字还能想到closure;它的域名又没有被占用;而且对于搜索引擎来说也是个很不错的关键词，所以就有了它了."


安装clojurescript

~~~bash
brew install clojurescript
~~~

安装leiningen

~~~bash
brew install leiningen
~~~

开始运行

~~~bash
lein repl
~~~

出现如下画面,表示安装成功

![](http://7xs9oq.com1.z0.glb.clouddn.com/ss22fd084ede8f39cfdcd2269f5e619ffe.png){:height="50%" width="100%"}

## 特性
Clojure具有如下特性

### 延迟计算
其中的元素仅在需要的时候才求值

~~~Clojure
user=> (range 0 10)
(0 1 2 3 4 5 6 7 8 9)
~~~
上面的数列会被快速求出

~~~Clojure
user=> (range 0 10000000)
~~~
这样就需要一段时间了

~~~clojure
user=> （take ((range 0 10000000))
~~~
但是这个只需要take前10个元素，就特别快了

### reduce

reduce函数，接受3个参数，1个化简函数，1个初始值和1个集合。
reduce为集合中的每一个元素调用一次化简函数

因此以前一个循环加可以改写为

~~~Clojrue
(defn reduce-sum [numbers]
	(reduce (fn [acc x] (+ acc x)) 0 numbers))
~~~
因为+操作是现成的函数，也就是直接写为

~~~Clojrue
(defn reduce-sum [numbers]
	(reduce + numbers))
~~~

### flod
但是reduce还是逐个对元素进行操作，进阶版本，可以对元素进行分而治之，分组处理再合并
类似的，支持flod的操作，需要首先支持reduce操作。


## 本周总结
![脑图](http://7xs9oq.com1.z0.glb.clouddn.com/ss2ec098db93ca23d7837638693a03b2ec.png){:height="50%" width="100%"}
