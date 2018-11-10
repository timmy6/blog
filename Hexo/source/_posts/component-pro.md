---
title: iOS组件化及架构设计
date: 2018-11-10 20:59:01
category: iOS开发
---

> 网上组件化的文章很多。很多文章一提到组件化，就会说解耦，一说到解耦就会说路由或者runtime。好像组件化 == 解耦 == 路由/Runtime，然而这是一个非常错误的观念。持有这一观点的人，没有搞清楚在组件化中什么是想要结果，什么是过程。

#### 组件化和解耦
大家不妨先思考两个问题：
- 为何要进行组件化开发？
- 各个组件之间是否一定需要解耦？
采用组件化，是为了组件能单独开发，能单独开发App就能快速集成，所以单独开发是结果。要让组件能单独开发，组件必须职责单一，对于代码中已有模块，就需要用到重构和解耦的技术，所以重构和解耦是过程。那解耦是否是必须的过程？不一定。比如UIKit，我们用这个系统组件并没有使用任何解耦手段。问题来了，UIKit苹果可以独立开发，我们使用它为什么没用解耦手段？答案很简单，UIKit没有依赖我们的代码所以不用解耦。

>PS：我这里不纠结组件、服务、模块、框架的概念，网上对这些概念的定义五花八门，实际上把简单的事说复杂了。我这里只关心一件事，这一部分代码能否独立开发，能就叫组件，不能我管你叫什么

我们之所以要解耦才能独立开发，通常是出现了循环依赖。这时候当然可以无脑的用路由把两个组件的耦合解开，也可以独立开发。然而，这样做只是把强引用改成了弱引用，代码还是烂代码。站在重构的角度来说，A、B组件循环依赖就是设计有问题，要么应该重构A、B让依赖单向；要么应该抽离一个共用组件C，让A、B组件都只依赖于C。

![art1](https://upload-images.jianshu.io/upload_images/10333442-bc9a765d74cca9a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

如果我们每个组件都只是单向依赖其他组件，各个组件之间也就没有必要解耦。再换个角度说，如果一个组件职责不单一，即使跟其他组件解耦了，组件依然不能很好的工作。如何解耦只是重构过程中可选手段，代码设计的原则如依赖倒置、接口隔离、里氏替换，都可以指导我们写出好的组件。

**所以在组件化中重要的是让组件职责单一，职责单一的重要标志之一就是没有组件间的循环依赖。**

#### 架构图
一般来讲，App的组件可以分为三层，上层业务组件、中层UI组件、底层SDK组件

同一层之间的组件互相独立，上层的组件耦合下层的组件。一般来讲，底层SDK组件和中层UI组件都是独立的功能，不会出现同层耦合。

![art1](https://upload-images.jianshu.io/upload_images/10333442-7464e6a05db1b89e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

同层耦合一般出现在上层业务组件中，常见有两类，一类是controller之间的跳转，这个可以用路由解决；另一类是数据获取，数据获取需要通过重构抽离一个数据模块，耦合这个数据模块可以用协议，这样可以用依赖注入的方式隔离实现。通过协议隔离的好处是，上层业务可以不依赖其他业务的实现单独编译通过，这对上层业务组件的单独开发比较好。

这种方法把所有数据协议放在一起，简单粗暴，但每一个数据协议改动都会导致数据协议层重新发布，不是很合理，可以再进一步细分数据协议层。
业务组件耦个协议:
![art1](https://upload-images.jianshu.io/upload_images/10333442-bb5b6936a5dc0a85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

业务组件自己需要的协议:
![art1](https://upload-images.jianshu.io/upload_images/10333442-88d3fde3e1483d95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)


这种方法把数据协议层分成了多个，业务组件只依赖自己的数据协议。管理起来更麻烦一点，不过架构上更合理。

建议组件化前期用第一种方案，后期细化为第二种。

#### 包管理

要解除循环依赖，引入包管理技术cocoapods会让我们更有效率。pod不允许组件间有循环依赖，若有pod install时就会报错。

cocoapods，提供私有pod repo，使用时把自己的组件放在私有pod repo里，然后在Podfile里直接通过pod命令集成。一个组件对应一个私有pod，每个组件依赖自己所需要的三方库。多个组件联合开发的时候，可以再一个podspec里配置子模块，这样在每个组件自己的podspec里，只需要把子模块里的pod依赖关系拷贝过去就行了。

在多个组件集成时会有版本冲突的问题。比如登录组件（L）、广告组件（A）都依赖了埋点组件（O），L依赖O的1.1版本，A依赖O的1.2版本，这时候集成就会报错。为了解决这个错误，在组件间依赖时，不写版本号，版本号只在APP集成方写。即podfile里引用所有组件，并写上版本号，.podspec里不写版本号。

这样做既可以保证APP集成方的稳定性，也可以解决组件依赖的版本冲突问题。这样做的坏处是，所有组件包括App集成方，在使用其他组件时，都必须使用其他组件最新的API，这会造成额外的升级工作量。如果不想接受组件升级最新api的成本，可以私有化一个三方库自己维护。

*组件开发完毕后告诉集成方，目前的组件稳定版本是多少，引用的三方库稳定版本集成方自己去决定*

![art1](https://upload-images.jianshu.io/upload_images/10333442-c52c37c81a0939c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
推荐的组件版本管理方式

另一种版本管理的方式，是在podspec里写依赖组件的版本号，podfile里不写组件依赖的版本，然后通过内部沟通来解决版本冲突的问题。我认为虽然也能做，但有很多弊端：
- 作为App集成方，没办法单独控制依赖的三方库版本。三方库升级会更复杂
- 每个依赖的三方库，都应该做了完整的单元测试，才能被集成到App中。所以正确的逻辑不是组件内测试过三方库没问题就在组件内写死版本号，而是这个三方库经过我们测试后，可以在我们系统中使用XX版本。
- 在工程中就没有一个地方能完整知道所有的pod组件，而App集成方有权利知道这一点
- 沟通成本高

![art1](https://upload-images.jianshu.io/upload_images/10333442-8eb0637a4286d1e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)


*以上，就是组件化的所有东西。你可能会奇怪，解耦在组件化过程中有什么用。答案是解耦是为了更好的实现组件的单一职责，解耦的作用在架构设计中谈。需要再次强调，组件化 ≠ 解耦。*
如果非要给组件化下一个定义，我的理解是：
> 组件化意味着重构，目的是让每个组件职责单一

#### 关于架构设计
在我看来，iOS客户端架构主要为了解决两个问题，一是解决大型项目分组件开发的效率的问题，二是解决单进程App的稳定性的问题。
设计到架构设计的都是大型App，小型App主要是业务的堆叠。很多公司在业务初期都不会考虑架构，在业务发展到一定规模的时候，才会重新审视架构混乱带来的开发效率和业务稳定性瓶颈。这时候就会引入组件化的概念，我们常常面临的是对已有项目的组件化，这一过程会异常困难。

#### 组件拆分原则
对老工程的组件拆分，我的办法是，从底层开始拆。SDK>  模块 > 业务 。如果App没有SDK可以抽离，就从模块开始拆，不要为了抽离SDK而抽离。常见的误区是，大家一拿到代码就把公共函数提出来作为共用框架，起的名字还特别接地气，如XXCommon。
事实上，这种框架型SDK，是最鸡肋的组件，原因是它实用性很小，无非就是减少了点冗余代码。而且在架构能力不强的情况下，它很容易变成“垃圾堆”，什么东西都想往里面放，后面越来越庞大。所以，开始拆分架构的时候，尽量以业务优先，比如先拆分享模块。

*如果两个组件中有共同的函数，前期不要想着提出来，改个名字让它冗余是更好的办法。* 如果共同耦合的是一个静态库，可以利用动态库的隔离性封装静态库，具体方法可以网上找。

#### 响应式
基础组件常常要在系统启动时初始化，或者接受App生命周期时间。这就引出了个问题，如何给appDelegate瘦身？比如我们现在有两个基础组件A、B，他们都需要监听App生命周期事件，传统的做法是，A、B两个组件都提供一些函数在appDelegate中调用。但这样做的坏处是，如果某一天我不想引入B组件了，还得去改appDelegate代码。理想的方式是，基础组件的使用不需要在appDelegate里写代码

为了实现基础组件与appDelegate分离，得对appDelegate改造。首先得提出一个观点，苹果的appDelegate设计的有问题，它在用代理模式解决观察者模式的问题。在《设计模式》中，代理模式的设计意图定义是：为其他对象提供一种代理以控制对这个对象的访问。反过来看appDelegate你会发现，它大部分代理函数都没有办法控制application，如applicationDidBecomeActive。applicationDidBecomeActive这种事件常常需要多个处理者，这种场景用观察者模式更适合。而openURL需要返回BOOL值，才需要使用代理模式。App生命周期事件虽然可以用监听通知获取，但用起来不如响应式监听信号方便。

基于响应式编程的思想，我写了一个TLAppEventBus，提供属性来监听生命周期事件。我并不喜欢庞大的ReactiveObjectC，所以我通过category实现了简单的响应式，用户只需要监听需要的信号即可。在TLAppEventBus里，我默认提供了8个系统事件用来监听，如果有其他的系统事件需要监听，可以使用扩展的方法，给TLAppEventBus添加属性（见文末Demo）。

#### 路由
对于Appdelegate中的openURL的事件，苹果使用代理模式并没有问题，但我们常常需要在openURL里面写if-else区分事件的处理者，这也会造成多个URL处理模块耦合在Appdelegate中。我认为appdelegate中的openURL应该用路由转发的方式来解耦。

openURL代理需要同步返回处理结果，但网上开源的路由框架能同步返回结果的。所以我这边实现了一个能同步返回结果的路由TLRouter，同时支持了注册scheme。注册scheme这一特性，在第三方分享的场景下会比较有用（见文末Demo）。

另外，网上大部分方案都搞错了场景。以蘑菇街的路由方案为例（好像iOS的路由就是他们提出来的？），蘑菇街认为路由主要有两个作用，一是发送数据让路由接收者处理，二是返回对象让路由发送者继续处理。我不禁想问，这是路由吗？不妨先回到URL的定义

> URL: 统一资源标识符（Uniform Resource Locator,统一资源定位符)是一个用于标识某一互联网资源名称的字符串

openURL就是在访问资源，在浏览器中，openURL意味着打开一个网页，openURL的发起者并不关心打开的内容是什么，只关心打开的结果。所以苹果的openURL Api 就只返回了除了结果YES/NO，没有返回一个对象。所以，我对openURL这一行为定义如下

> openURL：访问资源，返回是否访问成功

那把蘑菇街的路由，返回的对象改成BOOL值就可以了么？我认为还不够。对于客户端的路由，使用的实际上是通知的形式在解耦，带来的问题是路由的注册代码散落在各地，所以路由方案必须要配路由文档，要不然开发者会不知道路由在干嘛。

有没有比文档更好的方式呢？我的思路是：用schema区分路由职责
系统的openURL只干了两件事：打开App ，比如打开微信
> [[UIApplicationsharedApplication] 
> openURL:[NSURLURLWithString:@"weixin://"]];
还有就是打开网页，比如
> [[UIApplicationsharedApplication] openURL:
> [NSURLURLWithString:@"https://www.baidu.com"]];

两者的共性是页面切换。所以我这边设计的路由openURL，只扩充了controller跳转的功能，比如打开登录页
> [TLRouter openURL:@"innerJump://account/login"];

只扩充了controller跳转的功能好处是让路由的职责更单一，同时也更符合苹果对openURL的定义。工程师在看到url schema的时候就知道他的作用，避免反复查看文档。
*对于数据的传递，我认为不应该用路由的方式。相比路由，响应式会是更好的选择。*用响应式的思想，提供属性给相关组件去监听，代码逻辑会更清晰。至于响应式会带来耦合属性这个问题，我在组件化的部分已经讨论过。

#### 依赖注入
依赖注入用于上层业务解耦中，有时多个业务需要用到同一个数据组件。这时可以使用IoC（Inversion of Control）的思想，让上层业务依赖IoC容器，IoC容器创建数据组件，并注入到容器里。具体实现可以通过iOS的协议来实现IoC容器和对象的接口一致，注入对象的获取通过runtime实例化。这样做有个缺点是类名会写死在IoC容器中，如果要动态配置，可以把类名放在plist中让IoC容器去读它。（见Demo TLUserModuleContainer类）

#### App配置
有时候我们需要组件的跨App复用，在App集成组件时，能够不改代码只改配置是最理想的方式。使用组件+plist配置是一个方案，具体做法是把A组件的配置放在A.plist中，在A组件内写死要读取A.plist。
*以配置代替硬编码，防止对代码的侵入，是一个很好的思路。* 设想一下，如果我们可以通过配置在决定App是否使用组件、也可通过配置来改变组件和app所需的参数，那运维可以代替app开发来出包，这对效率和稳定性都会有提升。为了实现这一效果，我使用了OC的runtime来动态注册组件。需要在didfinishLaunch初始化的组件，可以实现代理 - (void)initializeWhenLaunch; 这样，自动初始化函数，就可以通过runtime+plist里配置的class name自动初始化。组件需要初始化的代码，可以在自己的initializeWhenLaunch里做。

由于路由只扩充了controller跳转的功能，所以路由注册这一行为也可进行一次抽象，把不同的部分放在plist配置文件，相同的放到runtime里做。这样做还有个好处是，程序内的路由跳转在一个plist里可以都可以看到
![art1](https://upload-images.jianshu.io/upload_images/10333442-8e60870cb8ed52f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

#### iOS解耦工具Tourelle
Tourelle，是根据上面的思路写的一个开源项目[https://github.com/zhudaye12138/Tourelle](https://github.com/zhudaye12138/Tourelle)可以通过pod集成  pod 'Tourelle'。下面介绍一下他的使用方式

#### TLAppEventBus
TLAppEventBus通过接收系统通知来获取app生命周期事件，收到生命周期事件后改变对应属性的值。默认提供了didEnterBackground等八个属性，可以使用响应式函数来监听 
- (void)observeWithBlock:(TLObservingBlock)block; 
```objc
[TLAppEventBus.shared.didBecomeActive observeWithBlock:^(idnewValue) {

        //do some thing

    }];
```
需要注意，如果在其它地方使用observeWithBlock，需要设置属性的owner，否则没有办法监听到。这里不用单独设置是因为在TLAppEventBus里已设置好
TLAppEventBus使用前需要调用 - (void)start; 如果需要监听更多的事件，可以调用
- (void)startWithNotificationMap:(NSDictionary *)map; 
```objc
NSMutableDictionary *defaultMap = [NSMutableDictionary dictionaryWithDictionary:[TLAppEventBus defaultNotificationMap]]; //获取默认map

    [defaultMapsetObject:KDidChangeStatusBarOrientation forKey:UIApplicationWillChangeStatusBarOrientationNotification]; //添加新的事件

    [TLAppEventBus.shared startWithNotificationMap:defaultMap];//开启EventBus
```
添加新事件需要用分类添加TLAppEventBus的属性，添加后就可正常使用了
```objc
-(void)setDidChangeStatusBarOrientation:(NSNotification*)didChangeStatusBarOrientation {

    objc_setAssociatedObject(self, (__bridge const void *)KDidChangeStatusBarOrientation , didChangeStatusBarOrientation, OBJC_ASSOCIATION_RETAIN_NONATOMIC);

}

-(NSNotification*)didChangeStatusBarOrientation {

    returnobjc_getAssociatedObject(self, (__bridge const void *)KDidBecomeActive);

}
```
#### TLRouter
路由支持两种注册方式，一种只写schema，一种写url路径
```objc
[TLRouter registerURL:@"wx1234567://" hander:^(TLRouterURL *routeURL, void (^callback)(BOOL result)) {       

        //do something     

}]//注册schema

[TLRouter registerURL:@"InnerJump://account/login" hander:^(TLRouterURL *routeURL, void (^callback)(BOOL result)) {

                //do something

 }]//注册url路径
```
支持同步 & 异步获取返回值，其中异步转同步内部通过semaphore实现
```objc
+(void)openURL:(NSString*)url callback:(void(^)(BOOLresult))callback;

+(BOOL)openURL:(NSString*)url;
```

另外openURL除了支持url中带参数，也支持参数放在字典中

```objc
+(BOOL)openURL:(NSString*)url param:(NSDictionary *)param;
```
#### TLAppLaunchHelper
TLAppLaunchHelper有两个函数，一个用来初始化组件。该函数会读取AutoInitialize.plist中的classes，通过runtime + 自动初始化协议完成初始化
![art1](https://upload-images.jianshu.io/upload_images/10333442-501afe909e01028d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

另一个函数用来自动注册路由，该函数会读取AutoRegistURL.plist完成路由注册。其中controller代表类名，params代表默认参数，如果openURL传的参数与默认参数不符合，路由会报错
```objc
-(void)autoRegistURL;
```

![art1](https://upload-images.jianshu.io/upload_images/10333442-829c98cbb704af7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/946/format/webp)
路由注册时，并不决定controller跳转的方式。注册者只是调用presentingSelf方法，跳转方式由controller中presentingSelf方法决定。
```objc
-(BOOL)presentingSelf {

    UINavigationController *rootVC = (UINavigationController *) APPWINDOW.rootViewController;

    if(rootVC) {

        [rootVCpushViewController:self animated:YES];

        returnYES;

   }

    return NO;

}
```

#### 耦合检测工具
针对既有代码的组件化重构，我这边开发了一个耦合检测工具，目前只支持OC。

耦合检测工具的原理是这样：工具认为工程中一级文件夹由组件构成，比如A工程下面有aa、bb、cc三个文件夹，aa、bb、cc就是三个待检测的组件。耦合检测分三步，第一步通过正则找到组件内.h文件中所有关键字（包括函数、宏定义和类）。第二步通过找到的组件内关键字，再通过正则去其它组件的.m中找是否使用了该组件的关键字，如果使用了，两个组件就有耦合关系。第三步，输出耦合检测报告

代码：开源中....

#### 总结
本文给出了组件化的定义：组件化意味着重构，目的是让每个组件职责单一以提升集成效率。包管理技术Pod是组件化常用的工具，iOS组件依赖及组件版本号确定，都可以用pod实现。整个iOS工程的组件通常分为3层，业务组件、模块组件和SDK组件。在老工程重构时，优先抽离SDK组件，切记不要写XXCommon让它变成垃圾堆。

关于解耦的技术，appldegate适合用观察者模式替换代理模式，路由只用来做controller之间的跳转，上层业务组件的解耦更多是靠重构而不是全用路由。工程的组件和路由都可通过runtime + 配置的形式自动注册，这样做维护和集成都会很方便。

Demo地址：https://github.com/zhudaye12138/Tourelle

> 本文转载至：[朱大爷](https://www.jianshu.com/p/d88aef8e29a4)