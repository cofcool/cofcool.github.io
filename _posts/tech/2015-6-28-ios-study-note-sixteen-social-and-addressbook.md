---
layout: post
category : Development
title : ios学习系列16--社交和通讯录
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第十六部分，关于社交和通讯录的一些内容。

### 社交分享

ios自带社交分享框架：Social.framework，支持的分享平台：Twitter，Facebook，Flickr，Vimeo，新浪微博，腾讯微博。

使用：

	// 判断服务是否可用
	[SLComposeViewController isAvailableForServiceType:SLServiceTypeSinaWeibo]
	
	// 弹出分享内容输入界面
	SLComposeViewController *cc = [SLComposeViewController composeViewControllerForServiceType:SLServiceTypeSinaWeibo];
	[self presentViewController:cc animated:YES completion:nil];
	
	// 额外设置
	[cc setInitialText:@"测试文字"]; // 初始化文字
	[cc addImage:[UIImage imageNamed:@"lufy"]]; // 配图

###2. 通讯录

在iOS中，有2个框架可以访问用户的通讯录:

1. AddressBookUI.framework,提供了联系人列表界面、联系人详情界面、添加联系人界面等，一般用于选择联系人。

2. AddressBook.framework，纯C语言的API，仅仅是获得联系人数据，没有提供UI界面展示，需要自己搭建联系人展示界面，里面的数据类型大部分基于Core Foundation框架，使用比较困难。

从iOS6开始，需要得到用户的授权才能访问通讯录，因此在使用之前，需要检查用户是否已经授权。获得通讯录的授权状态：

	ABAddressBookGetAuthorizationStatus()
	
	// 授权状态
	kABAuthorizationStatusNotDetermined //用户还没有决定是否授权你的程序进行访问
	kABAuthorizationStatusRestricted // iOS设备上的家长控制或其它一些许可配置阻止程序与通讯录数据库进行交互
	kABAuthorizationStatusDenied // 用户明确的拒绝了你的程序对通讯录的访问
	kABAuthorizationStatusAuthorized // 用户已经授权给你的程序对通讯录进行访问

	
1. 申请访问通讯录(申请通讯录访问授权的代码，通常放在AppDelegate中)

		// 实例化通讯录对象
		ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, NULL);
		ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error) {
		    if (granted) {
		        NSLog(@"授权成功！");
		    } else {
		        NSLog(@"授权失败!");
		    }
		});
		CFRelease(addressBook);

2. 获得所有的联系人数据

		// 获取所有联系人记录
		CFArrayRef array = ABAddressBookCopyArrayOfAllPeople(addressBook);
		NSInteger count = CFArrayGetCount(array);
		
		for (NSInteger i = 0; i < count; ++i) {
		    // 取出一条记录
		    ABRecordRef person = CFArrayGetValueAtIndex(array, i);
		    
		    // 取出个人记录中的详细信息
		    // 名
		    CFStringRef firstNameLabel = ABPersonCopyLocalizedPropertyName(kABPersonFirstNameProperty);
		    CFStringRef firstName = ABRecordCopyValue(person, kABPersonFirstNameProperty);
		    CFStringRef lastNameLabel = ABPersonCopyLocalizedPropertyName(kABPersonLastNameProperty);
		    // 姓
		    CFStringRef lastName = ABRecordCopyValue(person, kABPersonLastNameProperty);
		    NSLog(@"%@ %@ - %@ %@", lastNameLabel, lastName, firstNameLabel, firstName);
		}

3. CoreFoundation 与 Foundation之间的桥接

		// 1. 获取通讯录引用
		ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, nil);
		// 2. 获取所有联系人记录
		NSArray *array = (__bridge NSArray *)(ABAddressBookCopyArrayOfAllPeople(addressBook));
		for (NSInteger i = 0; i < array.count; i++) {
		    // 取出一条记录
		    ABRecordRef person = (__bridge ABRecordRef)(array[i]);
		    
		    // 取出个人记录中的详细信息
		    NSString *firstNameLabel = (__bridge NSString *)(ABPersonCopyLocalizedPropertyName(kABPersonFirstNameProperty));
		    NSString *firstName = (__bridge NSString *)(ABRecordCopyValue(person, kABPersonFirstNameProperty));
		    NSString *lastNameLabel = (__bridge NSString *)(ABPersonCopyLocalizedPropertyName(kABPersonLastNameProperty));
		    NSString *lastName = (__bridge NSString *)(ABRecordCopyValue(person, kABPersonLastNameProperty));
		    
		    NSLog(@"%@ %@ - %@ %@", lastNameLabel, lastName, firstNameLabel, firstName);
		}
		
		CFRelease(addressBook);

4. 多重属性

   联系人的有些属性值就没这么简单，一个属性可能会包含多个值，比如邮箱，分为工作邮箱、住宅邮箱、其他邮箱等，比如电话，分为工作电话、住宅电话、其他电话等。如果是复杂属性，那么ABRecordCopyValue函数返回的就是ABMultiValueRef类型的数据，例如邮箱或者电话：

		// 取电话号码
		ABMultiValueRef phones = ABRecordCopyValue(person, kABPersonPhoneProperty);
		// 取记录数量
		NSInteger phoneCount = ABMultiValueGetCount(phones);
		
		// 电话标签
		CFStringRef phoneLabel = ABMultiValueCopyLabelAtIndex(phones, i);
		
		// 本地化电话标签
		CFStringRef phoneLocalLabel = ABAddressBookCopyLocalizedLabel(phoneLabel);
		
		// 电话号码
		CFStringRef phoneNumber = ABMultiValueCopyValueAtIndex(phones, i);

5. 添加联系人的步骤

   * 通过ABPersonCreate函数创建一个新的联系人（返回ABRecordRef）
   * 通过ABRecordSetValue函数设置联系人的属性
   * 通过ABAddressBookAddRecord函数将联系人添加到通讯录数据库中
   * 通过ABAddressBookSave函数保存刚才所作的修改

   可以通过ABAddressBookHasUnsavedChanges函数判断是否有未保存的修改,当决定是否更改通讯录数据库后,你可以分别使用 **AbAddressBookSave **或 **ABAddressBookRevert** 方式来保存或放弃更改。

6. 添加群组的步骤，与添加联系人一致。

   * 通过ABPersonCreate函数创建一个新的组（返回ABRecordRef）
   * 通过ABRecordSetValue函数设置组名
   * 通过ABAddressBookAddRecord函数将组添加到通讯录数据库中
   * 通过ABAddressBookSave函数保存刚才所作的修改

 
  
7. 操作联系人的头像,想操作联系人的头像，有以下函数:

		BPersonHasImageData // 判断通讯录中的联系人是否有图片
		ABPersonCopyImageData // 取得图片数据(假如有的话)
		ABPersonSetImageData  // 设置联系人的图片数据 
