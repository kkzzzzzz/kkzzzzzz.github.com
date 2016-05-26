---
layout: post
title: "disruptor学习笔记"
description: ""
category: [study]
tags: [disruptor]
---
{% include JB/setup %}



# disruptor学习分享

## 是什么？
LMAX在线交易出品的一个高效的无锁并发框架
它高效核心在于其无锁队列RingBuffer的独特设计。
它可以用来进行线程之间的数据交互。
## 老生重谈：锁
并发编程中，为了保证准确性，引入了锁的机制，包括乐观锁，悲观锁等。有锁就涉及到资源的竞争，竞争就可能出现死锁，这样的情况下，你只能重启你的机器了。
### 考虑一个简单自增的问题：
从1加到10亿，（测试机器 Mac Air）

* 单线程简单自增，耗时5S左右
* 单线程加锁自增，仅仅简单加锁，没有竞争，耗时40S左右，慢一个数量级

考虑并发

* 两个线程简单自增，耗时减少，但是结果无法保证准确性，比如存在脏读等问题。
* 两个线程加锁自增，按照理解，时间应该减半，但是因为引入了锁机制，导致竞争，实际时间更加长。

### disruptor怎么做

disruptor在需要保证线程安全的地方，用到了CAS操作，这是一个CPU级别的指令，类似于乐观锁，即Campare and Set/Swap. JAVA 从1.5版本新引入了AtomicLong等支持CAS指令的数据结构。
测试代码可以看出来，在引入了AtomicLong的情况下，单线程，耗时为18S左右，多线程，耗时26S左右。

 [测试代码](https://github.com/kkzzzzzz/java/blob/master/src/main/java/IncreaseLockSample/SingleThreadApp.java)

## 说说数据结构：链表 or 数组
既然是数据交换，那就存在一个产生数据（producer），一个消费数据(consumer)，和数据存储(RingBuffer)，数据存储，我们可以理解，应该是一个队列，数据先到先处理，在java类库中，提供了例如LinkedBlockingQueue、ArrayBlockingQueue等，而disruptor之所以高效，是因为它没有直接使用java类库中提供的队列，而是自己写的RingBuffer。

它有什么特点呢？

* 顾名思义，它是一个环，准确说，是一个用数组实现的环形队列。
* 不像传统队列，维护对头，队尾，它只有Sequencer,指向下一个可用的数据缓存区
* 新产生的数据对原来数据进行写覆盖，不进行remove操作。
* 队列大小一定是2的N次方

它有什么好处呢？

* 相比于链表，它寻址更快，时间复杂度控制在O（n）
* 在初始化时，就已经分配好了内存，而且新产生的只覆盖，所以更少的GC
* sequece 一直自增，进行位操作可以快速定位到实际slot , sequece & （array length－1） = array index，比如一共有8槽，9 &（8－1）= 1

[对比例子代码](https://github.com/kkzzzzzz/java/blob/master/src/main/java/DisruptorSample/CompareDemo.java)

## 说说硬件：缓存
### 伪共享
CPU和内存之间存在着多级缓存，我们都知道越靠近CPU的缓存越快，存储速度依次排列为L1,L2,L3,Memery,但是他们的存储空间大小依次排列为倒序，因此我们不能把所有数据都放在L1，我们需要把我们的数据从Memery中Load到cache里面，从而进行访问。
下图有一些数据：

![cpu与缓存](http://7xs9oq.com1.z0.glb.clouddn.com/ss54c13570c4f3b6c79954a948a5455575.png-960.jpg)

如果你想让你端到端的延时为10ms,那你mermery中load耗时80ns相对来说是一个比较重的操作。
更为严重的问题是，我们的数据在缓存中，不是独立项存储的，你可以想象缓存为一个阵列，由多个缓存行组成，缓存行大小根据机器不同，有差异，常见为64个字节。每次LRU（或者其他算法）的时候，它会把你目标数据的相邻数据也load进来，放入一整行的缓存中。

![load](http://7xs9oq.com1.z0.glb.clouddn.com/ssf92529d68a5968d432b021d912135c7f.png-960.jpg)

现在假设我们要操作A.B两个数据，他们正好在内存中是紧挨着的，线程1想要对数据A进行写入操作，它把A从Memery中load到L1来，相应的B也被免费的load到L1中。

![update](http://7xs9oq.com1.z0.glb.clouddn.com/ss6a7047e65a6f53b2eb5f128abfde9c7b.png-960.jpg)

现在线程1需要对A进行写入，同时线程2需要对B进行写入，他们需要争夺对这个缓存行的所有权，加入线程1成功对A进行了写入，那线程2需要对自己的缓存置为失效。
通过这样的一个方式，说明了两个不相关的线程，本来操作自己的数据，但是因为另外一个线程对自己数据的更改，导致自己的数据需要重新从Memery中load，这样会把自己的整体速度给拖慢。

这就是**伪共享**，因为每次你访问A的同时，你也会得到B,而且每次你访问B，同时你也会得到A。他们仿佛是一体的，但是实际没有任何关系。

### disruptor怎么做
引入缓存行填充机制，在RingBuffer中，需要有一个指向当前数据区的序列号（Sequencer），在有生成者和消费者对RingBuffer进行，数据读写的时候，我们对这个序列号进行缓存行填充机制，保证一个序列号在内存中，占有一个缓冲行。
![源码](http://7xs9oq.com1.z0.glb.clouddn.com/ssf0f3cb68d1921ae442ed9f9c3eb0ca5f.png-960.jpg)
通过代码演示，我们也可以看到的确存在差异，而且随着读写的线程变多，这样的差距越大。

[false share code](https://github.com/kkzzzzzz/java/blob/master/src/main/java/DisruptorSample/FalseSharing.java)
官方给到的代码，没有进行完全填充（也就是没有沾满一个缓存行），我自己写的例子有进行改进。

## 等待策略
在典型的消费者/生成者模型中，会存在等待现象，disruptor提供了以下的几种等待策略：

* BlockingWaitStrategy 是最低效的策略，但其对CPU的消耗最小并且在各种不同部署环境中能提供更加一致的性能表现
* SleepingWaitStrategy 的性能表现跟 BlockingWaitStrategy 差不多，对 CPU 的消耗也类似，但其对生产者线程的影响最小，适合用于异步日志类似的场景；
* YieldingWaitStrategy 的性能是最好的，适合用于低延迟的系统。在要求极高性能且事件处理线数小于 CPU 逻辑核心数的场景中，推荐使用此策略；例如，CPU开启超线程的特性。
经测试前两种效果差不多,延迟在微秒以内,可以忽略,cpu占用不高,YieldingWaitStrategy模式队列空闲时CPU达到100%,不适合

## Demo

* 简单例子
直接使用ringbuffer

	public static void main(String[] args) {
		int size = 1<<10;
		ExecutorService executors = Executors.newCachedThreadPool();
		//创建一个disruptor，指定ringbuffer的size和处理数据的factory
		Disruptor<TestObject> disruptor = new Disruptor<TestObject>(new TestObjectFactory(), size, executors);
		//disruptor里面设置一个处理方式
		disruptor.handleEventsWith(new TestObjectHandler());
		RingBuffer<TestObject> ringBuffer = disruptor.start();
		for (long i = 0; i < 1000; i++) {
		//下一个可以用的序列号
			long seq = ringBuffer.next();
			try {
				//这个序列号的slot 放入数据
				TestObject valueEvent = ringBuffer.get(seq);
				valueEvent.setValue(i);
			} finally {
				//发布通知，并且这一步一定要放在finally中，因为调用了ringBuffer.next(),就一定要发布，否则会导致disruptor状态的错乱
				ringBuffer.publish(seq);
			}
		}
		disruptor.shutdown();
		executors.shutdown();
	}

* 复杂例子
![多消费者](http://7xs9oq.com1.z0.glb.clouddn.com/ss101c2398bb10ddd4afea9d0ce5b6d452.png-960.jpg)
这个例子中，我们需要有不同的消费者，并且有些消费者之间存在依赖关系，有些消费者之间可以并行处理。

	public static void main(String[] args) throws InterruptedException {
		long beginTime=System.currentTimeMillis();
		int bufferSize=4;
		ExecutorService executor= Executors.newFixedThreadPool(10);//大于consumer的数量

		Disruptor<TestObject> disruptor = new Disruptor<TestObject>(new TestObjectFactory(), bufferSize, executor, ProducerType.SINGLE, new BusySpinWaitStrategy());
		//使用disruptor创建消费者AnalysisHandler,CalcHandler，两个可以并行执行
		EventHandlerGroup<TestObject> handlerGroup=disruptor.handleEventsWith(new TestObjectAnalysisHandler(),new TestObjectCalcHandler());
		//声明在AnalysisHandler,CalcHandler完事之后执行NotifyHandler
		EventHandlerGroup<TestObject> then = handlerGroup.then(new TestObjectNotifyHandler());
		//最终调用写入DB的handler,这里有启用多个线程，进行数据的写入
		then.thenHandleEventsWithWorkerPool(new TestObjectDBHandler(),new TestObjectDBHandler());
		//上面的也可以直接通过链式调用
		//disruptor.handleEventsWith(new TestObjectAnalysisHandler(),new TestObjectCalcHandler()).then(new TestObjectNotifyHandler()).thenHandleEventsWithWorkerPool(new TestObjectDBHandler(),new TestObjectDBHandler());
		disruptor.start();//启动
		CountDownLatch latch=new CountDownLatch(1);
		//生产者准备
		executor.submit(new TestObjectPublisher(latch, disruptor));
		latch.await();//等待生产者完事.
		disruptor.shutdown();
		executor.shutdown();
		System.out.println("总耗时:"+(System.currentTimeMillis()-beginTime));
	}

[代码地址](https://github.com/kkzzzzzz/java/blob/master/src/main/java/DisruptorSample/ComplexDemo.java)
## 应用场景:
个人思考下来，它适合一切异步环境，但是对于并发量小的场景不一定需要。在log4j2中，已经使用了disruptor进行日志记录。同样是用异步，选择disruptor会更快。

1. 在一些获取验证码，发短信的场景下，对实时性要求不够，如果收不到，用户可以再次要求重发。
2. 对于一些奖品，卡券的发放，在高峰期，可以只入队，在之后用异步的方式慢慢发放。
3. 对于比较复杂的逻辑可以进行并发操作

## 总结：
disruptor作为一个高并发框架，从CPU层面对整个代码进行优化。具有如下特点

1. 队列使用数组结构，而不是使用传统的链表结构，寻址更快
2. 新生产的对象采用覆盖的方式(不是传统阻塞队列，删除->添加的逻辑)，减少GC回收的负担
3. 从CPU层面优化，对Sequencer进行内存分配补齐，消除Java伪共享(cpu缓存行)
4. 多个线程同时访问，由于他们都通过序号器Sequencer访问ringBuffer，通过CAS取代了加锁和同步块，这也是并发编程的一个指导性原则：把同步块最小化到一个变量上。

## 参考文献
* [False Sharing](http://mechanical-sympathy.blogspot.com/2011/07/false-sharing.html)
* [Getting-Started](https://github.com/LMAX-Exchange/disruptor/wiki/Getting-Started)

## Q&A
* Q1:通过Sequencer产生的序列号，一直处于自增状态，是否会爆掉
* A1:[不会，long的范围最大可以达到9223372036854775807，一年365 * 24 * 60 * 60 = 31536000秒，每秒产生1W条数据，也可以使用292年](https://github.com/LMAX-Exchange/disruptor/issues/154#event-671348970)
