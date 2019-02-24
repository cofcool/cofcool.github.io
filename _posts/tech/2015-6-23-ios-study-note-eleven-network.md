---
layout: post
category : Development
title : ios学习系列11--网络技术
tagline: "ios学习笔记"
tags : [ios]
---
{% include JB/setup %}

本篇是第十一部分，关于网络的一些简单内容。

### 网络

1. JSON

   JSON是一种轻量级的数据格式，一般用于数据交互服务器返回给客户端的数据，一般都是JSON格式或者XML格式（文件下载除外）。JSON的格式很像OC中的字典和数组（标准JSON中key必须用双引号）：
   
		{"name" : "jack", "age" : 10}
		{"names" : ["jack", "rose", "jim"]}

   **<1> JSON解析** 
   
   ![image]({{ site.url }}/public/upload/images/ios_29.png)
   
   在iOS中，JSON的常见解析方案有4种：第三方框架：JSONKit、SBJson、TouchJSON（性能从左到右，越差），苹果原生（自带）：NSJSONSerialization（性能最好）。
 
   NSJSONSerialization：
   
		// JSON数据OC对象
		+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;
		
		// OC对象JSON数据 
		+ (NSData *)dataWithJSONObject:(id)obj options:(NSJSONWritingOptions)opt error:(NSError **)error;

   解析来自服务器的JSON:
   
   ![image]({{ site.url }}/public/upload/images/ios_30.png)
   
2. XML

   XML是Extensible Markup Language的简称，译作“可扩展标记语言”，跟JSON一样，也是常用的一种用于交互的数据格式，一般也叫XML文档（XML Document）。
   
   XML举例：
   
		<videos>
		    <video name="v1" length="31" />
		    <video name="v2" length="21" />
		    <video name="v3" length="35" />
       </videos>

   一个常见的XML文档一般由以下部分组成：文档声明、元素（Element）、属性（Attribute）。在XML文档的最前面，必须编写一个文档声明，用来声明XML文档的类型：
   
       <?xml version="1.0" encoding="UTF-8" ?>

   **<1> 元素** 一个元素包括了开始标签和结束标签。
   
		拥有元素内容：<video>v1</video>
		没有元素内容：<video></video>
		没有元素内容的简写：<video/> 

   一个元素可以嵌套若干个子元素（不能出现交叉嵌套）,规范的XML文档最多只有1个根元素，其他元素都是根元素的子孙元素。

		<videos>
		    <video>
		         <name>v1</name>
		         <length>31</length>
		    </video>
		</videos>

   **需要注意的是：XML中的所有空格和换行，都会当做具体内容处理。**

   **<2> 属性** 一个元素可以拥有多个属性
   
       <video name="v1" length="31" />
 
   video元素拥有name和length两个属性，属性值必须用双引号"" 或者 单引号'' 括住。属性表示的信息也可以用子元素来表示。

   **<3> 解析** XML的解析方式有2种：DOM：一次性将整个XML文档加载进内存，比较适合解析小文件；SAX：从根元素开始，按顺序一个元素一个元素往下解析，比较适合解析大文件。
   
   ios中XML的解析：
   
   **苹果原生**：NSXMLParser：SAX方式解析，使用简单。
   
   **第三方框架**：
   
   libxml2：纯C语言，默认包含在iOS SDK中，同时支持DOM和SAX方式解析
   
   GDataXML：DOM方式解析，由Google开发，基于libxml2
   
   **XML解析方式的选择建议**
   
   大文件：NSXMLParser、libxml2，小文件：GDataXML

   1. NSXMLParser 采取的是SAX方式解析，特点是事件驱动，下面情况都会通知代理：当扫描到文档（Document）的开始与结束；当扫描到元素（Element）的开始与结束。


			// 传入XML数据，创建解析器
			NSXMLParser *parser = [[NSXMLParser alloc] initWithData:data];
			// 设置代理，监听解析过程
			parser.delegate = self;
			// 开始解析
			[parser parse];
 
      NSXMLParserDelegate:
   
			// 当扫描到文档的开始时调用（开始解析）
			- (void)parserDidStartDocument:(NSXMLParser *)parser;
			
			// 当扫描到文档的结束时调用（解析完毕）
			- (void)parserDidEndDocument:(NSXMLParser *)parser;
			
			// 当扫描到元素的开始时调用（attributeDict存放着元素的属性）
			- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict;
			
			// 当扫描到元素的结束时调用
			- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName;

3. 文件上传下载

   **<1> 小文件下载** 如果文件比较小，下载方式会比较多。直接用NSData的*+ (id)dataWithContentsOfURL:(NSURL \*)url*；利用NSURLConnection发送一个HTTP请求去下载；如果是下载图片，还可以利用SDWebImage框架。

   **<2> HTTP Range** 通过设置请求头Range可以指定每次从网路下载数据包的大小。
   
   Range示例
   
		bytes=0-499			从0到499的头500个字节
		bytes=500-999		从500到999的第二个500字节
		bytes=500-			从500字节以后的所有字节
		
		bytes=-500			最后500个字节
		bytes=500-599,800-899	同时指定几个范围

   总结：
   
   分隔：前面的数字表示起始字节数，后面的数组表示截止字节数，没有表示到末尾。
   
   分组：可以一次指定多个Range，很少用。

   **<3> 压缩框架**
   
   1. SSZipArchive：使用时需要引入libz.dylib框架 。
   
4. HTTP

   **<1> 通信过程**
   
   1. 请求 客户端 --> 服务器。请求内容：**请求行**：请求方法\HTTP协议\请求资源路径；**请求头**：描述客户端的信息；**请求体**：POST请求才会使用，存放具体数据。
   2. 响应 服务器 --> 客户端。响应的内容：**状态行**：响应行，状态码；**响应头**：服务器信息，返回数据的类型，返回数据的长度；**实体内容**：响应体，返回给客户端的具体内容。
   3. 请求方法 **GET** ：参数拼接在URL后面，参数有限制；**POST** ：参数都在请求体，参数没有限制。
   4. 发送请求的手段
   
			// NSURLConnection
			// 发送一个同步请求
			+ (NSData *)sendSynchronousRequest:(NSURLRequest *)request returningResponse:(NSURLResponse **)response error:(NSError **)error;
			
			// 发送一个异步请求
			+ (void)sendAsynchronousRequest:(NSURLRequest *)request queue:(NSOperationQueue *)queue completionHandler:(void (^)(NSURLResponse *response, NSData *data, NSError *connectionError)) handler;
			
			// 代理的方法(异步)
			[NSURLConnection connectionWithRequest:request delegate:self];
			[[NSURLConnection alloc] initWithRequest:request delegate:self];
			[[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:YES];
			
			NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
			[conn start];
