---
layout: post
category : Development
title : ios学习系列15--蓝牙
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十五部分，关于蓝牙的一些内容。

### 蓝牙

iOS中提供了4个框架用于实现蓝牙连接

1. GameKit.framework（用法简单），只能用于iOS设备之间的连接，多用于游戏（比如五子棋对战），从iOS7开始过期。并且只能用于同一个应用程序之间的连接,因此最好别利用蓝牙发送比较大的数据.


2. MultipeerConnectivity.framework，只能用于iOS设备之间的连接，从iOS7开始引入，主要用于文件共享（仅限于沙盒的文件）。

3. ExternalAccessory.framework，可用于第三方蓝牙设备交互，但是蓝牙设备必须经过苹果MFi认证（国内较少）。

4. CoreBluetooth.framework（时下热门），可用于第三方蓝牙设备交互，必须要支持蓝牙4.0。硬件至少是4s，系统至少是iOS6。蓝牙4.0以低功耗著称，一般也叫BLE（Bluetooth Low Energy）。目前应用比较多的案例：运动手坏、嵌入式设备、智能家居。

#####1. GameKit的蓝牙开发步骤

	// 显示可以连接的蓝牙设备列表
	GKPeerPickerController *ppc = [[GKPeerPickerController alloc] init];
	ppc.delegate = self;
	[ppc show];
	
	// 在代理方法中监控蓝牙的连接
	- (void)peerPickerController:(GKPeerPickerController *)picker didConnectPeer:(NSString *)peerID toSession:(GKSession *)session 
	{
	    NSLog(@"连接到设备：%@", peerID);
	    // 关闭蓝牙设备显示界面
	    [picker dismiss];
	    // 设置接收到蓝牙数据后的监听器
	    [session setDataReceiveHandler:self withContext:nil];
	    // 保存session
	    self.session = session;
	}
	
	// 处理接收到的蓝牙数据
	- (void) receiveData:(NSData *)data fromPeer:(NSString *)peer inSession: (GKSession *)session context:(void *)context 
	{
	    // 处理
	}
	
	// 利用GKSession给其他设备发送数据
	// 给指定的连接设备发送数据
	- (BOOL)sendData:(NSData *) data toPeers:(NSArray *)peers withDataMode:(GKSendDataMode)mode error:(NSError **)error;
	
	// 给所有连接的设备发送数据
	- (BOOL)sendDataToAllPeers:(NSData *) data withDataMode:(GKSendDataMode)mode error:(NSError **)error;

####2. Core Bluetooth

Core Bluetooth测试比较麻烦，正常情况下，得至少有2台真实的蓝牙4.0设备。iOS模拟器测试蓝牙4.0程序：买一个CSR蓝牙4.0 USB适配器，插在Mac上；在终端输入**sudo nvram bluetoothHostControllerSwitchBehavior="never"**；重启Mac，用Xcode 4.6调试代码，将程序跑在iOS 6.1的模拟器上，（苹果把iOS 7.0模拟器对BLE的支持移除掉了）。使用场景：运动手环、智能家居、嵌入式设备等等（金融刷卡器、心电测量器）。

Core Bluetooth的核心结构图:

![image]({{ site.url }}/public/upload/images/ios_32.png)

每个蓝牙4.0设备都是通过服务（Service）和特征（Characteristic）来展示自己的，一个设备必然包含一个或多个服务，每个服务下面又包含若干个特征。特征是与外界交互的最小单位，比如说，一台蓝牙4.0设备，用特征A来描述自己的出厂信息，用特征B来收发数据。服务和特征都是用UUID来唯一标识的，通过UUID就能区别不同的服务和特征。设备里面各个服务(service)和特征(characteristic)的功能，均由蓝牙设备硬件厂商提供，比如哪些是用来交互(读写)，哪些可获取模块信息(只读)等。

开发步骤：

1. 建立中心设备
2. 扫描外设（Discover Peripheral）
3. 连接外设(Connect Peripheral)
4. 扫描外设中的服务和特征(Discover Services And Characteristics)
5. 利用特征与外设做数据交互(Explore And Interact)
6. 断开连接(Disconnect)
