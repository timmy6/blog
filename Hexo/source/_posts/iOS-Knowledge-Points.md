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


#### 6.设置两个View居中对齐
```objc
view1.center = view2.center;    //设置了之后，可以不用设置view1的x和y.
```

#### 7.weak和assign修饰符的区别
* 相同点 :  都是弱引用声明类型
* 不同点：如果用weak声明的变量在栈中会自动清空，赋值为nil。如果用assign声明的变量在栈中可能不会自动赋值为nil，就会造成野指针错误。

#### 8.layoutSubviews在以下情况会被调用
- init初始化不会触发layoutSubviews.
- addSubview会触发layoutSubviews.
- 设置view的Frame会触发layoutSubViews，当然前提是Frame的值前后发生了变化.
- 滚动一个UIScrollView会触发layoutSubviews
- 旋转Screen会触发父UIView上的layoutSubviews
- 改变一个UIView大小的时候，也会触发父UIView上的layoutSubviews事件。
- 直接调用setLayoutSubViews.

#### 9.Block的用法
~ Block是iOS开发中一种比较特殊的数据结构，它可以保存一段代码，在合适的地方再调用，具有语法简介、回调方便、变成思路清晰、执行高效率等优点。

1.Block定义及使用
```objc
返回值类型 (^block变量名)(形参列表) = ^(形参列表) {

};

//调用Block保存的代码
block变量名(实参);
```

2.在项目中使用格式(一般声明在@interface之外).
在项目中，通常会重新定义Block的类型的别名，然后用别名来定义Block的类型
```objc
//定义block类型
typedef void (^Block)(int);
//定义block
Block block = ^(int a){};
//调用block
block(3);
```

#### 10.UITableViewCell高度设置
设置UITableView高度，有两种方式：
第一种：
```objc
self.tableView.rowHeight = 88;
```

第二种:是实现UITableViewDelegate中的

```objc
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    // return xxx
}
```
第一种方式指定了UITableView的cell高度为88，对于固定高度的cell，**强烈建议**使用这种，而非第二种，可以保证不必要的高度计算和调用。rowHeight属性的默认值是44，所以一个空的UITableView显示成那个样子。需要注意的是，实现了第二种方法后，rowHeight的设置将无效。所以，第二种方法适用于具有多种cell高度的UITableView。

#### 11.UITableView的estimatedRowHeight
这个属性 iOS7 就出现了， 文档是这么描述它的作用的：
> If the table contains variable height rows, it might be expensive to calculate all their heights when the table loads. Using estimation allows you to defer some of the cost of geometry calculation from load time to scrolling time.

恩，听上去蛮靠谱的。我们知道，UITableView 是个 UIScrollView，就像平时使用 UIScrollView 一样，加载时指定 contentSize 后它才能根据自己的 bounds、contentInset、contentOffset 等属性共同决定是否可以滑动以及滚动条的长度。而 UITableView 在一开始并不知道自己会被填充多少内容，于是询问 data source 个数和创建 cell，同时询问 delegate 这些 cell 应该显示的高度，这就造成它在加载的时候浪费了多余的计算在屏幕外边的 cell 上。和上面的 rowHeight 很类似，设置这个估算高度有两种方法：
```objc
self.tableView.estimatedRowHeight = 88;
// or
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath {
    // return xxx
}
```
有所不同的是，即使面对种类不同的 cell，我们依然可以使用简单的 estimatedRowHeight 属性赋值，只要整体估算值接近就可以，比如大概有一半 cell 高度是 44， 一半 cell 高度是 88， 那就可以估算一个 66，基本符合预期。
说完了估算高度的基本使用，可以开始吐槽了：
- 设置估算高度后，contentSize.height 根据“cell估算值 x cell个数”计算，这就导致滚动条的大小处于不稳定的状态，contentSize 会随着滚动从估算高度慢慢替换成真实高度，肉眼可见滚动条突然变化甚至“跳跃”。
- 若是有设计不好的下拉刷新或上拉加载控件，或是 KVO 了 contentSize 或 contentOffset 属性，有可能使表格滑动时跳动。
- 估算高度设计初衷是好的，让加载速度更快，那凭啥要去侵害滑动的流畅性呢，用户可能对进入页面时多零点几秒加载时间感觉不大，但是滑动时实时计算高度带来的卡顿是明显能体验到的，个人觉得还不如一开始都算好了呢（iOS8更过分，即使都算好了也会边划边计算）