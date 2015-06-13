---
layout: post
category : Development
title : ios学习系列2--滚动视图和表视图相关知识
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}


最近把之前学习ios做的笔记整理了一下，笔记的主要内容是对以前看过的ios视频和ppt的内容做了总结以及自己在学习和开发中的一些经验教训，内容比较杂乱，都是零碎知识，希望留作参考。

本篇是第二部分，关于滚动视图和表视图的一些内容。

##1. 滚动视图

####1. UIScrollView常见属性：

	@property(nonatomic) CGPoint contentOffset; //滚动视图的初试滚动点
	@property(nonatomic) CGSize contentSize; //表示UIScrollView内容的尺寸，滚动范围
	@property(nonatomic) UIEdgeInsets contentInset; //能够在UIScrollView的四周增加额外的滚动区域

如下图所示：

![image]({{ site.url }}/public/upload/images/ios_2.jpg)

当用用户缩放UIScrollView中的内容时，会触发UIScrollView的代理方法。调用**- (UIView \*)viewForZoomingInScrollView:(UIScrollView *)scrollView**方法，返回需要缩放的UI控件。

####2. 定时器

NSTimer ： 设置时间执行指定任务，可以定时执行指定任务。开启定时任务：

    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)seconds target:(id)target selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)repeats;

执行**[[NSRunLoop mainRunLoop] addTimer:(NSTimer \*)aTimer forMode:(NSString \*)mode]**在主线程执行定时器任务。可以调用**- (void)invalidate**方法停止定时器，一旦停止不能继续执行，要想继续执行需要重新创建。

####3. 协议的使用

1. 定义协议

   1. 定义protocol,可选参数：optional 代理对象可以不实现该方法，required 代理对象必须实现该方法，默认是optional。
   
   2. 增加代理属性：@property (weak, nonatomic) id<AppInfoViewDelegate> delegate。定义协议的名称为“**类名+Delegate**”。
   
   3. 给代理发消息，调用代理的方法（需要判断代理对象是否实现了该方法）。协议方法的命名规范：**方法名需要去掉该类的类名前缀 + 该方法的行为，并且将自己作为参数**。
   
2. 使用代理

   1. 声明协议
   
   2. 设置代理对象
   
   3. 实现协议方法
   
####4. 其它

在定义属性时，会生成getter和setter方法，还会生成一个带下划线的成员变量。但是如果该属性是readonly，只会生成getter方法，同时没有带下划线的成员变量。

##2. 表视图

####1. UITableViewCell

1. **UITableViewCell的重用：**iOS设备的内存有限，如果用UITableView显示大量数据，需要创建大量的UITableViewCell对象，从而耗尽ios设备的内容。因此，需要重复利用UITableViewCell对象。当滚动列表时，部分UITableViewCell会移出窗口，UITableView会将窗口外的UITableViewCell放入一个对象池中，每一个cell有一个标识符，可以根据标识符来识别对应的cell对象。当UITableView要求dataSource返回UITableViewCell时，先通过一个字符串标识到对象池中查找对应类型的UITableViewCell对象，如果有，就重用，如果没有，就传入这个字符串标识来初始化一个UITableViewCell对象。

	代码：
	
	      // 1.定义一个cell的标识
	      static NSString *ID = @”njcell";
	      
	      // 2.从缓存池中取出cell
	      UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
	      
	      // 3.如果缓存池中没有cell
	      if (cell == nil) {
	          cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];
	      }

2. cell的背景颜色设置：backgroundColor 和 backgroundView都可以设置cell的背景，但是backgroundView 的优先级比 backgroundColor的高。

3. 当每一行的cell的高度不一致的时候就使用代理方法设置cell的高度，当每一行的cell高度一致的时候使用属性设置cell的高度“**self.tableView.rowHeight = 60**”。

4. 刷新指定行，调用：

        - (void)reloadRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;

5. 载入nib文件：

        - (NSArray *)loadNibNamed:(NSString *)name owner:(id)owner options:(NSDictionary *)options;

6. 视图创建：

        // 只有通过代码创建控件时才会调用
        - (id)init;
        
        // 控件通过xib或者storyboard创建的时候就会调用
        - (void)awakeFromNib;        
        - (id)initWithCoder:(NSCoder *)aDecoder; 
        
        // 代码创建cell，构造方法(在初始化对象的时候会调用)一般在这个方法中添加需要显示的子控件
        - (id)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier; 
        
7. 创建复杂的cell视图时，把cell子控件的frame和数据分开设置

8. 计算文本的高度

        // 文本用什么字体字号计算
	    NSDictionary *dict = @{NSFontAttributeName : systemFontOfSize};
	    // 最大能够容纳文本的范围
	    CGSize maxSize = CGSizeMake(MAXFLOAT, MAXFLOAT);
	    // 如果将来计算的文字的范围超出了指定的范围,返回的就是指定的范围
	    // 如果将来计算的文字的范围小于指定的范围, 返回的就是真实的范围
	    CGSize textSize =  [text boundingRectWithSize:maxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:dict context:nil].size;
	    
####2. 通知

1. 每一个应用程序都有一个通知中心NSNotificationCenter，负责协助不同对象之间的消息通信。任何一个对象都可以向通知中心发布通知，描述需要的发出的信息，对通知消息感兴趣的对象可以申请在某个特定通知发布的是时候收这个通知。一个完整的通知一般包含三个属性：通知的名称、通知发布者、一些额外的信息内容。通知中心提供了一个方法来注册一个监听通知的监听器，通知中心不会保留监听对象，再通知中心注册过的对象，必须在该对象释放前取消注册。否则，在对象销毁之后，该监听器还可能监听通知，但是监听器的监听对象已经不存在，导致应用程序奔溃。

2. UIDevice，该类提供了一个单例对象，可以通过它来获取设备信息，比如电池信息，设备类型，设备系统等，通过**[UIDevice currentDevice]**获取单例对象。

####3. 图片文字的处理

   1. 在设置一个较小图片作为一个较大的控件的背景时，图片会发生较大的拉伸导致变形。因此，想要图片显示正常，需要把图片的局部像素进行拉伸填充。代码示例：
   
		   + (UIImage *)resizableImageWithName:(NSString *)imageName
		{
		    
		    // 加载原有图片
		    UIImage *norImage = [UIImage imageNamed:imageName];
		    // 获取原有图片的宽高的一半
		    CGFloat w = norImage.size.width * 0.5;
		    CGFloat h = norImage.size.height * 0.5;
		    // 生成可以拉伸指定位置的图片
		    UIImage *newImage = [norImage resizableImageWithCapInsets:UIEdgeInsetsMake(h, w, h, w) resizingMode:UIImageResizingModeStretch];
		    
		    return newImage;
	    }
	    
2. 计算文字高度，返回文字占用的高度

		- (CGSize)sizeWithFont:(UIFont *)font maxSize:(CGSize)maxSize
		{
		    NSDictionary *dict = @{NSFontAttributeName: font};
		    CGSize textSize = [self boundingRectWithSize:maxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:dict context:nil].size;
		    return textSize;
		}
		
####4. 键盘退出

键盘状态改变时，系统会发出对应的通知，并且会附带跟键盘相关事务额外信息。例如：
  
		UIKeyboardFrameBeginUserInfoKey // 键盘刚开始的frame
		UIKeyboardFrameEndUserInfoKey // 键盘最终的frame(动画执行完毕后)
		UIKeyboardAnimationDurationUserInfoKey // 键盘动画的时间
		UIKeyboardAnimationCurveUserInfoKey // 键盘动画的执行节奏(快慢）
        
####5. 控件无法点击

 1. 查看是否调用添加的方法
 2. frame为空(没有设置frame)
 3. hidden属性是否为yes
 4. alpha <=0.1
 5. 没有添加到父控件中
 6. 查看父控件有没有以上几点