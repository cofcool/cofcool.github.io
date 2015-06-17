---
layout: post
category : Development
title : ios学习系列7--触摸事件
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第六部分，关于触摸事件的一些内容。

ios的事件可分为三大类型：触摸事件，加速事件，远程事件。本章主要是触摸事件的相关内容。

1. 响应者对象

   在iOS中不是任何对象都能处理事件，只有继承了UIResponder的对象才能接收并处理事件。我们称之为“响应者对象”。UIApplication、UIViewController、UIView都继承自UIResponder，因此它们都是响应者对象，都能够接收并处理事件。
   
   UIResponder内部提供了以下方法来处理事件
   
   * 触摸事件
  
		 - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
		 - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
		 - (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
		 - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;

   * 加速计事件
  
		 - (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
		 - (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
		 - (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;

   * 远程控制事件
  
		 - (void)remoteControlReceivedWithEvent:(UIEvent *)event;

2. UIView的触摸事件处理

   UIView是UIResponder的子类，可以覆盖下列4个方法处理不同的触摸事件(touches中存放的都是UITouch对象)：

	  * 一根或者多根手指开始触摸view，系统会自动调用view的下面方法
	  
	        - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
	
	  * 一根或者多根手指在view上移动，系统会自动调用view的下面方法（随着手指的移动，会持续调用该方法）
	  
			- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
	
	  * 一根或者多根手指离开view，系统会自动调用view的下面方法
	
			- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event
	
	  * 触摸结束前，某个系统事件(例如电话呼入)会打断触摸过程，系统会自动调用view的下面方法
	  
	        - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event

3. UITouch

   当用户用一根触摸屏幕时，会创建一个与手指相关联的UITouch对象，该对象保存着跟手指相关的信息，比如触摸的位置、时间、阶段。当手指移动时，系统会更新同一个UITouch对象，使之能够一直保存该手指在的触摸位置，当手指离开屏幕时，系统会销毁相应的UITouch对象。

4. UIEvent

   每产生一个事件，就会产生一个UIEvent对象，UIEvent是事件对象，记录事件产生的时刻和类型。

5. 触摸事件

   **<1> 一次完整的触摸过程：**
   
		// 触摸开始
	    - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
		// 触摸移动
		- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
		// 触摸结束
		- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event
		// 触摸取消（可能会经历）
		- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event

   4个触摸事件处理方法中，都有touches和event两个参数,在一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数。根据touches中UITouch的个数可以判断出是单点触摸还是多点触摸。

   **<2> 事件的产生和传递**
    
   发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中。UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）。主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步，找到合适的视图控件后，就会调用视图控件的touches方法来作具体的事件处理。**响应者链条**示意图:

   ![image]({{ site.url }}/public/upload/images/ios_20.png)
   
   如果view的控制器存在，就传递给控制器；如果控制器不存在，则将其传递给它的父视图。在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理。如果window对象也不处理，则其将事件或消息传递给UIApplication对象。如果UIApplication也不能处理该事件或消息，则将其丢弃。

6. 手势识别

    为了完成手势识别，必须借助于手势识别器**UIGestureRecognizer**。利用UIGestureRecognizer，能轻松识别用户在某个view上面做的一些常见手势。UIGestureRecognizer是一个抽象类，定义了所有手势的基本行为，使用它的子类才能处理具体的手势：

	    UITapGestureRecognizer(敲击)
	    UIPinchGestureRecognizer(捏合，用于缩放)
	    UIPanGestureRecognizer(拖拽)
	    UISwipeGestureRecognizer(轻扫)
	    UIRotationGestureRecognizer(旋转)
	    UILongPressGestureRecognizer(长按)
  
   具体使用：
  
		// UITapGestureRecognizer的使用步骤如下：
		// 创建手势识别器对象
		UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] init];
		
		// 设置手势识别器对象的具体属性
		// 连续敲击2次
		tap.numberOfTapsRequired = 2;
		// 需要2根手指一起敲击
		tap.numberOfTouchesRequired = 2;
		
		// 添加手势识别器到对应的view上
		[self.iconView addGestureRecognizer:tap];
		
		// 监听手势的触发
		[tap addTarget:self action:@selector(tapIconView:)];

   手势识别状态变化示意图:
   
   ![image]({{ site.url }}/public/upload/images/ios_21.png)