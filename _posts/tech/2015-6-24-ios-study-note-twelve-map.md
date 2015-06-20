---
layout: post
category : Development
title : ios学习系列12--地图
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第十二部分，关于地图的一些内容。

### 地图

####1. CoreLocation框架

   CoreLocation框架中所有数据类型的前缀都是CL，并使用CLLocationManager对象来做用户定位。
   
   **<1> CLLocationManager** 
   
	// 开始用户定位
	- (void)startUpdatingLocation;
		
	// 停止用户定位
	- (void) stopUpdatingLocation;

   当调用了startUpdatingLocation方法后，就开始不断地定位用户的位置，中途会频繁地调用代理的方法,其中locations参数里面装着CLLocation对象。
   
	- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations;

   **<2> CLLocation** CLLocation用来表示某个位置的地理信息，比如经纬度、海拔等等。
   
	@property(readonly, nonatomic) CLLocationCoordinate2D coordinate; //经纬度
	@property(readonly, nonatomic) CLLocationDistance altitude; // 海拔
	@property(readonly, nonatomic) CLLocationDirection course;  // 路线，航向（取值范围是0.0° ~ 359.9°，0.0°代表真北方向）
	@property(readonly, nonatomic) CLLocationSpeed speed;  // 行走速度（单位是m/s）

   **<3> CLLocationManager**
   
	@property(assign, nonatomic) CLLocationDistance distanceFilter;  // 每隔多少米定位一次
	@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy;  // 定位精确度（越精确就越耗电）
		
	// 判断当前设备的定位功能是否可用
	+ (BOOL)locationServicesEnabled;
   
   **<4> CLLocationCoordinate2D** 一般用CLLocationCoordinate2DMake函数来创建CLLocationCoordinate2D。
   
	typedef struct {
		CLLocationDegrees latitude; // 纬度
		CLLocationDegrees longitude; // 经度
    } CLLocationCoordinate2D;
		
   **<5> CLGeocoder** 使用CLGeocoder可以完成“地理编码”和“反地理编码”。地理编码：根据给定的地名，获得具体的位置信息（比如经纬度、地址的全称等）；反地理编码：根据给定的经纬度，获得具体的位置信息。

	// 地理编码方法
	- (void)geocodeAddressString:(NSString *)addressString completionHandler:(CLGeocodeCompletionHandler)completionHandler;

	// 反地理编码方法
	- (void)reverseGeocodeLocation:(CLLocation *)location completionHandler:(CLGeocodeCompletionHandler)completionHandler;

   **<6> CLGeocodeCompletionHandler** 当地理\反地理编码完成时，就会调用CLGeocodeCompletionHandler。
   
	typedef void (^CLGeocodeCompletionHandler)(NSArray *placemarks, NSError *error);
		
   这个block传递2个参数：error ：当编码出错时（比如编码不出具体的信息）有值；placemarks ：里面装着CLPlacemark对象。

   **<7> CLPlacemark** 封装详细的地址位置信息
   
	@property (nonatomic, readonly) CLLocation *location; // 地理位置
	@property (nonatomic, readonly) CLRegion *region; // 区域
	@property (nonatomic, readonly) NSDictionary *addressDictionary; // 详细的地址信息
	@property (nonatomic, readonly) NSString *name; // 地址名称
	@property (nonatomic, readonly) NSString *locality; // 城市

####2. MapKit
  
   MapKit框架中所有数据类型的前缀都是MK,MapKit有一个比较重要的UI控件 ：MKMapView，专门用于地图显示.
  
   **<1> 跟踪显示用户的位置** 
   
   设置MKMapView的userTrackingMode属性可以跟踪显示用户的当前位置:
   
    MKUserTrackingModeNone ：不跟踪用户的位置;
    MKUserTrackingModeFollow ：跟踪并在地图上显示用户的当前位置
    MKUserTrackingModeFollowWithHeading ：跟踪并在地图上显示用户的当前位置，地图会跟随用户的前进方向进行旋转

   **<2> 地图的类型**
   
   可以通过设置MKMapView的mapViewType设置地图类型
   
	MKMapTypeStandard ：普通地图
	MKMapTypeSatellite ：卫星云图
	MKMapTypeHybrid ：普通地图覆盖于卫星云图之上 

   **<3> MKMapView的代理**
   
   常见的代理方法:
 
	// 调用非常频繁，不断监测用户的当前位置
	// 每次调用，都会把用户的最新位置（userLocation参数）传进来
	- (void)mapView:(MKMapView *)mapView didUpdateUserLocation:(MKUserLocation *)userLocation;

	//地图的显示区域即将发生改变的时候调用
	- (void)mapView:(MKMapView *)mapView regionWillChangeAnimated:(BOOL)animated;
	
	// 地图的显示区域已经发生改变的时候调用
	- (void)mapView:(MKMapView *)mapView regionDidChangeAnimated:(BOOL)animated;
	
   **<4> MKUserLocation** 大头针模型
   
	@property (nonatomic, copy) NSString *title; // 显示在大头针上的标题
	@property (nonatomic, copy) NSString *subtitle; // 显示在大头针上的子标题
	@property (readonly, nonatomic) CLLocation *location; // 地理位置信息(大头针钉在什么地方?)

   **<5> 设置地图的显示**
   
   通过MKMapView的下列方法，可以设置地图显示的位置和区域
   
	// 设置地图的中心点位置
	@property (nonatomic) CLLocationCoordinate2D centerCoordinate;
	- (void)setCenterCoordinate:(CLLocationCoordinate2D)coordinate animated:(BOOL)animated;
	
	// 设置地图的显示区域
	@property (nonatomic) MKCoordinateRegion region;
	- (void)setRegion:(MKCoordinateRegion)region animated:(BOOL)animated;

   **<6> MKCoordinateRegion**
   
   MKCoordinateRegion是一个用来表示区域的结构体，定义如下
   
	typedef struct {
	    CLLocationCoordinate2D center; // 区域的中心点位置
	    MKCoordinateSpan span; // 区域的跨度
    } MKCoordinateRegion;

MKCoordinateSpan的定义

	typedef struct {
	    CLLocationDegrees latitudeDelta; // 纬度跨度
	    CLLocationDegrees longitudeDelta; // 经度跨度
	} MKCoordinateSpan;

   **<7> 大头针**
   
	// 添加一个大头针
	- (void)addAnnotation:(id <MKAnnotation>)annotation;
	
	// 添加多个大头针
	- (void)addAnnotations:(NSArray *)annotations;
	
	// 移除一个大头针
	- (void)removeAnnotation:(id <MKAnnotation>)annotation;
	
	// 移除多个大头针
	- (void)removeAnnotations:(NSArray *)annotations;

   自定义大头针模型：
   
	#import <MapKit/MapKit.h>
	
	@interface TestAnnotation : NSObject <MKAnnotation>

	/** 坐标位置 */
	@property (nonatomic, assign) CLLocationCoordinate2D coordinate;

	/** 标题 */
	@property (nonatomic, copy) NSString *title; 

	/** 子标题 */
	@property (nonatomic, copy) NSString *subtitle;

	@end

   自定义大头针：
   
   * 设置MKMapView的代理
   * 实现下面的代理方法，返回大头针控件
     		 
		- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id <MKAnnotation>)annotation;
		
   * 根据传进来的(id <MKAnnotation>)annotation参数创建并返回对应的大头针控件
   
   ***代理方法的使用注意***：如果返回nil，显示出来的大头针就采取系统的默认样式。标识用户位置的蓝色发光圆点，它也是一个大头针，当显示这个大头针时，也会调用代理方法。因此，需要在代理方法中分清楚(id <MKAnnotation>)annotation参数代表自定义的大头针还是蓝色发光圆点。
   
   示例代码：
   
	- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
	{
	    // 判断annotation的类型
	    if (![annotation isKindOfClass:[TestAnnotation class]]) return nil;
	    
	    // 创建MKAnnotationView
	    static NSString *ID = @"id";
	    MKAnnotationView *annoView = [mapView dequeueReusableAnnotationViewWithIdentifier:ID];
	    if (annoView == nil) {
	        annoView = [[MKAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:ID];
	        annoView.canShowCallout = YES;
	    }
	    
	    // 传递模型数据
	    annoView.annotation = annotation;
	    
	    // 设置图片
	    TestAnnotation *testAnnotation = annotation;
	    annoView.image = [UIImage imageNamed:icon];
	    
	    return annoView;
	}

   地图上的大头针控件是MKAnnotationView，属性：
   
	@property (nonatomic, strong) id <MKAnnotation> annotation; // 大头针模型
	@property (nonatomic, strong) UIImage *image; // 显示的图片
	@property (nonatomic) BOOL canShowCallout; // 是否显示标注
	@property (nonatomic) CGPoint calloutOffset; // 标注的偏移量
	@property (strong, nonatomic) UIView *rightCalloutAccessoryView; // 标注右边显示什么控件
	@property (strong, nonatomic) UIView *leftCalloutAccessoryView; // 标注左边显示什么控件

   MKPinAnnotationView是MKAnnotationView的子类，MKPinAnnotationView比MKAnnotationView多了2个属性
   
	@property (nonatomic) MKPinAnnotationColor pinColor; // 大头针颜色
	@property (nonatomic) BOOL animatesDrop; // 大头针第一次显示时是否从天而降
