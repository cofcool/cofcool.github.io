#ios学习总结 October 20, 2015

在日常的工作中会遇到各种各样的问题，遇到问题时要记的把问题和对应的解决方案记录下来。

1. 一次性执行的代码：

		static dispatch_once_t onceToken;
		dispatch_once(&onceToken, ^{
			// 只执行1次的代码(这里面默认是线程安全的)
		});

2. 使用单例时，注意添加线程锁

		+ (instancetype)sharedTool
		{
			@synchronized(self) {
				if (!_instance) {
					_instance = [[self alloc] init];
				}
			}
		   return _instance;
		}

3. 简单的图片轮播器

   使用scrollView和NSTimer来实现一个简单的图片轮播器。
   
   		
   	    // 创建控件
	    UIScrollView *scrollView = [[UIScrollView alloc]initWithFrame:CGRectMake(0, 64, self.view.frame.size.width, self.view.frame.size.height / 4)];
	    scrollView.contentSize = CGSizeMake(self.scrollView.frame.size.width * 3, self.scrollView.frame.size.height);
	    scrollView.delegate = self;
	    scrollView.showsHorizontalScrollIndicator = NO;
	    scrollView.pagingEnabled = YES;
	    scrollView.bounces = NO;
	    self.scrollView = scrollView;
	    
	
	    //  添加scrollview的图片内容
	    for (int i = 1; i < 4; i++) {
	        UIImageView *imageView = [[UIImageView alloc]init];
	        CGFloat imageX = (i - 1) * self.scrollView.frame.size.width;
	        CGFloat imageY = 0.0f;
	        imageView.frame = CGRectMake(imageX, imageY, scrollView.frame.size.width, scrollView.frame.size.height);
	        imageView.image = [UIImage imageNamed:@"imageName"];
	        [scrollView addSubview:imageView];
	    }
	    [self addScrollTimer];
	    
	    
	    UIPageControl *pageControl = [[UIPageControl alloc]initWithFrame:CGRectMake(self.scrollView.frame.size.width - 80, CGRectGetMaxY(scrollView.frame) - 25, 60, 20)];
	    pageControl.currentPageIndicatorTintColor = [UIColor redColor];
	    pageControl.pageIndicatorTintColor = [UIColor whiteColor];
	    pageControl.numberOfPages = 3;
	    pageControl.currentPage = 0;
	    self.pageControl = pageControl;
	    
		#pragma mark - scrollview delegate
		- (void)scrollViewDidScroll:(UIScrollView *)scrollView  {
		    if (self.timer)  {
		        return;
		    }
		    CGPoint offset = scrollView.contentOffset;
		    CGFloat offsetX = offset.x;
		    CGFloat width = scrollView.frame.size.width;
		    int pageNum = (offsetX + 0.5f * width) / width;
		    
		    self.pageControl.currentPage = pageNum;
		}
		
		- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView  {
		    [self removeTimer];
		}
		
		- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate {
		    [self addScrollTimer];
		}
		
		/**
		 *  创建定时器
		 */
		- (void)addScrollTimer  {
		    self.timer = [NSTimer timerWithTimeInterval:timerTime target:self selector:@selector(nextPage) userInfo:nil repeats:YES];
		    [[NSRunLoop mainRunLoop]addTimer:self.timer forMode:NSRunLoopCommonModes];
		}
		
		/**
		 *  移除定时器
		 */
		- (void)removeTimer {
		    [self.timer invalidate];
		    self.timer = nil;
		}
		
		/**
		 *  切换page
		 */
		- (void)nextPage    {
		    
		    if (self.pageControl.currentPage == 2) {
		        self.pageControl.currentPage = 0;
		    }else {
		        self.pageControl.currentPage += 1;
		    }
		    
		    CGFloat width = self.scrollView.frame.size.width;
		    CGPoint offset = CGPointMake(width * self.pageControl.currentPage, 0.f); 
		    [self.scrollView setContentOffset:offset animated:YES];
		}
		
4. 获取方法的执行时间

	    #import <mach/mach_time.h>  // for mach_absolute_time() and friends  
	      
	    CGFloat BNRTimeBlock (void (^block)(void)) {  
	        mach_timebase_info_data_t info;  
	        if (mach_timebase_info(&info) != KERN_SUCCESS) return -1.0;  
	      
	        uint64_t start = mach_absolute_time ();  
	        block ();  
	        uint64_t end = mach_absolute_time ();  
	        uint64_t elapsed = end - start;  
	      
	        uint64_t nanos = elapsed * info.numer / info.denom;  
	        return (CGFloat)nanos / NSEC_PER_SEC;  
	      
	    } // BNRTimeBlock 
	    
5. coreData是每操作一次，上下文就会注销。因此，想要再次操作数据必须重新获取上coreData上下文。

		/**
		 *  保存用户数据
		 */
		- (void)saveData:(TestModel *)testModel    {
		    AppDelegate *golbalDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
		    NSManagedObjectContext *context = [golbalDelegate managedObjectContext];
		    NSManagedObject *object = [NSEntityDescription insertNewObjectForEntityForName:TestTable inManagedObjectContext:context];
		    NSError *error;
		    
		    // 保存新的数据
		    [object setValue:testModel.paramId forKey:@"test"];
		    
		    
		    if (![context save:&error]) {
		        NSLog(@"数据保存失败 - %@",[error localizedDescription]);
		    }
		}
		
		- (void)deleteAllDataObject {
		    AppDelegate *golbalDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
		    NSManagedObjectContext *context = [golbalDelegate managedObjectContext];
		    NSError *error;
		    
		    // 删除之前存储的数据
		    
		    NSFetchRequest *fetch = [[NSFetchRequest alloc]init];
		    NSEntityDescription *entity = [NSEntityDescription entityForName:TestTable inManagedObjectContext:context];
		    [fetch setEntity:entity];
		    NSArray *datas = [context executeFetchRequest:fetch error:&error];
		    if (!error && datas && [datas count]) {
		        
		        for (NSManagedObject *obj in datas) {
		            [context deleteObject:obj];
		        }
		        if (![context save:&error]) {
		            NSLog(@"delete - error:%@",error);
		        }
		    }
		}
		
		//  读取数据
		- (NSMutableArray *)readData   {
		    AppDelegate *golbalDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
		    NSManagedObjectContext *context = [golbalDelegate managedObjectContext];
		    NSFetchRequest *fetch = [[NSFetchRequest alloc]init];
		    NSEntityDescription *entity = [NSEntityDescription entityForName:TestTable inManagedObjectContext:context];
		    fetch.entity = entity;
		    
		    NSError *error;
		    NSArray *result = [context executeFetchRequest:fetch error:&error];
		    
		    NSMutableArray *datas = [NSMutableArray array];
		    for (NSArray *arr in result) {
			    NSString *paramId = [arr valueForKey:@"test"];
			     NSDictionary *testDict = [NSDictionary dictionaryWithObjectsAndKeys:paramId,@"test", nil];
			    [datas addobject:testDict];
		    }
		    
		    return datas;
		}
		
6. 如果想在NSString中显示%就需要用%%。

7. 在某些需要不同的控制器监听同一通知时，要确保新的控制器开始监听之前销毁之前创建的控制器中的通知。切换模块之后，原来的控制器并没有被销毁，因此通知中心还在监听不属于它的通知，导致其他模块的控制器无法监听本属于它们的通知，导致应用程序崩溃。在这种情况下，可以在视图显示和销毁时分别添加监听和销毁。

		- (void)viewWillAppear:(BOOL)animated   {
		    [super viewWillAppear:animated];
		    
		    // 监听通知
		    [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(testMethod:) name:TestNotification object:nil];
		}
		
		- (void)viewDidDisappear:(BOOL)animated {
		    [super viewDidDisappear:animated];
		    [[NSNotificationCenter defaultCenter]removeObserver:self];
		}
	
	如果切换控制器之后，之前的控制器也会销毁，那么可以在viewDidLoad方法中创建，在dealloc方法中销毁。
	
		- (void)viewDidLoad {
		    [super viewDidLoad];
		    // Do any additional setup after loading the view.
		    
		    // 监听通知
		    [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(testMethod:) name:name:TestNotification object:nil];
				    
		}
		
		- (void)dealloc {
		    [[NSNotificationCenter defaultCenter]removeObserver:self];
		}
		
8. 清空使用NSUserDefaults存储的数据

		NSString *appDomain = [[NSBundle mainBundle] bundleIdentifier];
		[[NSUserDefaults standardUserDefaults] removePersistentDomainForName:appDomain];
		
9. 在升级xcode7之后发现之前在storyboard中的UI不看不见，，原来升级之后Size Class发生了改变，修改为原来的设置之后，就可以显示了。（之后的Xcode7.0.1显示正常）

10. 在不同控制器之间传输数据尽可能使用block取代delegate，如果需要大量使用的话，可以考虑使用NSNotificationCenter.

11. 微信，易信分享sdk使用时，必须先设置url scheme，在ios9以上版本需设置白名单，允许对应identifier的url type通过。

12. UICollectionView拥有selected和deselected，以及highlighted和unhighlighted，来更改view的状态，另外也可以设置它是否多选状态。如果该view的数据不够一屏时上下不会滚动，可以设置alwaysBounceHorizontal（scroll view）属性来开启或关闭。(如果子类找找不到需要的成员属性，可以再父类中寻找)

13. UICollectionView的contentSize并不会随着内容的变化而马上发生改变，只有滑动到底的时候才会改变，因此要想实时获取contentSize的值，可以使用如下属性：customCollectionView.**collectionViewLayout.collectionViewContentSize.height**。UICollectionView和它对应的flowlayout紧密相连，当布局发生变化时，flowlayout会马上发生相应的改变。