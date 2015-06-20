---
layout: post
category : Development
title : ios学习系列14--传感器
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十四部分，关于传感器的一些内容。

### 传感器

传感器是一种感应\检测装置, 目前已经广泛应用于智能手机上。传感器的作用：用于感应\检测设备周边的信息。不同类型的传感器, 检测的信息也不一样。iPhone5中内置的传感器有：运动传感器\加速度传感器\加速计（Motion/Accelerometer Sensor），环境光传感器（Ambient Light Sensor），距离传感器（Proximity Sensor），磁力计传感器（Magnetometer Sensor），内部温度传感器（Internal Temperature Sensor），湿度传感器（Moisture Sensor），陀螺仪（Gyroscope）••••••

####1. 加速计

在iOS5以前：使用UIAccelerometer，用法非常简单，从iOS5开始：CoreMotion.framework。虽然UIAccelerometer已经过期，但由于其用法极其简单，很多程序里面都还有残留。Core Motion不仅能够提供实时的加速度值和旋转速度值，更重要的是，苹果在其中集成了很多算法。

使用：

	// 获得单例对象
	UIAccelerometer *accelerometer = [UIAccelerometer sharedAccelerometer];
	
	// 设置代理
	accelerometer.delegate = self;
	
	// 设置采样间隔
	accelerometer.updateInterval = 1.0/30.0; // 1秒钟采样30次
	
	// 实现代理方法
	// acceleration中的x、y、z三个属性分别代表每个轴上的加速度
	- (void)accelerometer:(UIAccelerometer *)accelerometer didAccelerate:(UIAcceleration *)acceleration
	
Core Motion获取数据的两种方式：push，实时采集所有数据，采集频率高；pull，在有需要的时候，再去主动采集数据。具体使用：

1. push

		// 创建运动管理者对象
		CMMotionManager *mgr = [[CMMotionManager alloc] init];
		
		// 判断加速计是否可用（最好判断）
		if (mgr.isAccelerometerAvailable) {
		    // 加速计可用
		}
		
		// 设置采样间隔
		mgr.accelerometerUpdateInterval = 1.0/30.0; // 1秒钟采样30次
		
		// 开始采样（采样到数据就会调用handler，handler会在queue中执行）
		- (void)startAccelerometerUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMAccelerometerHandler)handler;

2. pull

		// 创建运动管理者对象
		CMMotionManager *mgr = [[CMMotionManager alloc] init];
		
		// 判断加速计是否可用（最好判断）
		if (mgr.isAccelerometerAvailable) {
		     // 加速计可用 
		 }
		 
		// 开始采样
		- (void)startAccelerometerUpdates;
		
		// 在需要的时候采集加速度数据
		CMAcceleration acc = mgr.accelerometerData.acceleration;
		NSLog(@"%f, %f, %f", acc.x, acc.y, acc.z);


####2. 距离传感器

	// 开启距离感应功能
	[UIDevice currentDevice].proximityMonitoringEnabled = YES;
	// 监听距离感应的通知
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(proximityChange:) name:UIDeviceProximityStateDidChangeNotification object:nil];
	
	- (void)proximityChange:(NSNotificationCenter *)notification {
	    if ([UIDevice currentDevice].proximityState == YES) {
	        NSLog(@"某个物体靠近了设备屏幕"); // 屏幕会自动锁住
	    } else {
	        NSLog(@"某个物体远离了设备屏幕"); // 屏幕会自动解锁
	    }
	}
