---
layout: post
category : Development
title : ios学习系列3--应用程序相关知识
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}


####1. 从ios7开始，系统提供了2种管理状态栏的方式，控制器和UIApplication。隐藏状态栏：

1. 控制器：
  
	    - (UIStatusBarStyle)preferredStatusBarStyle;
	    - (BOOL)prefersStatusBarHidden;
    
2. UIApplication:

   修改Info.plist的“View controller-based status bar appearance”的值为**NO**。View controller-based status bar appearance项设为YES，则View controller对status bar的设置优先级高于application的设置；为NO则以application的设置为准，view controller的prefersStatusBarHidden方法无效，是根本不会被调用的。
   
       [[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:NO];
 
   另外UIApplication的方法openUrl:可以打电话、发短信、发邮件、打开网页以及其他app。
   
       // 打电话
       UIApplication *app = [UIApplication sharedApplication];
       [app openURL:[NSURL URLWithString:@"tel://10086"]];
       // 发短信
       [app openURL:[NSURL URLWithString:@"sms://10086"]];
       // 发邮件
       [app openURL:[NSURL URLWithString:@"mailto://cofcool@126.com"]];
       // 打开网页资源
       [app openURL:[NSURL URLWithString:@"http://www.cofcool.net"]];

####2. 正在运行的应用程序受到干扰时，UIApplication会通知它的代理对象处理这些系统事件。如：应用程序生命周期事件、系统事件、内存警告等。

![image]({{ site.url }}/public/upload/images/ios_3.png)
   
####3. ios程序启动之后创建的第一个视图控件就是UIWindow，接着创建控制器的View，最后将控制器的View添加到UIWindow上。一个ios程序能够显示到屏幕上，是因为有UIWindow，如果没有的话看不见任何UI元素。

![image]({{ site.url }}/public/upload/images/ios_4.png)
   
####4. 开发阶中分为开发调试和发布：

1. 开发调试阶段:是需要打印log调试程序的, 如果程序处于调试阶段,系统会为我们定义一个名称叫做DEBUG的宏。
2. 发布阶段:不需要打印log, 因为log很占用资源并且用户看不懂log,如果程序处于发布阶段,系统就会自动删除名称叫做DEBUG的宏。
	
		#ifdef DEBUG
		#define CCLog(...) NSLog(__VA_ARGS__)
		#else
		#define CCLog(...)
		#endif
		#endif
 
####5. 打电话

1. 最简单最直接的方式：直接跳到拨号界面
   
		NSURL *url = [NSURL URLWithString:@"tel://10086"];
		[[UIApplication sharedApplication] openURL:url];
		
   缺点:电话打完后，不会自动回到原应用，直接停留在通话记录界面。
2. 拨号之前会弹框询问用户是否拨号，拨完后能自动回到原应用
   
		NSURL *url = [NSURL URLWithString:@"telprompt://10086"];
		[[UIApplication sharedApplication] openURL:url];

   缺点:因为是私有API，所以可能不会被审核通过。
   
3. 创建一个UIWebView来加载URL，拨完后能自动回到原应用
   
		if (_webView == nil) {
			_webView = [[UIWebView alloc] initWithFrame:CGRectZero];
		}
		[_webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"tel://10086"]]];
		
   需要注意的是：这个webView千万不要添加到界面上来，不然会挡住其他界面。

####6. 发短信

1. 直接跳到发短信界面，但是不能指定短信内容，而且不能自动回到原应用。
  
		NSURL *url = [NSURL URLWithString:@"sms://10086"];
		[[UIApplication sharedApplication] openURL:url];

2. 如果想指定短信内容，那就得使用MessageUI框架,包含主头文件**#import <MessageUI/MessageUI.h>**。

		// 显示发短信的控制器
		MFMessageComposeViewController *vc = [[MFMessageComposeViewController alloc] init];
		// 设置短信内容
		vc.body = @"吃饭了没？";
		// 设置收件人列表
		vc.recipients = @[@"10086", @"1232132"];
		// 设置代理
		vc.messageComposeDelegate = self;
		// 显示控制器
		[self presentViewController:vc animated:YES completion:nil];
		
		// 代理方法，当短信界面关闭的时候调用，发完后会自动回到原应用
		- (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result
		{
		    // 关闭短信界面
		    [controller dismissViewControllerAnimated:YES completion:nil];
		    
		    if (result == MessageComposeResultCancelled) {
		        NSLog(@"取消发送");
		    } else if (result == MessageComposeResultSent) {
		        NSLog(@"已经发出");
		    } else {
		        NSLog(@"发送失败");
		    }
		}

####7. 发邮件

1. 用自带的邮件客户端，发完邮件后不会自动回到原应用。
   
		NSURL *url = [NSURL URLWithString:@"mailto://10086@qq.com"];
		[[UIApplication sharedApplication] openURL:url];

2. 使用MFMailComposeViewController控制器。
   
		MFMailComposeViewController *vc = [[MFMailComposeViewController alloc] init];
		// 设置邮件主题
		[vc setSubject:@"会议"];
		// 设置邮件内容
		[vc setMessageBody:@"今天下午开会吧" isHTML:NO];
		// 设置收件人列表
		[vc setToRecipients:@[@"102010@qq.com"]];
		// 设置抄送人列表
		[vc setCcRecipients:@[@"12234@qq.com"]];
		// 设置密送人列表
		[vc setBccRecipients:@[@"526789@qq.com"]];
		
		// 添加附件（一张图片）
		UIImage *image = [UIImage imageNamed:@"lufy.jpeg"];
		NSData *data = UIImageJPEGRepresentation(image, 0.5);
		[vc addAttachmentData:data mimeType:@"image/jepg" fileName:@"lufy.jpeg"];
		
		// 设置代理
		vc.mailComposeDelegate = self;
		// 显示控制器
		[self presentViewController:vc animated:YES completion:nil];

		// 邮件发送后的代理方法回调，发完后会自动回到原应用
		- (void)mailComposeController:(MFMailComposeViewController *)controller didFinishWithResult:(MFMailComposeResult)result error:(NSError *)error
		{
		    // 关闭邮件界面
		    [controller dismissViewControllerAnimated:YES completion:nil];

		    if (result == MFMailComposeResultCancelled) {
		        NSLog(@"取消发送");
		    } else if (result == MFMailComposeResultSent) {
		        NSLog(@"已经发出");
		    } else {
		        NSLog(@"发送失败");
		    }
		}

####8. 打开其他常见文件

如果想打开一些常见文件，比如html、txt、PDF、PPT等，都可以使用UIWebView打开，只需要告诉UIWebView文件的路径即可。

####9. 应用间跳转

想要在不同应用之间跳转，如A到B，在A中执行

	NSURL *url = [NSURL URLWithString:@"cofcool://test.cofcool.net"];
	[[UIApplication sharedApplication] openURL:url];
		
代码中openURL的参数为B的URL,可以在Info.plist或是项目设置的Info选项卡中设置。
   
![image]({{ site.url }}/public/upload/images/ios_24.png)
   
