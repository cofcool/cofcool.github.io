---
layout: post
category : Development
title : ios学习系列6--Quartz2D
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第六部分，关于Quartz2D的一些内容。


Quartz 2D是一个二维绘图引擎，同时支持iOS和Mac系统。Quartz 2D能完成的工作有绘制图形 : 线条\三角形\矩形\圆\弧等；绘制文字；绘制\生成图片(图像)；读取\生成PDF；截图\裁剪图片；自定义UI控件等。我们使用的大部分UI控件都是使用Quartz2D绘制出来的，因此Quartz2D在iOS开发中很重要的一个价值是：自定义view（自定义UI控件）。

####1. 图形上下文

图形上下文（Graphics Context）：是一个CGContextRef类型的数据。作用：保存绘图信息、绘图状态；决定绘制的输出目标（输出目标可以是PDF文件、Bitmap或者显示器的窗口上)。Quartz2D的API是纯C语言编写，由Core Graphics框架提供，数据类型和函数都是以**CG**作为前缀。Quartz2D提供了以下几种类型的Graphics Context：

  * Bitmap Graphics Context
  * PDF Graphics Context
  * Window Graphics Context
  * Layer Graphics Context
  * Printer Graphics Context

**自定义View：**

  * 新建一个类，继承自UIView
  * 实现- (void)drawRect:(CGRect)rect方法
  * 在这个方法中取得跟当前view相关联的图形上下文
  * 绘制相应的图形内容
  * 利用图形上下文将绘制的所有内容渲染显示到view上面

**drawRect:** 这个方法中取得跟view相关联的图形上下文，当view第一次显示到屏幕上时调用，调用view的setNeedsDisplay或者setNeedsDisplayInRect:也会调用drawRect:。在drawRect:方法中取得上下文后，就可以绘制东西到view上。View内部有个layer（图层）属性，drawRect:方法中取得的是一个Layer Graphics Context，因此，绘制的东西其实是绘制到view的layer上去了。View之所以能显示东西，完全是因为它内部的layer。

####2. 绘图步骤

1. 获得图形上下文

        CGContextRef ctx = UIGraphicsGetCurrentContext();

2. 拼接路径（下面代码是搞一条线段）

		CGContextMoveToPoint(ctx, 10, 10);
		CGContextAddLineToPoint(ctx, 100, 100);

3. 绘制路径

		CGContextStrokePath(ctx); // CGContextFillPath(ctx);

####3. 常用拼接路径函数

1. 新建一个起点

		void CGContextMoveToPoint(CGContextRef c, CGFloat x, CGFloat y)

2. 添加新的线段到某个点

		void CGContextAddLineToPoint(CGContextRef c, CGFloat x, CGFloat y)

3. 添加一个矩形

        void CGContextAddRect(CGContextRef c, CGRect rect)

4. 添加一个椭圆

		void CGContextAddEllipseInRect(CGContextRef context, CGRect rect)

5. 添加一个圆弧

		void CGContextAddArc(CGContextRef c, CGFloat x, CGFloat y,CGFloat radius, CGFloat startAngle, CGFloat endAngle, int clockwise)

####4. 常用绘制路径函数

一般以CGContextDraw、CGContextStroke、CGContextFill开头的函数，都是用来绘制路径的

1. Mode参数决定绘制的模式

        void CGContextDrawPath(CGContextRef c, CGPathDrawingMode mode)

2. 绘制空心路径

		void CGContextStrokePath(CGContextRef c)

3. 绘制实心路径

		void CGContextFillPath(CGContextRef c)

####5. 图形上下文栈的操作

1. 将当前的上下文copy一份,保存到栈顶(那个栈叫做”图形上下文栈”)

		void CGContextSaveGState(CGContextRef c)

2. 将栈顶的上下文出栈,替换掉当前的上下文

		void CGContextRestoreGState(CGContextRef c)

####6. 矩阵操作

利用矩阵操作，能让绘制到上下文中的所有路径一起发生变化

1. 缩放

		void CGContextScaleCTM(CGContextRef c, CGFloat sx, CGFloat sy)

2. 旋转
 
		void CGContextRotateCTM(CGContextRef c, CGFloat angle)

3. 平移

		void CGContextTranslateCTM(CGContextRef c, CGFloat tx, CGFloat ty)

####7. 内存管理

使用含有“Create”或“Copy”的函数创建的对象，使用完后必须释放，否则将导致内存泄露。使用不含有“Create”或“Copy”的函数获取的对象，则不需要释放。如果retain了一个对象，不再使用时，需要将其release掉。可以使用Quartz 2D的函数来指定retain和release一个对象。例如，如果创建了一个CGColorSpace对象，则使用函数CGColorSpaceRetain和CGColorSpaceRelease来retain和release对象。也可以使用Core Foundation的CFRetain和CFRelease。注意不能传递NULL值给这些函数。

####8. 图片水印

1. 开启一个基于位图的图形上下文

		void UIGraphicsBeginImageContextWithOptions(CGSize size, BOOL opaque, CGFloat scale)

2. 从上下文中取得图片（UIImage）

		UIImage* UIGraphicsGetImageFromCurrentImageContext();

3. 结束基于位图的图形上下文

		void UIGraphicsEndImageContext();

####9. 图片裁剪

将当前上下所绘制的路径裁剪出来（超出这个裁剪区域的都不能显示）

	void CGContextClip(CGContextRef c)

####10. 屏幕截图

调用view的layer的renderInContext:方法即可

    - (void)renderInContext:(CGContextRef)ctx;
