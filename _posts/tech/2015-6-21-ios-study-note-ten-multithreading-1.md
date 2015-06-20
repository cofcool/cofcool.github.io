---
layout: post
category : Development
title : ios学习系列10--多线程技术（一）
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十部分，关于多线程的一些内容。

### 多线程技术

1. 进程：进程是指在系统中正在运行的一个应用程序，每个进程之间是独立的，每个进程均运行在其专用且受保护的内存空间内。

2. 线程：线程是进程的基本执行单元，一个进程（程序）的所有任务都在线程中执行。1个进程要想执行任务，必须得有线程（每1个进程至少要有1条线程）。1个线程中任务的执行是串行的，如果要在1个线程中执行多个任务，那么只能一个一个地按顺序执行这些任务。也就是说，在同一时间内，1个线程只能执行1个任务。

3. 多线程：1个进程中可以开启多条线程，每条线程可以并行（同时）执行不同的任务。进程就像车间，线程像车间工人，线程在进程中执行任务。多线程技术可以提高程序的执行效率。

   **原理**：同一时间，CPU只能处理1条线程，只有1条线程在工作。多线程并发执行，其实是CPU快速地在多条线程之间调度。如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象。但是线程非常多时，CPU会在N多线程之间调度，CPU会累死，消耗大量的CPU资源，另外每条线程被调度执行的频次会降低，也就是线程的执行效率降低。
   
   **优点**：提高程序的执行效率，提高资源利用率（CPU、内存利用率等）
   
   **缺点**：开启线程需要占用一定的内存空间（默认情况下，主线程占用1M，子线程占用512KB），如果开启大量的线程，会占用大量的内存空间，降低程序的性能；线程越多，CPU在调度线程上的开销就越大；程序设计更加复杂：比如线程之间的通信、多线程的数据共享，增加了开发成本。

4. iOS中多线程的实现方案：

   ![image]({{ site.url }}/public/upload/images/ios_25.png)
   
5. NSThread

   **<1> 创建和启动线程**
   
		NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
		[thread start];
		
		//主线程相关用法
		+ (NSThread *)mainThread; // 获得主线程
		- (BOOL)isMainThread; // 是否为主线程
		+ (BOOL)isMainThread; // 是否为主线程

   **<2> 用法**
   
		// 获得当前线程
		NSThread *current = [NSThread currentThread];

		// 线程的调度优先级
		// 调度优先级的取值范围是0.0 ~ 1.0，默认0.5，值越大，优先级越高
		+ (double)threadPriority;
		+ (BOOL)setThreadPriority:(double)p;
		- (double)threadPriority;
		- (BOOL)setThreadPriority:(double)p;

		// 线程的名字
		- (void)setName:(NSString *)n;
		- (NSString *)name;

   **<3> 其他创建线程方式** 优点：简单快捷;缺点：无法对线程进行更详细的设置。
   
		// 创建线程后自动启动线程
		[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];

		// 隐式创建并启动线程
		[self performSelectorInBackground:@selector(run) withObject:nil];

   **<4> 线程的状态**
   
   ![image]({{ site.url }}/public/upload/images/ios_26.png)
   
   控制线程的状态：
   
		// 启动线程
		// 进入就绪状态 -> 运行状态。当线程任务执行完毕，自动进入死亡状态
		- (void)start; 

		// 阻塞（暂停）线程,进入阻塞状态
		+ (void)sleepUntilDate:(NSDate *)date;
		+ (void)sleepForTimeInterval:(NSTimeInterval)ti;

		// 强制停止线程,进入死亡状态,一旦线程停止了，就无法再次开启任务另外
		+ (void)exit;

   **<5> 多线程的安全隐患** 同一资源可能会被多个线程共享，也就是多个线程可能会访问同一块资源，如多个线程访问同一个对象、同一个变量、同一个文件。当多个线程访问同一资源时，很容易引发数据错乱和数据安全问题。
   
   安全隐患解决：**互斥锁**。
   
   **互斥锁使用格式**：@synchronized(锁对象) { // 需要锁定的代码  }。**注意：锁定1份代码只用1把锁，用多把锁是无效的**
   
   **优点**：能有效防止因多线程抢夺资源造成的数据安全问题
   
   **缺点**：需要消耗大量的CPU资源
   
   **互斥锁的使用前提**：多条线程抢夺同一块资源
   
   **线程同步**：多条线程按顺序地执行任务，互斥锁，就是使用了线程同步技术。

   **<6> 原子和非原子属性** OC在定义属性时有nonatomic和atomic两种选择:**atomic**：原子属性，为setter方法加锁（默认就是atomic）;**nonatomic**：非原子属性，不会为setter方法加锁。
   
		// atomic加锁原理
		@property (assign, atomic) int age;
		- (void)setAge:(int)age
		{
		    @synchronized(self) {
		        _age = age;
		    }
		}

   **nonatomic和atomic对比**:atomic：线程安全，需要消耗大量的资源;nonatomic：非线程安全，适合内存小的移动设备。在ios开发中，尽量避免使用多线程，把加锁、资源抢夺的业务逻辑交给服务器端处理，减小移动客户端的压力。因此，所有属性声明为**nonatomic**。
   
   **<7> 线程之间通信** 在1个进程中，线程往往不是孤立存在的，多个线程之间需要经常进行通信。线程间通信的体现：1个线程传递数据给另1个线程；在1个线程中执行完特定任务后，转到另1个线程继续执行任务。
   
		// 线程间通信常用方法
		- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
		- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;


6. GCD：全称是Grand Central Dispatch，纯C语言，提供了非常多强大的函数。

   **<1> 优势**
   
	1. GCD是苹果公司为多核的并行运算提出的解决方案
	2. GCD会自动利用更多的CPU内核（比如双核、四核）
	3. GCD会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
	4. 程序员只需要告诉GCD想要执行什么任务，不需要编写任何线程管理代码

   **<2>  任务和队列** GCD中有2个核心概念，任务：执行什么操作；队列：用来存放任务。GCD的使用2个步骤：定制任务，确定想做的事情。将任务添加到队列中，GCD会自动将队列中的任务取出，放到对应的线程中执行。任务的取出遵循队列的FIFO原则：先进先出，后进后出。

   **<3> 执行任务** 同步和异步的区别：同步：在当前线程中执行；异步：在另一条线程中执行。

   
		// 用同步的方式执行任务
		// queue：队列
		// block：任务
		dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
		
		// 用异步的方式执行任务
		dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
  
   **<4> 队列** GCD的队列可以分为2大类型:并发队列，串行队列。
   
   ![image]({{ site.url }}/public/upload/images/ios_27.png)
   
   1. 并发队列（Concurrent Dispatch Queue）
   
      * 可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）
      
      * 并发功能只有在异步（dispatch_async）函数下才有效
      
      GCD默认已经提供了全局的并发队列，供整个应用使用，不需要手动创建。
    
			// 使用dispatch_queue_create函数创建串行队列
			// 创建,第一个参数为队列名称，第二个参数为队列属性，一般用NULL
			dispatch_queue_t queue = dispatch_queue_create("net.cofcool.queue", NULL);
			// 非ARC需要释放手动创建的队列
			dispatch_release(queue); 
			
			// 使用主队列（跟主线程相关联的队列）
			// 主队列是GCD自带的一种特殊的串行队列
			// 放在主队列中的任务，都会放到主线程中执行
			// 使用dispatch_get_main_queue()获得主队列
			dispatch_queue_t queue = dispatch_get_main_queue();


    2. 串行队列（Serial Dispatch Queue）
     
       * 让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务
       
   **<5> 线程间通信** 
   
		// 从子线程回到主线程
		dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
		    // 执行耗时的异步操作...
            dispatch_async(dispatch_get_main_queue(), ^{
                // 回到主线程，执行UI刷新操作
            });
        });

   **<6> 延时执行** 
   
   iOS常见的延时执行有2种方式：
   
   1. 调用NSObject的方法
   
			// 2秒后再调用self的run方法   
			[self performSelector:@selector(run) withObject:nil afterDelay:2.0];


   2. 使用GCD函数
   
			dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
			    // 2秒后异步执行这里的代码...
			});

   3. 一次性代码
   
			// 使用dispatch_once函数能保证某段代码在程序运行过程中只被执行1次
			static dispatch_once_t onceToken;
			dispatch_once(&onceToken, ^{
			    // 只执行1次的代码(这里面默认是线程安全的)
			});

   **<7> 队列组** 分别异步执行2个耗时的操作，等2个异步操作都执行完毕后，再回到主线程执行操作。可以使用队列组来实现：
   
		dispatch_group_t group =  dispatch_group_create();
		dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
		    // 执行1个耗时的异步操作
		});
		dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
		    // 执行1个耗时的异步操作
		});
		dispatch_group_notify(group, dispatch_get_main_queue(), ^{
		    // 等前面的异步操作都执行完毕后，回到主线程...
		});

