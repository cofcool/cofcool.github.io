---
layout: post
category : Development
title : ios学习系列18--推送通知
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第十八部分，关于通知推送的一些内容。

ios提供了两种推送机制：本地推送通知（Local Notification），远程推送通知（Remote Notification）。

推送通知的展示效果：

* 在屏幕顶部显示一块横幅（显示具体内容）
* 在屏幕中间弹出一个UIAlertView（显示具体内容）
* 在锁屏界面显示一块横幅（锁屏状态下，显示具体内容）
* 更新app图标的数字（说明新内容的数量）
* 播放音效（提醒作用）

具体的显示效果，用户可以在通知中心设置，用户也可设置是否开启。发出推送通知时，如果程序正运行在前台，那么推送通知就不会被呈现出来。点击推送通知后，默认会自动打开发出推送通知的app。不管app打开还是关闭，推送通知都能如期发出。

####1. 本地通知

本地通知：不需要联网就能发出的推送通知（不需要服务器的支持）。通知的使用场景：常用来定时提醒用户完成一些任务，比如清理垃圾、记账、买衣服、看电影、玩游戏等。

步骤：

	// 创建本地推送通知对象
	UILocalNotification *ln = [[UILocalNotification alloc] init];
	
	// 设置本地推送通知属性
	//推送通知的触发时间（何时发出推送通知）
	@property(nonatomic,copy) NSDate *fireDate;
	// 推送通知的具体内容
	@property(nonatomic,copy) NSString *alertBody;
	// 锁屏界面显示的小标题（完整小标题：“滑动来” + alertAction）
	@property(nonatomic,copy) NSString *alertAction;
	// 音效文件名
	@property(nonatomic,copy) NSString *soundName;
	// app图标数字
	@property(nonatomic) NSInteger applicationIconBadgeNumber;
	
	// 调度本地推送通知（调度完毕后，推送通知会在特地时间fireDate发出）
	[[UIApplication sharedApplication] scheduleLocalNotification:ln];
	
	// 获得被调度的所有本地推送通知(等待发出的通知,已经发出且过期的推送通知就算调度结束，会自动从这个数组中移除)
	@property(nonatomic,copy) NSArray *scheduledLocalNotifications;
	
	// 取消调度本地推送通知
	- (void)cancelLocalNotification:(UILocalNotification *)notification;
	- (void)cancelAllLocalNotifications;
	
	// 立即发出本地推送通知(使用价值：app在后台运行的时候)
	- (void)presentLocalNotificationNow:(UILocalNotification *)notification;
	
	// 每隔多久重复发一次推送通知
	@property(nonatomic) NSCalendarUnit repeatInterval;
	
	// 点击推送通知打开app时显示的启动图片
	@property(nonatomic,copy) NSString *alertLaunchImage;
	
	// 附加的额外信息
	@property(nonatomic,copy) NSDictionary *userInfo;
	
	// 时区（一般设置为[NSTimeZone defaultTimeZone] ，跟随手机的时区）
	@property(nonatomic,copy) NSTimeZone *timeZone;

点击本地通知：

当用户点击本地推送通知，会自动打开app，这里有2种情况：

1. app并没有关闭，一直隐藏在后台，让app进入前台，并会调用AppDelegate的下面方法（并非重新启动app）

		- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification;

2. app已经被关闭（进程已死），启动app，启动完毕会调用AppDelegate的下面方法：

		- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
		
   其中，launchOptions参数通过UIApplicationLaunchOptionsLocalNotificationKey取出本地推送通知对象。

####2. 远程推送通知

远程通知，就是从远程服务器推送给客户端的通知（需要联网），远程推送服务，又称为APNs（Apple Push Notification Services）。传统获取数据的局限性：只要用户关闭了app，就无法跟app的服务器沟通，无法从服务器上获得最新的数据内容。远程推送通知可以解决以上问题，不管用户打开还是关闭app，只要联网了，都能接收到服务器推送的远程通知。

**使用须知**：所有的苹果设备，在联网状态下，都会与苹果的服务器建立长连接。

**DeviceToken处理流程**

![image]({{ site.url }}/public/upload/images/ios_33.png)

![image]({{ site.url }}/public/upload/images/ios_34.png)

远程推送流程：

![image]({{ site.url }}/public/upload/images/ios_35.png)

![image]({{ site.url }}/public/upload/images/ios_36.png)

####1. 注册远程推送通知

客户端如果想接收APNs的远程推送通知，必须先注册（得到用户的授权），一般在App启动完毕后就马上注册。

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
	    // 注册远程通知
	    UIRemoteNotificationType type = UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound;
	    [application registerForRemoteNotificationTypes:type];
	    
	    return YES;
	}

注册成功后会调用AppDelegate的下面方法，得到设备的deviceToken

	- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
	{
	    NSLog(@"%@", deviceToken);
	}

####2. 接受远程推送通知

当设备接收到远程推送通知时，如果程序是处于关闭状态，系统会在给用户展示远程推送通知的同时，将程序启动到后台，并调用AppDelegate的下面方法：

	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler;

可以在这个方法中做些数据下载操作，争取在用户点击通知前，就将数据下载完毕，下载完毕要调用completionHandler这个block，告知下载完毕。

    completionHandler(UIBackgroundFetchResultNewData);

####3. 点击远程推送通知

当用户点击远程推送通知，会自动打开app，这里有2种情况：

1. app并没有关闭，一直隐藏在后台，让app进入前台，并会调用AppDelegate的下面方法（并非重新启动app）。

		- (void)application:(UIApplication *)application didReceiveRemoteNotification:(UILocalNotification *)notification;

2. app已经被关闭（进程已死），启动app，启动完毕会调用AppDelegate的下面方法。
 
		- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
		
   其中，launchOptions参数通过UIApplicationLaunchOptionsRemoteNotificationKey取出远程推送通知对象。
