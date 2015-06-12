---
layout: post
category : Development
title : ios学习系列1--控件的基本知识
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}


最近把之前学习ios做的笔记整理了一下，笔记的主要内容是对以前看过的ios视频和ppt的内容做了总结以及自己在学习和开发中的一些经验教训，内容比较杂乱，都是零碎知识，希望留作参考。

本篇是第一部分，关于系统控件的一些内容。

1. 开发ios应用必备知识：

	![image]({{ site.url }}/public/upload/images/ios_1.png)
	
2. 初学者先学习OC的语法，等对OC理解之后再开始学习UI知识。
3. 每个控件都是一个UI对象。
4. UIViewController负责管理UIView，创建、显示、销毁UIView，负责监听UIView内部的事件，负责处理UIView与用户的交互。
5. 退出键盘的两种方式：1.resignFirstResponder 2.endEditing
6. 常用控件：

	控件  |  功能 | 控件 | 功能
	---- |  ---| --- | ---
	UIbutton |  按钮 | UILabel | 文本标签
	UITextField |  文本输入框 | UIImageView  | 图片显示
	UIProgressView | 进度条 | UISlider  | 滑块
	UISwitch  | 开关 | UISegmentControl  | 选项卡
	UIActivityIndicator  | 圈圈 | UIAlertView | 对话框（中间弹框）
	UIActionSheet  | 底部弹框 | UIScrollView | 滚动的控件
	UIPageControl  | 分页控件 | UITextView | 能滚动的文字显示控件
	UITableView | 表格 | UICollectionView  | 九宫格
	UIPickerView  | 选择器 | UIDatePicker | 日期选择器
	UIWebView  | 网页显示控件 | UIToolbar | 工具条
	UINavigationBar | 导航条 | ……
	
7. UIButton的状态：normal -> 默认情况；highlighted -> 高亮状态，按钮被按下去的时候，手指还未松开；disabled -> 不可用状态，代表按钮不可点击。
8. 为了保证高亮状态下的图片显示正常显示，必须设置按钮的类型为：custom。
9. UIImageView默认情况下不能点击。
10. 懒加载，将属性放在getter方法中初始化，在需要的时候创建

	    - (NSArray *)appList   {
        	if (!appList) {
        		NSString *path = [[NSBundle mainBundle]pathForResource:@"statuses" ofType:@"plist"];
                NSArray *tmpArray = [NSArray arrayWithContentsOfFile:path];
    
                NSMutableArray *array = [NSMutableArray arrayWithCapacity:tmpArray.count];
                for (NSDictionary *dict in tmpArray) {
			         [array addObject:[LFAppInfo appInfoWithDict:dict]];
                }
             _appList = [array mutableCopy];
        	}
        	return _appList;
        }

11. 在OC中不允许直接修改对象的“结构体属性”的成员变量，可通过中间变量来间接修改。
12. 使用枚举类型，避免在程序中出现魔法数字，使程序更易读。
13. transform属性：带有make单词的方法创建基于控件初始位置的形变；不带make的方法创建基于transform参数的形变。
14. OC中，使用弧度制，正数表示顺时针旋转，负数表示逆时针。
15. viewDidLoad是视图加载完成之后调用，通常在此方法中执行视图控制器的初始化工作。在此方法中一定要调用父类的该方法。
16. Tom猫分析:


	①创建动画播放方法**startAnimation:**把行为以字符串的方式传入，在该方法中创建一个可变数组，来储存该行为的全部图片。调用UIImageView的setAnimationImages:方法，把数组传入，设置好播放时间和重复次数之后，调用startAnimating即可播放动画。需要注意的时，播放结束后一定要清空数组。
	
	②每一个按钮创建一个方法，以其对应的行为命名，并在该方法中调用上述方法，把对应的行为以字符串的形式传入。
		
	*注意*：

	> 1. [UIImage imageNamed:imageName],这样创建的图片在使用完成后，不会直接释放掉，具体释放时间由系统确定，适用于使用小图片的场合。直接从沙盒通过图片路径加载图片，使用完即释放。
	> 2. Images.xcassets中的图片不能使用[[NSBundle mainBundle]pathForResource:imageName ofType:nil]方法来访问

	  代码：
	    
	    - (void)startAnimation:(NSString *)action {
	    // 如果正在动画则直接返回
		    if ([self.tom isAnimating]) return;
		    
		    NSMutableArray *tmpArray = [NSMutableArray array];
		    int count = [self.allPic[action] intValue];
		    
		    for (int  i = 0; i < count; i++) {
		        NSString *imageName = [NSString stringWithFormat:@"%@_%02d.jpg",action,i];
		        NSString *paht = [[NSBundle mainBundle]pathForResource:imageName ofType:nil];
		        UIImage *image = [UIImage imageWithContentsOfFile:paht];
		        [tmpArray addObject:image];
	    }
		    // 设置动画数组
		    [self.tom setAnimationImages:tmpArray];
		    
		    [self.tom setAnimationDuration:count * 0.075];
		    [self.tom setAnimationRepeatCount:1];
		    
		    // 开始动画
		    [self.tom startAnimating];
		    
		    //动画完成后清除数组
		    [self.tom performSelector:@selector(setAnimationImages:) withObject:nil afterDelay:self.tom.animationDuration];
	    }
	
17. ARC：ARC是苹果为了简化程序员对内存的管理，推出的、一套内存管理机制使用ARC机制，对象的申请和释放工作会在运行时由编译器自动在代码中添加retain和release。**strong**--> 强指针引用对象，在生命周期内不会被系统释放，OC对象默认都是强指针；**weak**--> 弱指针引用的对象，系统会立即释放，弱指针可以指向其他已经被强指针引用的对象。

18. 在ios的应用程序中，应用程序启动后，系统会创建一个运行循环监听用户的交互，当用户点击时，会向视图控制器发送点击消息。

    ![image]({{ site.url }}/public/upload/images/ios_0.png)

19. 九宫格布局：①创建多个小view通过循环来生成九宫格；②使用UICollectionView创建。在创建时，根据要显示的数据自动调整小格子的位置和数量。

20. 块动画：   
  ①首尾式动画：   

        [UIView beginAnimations:nil context:nil];
        [UIView setAnimationDuration:1.0];
        // 修改属性的动画代码
        // ......
        UIView commitAnimations];
        
	②块动画：
	
	    [UIView animateWithDuration:2.0 animations:^{
	        // 修改控件属性动画
	        label.alpha = 0.0;
	    } completion:^(BOOL finished) {
	        // 删除控件
	        [label removeFromSuperview];
	    }];

21. 字典转模型

	①字典转模型的好处：   
	 1. 降低代码的耦合度；   
	 2. 所有字典转模型部分的代码统一集中在一处处理，降低代码出错的几率
	 3. 在程序中直接使用模型的属性来操作，提高编码效率
	 4. 方法 ：
	 
            - (instancetype)initWithDict:(NSDictionary *)dict;
            + (instancetype)dataWithDict:(NSDictionary *)dict;

22. instancetype & id : 

	①instancetype 在类型表示上，跟id一样，可以表示任何对象类型   
	②instancetype 只能用在返回值类型上，不能像id一样用在参数类型上   
	③instancetype 比id多一个好处，编译器会检测instancetype的真实类型
	
23. 在模型中添加readonly属性，只会生成getter方法，同时没有成员变量

		@property (nonatomic, strong, readonly) UIImage *image;

		@interface LFAppInfo()
		{
    	    UIImage *_imageABC;
		}
		- (UIImage *)image
		{
    		if (!_imageABC) {
                _imageABC = [UIImage imageNamed:self.icon];
    		}
    		return _imageABC;
		}   		
在模型中合理地使用只读属性，可以进一步降低代码的耦合度。

24. UIView的封装

	①如果一个view内部的子控件比较多，一般会考虑自定义一个view，把它内部子控件的创建屏蔽起来，不让外界关心   
	②外界可以传入对应的模型数据给view，view拿到模型数据后给内部的子控件设置对应的数据
	
25. 使用代码显示九宫格时，行位置 = 总数 / 每行个数；列位置 = 总数 % 每行个数 。
26. 代码载入nib文件，loadNibName会将xib中定义的视图全部加载出来，并且返回一个数组，如果只有一个view的话，获取firstObject即可。使用xib时，可以再一个UIView类中使用类方法载入xib视图。

	    NSArray *array = [[NSBundle mainBundle] loadNibNamed:@"AppInfoView" owner:nil options:nil];
	    
	    // 载入xib视图
	    + (instancetype)appInfoView
		{
    		// loadNibNamed 会将名为AppInfoView中定义的所有视图全部加载出来，并且按照XIB中定义的顺序，返回一个视图的数组
    		NSArray *array = [[NSBundle mainBundle] loadNibNamed:@"AppInfoView" owner:nil options:nil];
    		return [array firstObject];
		}
		
27. 在字典转模型时，可以在模型中载入plist文件，创建一个类方法返回数组即可。

		+ (NSArray *)appInfoList
		{
			NSString *path = [[NSBundle mainBundle] pathForResource:@"app.plist" ofType:nil];
    		NSArray *array = [NSArray arrayWithContentsOfFile:path];
    
    		NSMutableArray *arrayM = [NSMutableArray array];
    		for (NSDictionary *dict in array) {
        		[arrayM addObject:[self appInfoWithDict:dict]];
         	 }
    		return arrayM;
		}
		
	
28. ios7修改状态栏调用 **- (UIStatusBarStyle)preferredStatusBarStyle**，返回状态栏样式。

29. KVC：

	使用KVC间接修改对象属性时，系统会自动判断对象的类型，并完成转换。调用对象的**setValue:forKeyPath:**方法来给属性赋值,也可以给对象的属性的属性进行赋值。KVC按照键值路径取值时，如果对象不包含指定的键值，会自动进入对象内部，查找对象属性。使用KVC也可以进行一些简单的运算，例如
	
	    NSNumber * min = [array valueForKeyPath:@"@min.age"];
	    
KVC性能不高，如果写错对象属性名称，程序在编译时不会报错，运行时才会报错。