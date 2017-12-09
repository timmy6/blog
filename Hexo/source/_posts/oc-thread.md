---
title: 多线程编程技术(Thread、Cocoa operations、GCD)
date: 2016-03-26 22:53:04
category: iOS开发
---
### 简介
在软件开发中，多线程编程技术被广泛应用，相信多线程任务对我们来说已经不再陌生了。有了多线程技术，我们可以同做多个事情，而不是一个一个任务地进行。比如：前端和后台作交互、大任务（需要耗费一定的时间和资源）等等。也就是说，我们可以使用线程把占据时间长的任务放到后台中处理，而不影响到用户的使用。

### 线程间通讯
有一个非常重要的队列，就是主队列。在这个队列中处理多点触控及所有与UI相关操作等等。它非常特殊，原因有两点。一是我们绝对不想它阻塞，我们不会将需要执行很长时间的任务放在主队列上执行。二是我们将其用于所有与UI相关的同步，也就是线程间通讯需要注意的地方。所有有可能会使屏幕UI发生变化的，都应放在主队列上执行。

** 线程的定义:**
>每个正在系统上运行的程序都是一个进程。每个进程包含一到多个线程。进程也可能是整个程序或者是部分程序的动态执行。线程是一组指令的集合，或者是程序的特殊段，它可以在程序里独立执行。也可以把它理解为代码运行的上下文。所以线程基本上是轻量级的进程，它负责在单个程序里执行多任务。通常由操作系统负责多个线程的调度和执行。

### IOS支持的多线程技术:

#### 一、Thread:
1)显示创建线程:NSThread
2)隐式创建线程:NSObject

#### 二、Cocoa operations:
NSOperation类十一个抽象类，所以我们必须使用它的两个子类。
1)NSInvocationOperation
2)NSBlockOperation
3)NSoperationQueue (继承于NSObject)

#### 三、Grand Central Dispatch (GCD)
1)GCD的创建
2)重复执行线程及一次性执行: dispatch_apply & dispatch_once
3)操作(串行)队列: dispatch_queue_create
4)GCD群组通知:dispatch_group_t
5)GCD实现计时器
6)后台运行
7)延迟执行

#### 四、比较多线程技术
我们可以使用NSThread 或 NSObject类去调用
##### 1)显示创建线程:NSThread
创建NSThread有两个办法
##### 1.1)创建之后需要使用start方法，才会执行方法:
```objc
NSThread *threadAlloc = [[NSThread alloc] initWithTarget:self selector:@selector(threadAlloc) object:nil];
[threadAlloc start];
```
##### 1.2)创建并马上执行方法
```objc
[NSThread detachNewThreadSelector:@selector(threadAlloc:) toTarget:self withObject:nil];
```
##### 2)隐式创建线程:NSObject
我们也可以使用NSObject类的方法直接调用方法:
```objc
[self performSelectorInBackground:@selector(threadAlloc) withObject:nil];
```
**取消线程发的方法**
实际上并没有真正提供取消线程的API。苹果提供了一个cancel的api，但它不能作用于取消线程，它只能改变线程的运行状态。我们可以使用它来进行条件判断。
```objc
- (void)threadCancel
{
    NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(threadCancelNow) object:nil];
    [thread start];
}

- (void)threadCancelNow
{
    int a = 0;
    while (![[NSThread currentThread] isCancelled]) {
        NSLog(@"a - %d", a);
        a++;
        if (a == 5000) {
            NSLog(@"终止循环");
            [[NSThread currentThread] cancel];
            break;
        }
    }
}
```
程序效果: 循环输出5000次，线程就会被终止
##### NSThread线程间通讯-调用主线程修改UI:
只需要传递一个selector和它的参数，withObject参数可以为nil，waitUntilDone代表是否要等待调用它的这个线程执行之后再将它从主队列调出，并在主队列上运行，通常设为NO，不需要等待。
```objc
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait; 
```
##### NSThread相关属相及方法:
```objc
// 获取/设置线程的名字
@property (copy) NSString *name NS_AVAILABLE(10_5, 2_0);

/**
 *  获取当前线程的线程对象
 *
 *  通过这个属性可以查看当前线程是第几条线程，主线程为1。
 *  可以看到当前线程的序号及名字，主线程的序号为1，依次叠加。
 */
+ (NSThread *)currentThread;

// 线程休眠（秒）
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;

// 线程休眠，指定具体什么时间休眠
+ (void)sleepUntilDate:(NSDate *)date;

// 退出线程 
// 注意：这里会把线程对象销毁！销毁后就不能再次启动线程，否则程序会崩溃。
+ (void)exit;
```

#### Cocoa operations

##### 1)NSInvocationOperation
创建NSInvocationOperation线程，附带一个NSString参数:
```objc
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationAction:) object:@"abc"];
// 需要启动线程，默认是不启动的。
[operation start];
```
如果在创建时定义了参数，那么接受的时候，可以对sender进行转换，如字符串、数组等:
```objc
- (void)invocationAction:(NSInvocationOperation *)sender
{
    NSLog(@"sender - %@", sender);      // 输出params
    NSString *str = (NSString *)sender;
    NSLog(@"str - %@e", str);           // params
}
```
附带一提，线程的普通创建一般为并发执行的，因为串行队列是需要显式创建的，如没看见此类代码，那么即是并发队列线程，因此，上述代码也就是并发线程。关于并发和串行队列（线程），我将会在下面详细说明，我们继续往下看。

**你也可以使用NSOperationQueue来创建一个线程队列，用来添加子线程：**
```objc
NSOperationQueue *invocationQueue = [[NSOperationQueue alloc] init];

// 线程A
NSInvocationOperation *invocationQ1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationAction:) object:@"invocationQ1"];

// 线程B
NSInvocationOperation *invocationQ2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationAction:) object:@"invocationQ2"];

// 往invocationQueue添加子线程
[invocationQueue addOperations:@[invocationQ1, invocationQ2] waitUntilFinished:YES];
```
##### 2)NSBlockOperation
**创建NSBlockOperation**
```objc
NSBlockOperation *blockOperation = [NSBlockOperation
                                    blockOperationWithBlock:^{
                                        [NSThread sleepForTimeInterval:2];
                                        NSLog(@"one - %@", [NSThread currentThread]);
                                    }];;// 执行线程任务
[blockOperation start];
```
注意：这会在当前的线程中执行，因为它是根据调用的线程所决定的。
比方说你在主线程中运行它，那么它就是在主线程中执行任务。如果你是在子线程中运行它，那么它就是在子线程中执行任务。

做个简单的实验，我们新建一条子线程，然后在子线程里调用NSBlockOperation
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSBlockOperation *blockOperation = [NSBlockOperation
                                        blockOperationWithBlock:^{
                                            [NSThread sleepForTimeInterval:2];
                                            NSLog(@"one - %@", [NSThread currentThread]);
                                            // print: one - <NSThread: 0x7f8ac2e1d0b0>{number = 2, name = (null)}
                                        }];;
    
    [blockOperation start];
});
```
它将打印：one - <NSThread: 0x7f8ac2e1d0b0>{number = 2, name = (null)}，因此这个理论是正确的

我们也可以使它并发执行，通过使用addExecutionBlock方法添加多个Block，这样就能使它在主线程和其它子线程中工作。
```objc
NSBlockOperation *blockOperation = [NSBlockOperation
                                    blockOperationWithBlock:^{
                                        NSLog(@"one - %@", [NSThread currentThread]);
                                    }];;

[blockOperation addExecutionBlock:^{
    NSLog(@"two - %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"three - %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"four - %@", [NSThread currentThread]);
}];

[blockOperation start];
```

它将打印:
```objc
two - <NSThread: 0x7fea8a70b000>{number = 3, name = (null)}
one - <NSThread: 0x7fea8a558a40>{number = 4, name = (null)}
four - <NSThread: 0x7fea8a406b90>{number = 1, name = main}
three - <NSThread: 0x7fea8a436e40>{number = 2, name = (null)}
```
大家都看到，即使我们通过使用addExecutionBlock方法使它并发执行任务，但是它也依旧会在主线程执行，因此我们就需要使用NSOperationQueue了。

##### NSOperationQueue
这里介绍一下NSOperation的依赖关系，依赖关系会影响线程的执行顺序：
```objc
// 创建操作队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];

// 线程A
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op1");
    [NSThread sleepForTimeInterval:2];
}];

// 线程B
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op2");
}];

// 线程C
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op3");
    [NSThread sleepForTimeInterval:2];
}];

// 线程B依赖线程C，也就是等线程C执行完之后才会执行线程B
[op2 addDependency:op3];
// 线程C依赖线程A，同上，只不过依赖对象改成了线程A
[op3 addDependency:op1];

// 为队列添加线程
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
```
当你没添加依赖时,队列是并行执行的。
**注意：依赖关系可以多重依赖，但不要建立循环依赖。**

###### Cocoa operations线程间通信－调用主线程修改UI：
```objc
// 创建线程对象（并发）
NSBlockOperation *blockOperation = [[NSBlockOperation alloc] init];

// 添加新的操作
[blockOperation addExecutionBlock:^{
    NSLog(@"two - %@", [NSThread currentThread]);
}];

// 添加新的操作
[blockOperation addExecutionBlock:^{
    NSLog(@"three - %@", [NSThread currentThread]);
    // 在主线程修改UI
    NSOperationQueue *queue = [NSOperationQueue mainQueue];
    [queue addOperationWithBlock:^{
        [self editUINow];
    }];
}];

[blockOperation start];
```

**NSoperation方法及属性:**
```objc
// 设置线程的最大并发数
@property NSInteger maxConcurrentOperationCount;

// 线程完成后调用的Block
@property (copy) void (^completionBlock)(void);

// 取消线程
- (void)cancel;
```
只列举上面那些，其他的方法就不全列出来了。
**注意：在NSOperationQueue类中，我们可以使用cancelAllOperations方法取消所有的线程。这里需要说明一下，不是执行cancelAllOperations方法时就会马上取消，是等当前队列执行完，下面的队列不会再执行。**

#### Grand Central Dispatch (GCD)

##### 1)GCD异步线程的创建
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"线程 - %@", [NSThread currentThread]);
});
```
GCD也可以创建同步的线程，只需要把async改成sync即可。

##### 2)重复执行线程: dispatch_apply
以下代码会执行4次:
```objc
dispatch_apply(4, DISPATCH_QUEUE_PRIORITY_DEFAULT, ^(size_t index) {
    // index则为执行的次数 0开始递增
    NSLog(@"one - %ld", index);
});
```
index参数为执行的次数，从0开始递增。

其中需要注意的是，每次执行都会开辟一条子线程，因为是异步的原因，它们不会是顺序的。
```objc
[657:159159] one - 0, thread - <NSThread: 0x100110b50>{number = 1, name = main}
[657:159191] one - 2, thread - <NSThread: 0x103800000>{number = 2, name = (null)}
[657:159192] one - 3, thread - <NSThread: 0x100112b90>{number = 3, name = (null)}
[657:159190] one - 1, thread - <NSThread: 0x100501180>{number = 4, name = (null)}
```
然而，GCD还有一次性执行的方法:
```objc
dispatch_once_t once;
dispatch_once(&once, ^{
    NSLog(@"once - %@", [NSThread currentThread]); // 主线程
});
```
它通常用于创建单列

##### 3)操作队列:dispatch_queue_create
使用GCD也能创建串行队列，具体代码如下:
```objc
/**
 *  GCD创建串行队列
 *
 *  @param "com.GarveyCalvin.queue"  队列字符串标识
 *  @param DISPATCH_QUEUE_CONCURRENT 可选的，可以是NULL
 *
 *  @return dispatch_queue_t
 */
dispatch_queue_t queue = dispatch_queue_create("com.GarveyCalvin.queue", DISPATCH_QUEUE_CONCURRENT);

// 线程A
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:1];
    NSLog(@"sleep async - %@", [NSThread currentThread]);
});

// 线程B
dispatch_barrier_async(queue, ^{
    [NSThread sleepForTimeInterval:3];
    NSLog(@"sleep barrier2 - %@", [NSThread currentThread]);
});

// 线程C
dispatch_async(queue, ^{
    NSLog(@"async");
});
```
运行效果：以上会先执行 线程A－》线程B－》线程C，它是一个串行队列。

dispatch_queue_create的第二个参数：

1）DISPATCH_QUEUE_SERIAL(串行)

2）DISPATCH_QUEUE_CONCURRENT(并发)

##### 4)GCD群组通知: dispatch_group_t
GCD的高级用法，等所有线程都完成工作后，再作通知。
```objc
// 创建群组
dispatch_group_t group = dispatch_group_create();

// 线程A
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"group1");
    [NSThread sleepForTimeInterval:2];
});

// 线程B
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"group2");
});

// 待群组里的线程都完成之后调用的通知
dispatch_group_notify(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"group success");
});
```
群组里的线程也是并行队列。线程A和线程B都执行完之后，会调用通知打印group success。

##### 5)GCD实现计时器
```objc
__block int time = 30;
CGFloat reSecond = 1.0;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, reSecond * NSEC_PER_SEC, 0);
dispatch_source_set_event_handler(timer, ^{
    time--;
    NSLog(@"%d", time);
    if (time == 0) {
        dispatch_source_cancel(timer);
    }
});
dispatch_resume(timer);
```
代码效果：创建了一个计时器，计时器运行30秒，每过一秒会调用一次block，我们可以在block里面写代码。因为dispatch_source_t默认是挂起状态，因此我们使用时需要使用dispatch_resume方法先恢复，不然线程不会执行。

**GCD线程间通信-调用主线程修改UI:**
有时候我们请求后台作数据处理，数据处理是异步的，数据处理完成后需要更新UI，这时候我们需要切换到主线程修改UI，例子如下：
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"异步数据处理 - %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"数据处理完成");
    
    // 调用主线程更新UI
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"更新UI - %@", [NSThread currentThread]);
        [self editUINow];
    });
});
```
##### GCD方法及属性:
```objc
// 获取主线程
dispatch_get_main_queue()

// 创建队列：第一个参数是队列的名称，它会出现在调试程序等之中，是个内部名称。第二个参数代表它是串行队列还是并并发队列，NULL代表串行队列。
dispatch_queue_t dispatch_queue_create(const char *label, dispatch_queue_attr_t attr);

// 创建异步调度队列
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);

// 恢复队列
void dispatch_resume(dispatch_object_t object);

// 暂停队列
void dispatch_suspend(dispatch_object_t object);
```
小结：本文主要介绍了IOS三种线程对比及其使用方法。需要特别注意的是，在修改任何有关于UI的东西，我们必须要切换至主线程，在主线程里修改UI，避免不必要的麻烦产生。苹果是推荐我们使用GCD，因为GCD是这三种里面抽象级最高的，使用起来也简单，也是消耗资源最低的，并且它执行效率比其它两种都高。因此，能够使用GCD的地方，尽量使用GCD。
###### 后台运行
```objc
AppDelegate.h
@interface AppDelegate ()

@property (assign, nonatomic) UIBackgroundTaskIdentifier backGroundUpdate;

@end

AppDelegate.m
- (void)applicationDidEnterBackground:(UIApplication *)application {
    [self beginBackGroundUpdate];
   // 需要长久运行的代码
    [self endBackGroundUpdate];
}

- (void)beginBackGroundUpdate
{
    self.backGroundUpdate = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        [self endBackGroundUpdate];
    }];
}

- (void)endBackGroundUpdate
{
    [[UIApplication sharedApplication] endBackgroundTask:self.backGroundUpdate];
    self.backGroundUpdate = UIBackgroundTaskInvalid;
}
```
建议大家在真机上测试，因为我在模拟器测试了24分钟还有效。
##### 延迟执行
如果我们想要某段代码延迟执行，那么可以使用dispatch_after ，但是有一个缺点是，当提交代码后（代码执行后），我们不能取消它，它将会运行。另外，我们可以使用 NSTimer 进行延时操作，值得一提，它是可以被取消的。
```objc
dispatch_time_t time_t = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(time * NSEC_PER_SEC));
dispatch_queue_t queue = dispatch_get_main_queue();
dispatch_after(time_t, queue, ^{
    NSLog(@"hahalo");
});
```
#### 比较多线程技术:

##### 一、Thread:
优点:量级较轻。
缺点:需要自己管理线程的生命周期，线程同步。线程同步对数据加锁会有一定的系统开销。
#### 二、Cocoa operations:
优点:不需要关心线程管理，数据同步的事情，可以把精力放在自己需要执行的操作上。
#### 三、Grand Central Dispatch (GCD)
优点:GCD基于C的API，非常底层，充分利用多核，能够轻松在多核系统上高效运行并发代码，也是苹果推荐使用的多线程技术。

本文参考:
[GCD的另一个用处是可以让程序在后台较长久的运行](http://blog.sina.com.cn/s/blog_af73e7a70102uykk.html)
[全面掌握iOS多线程攻略 ](http://mobile.51cto.com/iphone-403490.htm)
[iOS多线程的初步研究（一）-- NSThread ](http://www.cnblogs.com/sunfrog/p/3243230.html)