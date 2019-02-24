---
layout: post
category : Development
title : ios学习系列17--iPad开发
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第十七部分，关于iPad开发的一些内容。

##1. UIPopoverController

UIPopoverController是iPad开发中常见的一种控制器（在iPhone上不允许使用），跟其他控制器不一样的是，它直接继承自NSObject，并非继承自UIViewController。它只占用部分屏幕空间来呈现信息，而且显示在屏幕的最前面。

使用步骤：

1. 设置内容控制器，由于UIPopoverController直接继承自NSObject，不具备可视化的能力，因此UIPopoverController上面的内容必须由另外一个继承自UIViewController的控制器来提供，这个控制器称为“内容控制器”。

2. 设置内容的尺寸，显示出来占据多少屏幕空间

3. 设置显示的位置，从哪里弹出


####1. 设置内容控制器

设置内容控制器有3种方法，在初始化UIPopoverController的时候传入一个内容控制器。

	- (id)initWithContentViewController:(UIViewController *)viewController;
	
	@property (nonatomic, retain) UIViewController *contentViewController;
	
	- (void)setContentViewController:(UIViewController *)viewController animated:(BOOL)animated;

####2. 设置内容的尺寸

设置内容的尺寸有2种方法

	@property (nonatomic) CGSize popoverContentSize;
	
	- (void)setPopoverContentSize:(CGSize)size animated:(BOOL)animated;

####3. 设置显示的位置

设置显示的位置有2种方法：

1. 围绕着一个UIBarButtonItem显示（箭头指定那个UIBarButtonItem）

		- (void)presentPopoverFromBarButtonItem:(UIBarButtonItem *)item permittedArrowDirections:(UIPopoverArrowDirection)arrowDirections animated:(BOOL)animated;
		
		// item ：围绕着哪个UIBarButtonItem显示
		// arrowDirections ：箭头的方向
		// animated ：是否通过动画显示出来

2. 围绕着某一块特定区域显示（箭头指定那块特定区域）

		- (void)presentPopoverFromRect:(CGRect)rect inView:(UIView *)view permittedArrowDirections:(UIPopoverArrowDirection)arrowDirections animated:(BOOL)animated;
		
		// rect ：指定箭头所指区域的矩形框范围（位置和尺寸）
		// view ：rect参数是以view的左上角为坐标原点（0，0）
		// arrowDirections ：箭头的方向
		// animated ：是否通过动画显示出来

3. 如果想让箭头指向某一个UIView的做法有2种做法，比如指向一个button。

		[popover presentPopoverFromRect:button.bounds inView:button permittedArrowDirections:UIPopoverArrowDirectionDown animated:YES];
		
		[popover presentPopoverFromRect:button.frame inView:button.superview permittedArrowDirections:UIPopoverArrowDirectionDown animated:YES];

####4. 通过内容控制器设置内容尺寸

UIViewController的属性，在iOS7之前

	@property (nonatomic,readwrite) CGSize contentSizeForViewInPopover;

从iOS7开始

	@property (nonatomic) CGSize preferredContentSize;

UIPopoverController常用属性：

    // 代理对象
    @property (nonatomic, assign) id <UIPopoverControllerDelegate> delegate;
	
	// 是否可见
	@property (nonatomic, readonly, getter=isPopoverVisible) BOOL popoverVisible;
	
	// 箭头方向
	@property (nonatomic, readonly) UIPopoverArrowDirection popoverArrowDirection;
	
	// 关闭popover（让popover消失）
	- (void)dismissPopoverAnimated:(BOOL)animated;

####5. 防止点击UIPopoverController区域外消失

默认情况下,只要UIPopoverController显示在屏幕上，UIPopoverController背后的所有控件默认是不能跟用户进行正常交互的。点击UIPopoverController区域外的控件，UIPopoverController默认会消失。

要想点击UIPopoverController区域外的控件时不让UIPopoverController消失，解决办法是设置passthroughViews属性。

    @property (nonatomic, copy) NSArray *passthroughViews;

这个属性是设置当UIPopoverController显示出来时，哪些控件可以继续跟用户进行正常交互。这样的话，点击区域外的控件就不会让UIPopoverController消失了。
