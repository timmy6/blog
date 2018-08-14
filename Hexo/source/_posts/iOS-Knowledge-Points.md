---
title: iOS 个人实际开发知识点集锦(长久更新)
date: 2016-06-05 14:52:01
category: iOS开发
---

#### 1.初始化一个与屏幕大小相同的View
**(1)**
```objc
	self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
	//[UIScreen mainScreen].bounds是整个屏幕的大小，初始化时将window大小设置为整个屏幕;
```
**(2)**
```objc
	self.webView = [[View alloc] initWithFrame:self.view.bounds]
	[self.view addSubView:self.webView]
```

#### 2.跳转界面
**(1)**
```objc
	TestViewController *controller = [[TestViewController alloc] init];
	[self presentViewController:controller animated:true completion:NULL];
```

#### 3.设置默认布局开始位置(UIViewController 的 edgeForExtendedLayout属性)
在iOS 7中，引入了一个新的属性，edgeForExtendedLayout。这个属性的默认值是UIRectEdgeAll。当你的容器是UINavigationController的shih，默认的布局就是从顶部开始的。这就是为什么你设置的空间都往上漂移了66的原因。
```objc
@property(nonatomic,assign) UIRectEdge edgesForExtendedLayout NS_AVAILABLE_IOS(7_0);	//defaults to UIRectEdgeAll
```
**解决方法一:改变edgesForExtendedLayout**
```objc
	self.edgesForExtendedLayout = UIRectEdgeNone;
```
将edgesForExtendedLayout属性设置为UIRectEdgeNone，这样布局就是从导航栏下面开始了。
**解决方法二:导航栏半透明属性设置为No**
```objc
	@property(nonatomic,assign,getter=isTranslucent) BOOL translucent NS_AVAILABLE_IOS(3_0) UI_APPEARANCE_SELECTOR; //Default is NO on iOS 6 and earlier.
```
在iOS 6之前(包括iOS 6)translucent默认就是NO，在iOS 7默认就是YES了。
```objc
	self.navigationController.navigationBar.translucent = NO;
	self.automaticallyAdjustsScrollViewInsets = NO;
```
将导航栏的半透明属性关闭掉，布局也是从导航栏下面开始了。

#### 4.加载本地plist文件
```objc
	[NSArray arrayWithContentOfFile:[[NSBundle mainBundle] pathForResource:@“文件名.plist” ofType:nil]]
```

#### 5.视图控制对象声明周期
**init**	初始化程序
**viewDidLoad** 	加载视图
**viewWillAppear**	UIViewController对象的视图即将加入窗口时调用;
**viewDidApper**	UIViewController对象的视图已经加入到窗口时调用;
**viewDidDisappear**	UIViewController对象的视图已经消失、被覆盖或是隐藏时调用；
**viewVillUnload**	当内存过低时，需要释放一些不需要使用的视图时，即将释放时调用;
**viewDidUnload**	当内存过低，释放一些不需要的视图时调用。
视图控制对象通过alloc和init来创建，但是视图控制对象不会在创建的那一刻就马上创建相应的视图，而是等到需要使用的时候才通过调用loadView来创建，这样的做法能提高内存的使用率。比如，当某个标签有很多UIViewController对象，那么对于任何一个UIViewController对象的视图，只有相应的标签被选中时才会被创建出来。
比如下面代码:
```objc
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil  
{  
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];  
    if (self) {  
        // Custom initialization  
        UITabBarItem *tbi = [self tabBarItem];  
        [tbi setTitle:@"CurrentTime"];  
        [[self view ] setBackgroundColor:[UIColor yellowColor]];  
        }  
    return self;  
}
```
我们将UIViewController的init方法中访问的实例变量view在init中将背景设置为黄色，运行程序，我们能发现背景的确变成了黄色，但是，在我们还没有需要使用视图的时候，该视图已经加载好了，在UIViewController的初始方法中访问实例变量view，会导致延迟载入机制失效，这个问题看上去不是很严重，但是如果考虑到内存过低警告，那么问题就大了。。。
运行程序，选择模拟器中的硬件－>模拟内存过低警告，我们会发现，原本设置的黄色背景不见了，这是因为，内存过低，视图控制对象会在发出内存过低警告时收到didReceiveMemoryWarning消息，该方法默认实现，检查视图控制对象的视图是否可见，如果不可见，则释放掉，下次在加载该视图时就不会执行init方法，而是只执行viewDidLoad方法，所以需要将[[selfview ] setBackgroundColor:[UIColoryellowColor]];放到viewDidLoad中，这样如果视图因为内存过低被释放掉了，下次需要使用到该视图的时候，程序会默认取执行该视图的viewDidLoad方法，这样背景颜色就又出来了。

