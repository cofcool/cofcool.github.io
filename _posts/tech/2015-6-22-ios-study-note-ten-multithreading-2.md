---
layout: post
category : Development
title : ios学习系列10--多线程技术（二）
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十部分，关于多线程的一些内容。

### 多线程技术

1. 单例模式

   永远只分配一块内存来创建对象,提供一个类方法, 返回内部唯一的一个对象(一个实例),最好保证init方法也只初始化一次。

   **<1> 作用** 可以保证在程序运行过程，一个类只有一个实例，而且该实例易于供外界访问，从而方便地控制了实例个数，并节约系统资源。

   **<2> 使用场合** 在整个应用程序中，共享一份资源（这份资源只需要创建初始化1次）
   
   **<3> 使用** 单例模式在ARC\MRC环境下的写法有所不同，需要编写2套不同的代码，可以用宏判断是否为ARC环境：
   
		#if __has_feature(objc_arc)
		// ARC
		#else
		// MRC
		#endif

   
   1. ARC 
   
      * 在.m中保留一个全局的static的实例
    
            static id _instance;

      * 重写allocWithZone:方法，在这里创建唯一的实例（注意线程安全）
    
			+ (id)allocWithZone:(struct _NSZone *)zone
			{
			    @synchronized(self) {
			        if (!_instance) {
			            _instance = [super allocWithZone:zone];
			        }
			    }
			    return _instance;
			}

      * 提供1个类方法让外界访问唯一的实例
      
			+ (instancetype)sharedSoundTool
			{
			    @synchronized(self) {
			        if (!_instance) {
			            _instance = [[self alloc] init];
			        }
			    }
			    return _instance;
			}

   2. 非ARC
     
      * 实现copyWithZone:方法
      
			+ (id)copyWithZone:(struct _NSZone *)zone
			{
			    return _instance;
			}

      * 实现内存管理方法
      
			- (id)retain { return self; }
			- (NSUInteger)retainCount { return 1; }
			- (oneway void)release {}
			- (id)autorelease { return self; }

2. NSOperation

   NSOperation是个抽象类，并不具备封装操作的能力，必须使用它的子类。使用NSOperation子类的方式有3种：NSInvocationOperation、NSBlockOperation、自定义子类继承NSOperation，实现内部相应的方法。

   **<1> NSInvocationOperation** 
   
		// 创建NSInvocationOperation对象
		- (id)initWithTarget:(id)target selector:(SEL)sel object:(id)arg;
		
		// 调用start方法开始执行操作,一旦执行操作，就会调用target的sel方法
		- (void)start;


   **注意**：默认情况下，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作，只有将NSOperation放到一个NSOperationQueue中，才会异步执行操作。

   **<2> NSBlockOperation**
   
		// 创建NSBlockOperation对象
		+ (id)blockOperationWithBlock:(void (^)(void))block;
		
		// 通过addExecutionBlock:方法添加更多的操作
		- (void)addExecutionBlock:(void (^)(void))block;

   **注意**：只要NSBlockOperation封装的操作数 > 1，就会异步执行操作。

   **<3> NSOperationQueue** NSOperation可以调用start方法来执行任务，但默认是同步执行的。如果将NSOperation添加到NSOperationQueue（操作队列）中，系统会自动异步执行NSOperation中的操作。
   
		// 添加操作到NSOperationQueue中
		- (void)addOperation:(NSOperation *)op;
		- (void)addOperationWithBlock:(void (^)(void))block;

   **<4> 最大并发数** 同时执行的任务数。比如，同时开3个线程执行3个任务，并发数就是3。

		// 最大并发数的相关方法
		- (NSInteger)maxConcurrentOperationCount;
		- (void)setMaxConcurrentOperationCount:(NSInteger)cnt;

   **<5> 队列的取消、暂停、恢复** 
   
		// 取消队列的所有操作,也可以调用NSOperation的- (void)cancel方法取消单个操作
		- (void)cancelAllOperations;
		
		// 暂停和恢复队列
		- (void)setSuspended:(BOOL)b; // YES代表暂停队列，NO代表恢复队列
		- (BOOL)isSuspended;

   **<6> 操作优先级** 
   
		// 设置NSOperation在queue中的优先级，可以改变操作的执行顺序
		- (NSOperationQueuePriority)queuePriority;
		- (void)setQueuePriority:(NSOperationQueuePriority)p;

   > // 优先级的取值（优先级越高，越先执行） 
   
   > NSOperationQueuePriorityVeryLow = -8L,
   
   > NSOperationQueuePriorityLow = -4L,
   
   > NSOperationQueuePriorityNormal = 0,
   
   > NSOperationQueuePriorityHigh = 4,
   
   > NSOperationQueuePriorityVeryHigh = 8

   **<7> 操作依赖** NSOperation之间可以设置依赖来保证执行顺序。可以在不同queue的NSOperation之间创建依赖关系(不能互相依赖)。在任务执行中，先满足依赖关系，然后再从NSOperation中选择优先级最高的那个执行。比如一定要让操作A执行完后，才能执行操作B，可以这么写：
   
       // 操作B依赖于操作A
       [operationB addDependency:operationA]; 

   **<8> 自定义NSOperation** 重写- (void)main方法，在里面实现想执行的任务。**注意点：自己创建自动释放池（因为如果是异步操作，无法访问主线程的自动释放池）；经常通过- (BOOL)isCancelled方法检测操作是否被取消，对取消做出响应。**