---
layout: post
title: "linux command"
description: ""
category:
tags: []
---
{% include JB/setup %}

# awk

## 如何快速删除一个文件里面的重复行？

~~~shell
awk '!a[$0]++' 文件名
~~~

## 命令格式

~~~shell
awk [ -F field-separator ] [ -v var=value ] [ 'program' | -f progfile ] [ file ...  ]
~~~~

* 第一个为可选项 表示分隔符，默认是以空格b进行分割
* 第二个为可选项 表示初始变量的赋值，
* 第三个为可选项 表示编码的程序，支持从文件中读取 （-f）程序一般是Pattern { Action }这种格式，与指定模式（Pattern）相匹配，或包含与该模式匹配的字段，那么执行相应的操作（Action）。
* 第四个为可选项 表示从哪个文件中读取文件，当没有文件名时 ，接收标准输入


## 例子

### 实例1：打印文件中第一个字符
~~~shell
awk '{print  $2}' pay.txt
~~~~
简单输出，动作需要用{}包起来
### 实例2：简单判断输出
~~~shell
awk '$3 >0 { print $1, $2 * $3 }' pay.txy
~~~~
第三个变量大于0，则输出第一个变量，第二个和第三个变量的乘积

## 解释
~~~shell
'!a[$0]++'
~~~

我们这里写的 awk 命令是!a[$0]++，意思是，首先创建一个 map 叫a，然后用当前行的全文$0作为 map 的 key，
a[$0]第一次是获取到初始值为0，Parttern等于!0 即为真，默认输出当前记录，
++表示先取值，再加，也就是a[$0]的value++，则为1，
下次遇到$0的时候，就获取到了非0的value，Parttern等于!1 即为假，不输出。


# grep
## 命令功能
配合正则表达式，用于搜索特定字符。
配合其他命令可以产生强大的功能。
## 命令格式
~~~shell
  grep [OPTIONS] PATTERN [FILE...]
~~~
* 第一个参数为可选项 表示命令的选项
* 第二个参数为必选项 表示匹配的模式
* 第三个参数为可选项，文件名，当没有文件名时 ，接收标准输入

## 常用支持的参数

~~~shell
-a   --text   #不要忽略二进制的数据。   
-A<显示行数>   --after-context=<显示行数>   #除了显示符合范本样式的那一列之外，并显示该行之后的内容。   
-b   --byte-offset   #在显示符合样式的那一行之前，标示出该行第一个字符的编号。   
-B<显示行数>   --before-context=<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前的内容。
--color  #高亮匹配的内容  
-c    --count   #计算符合样式的列数。   
-C<显示行数>    --context=<显示行数>或-<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前后的内容。    
-f<规则文件>  --file=<规则文件>   #指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。   
-i    --ignore-case   #忽略字符大小写的差别。   
-n   --line-number   #在显示符合样式的那一行之前，标示出该行的列数编号。   
-r   --recursive   #此参数的效果和指定“-d recurse”参数相同。     
-w   --word-regexp   #只显示全字符合的列。   
-x    --line-regexp   #只显示全列符合的列。   
-y   #此参数的效果和指定“-i”参数相同。
~~~

## 例子
### 实例1：查找指定进程
~~~shell
ps -ef | grep java
~~~
查找java 进程

### 实例2：查找文件内某个字符上下文

~~~shell
grep "error" -A5 -B5 app.log
~~~

查找app.log文件中， 出现error字符的地方，并且打印上下5行
### 实例3：高亮字符
~~~shell
 grep "error" --color -A5 -B5 app.log
~~~

### 实例4：从文件中读取关键词进行搜索

~~~shell
cat dpidlist.txt | grep -f targetid.txt
~~~
从dpidlist.txt文件中，输出targetid.txt含有关键词的内容行

### 实例5：递归的从文件/目录中查找匹配的字符
~~~shell
grep "DTO" -r DP/cip-river-sand
~~~

从DP/cip-river-sand文件下查找 “DTO” 如果是目录，则查里面的文件以及目录，如果是文件，则匹配里面的内容

## 总结
~~~shell
grep 'word' 文件名
grep 'word' 文件1 文件2 文件3
grep '字符串1 字符串2'  文件名
cat 某个文件 | grep '某个东西'
command | grep '某个东西'
command 选项1 | grep '数据'
~~~
