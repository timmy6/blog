---
title: iOS架构：MVVM设计模式 + RAC响应式编程
date: 2019-02-27 09:49:01
category: iOS开发
---
### 一：为什么要使用MVVM?
为什么要用MVVM？只是因为它不会让我时常懵逼。



每次做完项目过后，都会被自己庞大的 ViewController 代码吓坏，不管是什么网络请求、网络数据处理、跳转交互逻辑统统往 ViewController 里面塞，就算是自己写的代码，也不敢直视。我不得不思考是不是MVC模式太过落后了，毕竟它叫做 Massive View Controller，其实说 MVC 落后不太合理，说它太原始了比较合适。



MVC 模式的历史非常的久远，它其实不过是对工程代码的一种模块化，不管是 MVVM、MVCS、还是听起来就毛骨悚然的 VIPER，都是对 MVC 标准的三个模块的继续划分，细分下去，使每个模块的功能更加的独立和单一，而最终目的都是为了提升代码的规范程度，解耦，和降低维护成本。具体用什么模式需要根据项目的需求来决定，而这里笔者就 MVVM 架构和设计，浅谈拙见。

### 二：MVVM模块划分
传统的MVC模式分为：Model、View、Controller。Model 是数据模型，有胖瘦之分，View 负责界面展示，而 Controller 就负责剩下的逻辑和业务。



MVVM 模式多了一个 ViewModel，它的作用是为 Controller 减负，将Controller里面的逻辑（主要是弱业务逻辑）转移到自身，其实它涉及到的工作不止是这些，还包括页面展示数据的处理等。（后序章节会有具体讲解）

![shortcuts](/uploads/mvvm.png)

我的设计是这样的：
- 一个 View 对应一个 ViewModel，View 界面元素属性与 ViewModel 处理后的数据属性绑定
- Model 只是在有网络数据的时候需要创建，它的作用只是一个数据的中专站，也就是一个极为简介的瘦 Model
- 这里弱化了 Model 的作用，而将对网络数据的处理的逻辑放在 ViewModel 中，也就是说，只有在有网络数据展示的 View 的 ViewModel 中，才会看见 Model 的影子，而处理过后的数据，将变成 ViewModel 的属性，注意一点，这些属性一定要尽量“直观”，比如能写成UIImage 就不要写成 URL
- ViewModel 和 Model 可以视情况看是否需要属性绑定
- Controller 的作用就是将主View通过与之对应的 ViewModel 初始化，然后添加到 self.view，然后就是监听跳转逻辑触发等少部分业务逻辑，当然，ViewController 的路由跳转还可以解放出来。

**注意：这里面提到的绑定，其实就是对属性的监听，当属性变化时，监听者做一些逻辑处理，由此涉及到一个框架————RAC**

### ReactiveCocoa
RAC是一个强大的工具，它和MVVM模式的结合使用只能用一个词形容 ——— 完美。


当然，有些开发者不太愿意用这些东西，大概是因为他们觉得这破坏了代理、通知、监听、闭包等的逻辑观感。但是笔者 MVVM 搭建思路里面会涉及大量的属性绑定、事件传递，运用 RAC 能大量简化代码，使逻辑更加的清晰。

在这之前，如果你没有用过RAC，请先Google搜索：RAC
大致的了解一下RAC过后，便可以往下（^）

四：MVVM模块具体实现
这是要实现的界面：

#### 1.Model
这里弱化了 Model，只是做为数据模型使用。只有在 View 需要显示网络数据的时候，对应的 ViewModel 里面才有 Model 的相关处理。

#### 2.ViewModel
在实际开发当中，一个 View 对应一个 ViewModel，主 View 对应并且绑定一个主 ViewModel。



主 ViewModel 承担了网络请求、点击事件协议、初始化子 ViewModel 并且给子 ViewModel 的属性赋初值；网络请求成功返回数据过后，主 ViewModel 还需要给子 ViewModel 的属性赋予新的值。



主 ViewModel 的观感是这样的：

```objc
@interface MineViewModel : NSObject

//viewModel
@property (nonatomic, strong) MineHeaderViewModel *mineHeaderViewModel;
@property (nonatomic, strong) NSArray<MineTopCollectionViewCellViewModel *> *dataSorceOfMineTopCollectionViewCell;
@property (nonatomic, strong) NSArray<MineDownCollectionViewCellViewModel *> *dataSorceOfMineDownCollectionViewCell;

//RACCommand
@property (nonatomic, strong) RACCommand *autoLoginCommand;

//RACSubject
@property (nonatomic, strong) RACSubject *pushSubject;

@end
```

其中，RACCommand 是放网络请求的地方，RACSubject 相当于协议，这里用于点击事件的代理，而 ViewModel 下面的一个 ViewModel 属性和三个装有 ViewModel 的数组需要着重说一下。



在iOS开发中，我们通常会自定义 View，而自定义的 View 有可能是继承自 UICollectionviewCell（UITableViewCell、UITableViewHeaderFooterView 等），当我们自定义一个 View 的时候，这个 View 不需要复用且只有一个，我们就在主 ViewModel 声明一个子 ViewModel 属性，当我们自定义一个需要复用的 cell、item、headerView 等的时候，我们就在主 ViewModel 中声明数组属性，用于储存复用的 cell、item 的 ViewModel，中心思想仍然是一个 View 对应一个 ViewModel。



在.m文件中，对这些属性做懒加载处理，并且将 RACCommand 和 RACSubject 配置好，方便之后在需要的时候触发以及调用，代码如下：
```objc
@implementation MineViewModel

- (instancetype)init
{
    self = [super init];
    if (self) {
        [self initialize];
    }
    return self;
}
- (void)initialize {
    [self.autoLoginCommand.executionSignals.switchToLatest subscribeNext:^(id responds) {
        //处理网络请求数据
        ......
    }];
}

#pragma mark *** getter ***
- (RACSubject *)pushSubject {
    if (!_pushSubject) {
        _pushSubject = [RACSubject subject];
    }
    return _pushSubject;
}
- (RACCommand *)autoLoginCommand {
    if (!_autoLoginCommand) {
        _autoLoginCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
            return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
                NSDictionary *paramDic = @{......};
                [Network start:paramDic success:^(id datas) {
                    [subscriber sendNext:datas];
                    [subscriber sendCompleted];
                } failure:^(NSString *errorMsg) {
                    [subscriber sendNext:errorMsg];
                    [subscriber sendCompleted];
                }];
                return nil;
            }];
        }];
    }
    return _autoLoginCommand;
}
- (MineHeaderViewModel *)mineHeaderViewModel {
    if (!_mineHeaderViewModel) {
        _mineHeaderViewModel = [MineHeaderViewModel new];
        _mineHeaderViewModel.headerBackgroundImage = [UIImage imageNamed:@"BG"];
        _mineHeaderViewModel.headerImageUrlStr = nil;
        [[[RACObserve([LoginBackInfoModel shareLoginBackInfoModel], headimg) distinctUntilChanged] takeUntil:self.rac_willDeallocSignal] subscribeNext:^(id x) {
            if (x == nil) {
                _mineHeaderViewModel.headerImageUrlStr = nil;
            } else {
                _mineHeaderViewModel.headerImageUrlStr = x;
            }
        }];
        ......
    return _mineHeaderViewModel;
}
- (NSArray<MineTopCollectionViewCellViewModel *> *)dataSorceOfMineTopCollectionViewCell {
    if (!_dataSorceOfMineTopCollectionViewCell) {
        MineTopCollectionViewCellViewModel *model1 = [MineTopCollectionViewCellViewModel new];
        MineTopCollectionViewCellViewModel *model2 = [MineTopCollectionViewCellViewModel new];
        ......
        _dataSorceOfMineTopCollectionViewCell = @[model1, model2];
    }
    return _dataSorceOfMineTopCollectionViewCell;
}
- (NSArray<MineDownCollectionViewCellViewModel *> *)dataSorceOfMineDownCollectionViewCell {
    if (!_dataSorceOfMineDownCollectionViewCell) {
        ......
    }
    return _dataSorceOfMineDownCollectionViewCell;
}

@end
```

为了方便，我直接将以前写的一些代码贴上来了，不要被它的长度吓着了，你完全可以忽略内部实现，只需要知道，这里不过是实现了 RACCommand 和 RACSubject 以及初始化子 ViewModel。

是的，主 ViewModel 的主要工作基本上只有这三个。

关于属性绑定的逻辑，我将在之后讲到。
我们先来看看子 ViweModel 的观感：

```objc

@interface MineTopCollectionViewCellViewModel : NSObject
@property (nonatomic, strong) UIImage *headerImage;
@property (nonatomic, copy) NSString *headerTitle;
@property (nonatomic, copy) NSString *content;
@end
```
我没有贴.m里面的代码，因为里面没有代码（嘿嘿）。
接下来说说，为什么我设计的子 ViewModel 只有几个单一的属性，而主 ViewModel 却有如此多的逻辑。

首先，我们来看一看 ViewModel 的概念，Model 是模型，所以 ViewModel 就是视图的模型。而在传统的 MVC 中，瘦 Model 叫做数据模型，其实瘦 Model 叫做 DataModel 更为合适；而胖 Model 只是将网络请求的逻辑、网络数据处理的逻辑写在了里面，方便于 View 更加便捷的展示数据，所以，胖 Model 的功能和 ViewModel 大同小异，笔者把它叫做“少根筋的 ViewModel”。

这么一想，我们似乎应该将网络数据处理的逻辑放在子 ViewModel 中，来为主 ViewModel 减负。

笔者也想这么做。
但是有个问题，举个简单的例子，比如这个需求：

![shortcuts](/uploads/mvvm_user_info.png)

一般的思路是自定义一个 CollectionviewCell 和一个 ViewModel，因为它们的布局是一样的，我们需要在主 ViewModel 中声明一个数组属性，然后放入两个 ViewModel，分别对应两个 Cell。

image 和 title 这种静态数据我们可以在主ViewModel中为这两个子ViewModel赋值，而下方的具体额度和数量来自网络，网络请求下来的数据通常是:
```java
{
    balance:"100"
    redPacket:"3"
}
```
我们需要把”100“转化为”100元“，”3“转化为”3个“。
这个网络数据处理逻辑按正常的逻辑来说是应该放在 ViewModel 中的，但是有个问题，我们这个 collectionviewcell 是复用的，它的 ViewModel 也是同一个，而处理的数据是两个不同的字段，我们如何区分？而且不要忘了，网络请求成功获得的数据是在主 ViewModel 中的，还涉及到传值。再按照这个思路去实现必然更为复杂，所以我干脆一刀切，不管是静态数据还是网络数据的处理，通通放在主 ViewModel 中。

这样做虽然让主 ViewModel 任务繁重，子 ViewModel 过于轻量，但是带来的好处却很多，一一列举：
- 在主 ViewModel 的懒加载中，实现对子 ViewModel 的初始化和赋予初值，在RACCommand 中网络请求成功过后，主 ViewModel 需要再次给子 ViewModel 赋值。赋值条理清晰，两个模块。
- 子 ViewModel 只放其对应的 View 需要的数据属性，作用相当于 Model，但是比 Model 更加灵活，因为如果该 View 内部有着一些点击事件等，我们同样可以在子 ViewModel 中添加RACSubject（或者协议）等，子 ViewModel 的灵活性很高。
- 不管是静态数据还是网络数据统一处理，所有子 ViewModel 的初始化和属性赋值放在一块儿，所有网络请求放在一块儿，所有 RACSubject 放在一块儿，结构更加清晰，维护方便。

#### 3.View
之前讲到，ViewModel 和 Model 交互的唯一场景是有网络请求数据需要展示的情况，而 View 和 ViewModel 却是一一对应，绑不绑定需要视情况而定。下面详细介绍。

自定义View这里分两种情况，分别处理：

##### （1）非继承有复用机制的 View（不是继承 UICollectionviewCell 等）

这里以界面的主 View 为例

.h
```java
- (instancetype)initWithViewModel:(MineViewModel *)viewModel;
```
该 View 需要和 ViewModel 绑定，实现相应的逻辑和触发事件，并且保证 ViewModel 的唯一性。
.m
这里就不贴代码了，反正 View 与 ViewModel 的交互无非就是触发网络请求、触发点击事件、将 ViewModel 的数据属性展示在界面上。


##### （2）继承有复用机制的 View（UICollectionviewCell 等）

最值得注意的地方就是 cell、item 的复用机制问题了。
我们在自定义这些 cell、item 的时候，并不能绑定相应的 ViewModel，因为它的复用原理，将会出现多个 cell（item）的 ViewModel 一模一样，在这里，笔者使用了如下的解决方案：
首先，在自定义的 cell（item）.h 中声明一个 ViewModel 属性。
```objc
#import <UIKit/UIKit.h>
#import "MineTopCollectionViewCellViewModel.h"

@interface MineTopCollectionViewCell : UICollectionViewCell
@property (nonatomic, strong) MineTopCollectionViewCellViewModel *viewModel;
@end
```

然后，在该属性的setter方法中给该cell的界面元素赋值：
```objc
#pragma mark *** setter ***
- (void)setViewModel:(MineTopCollectionViewCellViewModel *)viewModel {
    if (!viewModel)  return;
    _viewModel = viewModel;

    RAC(self, contentLabel.text) = [[RACObserve(viewModel, content) distinctUntilChanged] takeUntil:self.rac_willDeallocSignal];
    self.headerImageView.image = viewModel.headerImage;
    self.headerLabel.text = viewModel.headerTitle;
}
```
**ps:这里再次看到RAC（）和RACObserve（）这两个宏，这是属性绑定，如果你不懂，可以先不用管，在后面我会讲解一下我的属性绑定思路，包括不使用 ReactiveCocoa 达到同样的效果。**

重写setter的作用大家应该知道吧，就是在 collectionView 的协议方法中写到：
```objc
cell.viewModel = self.viewModel.collectionCellViewModel;
```
的时候，能够执行到该setter方法中，改变该cell的布局。

好吧，这就是精髓，废话不说了。

想了一下，还是贴上主 View 的 .m 代码吧：
```objc
@interface MineView () <UICollectionViewDelegate, UICollectionViewDataSource, UICollectionViewDelegateFlowLayout>
@property (nonatomic, strong) UICollectionView *collectionView;
@property (nonatomic, strong) MineViewModel *viewModel;
@end

@implementation MineView
- (instancetype)initWithViewModel:(MineViewModel *)viewModel
{
    self = [super init];
    if (self) {
        self.backgroundColor = [UIColor colorWithRed:243/255.0 green:244/255.0 blue:245/255.0 alpha:1];
        self.viewModel = viewModel;
        [self addSubview:self.collectionView];

        [self setNeedsUpdateConstraints];
        [self updateConstraintsIfNeeded];

        [self bindViewModel];
    }
    return self;
}
- (void)updateConstraints {
    [self.collectionView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.mas_equalTo(self);
    }];
    [super updateConstraints];
}
- (void)bindViewModel {
    [self.viewModel.autoLoginCommand execute:nil];
}

#pragma mark *** UICollectionViewDataSource ***
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView {
    return 3;
}
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section {
    if (section == 1) return self.viewModel.dataSorceOfMineTopCollectionViewCell.count;
    ......
}
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
    if (indexPath.section == 1) {
        MineTopCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:[NSString stringWithUTF8String:object_getClassName([MineTopCollectionViewCell class])] forIndexPath:indexPath];
            cell.viewModel = self.viewModel.dataSorceOfMineTopCollectionViewCell[indexPath.row];
            return cell;
    }
    ......
}
- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath {
    ......
}

#pragma mark *** UICollectionViewDelegate ***
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath {
    [self.viewModel.pushSubject sendNext:nil];
}

#pragma mark *** UICollectionViewDelegateFlowLayout ***
    ......
#pragma mark *** Getter ***
- (UICollectionView *)collectionView {
    if (!_collectionView) {
        ......
    }
    return _collectionView;
}
- (MineViewModel *)viewModel {
    if (!_viewModel) {
        _viewModel = [[MineViewModel alloc] init];
    }
    return _viewModel;
}

@end
```
##### Controller
这家伙已经解放了。
```objc
@interface MineViewController ()
@property (nonatomic, strong) MineView *mineView;
@property (nonatomic, strong) MineViewModel *mineViewModel;
@end

@implementation MineViewController

#pragma mark *** life cycle ***
- (void)viewDidLoad {
    [super viewDidLoad];
    self.hidesBottomBarWhenPushed = YES;
    [self.view addSubview:self.mineView];
    [self bindViewModel];
}
- (void)updateViewConstraints {
    [self.mineView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.mas_equalTo(self.view);
    }];
    [super updateViewConstraints];
}
- (void)bindViewModel {
    @weakify(self);
    [[self.mineViewModel.pushSubject takeUntil:self.rac_willDeallocSignal] subscribeNext:^(NSString *x) {
        @strongify(self);
        [self.navigationController pushViewController:[LoginViewController new] animated:YES];
    }];
}

#pragma mark *** getter ***
- (MineView *)mineView {
    if (!_mineView) {
        _mineView = [[MineView alloc] initWithViewModel:self.mineViewModel];
    }
    return _mineView;
}
- (MineViewModel *)mineViewModel {
    if (!_mineViewModel) {
        _mineViewModel = [[MineViewModel alloc] init];
    }
    return _mineViewModel;
}
@end
```

是不是非常清爽，清爽得甚至怀疑它的存在感了（_）。

### 五：附加描述
#### 1、绑定思想
我想，懂一些 RAC 的人都知道属性绑定吧，RAC（，）和 RACObserve（，），这是最常用的，它的作用是将 A 类的 a 属性绑定到 B 类的 b 属性上，当 A 类的 a 属性发生变化时，B 类的 b 属性会自动做出相应的处理变化。


这样就可以解决相当多的需求了，比如：用户信息展示界面->登录界面->登录成功->回到用户信息展示界面->展示用户信息


以往我们的做法通常是，用户信息展示界面写一个通知监听->登录成功发送通知->用户信息展示界面刷新布局


当然，也可以用协议、闭包什么的。而使用 RAC 的属性绑定、属性联合等一系列方法，将会有事半功倍的效果，充分的降低了代码的耦合度，降低维护成本，思路更清晰。


在上面这个需求中，需要这样做:


将用户信息展示 View 的属性，比如 self.name，self.phone 等与对应的 ViewModel 中的数据绑定。在主 ViewModel 中，为该子 ViewModel 初始化并赋值，用户信息展示 View 的内容就是这个初始值。当主 ViewModel 网络请求成功过后，再一次给该子 ViewModel 赋值，用户信息展示界面就能展示相应的数据了。



而且，我们还可以做得更好，就像我以上的代码里面做的），将 View 的展示内容与 ViewModel 的属性绑定，将 ViewModel 的属性与 Model 的属性绑定，看个图吧：
![shortcuts](/uploads/mvvm2.png)
只要 Model 属性一变，传递到 View 使界面元素变化，全自动无添加。有了这个东西过后，以后 reloadData 这个方法可能见得就比较少了。

#### 2.整体逻辑梳理
- 1.进入 ViewController，懒加载初始化主 View（调用-initWithViewMdoel方法，保证主 ViewModel 唯一性），懒加载初始化主ViewModel。
- 2.进入主 ViewModel，初始化配置网络请求、点击逻辑、初始化各个子 ViewModel。
- 3.进入主 View，通过主 ViewModel 初始化，调用 ViewModel 中的对应逻辑和对应子ViewModel 展示数据。
- 4.ViewController 与 ViewModel 的交互主要是跳转逻辑等。

#### 3.创建自己的架构
其实在任何项目中，如果某一个模块代码量太大，我们完全可以自己进行代码分离，只要遵循一定的规则（当然这是自己定义的规则），最终的目的都是让功能和业务细化，分类。



这相当于在沙滩上抓一把沙，最开始我们将石头和沙子分开，但是后来，发现沙子也有大有小，于是我们又按照沙子的大小分成两部分，再后来发现沙子颜色太多，我们又把不同颜色的沙子分开……



在 MVVM 模式中，完全可以把 ViewModel 的网络请求逻辑提出来，叫做 NetworkingCenter；还可以把 ViewModel 中的点击等各种监听事件提出来，叫做 ActionCenter；还可以把界面展示的 View 的各种配置（比如在 tableView 协议方法中的写的数据）提出来，叫做 UserInterfaceConfigDataCenter；如果项目中需要处理的网络请求数据很多，我们可以将数据处理逻辑提出来，叫做 DataPrecessCenter ……



记住一句话：万变不离其宗。

### 六：结语
移动端的架构一直都是千变万化，没有万能的架构，只有万能的程序员，根据产品的需求选择相应的架构才是正确的做法。MVC 固然古老，但是在小型项目却依然实用；MVVM 虽然很强大，但是在有时候还是会增加代码量；VIPER 看似高大上，实际上应用场景比较少。在实际开发中，不拘泥于某种架构或者将它们结合起来用才是正确的做法。
