---
layout: post
category : Development
title : ios学习系列4--控制器相关知识
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第四二部分，关于控制器的一些内容。

####1. 控制器加载相关过程

1. 控制器加载

   ![image]({{ site.url }}/public/upload/images/ios_5.png)

2. 内存警告处理

   ![image]({{ site.url }}/public/upload/images/ios_6.png)

3. 生命周期

   ![image]({{ site.url }}/public/upload/images/ios_7.png)
   
####2. 控制器创建
   
   控制器的创建方式：通过storyboard创建，直接创建，指定xib文件创建。
   
   ![image]({{ site.url }}/public/upload/images/ios_8.png)
   
  1. 直接创建：
  
          UIViewController *vc = [[UIViewController alloc] init];

  2. 从xib创建
  
          UIViewController *vc = [[UIViewController alloc] initWithNibName:@”nibView" bundle:nil];

  3. 从storyboard创建
 
     1. 加载storyboard文件
         
            UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Test" bundle:nil];
            
      2. 初始化
      
			  // 初始化“初始控制器”（箭头所指的控制器)
			  UIViewController *vc = [storyboard instantiateInitialViewController];
			  // 通过一个标识初始化对应的控制器
			  UIViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@”vc"];

创建view的几种情况：

* 没有xib和storyboard是，会创建一个空白的view作为控制器的view。
* 通过storyboard创建时，会创建箭头指向控制器的view作为view。如果重写了控制器的loadView方法，就不会创建storyboard中描述的view作为控制器的view，而是创建一个空白的view作为控制器的view。
* 如果指定xib，会创建xib中描述的view作为控制器的view。
* 如果有同名的xib，会创建xib中描述的view作为控制器的view。
* 有同名的去掉Controller的xib，会自动找到该xib的view作为控制器的view。
* 如果重写了控制器的loadView方法，就不会去加载创建同名去掉Controller的xib和同名的xib，而是创建一个空白的view作为控制器的view。

####3.导航控制器

1. UINavigationController以栈的形式保存子控制器
2. 使用push方法能将某个控制器压入栈

        - (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;

3. 使用pop方法可以移除控制器

		// 将栈顶的控制器移除
		- (UIViewController *)popViewControllerAnimated:(BOOL)animated;
		// 回到指定的子控制器
		- (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;
		// 回到根控制器（栈底控制器）
		- (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;

4. storyboard中连接界面负责跳转的线，都是一个UIStoryboardSegue对象（简称Segue）。每一个segue对象都有三个属性：

    ![image]({{ site.url }}/public/upload/images/ios_9.png)      

	    // 唯一标识
	    @property (nonatomic, readonly) NSString *identifier;
        // 来源控制器
        @property (nonatomic, readonly) id sourceViewController;
        // 目标控制器
        @property (nonatomic, readonly) id destinationViewController;
		
	根据segue的执行时刻，可以分为两大类型：1. 自动型：点击某个控件之后，自动执行segue完成界面跳转；2. 手动型：需要通过代码手动执行segue才能完成跳转。

   在需要对跳转进行判断时调用**- (void)performSegueWithIdentifier:(NSString *)identifier
                            sender:(id)sender**方法，可以根据传入segue的标识符进行判断，满足一定条件之后才会进行跳转。该方法的执行过程：
                            
		1. 根据identifier去storyboard中找到对应的线，新建UIStoryboardSegue对象
		2. 设置Segue对象的sourceViewController（来源控制器）
		3. 新建并且设置Segue对象的destinationViewController（目标控制器）

    不同控制器之间数据传递，在控制器跳转时系统会自动调用**- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender**，可以在该方法中根据segue的标识符来判断目的控制器从而完成跳转。过程如下：
    
	    调用Segue对象的- (void)perform;方法开始执行界面跳转操作
	    取得sourceViewController所在的UINavigationController
	    调用UINavigationController的push方法将destinationViewController压入栈中，完成跳转


5. 不同控制器的数据传递

   控制器之间的数据传递主要有两种：顺传和逆传。顺传指数据传递方向和控制器跳转方向一致，逆传是指数据传递方向和控制器跳转方向相反。顺传可以调用源控制器的**prepareForSegue:sender:**方法来传递数据；逆传需要使用代理，控制器跳转时，让源控制器成为目的控制器的代理，在目的控制器中调用源控制器的代理方法，通过代理方法的参数传递数据到源控制器。
   
6. 通过modal来切换控制器

    任何控制器都可以通过modal的形式展示出来，默认效果为新控制器从屏幕的最底部往上钻，直到盖住之前的控制器为止，需要注意的是，在modal出来的控制器中不能调用push方法切换控制器。方法：

		// 以Modal的形式展示控制器
		- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion;
		// 关闭当初Modal出来的控制器
		- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^)(void))completion;
