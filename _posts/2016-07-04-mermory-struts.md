---
layout: post
title: "jvm内存对象学习"
description: ""
category: [study]
tags: [java]
---
{% include JB/setup %}

今天在学习JVM内存模型中，学习到线程之间共享数据这块，涉及到主内存和工作内存之间数据的交换和线程之间数据共享的问题。其中说到数据原子性的时候，说在32位JVM中，如果对long型等64位数据进行读写时，会分成2次32位的读写操作，破坏原子性。

看到这里我就迷惑了，大学学习C+ +课程的时候，记得数据长度在不同机型上会有不一样，并不是绝对的，所以一段代码可能并不是放之四海而皆准的，和编译器有关。难道java里面所有的数据长度都是固定的和jvm无关？   

查询相关资料[https://en.wikibooks.org/wiki/Java_Programming/Primitive_Types]
发现对JAVA来说对，的确每个基本数据大小和长度是固定的，
例如double和long是64位的

顺便了解了一下一个实例化的对象在内存中存储的信息：

1. 对象的头部，包括分代年龄，锁记录，hashcode等
2. 实例数据，也就是对象的成员变量，包括父类继承的和子类定义的
3. 第三部分是对齐填充，比如要求数据的大小是8字节的整数倍


针对实例数据这块，也就是对象的内存布局可以展开讲一下

1. 空对象和类实例成员变量
	空对象，指的非内部类，没有实例属性的类。Object 类或者直接继承 Object 没有添加任何实例成员的类。
	空对象的不包含任何成员变量，其大小即对象头大小:

	• 在 32 位 JVM 上，占用 8 字节；
	• 在未开启 UseCompressedOops 的 64 位 JVM 上，16 字节。
	• 在开启 UseCompressedOops 的 64 位 JVM 上，12 + 4 = 16；

2. 对象实例成员重排序
	实例成员变量紧随对象头。每个成员变量都尽量使本身的大小在内存中尽量对齐。
	比如 int 按 4 位对齐，long 按 8 位对齐。为了内存紧凑，实例成员在内存中的排列和声明的顺序可能不一致，实际会按以下顺序排序:这样做可尽量节省空间。
	1. doubles and longs
	2. ints and floats
	3. shorts and chars
	4. booleans and bytes
	5. references

3. 父类和子类的实例成员
	父类和子类的成员变量分开存放，先是父类的实例成员。父类实例成员变量结束之后，按4位对齐，随后接着子类实例成员变量。

~~~java
class A {
   byte a;
}

class B extends A {
   byte b;
}
~~~

内存结构如下:

|	32 bit                   |64bit +UseCompressedOops|
|--------------------------|------------------------|
|[HEADER:  8 bytes]  8     |  [HEADER: 12 bytes] 12|
|[a:       1 byte ]  9     |  [a:       1 byte ] 13|
|[padding: 3 bytes] 12     |  [padding: 3 bytes] 16|
|b:        1 byte ] 13     |  [b:       1 byte ] 17|
|[padding: 3 bytes] 16     |  [padding: 7 bytes] 24|

如果子类首个成员变量是 long 或者 double 等 8 字节数据类型，而父类结束时没有 8 位对齐。会把子类的小于 8 字节的实例成员先排列，直到能 8 字节对齐。


~~~java
class A {
  byte a;
 }

 class B extends A{
    long b;
    short c;  
    byte d;
}
~~~

内存结构如下:


|	32 bit                 | 64bit +UseCompressedOops|
|------------------------|-------------------------|
|[HEADER:  8 bytes]  8   |    [HEADER:  8 bytes] 12|
|[a:       1 byte ]  9   |    [a:       1 byte ] 13|
|[padding: 3 bytes] 12   |    [padding: 3 bytes] 16|
|[c:       2 bytes] 14   |    [b:       8 bytes] 24|
|[d:       1 byte ] 15   |    [c:       4 byte ] 28|
|[padding: 1 byte ] 16   |    [d:       1 byte ] 29|
|[b:       8 bytes] 24   |    [padding: 3 bytes] 32|

上面的示例中，在 32 位的 JVM 上，B 的 2 个实例成员 c, d 被提前了。
