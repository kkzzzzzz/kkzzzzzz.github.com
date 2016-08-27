---
layout: post
title: "threadlocal学习笔记"
description: ""
category:
tags: []
---
{% include JB/setup %}



## 概述

threadlocal 字面意思，线程本地，进一步理解，即线程持有的，每个线程保存自己线程的变量值，相互之间不会影响。所以threadlocal其实就是一个特殊类型的变量，可以理解为线程类的变量。

每个线程维护自己的变量，这需要占用更多的存储空间，但是相应的，对于这个变量，线程之间不需要去关注协同操作带来的同步问题，是一种比较典型的通过空间换却时间的方案，但是这种变量仅限于，每个线程之间的操作相互不影响的。


## 用法

它主要有几下几个方法

### 初始化

Returns the current thread's "initial value" for this thread-local variable.

~~~java
protected T	initialValue()
~~~

这是设定一个变量的初始值，举例说明：

~~~java

public class LocalValue{

    private static final ThreadLocal<Long> TIME_THREADLOCAL  = new ThreadLocal<Long>(){
        @Override
        protected Long initialValue() {
            return System.currentTimeMillis();
        }
    };
}

~~~

该方法为一个protected的方法，就是希望大家在初始化的时候，直接重写它，给你的变量一个初始值。
例子中LocalValue类定义了一个threadlocal的变量（支持泛型，因此你可以给你的这个类绑定任何你想放入的数据）
从这里看出来，它的定义和我们一般的定义是及其相似的

~~~java

public class ShareValue {

	private static Long shareValue = 10L;
}

~~~

通过这样的一个方式，我们可以初始化一个threadlocal的变量，并且可以设定它的默认值。

这里注意到我们设置这个值用到了 **static** 关键字，因为这个值本身定位就是被多个线程访问。



### 获取


Returns the value in the current thread's copy of this thread-local variable.

~~~java
T	get()
~~~
这是一个无参数的方法，到这里我们就纳闷了，这是怎么获得值的呢？

参看源码

~~~java

/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
	 //原来这里拿到了访问线程
    Thread t = Thread.currentThread();

    //通过线程拿到了一个TreadLoaclMap的内部类
    ThreadLocalMap map = getMap(t);

    if (map != null) {
    	  //这里的Entry类似HashMap的实现，是一个数组
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}    
~~~
通过源码，我们可以知道，原来TreadLocal是有一个TreadLocalMap的内部类来维护各个线程的内部变量值，而ThreadLocalMap 是绑定在Thread.threadLocals 变量上的，如果没有则为null，如果有值则在 initialValues时调用createMap方法给它赋值

~~~java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
~~~


### 移除

Removes the current thread's value for this thread-local variable.

~~~jvav
void	remove()
~~~
估计是老样子，直接拿线程移除

~~~java
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }
~~~

### 设置新值

Sets the current thread's copy of this thread-local variable to the specified value.

~~~
void	set(T value)
~~~
直接看set的核心代码

~~~java
private void set(ThreadLocal key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    // 这里用了一个与操作，简单粗暴的定位放置的slot
    int i = key.threadLocalHashCode & (len-1);

	 // 这个slot里面没有，就用线性探测的方法，去找下一个slot，这里解决hash冲突的方法是开放地址法，区别于hashMap的单链表法
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
~~~

## 思考
通过用法的分析，我们可能会觉得就是ThreadLocal里面维护的一个map，用当前线程作为key，来存储不同线程线程的value，直接用map也可以实现，但是这里有两个问题

1. 直接使用map，作为类的成员变量，不可避免的有多线程问题。而这时threadLocal所避免的
2. 用Thread当key，区别于WeakReference ,除非手动调用remove，否则即使线程退出,也无法回收

## 总结

1. 在一个类中使用了static成员变量的时候，一定要考虑，多个线程需要独享自己的 static 成员变量吗？如果需要考虑，那就请用 ThreadLocal 吧！
2. 如果有多个值需要使用，设置多个ThreadLocal变量，因为一个ThreadLocal只可以持有一个值
