---
layout: post
category : Development
title : ios学习系列9--核心动画
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第九部分，关于核心动画的一些内容。


Core Animation，即为核心动画，它是一组非常强大的动画处理API，使用它能做出非常炫丽的动画效果，而且往往是事半功倍。也就是说，使用少量的代码就可以实现非常强大的功能。Core Animation可以用在Mac OS X和iOS平台。Core Animation的动画执行过程都是在后台操作的，不会阻塞主线程。要注意的是，Core Animation是直接作用在CALayer上的，并非UIView。

####1. 使用步骤

  1. 使用它需要先添加QuartzCore.framework框架和引入主头文件<QuartzCore/QuartzCore.h>(iOS7不需要)；
  2. 初始化一个CAAnimation对象，并设置一些动画相关属性；
  3. 通过调用CALayer的addAnimation:forKey:方法增加CAAnimation对象到CALayer中，这样就能开始执行动画了；
  4. 通过调用CALayer的removeAnimationForKey:方法可以停止CALayer中的动画。

####2. 结构

CAAnimation是所有动画对象的父类，负责控制动画的持续时间和速度，是个抽象类，不能直接使用，应该使用它具体的子类。属性：
	
	duration：动画的持续时间
	repeatCount：动画的重复次数
	repeatDuration：动画的重复时间
	removedOnCompletion：默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillModeForwards
	fillMode：决定当前对象在非active时间段的行为。比如动画开始之前,动画结束之后
	beginTime：可以用来设置动画延迟执行时间，若想延迟2s，就设置为CACurrentMediaTime()+2，CACurrentMediaTime()为图层的当前时间
	timingFunction：速度控制函数，控制动画运行的节奏
	delegate：动画代理

![image]({{ site.url }}/public/upload/images/ios_22.png)

####3. 子类

1. **CAPropertyAnimation**，是CAAnimation的子类，也是个抽象类，要想创建动画对象，应该使用它的两个子类：CABasicAnimation和CAKeyframeAnimation。

   属性解析：
   
   1. keyPath：通过指定CALayer的一个属性名称为keyPath(NSString类型)，并且对CALayer的这个属性的值进行修改，达到相应的动画效果。比如，指定@”position”为keyPath，就修改CALayer的position属性的值，以达到平移的动画效果。

2. **CABasicAnimation**

   属性解析：
   
   1. fromValue：keyPath相应属性的初始值。
   2. toValue：keyPath相应属性的结束值。
   3. 随着动画的进行，在长度为duration的持续时间内，keyPath相应属性的值从fromValue渐渐地变为toValue。
   4. **如果fillMode=kCAFillModeForwards和removedOnComletion=NO，那么在动画执行完毕后，图层会保持显示动画执行后的状态。但在实质上，图层的属性值还是动画执行前的初始值，并没有真正被改变。比如，CALayer的position初始值为(0,0)，CABasicAnimation的fromValue为(10,10)，toValue为(100,100)，虽然动画执行完毕后图层保持在(100,100)这个位置，实质上图层的position还是为(0,0)。**
   
3. **CAKeyframeAnimation**，CApropertyAnimation的子类，跟CABasicAnimation的区别是：CABasicAnimation只能从一个数值(fromValue)变到另一个数值(toValue)，而CAKeyframeAnimation会使用一个NSArray保存这些数值。

   属性解析：
  
   1. values：就是上述的NSArray对象。里面的元素称为”关键帧”(keyframe)。动画对象会在指定的时间(duration)内，依次显示values数组中的每一个关键帧。
   2. path：可以设置一个CGPathRef\CGMutablePathRef,让层跟着路径移动。path只对CALayer的anchorPoint和position起作用。如果你设置了path，那么values将被忽略。
   3. keyTimes：可以为对应的关键帧指定对应的时间点,其取值范围为0到1.0,keyTimes中的每一个时间值都对应values中的每一帧.当keyTimes没有设置的时候,各个关键帧的时间是平分的。
   4. CABasicAnimation可看做是最多只有2个关键帧的CAKeyframeAnimation。
  
4. **CAAnimationGroup**，CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行。

   属性解析：
  
   1. animations：用来保存一组动画对象的NSArray。
   2. 默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间。

5. **CATransition**，CAAnimation的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。iOS比Mac OS X的转场动画效果少一点。UINavigationController就是通过CATransition实现了将控制器的视图推入屏幕的动画效果。

    属性解析：
  
   1. type：动画过渡类型。
   2. subtype：动画过渡方向。
   3. startProgress：动画起点(在整体动画的百分比)。
   4. endProgress：动画终点(在整体动画的百分比)。

