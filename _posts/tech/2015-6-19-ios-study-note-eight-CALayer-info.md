---
layout: post
category : Development
title : ios学习系列8--CALayer
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第八部分，关于CALayer的一些内容。


> The CALayer class manages image-based content and allows you to perform animations on that content. Layers are often used to provide the backing store for views but can also be used without a view to display content. A layer’s main job is to manage the visual content that you provide but the layer itself has visual attributes that can be set, such as a background color, border, and shadow. 

从上述文档中，可以看到CALayer主要负责显示内容，通过UIView的layer属性来调整UIView的显示效果，包括背景颜色、边框、阴影等和控制内容的动画效果等。CALayer是被定义在QuartzCore框架中的，因此要想使用CALayer，先导入QuartzCore框架。

常用属性：

	// 宽度和高度
	@property CGRect bounds;
	
	// 位置(默认指中点，具体由anchorPoint决定)
	@property CGPoint position;
	
	// 锚点(x,y的范围都是0-1)，决定了position的含义
	@property CGPoint anchorPoint;
	
	// 背景颜色(CGColorRef类型)
	@property CGColorRef backgroundColor;
	
	// 形变属性
	@property CATransform3D transform;
	
	// 边框颜色(CGColorRef类型)
	@property CGColorRef borderColor;
	
	// 边框宽度
	@property CGFloat borderWidth;
	
	// 圆角半径
	@property CGColorRef borderColor;
	
	// 内容(比如设置为图片CGImageRef)
	@property(retain) id contents;

UIView之所以能显示在屏幕上，完全是因为它内部的一个图层,在创建UIView对象时，UIView内部会自动创建一个图层(即CALayer对象)，通过UIView的layer属性可以访问这个层。当UIView需要显示到屏幕上时，会调用drawRect:方法进行绘图，并且会将所有内容绘制在自己的图层上，绘图完毕后，系统会将图层拷贝到屏幕上，于是就完成了UIView的显示。换句话说，UIView本身不具备显示的功能，是它内部的图层才有显示功能。通过操作CALayer对象可以方便的调整UIView的一些外观属性。需要注意的是CALayer不能处理用户的触摸事件，而UIView可以。CALayer有2个非常重要的属性：position和anchorPoint。

  * @property CGPoint position：该属性用来设置CALayer在父层中的位置，以父层的左上角为原点(0, 0)
	
  * @property CGPoint anchorPoint：该属性称为“定位点”、“锚点”，决定着CALayer身上的哪个点会在position属性所指的位置，以自己的左上角为原点(0, 0)，它的x、y取值范围都是0~1，默认值为（0.5, 0.5）

每一个UIView内部都默认关联着一个CALayer，我们可用称这个Layer为Root Layer（根层）。所有的非Root Layer，也就是手动创建的CALayer对象，都存在着隐式动画(*当对非Root Layer的部分属性进行修改时，默认会自动产生一些动画效果，而这些属性称为Animatable Properties(可动画属性)*)。常见的Animatable Properties：

    bounds：用于设置CALayer的宽度和高度。修改这个属性会产生缩放动画。
	backgroundColor：用于设置CALayer的背景色。修改这个属性会产生背景色的渐变动画。
	position：用于设置CALayer的位置。修改这个属性会产生平移动画。
	    
可以通过动画事务(CATransaction)关闭默认的隐式动画效果。
	 
	[CATransaction begin];
	[CATransaction setDisableActions:YES];
	self.myview.layer.position = CGPointMake(10, 10);
	[CATransaction commit];

-----

自定义图层方法：

####1. 方法1

1. 创建一个CALayer的子类，然后覆盖drawInContext:方法，使用Quartz2D API进行绘图。
2. 在.m文件中覆盖drawInContext:方法，在里面绘图。

		// 绘制一个实心三角形
		- (void)drawInContext:(CGContextRef)ctx {
		     // 设置为蓝色
		     CGContextSetRGBFillColor(ctx, 0, 0, 1, 1);
		     
		     // 设置起点
		     CGContextMoveToPoint(ctx, 50, 0);
		     // 从(50, 0)连线到(0, 100)
		     CGContextAddLineToPoint(ctx, 0, 100);
		     // 从(0, 100)连线到(100, 100)
		     CGContextAddLineToPoint(ctx, 100, 100);
		     // 合并路径，连接起点和终点
		     CGContextClosePath(ctx);
		     
		     // 绘制路径
		     CGContextFillPath(ctx);
		 }
		 
3. 在控制器中添加图层到屏幕上，一定要调用setNeedsDisplay方法，这样才会触发drawInContext:方法，从而进行绘图。

		MYLayer *layer = [MYLayer layer];
		// 设置层的宽高
		layer.bounds = CGRectMake(0, 0, 100, 100);
		// 设置层的位置
		layer.position = CGPointMake(100, 100);
		// 开始绘制图层
		[layer setNeedsDisplay];
		[self.view.layer addSublayer:layer];
		
####2. 方法2

设置CALayer的delegate，然后让delegate实现drawLayer:inContext:方法，当CALayer需要绘图时，会调用delegate的drawLayer:inContext:方法进行绘图。**需要注意的是**：不能再将某个UIView设置为CALayer的delegate，因为UIView对象已经是它内部根层的delegate，再次设置为其他层的delegate就会出问题。

1. 在控制器中创建新的层，设置delegate，然后添加到控制器的view的layer中。

		CALayer *layer = [CALayer layer];
		// 设置delegate
		layer.delegate = self;
		// 设置层的宽高
		layer.bounds = CGRectMake(0, 0, 100, 100);
		// 设置层的位置
		layer.position = CGPointMake(100, 100);
		// 开始绘制图层
		[layer setNeedsDisplay];
		[self.view.layer addSublayer:layer];
		
2. 让CALayer的delegate(前面设置的控制器)实现drawLayer:inContext:方法。

		// 画一个矩形框
		- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
		     // 设置蓝色
		     CGContextSetRGBStrokeColor(ctx, 0, 0, 1, 1);
		     
		     // 设置边框宽度
		     CGContextSetLineWidth(ctx, 10);
		     
		     // 添加一个跟层一样大的矩形到路径中
		     CGContextAddRect(ctx, layer.bounds);
		     
		     // 绘制路径
		     CGContextStrokePath(ctx);
		 } 
		 
####3. 总结

当UIView需要显示时，它内部的层会准备好一个CGContextRef(图形上下文)，然后调用delegate(这里就是UIView)的drawLayer:inContext:方法，并且传入已经准备好的CGContextRef对象。而UIView在drawLayer:inContext:方法中又会调用自己的drawRect:方法，从而完成绘制。平时在drawRect:中通过UIGraphicsGetCurrentContext()获取的就是由图层传入的CGContextRef对象，在drawRect:中完成的所有绘图都会填入图层的CGContextRef中，然后被拷贝至屏幕显示出来。