---
layout: post
category : Development
title : ios学习系列13--多媒体
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十三部分，关于多媒体的一些内容。

### 多媒体

####1. UIImagePickerController

   该控制器的属性：
   
    UIImagePickerControllerSourceTypeCamera // 相机
    UIImagePickerControllerSourceTypePhotoLibrary // 相册

   拍照：

	UIImagePickerController *picker = [[UIImagePickerController alloc] init];
	switch (buttonIndex) {
		case 0:
		    // 拍照
		    picker.sourceType = UIImagePickerControllerSourceTypeCamera;
		    break;
		case 1:
		    // 相册
		    picker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
		    break;
	}
		
	// 设置代理
	picker.delegate = self;

	// 展示控制器
	[self presentViewController:picker animated:YES completion:nil];

   拍照或者从相册取图片完毕后，就会通知代理
   
	- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
	{
		_imageView.image = info[UIImagePickerControllerOriginalImage];
		    
		[picker dismissViewControllerAnimated:YES completion:nil];
	}

####2. MPMoviePlayerController

MPMoviePlayerController继承自NSObject，它内部有个view用来展示视频内容，添加其他控制器的view上面即可显示。MPMoviePlayerController可以播放的视频格式有以下两种：H.264、MPEG-4 Part 2 video，支持的文件扩展名为：avi,mkv,mov,m4v,mp4等。

	// 加载视频资源
	NSString *urlString = [[NSBundle mainBundle] pathForResource:@"sample_iTunes" ofType:@"mov"];
	NSURL *url = [NSURL fileURLWithPath:urlString];

	// 创建播放器
	_player = [[MPMoviePlayerController alloc] initWithContentURL:url];
	
	// 设置尺寸
	_player.view.frame = self.view.bounds;
	_player.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;

	// 添加到控制器的view上
	[self.view addSubview:_player.view];

	// 播放
	[_player play];
	
	// 监听播放状态的改变
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(videoStateChange) name:MPMoviePlayerPlaybackStateDidChangeNotification object:_player];

	// 监听播放器结束全屏
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(exitFullscreen) name:MPMoviePlayerDidExitFullscreenNotification object:_player];
	
	// 截取视频中的图片
	- (void)requestThumbnailImagesAtTimes:(NSArray *)playbackTimes timeOption:(MPMovieTimeOption)option;

MPMoviePlayerViewController继承自UIViewController，它内部封装了一个MPMoviePlayerController,只能全屏播放。简单使用：

	MPMoviePlayerViewController *play = [[MPMoviePlayerViewController alloc] initWithContentURL:url];

####3. 音效

	// 1.获得音效文件的路径
	NSURL *url = [[NSBundle mainBundle] URLForResource:@"test.wav" withExtension:nil];
	
	// 2.加载音效文件，得到对应的音效ID
	SystemSoundID soundID = 0;
	AudioServicesCreateSystemSoundID((__bridge CFURLRef)(url), &soundID);
	
	// 3.播放音效
	AudioServicesPlaySystemSound(soundID);

播放时，音效文件只需要加载1次。音效播放常见函数总结：

	// 加载音效文件
	AudioServicesCreateSystemSoundID(CFURLRef inFileURL, SystemSoundID *outSystemSoundID)
	
	// 释放音效资源
	AudioServicesDisposeSystemSoundID(SystemSoundID inSystemSoundID)
	
	// 播放音效
	AudioServicesPlaySystemSound(SystemSoundID inSystemSoundID)
	
	// 播放音效带点震动
	AudioServicesPlayAlertSound(SystemSoundID inSystemSoundID)

音效格式:

  | 音频格式 | 硬件解码 | 软件解码 |
  | ------- | ----- | ------ |
  | AAC | YES | YES |
  | ALAC | YES | YES |
  | HE-AAC | YES |  |
  | iLBC | | YES |
  | IMA4 |  | YES|
  | Linea PCM |  | YES |
  | MP3 | YES | YES |
  | CAF | YES | YES|
  
硬件解码器一次只能对一个音频文件解码。在实际应用中通常使用非压缩的音频格式（AIFF）或者CAF音频格式，从而减低系统在音频解码上的消耗，达到省电的目的。

音频转换工具：

	// 转换aiff格式
	afconvert -f AIFF -d I8 filename
	
	// 转换caf格式
	afconvert -f caff -d aac -b 32000 filename
	
	// 批量转换
	find . -name '*.mp3' -exec afconvert -f caff -d aac -b 32000 {} \;

####4. 音乐

音乐播放用到一个叫做AVAudioPlayer的类, 能够用于播放本地音频文件。AVAudioPlayer常用方法:

	// 加载音乐文件
	- (id)initWithContentsOfURL:(NSURL *)url error:(NSError **)outError;
	- (id)initWithData:(NSData *)data error:(NSError **)outError;
	
	// 准备播放（缓冲，提高播放的流畅性）
	- (BOOL)prepareToPlay;
	
	// 播放（异步播放）
	- (BOOL)play;
	
	// 暂停
	- (void)pause;
	
	// 停止
	- (void)stop;
	
	// 是否正在播放
	@property(readonly, getter=isPlaying) BOOL playing;
	
	// 时长
	@property(readonly) NSTimeInterval duration;
	
	// 当前的播放位置
	@property NSTimeInterval currentTime;

流媒体播放，框架：DOUAudioStreamer：豆瓣出品；StreamingKit；FreeStreamer。


---
**多媒体总结**

1. 音效播放（短时间的音频文件）

   * AudioServicesCreateSystemSoundID
   * AudioServicesPlaySystemSound

2. 音乐播放（长时间的音频文件）

   * AVAudioPlayer，只能播放本地的音频文件。

   * AVPlayer，能播放本地、远程的音频、视频文件，基于Layer显示，得自己去编写控制面板。
   * MPMoviePlayerController，能播放本地、远程的音频、视频文件，自带播放控制面板（暂停、播放、播放进度、是否要全屏）。
   * MPMoviewPlayerViewController，能播放本地、远程的音频、视频文件，内部是封装了MPMoviePlayerController，播放界面默认就是全屏的。如果播放功能比较简单，仅仅是简单地播放远程、本地的视频文件，建议用这个。
   * DOUAudioStreamer，能播放远程、本地的音频文件，监听缓冲进度、下载速度、下载进度。

3. 视频播放

   * 音乐播放中的2、3、4条。
   * VLC
   * FFmpeg
   * kxmovie
   * Vitamio

4. 录音
 
   * AVAudioRecorder
