---
layout: post
category : Development
title : ios学习总结 August 5, 2015
tagline: "ios学习总结"
tags : [ios,开发]
---
{% include JB/setup %}

在日常的工作中会遇到各种各样的问题，遇到问题时要记的把问题和对应的解决方案记录下来。

1. [UIScreen mainScreen] currentMode].size  获取当前设备的分辨率

2. 检测应用程序是否是第一次启动，如果是则展示app介绍图片。

	    self.window = [[UIWindow alloc]init];
	    self.window.frame = [UIScreen mainScreen].bounds;
	    
	    [self.window makeKeyAndVisible];
	    
	    // 检测
	    if (![[NSUserDefaults standardUserDefaults]boolForKey:@"firstLaunch"]) {
	        [[NSUserDefaults standardUserDefaults]setBool:YES forKey:@"firstLaunch"];
	        
	        UIViewController *vc = [[UIViewController alloc]init];
	        vc.view.frame = [[UIScreen mainScreen]bounds];
	        vc.view.backgroundColor = [UIColor redColor];
	        
	        self.window.rootViewController = vc;
	        
	        NSLog(@"first");
	    }else {
	        UIViewController *vc = [[UIViewController alloc]init];
	        vc.view.frame = [[UIScreen mainScreen]bounds];
	        vc.view.backgroundColor = [UIColor blueColor];
	        
	        self.window.rootViewController = vc;
	        
	        NSLog(@"no");
	    }
	    
3. 隐藏导航栏上的返回按钮

		[self.navigationItem.backBarButtonItem setTitle:@""];
		[self.navigationItem setHidesBackButton:YES];
    	
4. 在设置手动型segue时，需要判断条件来跳转的场景，直接从控制器拖线到另一控制器，segue设置为push，然后使用代码判断。
5. 销毁modal出来的视图，在该视图的控制器中执行

		[self dismissViewControllerAnimated:YES completion:nil];
		
6. tableView设置默认选中第一行

	    NSIndexPath *firstIndex = [NSIndexPath indexPathForRow:0 inSection:0];
	    [self.tableView selectRowAtIndexPath:firstIndex animated:NO scrollPosition:UITableViewScrollPositionTop];
	    [self tableView:self.tableView didSelectRowAtIndexPath:firstIndex];
	    
7. 没有自定义cell的情况，可以在自定义cell的类中，重写layoutSubviews来进行微调。例如：

		- (void)layoutSubviews  {
		    [super layoutSubviews];
		    
		    CGFloat width = self.frame.size.height * 0.8;
		    self.imageView.frame = CGRectMake(10, (self.frame.size.height - width) * 0.5, width, width);
		    self.imageView.contentMode = UIViewContentModeScaleAspectFit;
		}
		
8. 隐藏UITableView空Cell的Separator Lines

		1. 自定义cell
		self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
		
		2. 添加footerView
		UIView *footer =[[UIView alloc] initWithFrame:CGRectZero]; 
		self.tableView.tableFooterView = footer; 
		
9. 选中cell，每次只选中一个

		NSArray *cells = [tableView visibleCells];
	    for (UITableViewCell *tmpCell in cells) {
	        if (tmpCell.isSelected) {
	            tmpCell.accessoryView.hidden = NO;
	        }else {
	            tmpCell.accessoryView.hidden = YES;
	        }
	    }
	    
10. cell的子控件的frame尽量在layoutSubviews方法内设置，否则可能会出现宽度不足，控件位置混乱的问题。
11. 有时设置好了自定义控件的frame，该控件也可以正常显示，但是监听不到触摸事件。原因是因为控件内部的控件的frame设置有问题，尽量在layoutSubviews或initWithFrame中设置。
12. 自定义cell时，cell自带属性会消失，如果需要，需自行添加控件。
13. 遇到问题，考虑清楚问题的原因之后，再操作解决。
14. cell高度自动调整

		/**
		 *  计算文本占用的宽高
		 *
		 *  @param font    显示的字体
		 *  @param maxSize 最大的显示范围
		 *
		 *  @return 占用的宽高
		 */
		- (CGSize)sizeWithFont:(UIFont *)font maxSize:(CGSize)maxSize
		{
		    NSDictionary *dict = @{NSFontAttributeName: font};
		    CGSize textSize = [self boundingRectWithSize:maxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:dict context:nil].size;
		    return textSize;
		}
15. 图片拉伸

		/**
		 *  传入图片的名称,返回一张可拉伸不变形的图片
		 *
		 *  @param imageName 图片名称
		 *
		 *  @return 可拉伸图片
		 */
		 + (UIImage *)resizableImageWithName:(NSString *)imageName
		{
		    UIImage *norImage = [UIImage imageNamed:imageName];
		    CGFloat w = norImage.size.width * 0.5;
		    CGFloat h = norImage.size.height * 0.5;
		    UIImage *newImage = [norImage resizableImageWithCapInsets:UIEdgeInsetsMake(h, w, h, w) resizingMode:UIImageResizingModeStretch];
		    
		    return newImage;
		}
	
16. 自动回复

		- (NSDictionary *)autoReplays
		{
		    if (_autoReplays == nil) {
		        NSString *fullPath = [[NSBundle mainBundle] pathForResource:@"autoReplay.plist" ofType:nil];
		        _autoReplays = [NSDictionary dictionaryWithContentsOfFile:fullPath];
		    }
		    return _autoReplays;
		}
	
		- (NSString *)autoReplyWithContent:(NSString *)content  {
		    NSString *result = nil;
		    for (int i = 0; i  < content.length; i++) {
		        NSString *str = [content substringWithRange:NSMakeRange(i, 1)];
		        result = self.autoReplays[str];
		        if (result != nil) {
		            break;
		        }
		    }
		    if (result == nil) {
		        result = [NSString stringWithFormat:@"%@ ,bitch",content];
		    }
		    return result;
		}
	
17. 键盘弹出

		[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(keyboardWillChange:) name:UIKeyboardWillChangeFrameNotification object:nil];
		
		- (void)keyboardWillChange:(NSNotification *)notification  {
		    NSDictionary *dict = notification.userInfo;
		    CGRect keyboardFrame = [dict[UIKeyboardFrameEndUserInfoKey] CGRectValue];
		    CGFloat keyboardY = keyboardFrame.origin.y;
		    CGFloat duration = [dict[UIKeyboardAnimationDurationUserInfoKey]doubleValue];
		    CGFloat translationY = keyboardY - self.view.frame.size.height;
		    
		    [UIView animateWithDuration:duration delay:0.0 options:7 animations:^{
		        self.view.transform = CGAffineTransformMakeTranslation(0, translationY);
		    } completion:^(BOOL finished) {
		        
		    }];
		}