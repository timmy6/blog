---
title: iOS中static，const，extern相关的问题
date: 2018-11-20 20:59:01
category: iOS开发
---

### 一、 static
static分两种情况，修饰局部变量和全局变量。
我们首先要搞清楚生命周期和作用域的概念。
- *生命周期：*这个变量能存活多久，它所占用的内存什么时候分配，什么时候收回。
- *作用域：*说白了就是这个变量在什么区域是可见的，可以拿来用的。

#### static修饰局部变量
在函数或者说代码块内部声明的变量叫局部变量。

*局部变量*
局部变量是存储在栈区的，它的生命周期是整个代码块，作用域也是整个代码块，一旦出了这个代码块，存储局部变量的这个栈内存就会被回收，局部变量也就被销毁了。看一个例子：
![art1](https://upload-images.jianshu.io/upload_images/5796542-eeef5743e6099f5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/553/format/webp)
打印结果：
![art1](https://upload-images.jianshu.io/upload_images/5796542-565a909c8bf910fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/451/format/webp)

这个其实很好理解，局部变量a是在test方法的代码块内声明的，所以它的生命周期就是这个代码块，当我们调用完一次test方法后，局部变量a就被销毁了，不存在了。在下一次调用test方法时又在栈区重新申请了内存。
当我们用static修饰局部变量时，变量被称为静态局部变量，这个静态局部变量和全局变量，静态全局变量一样，是存储在静态存储区。**由于存储在静态存储区，所以这块内存直到程序结束才会销毁。也就是说，静态局部变量的生命周期是整个源程序。但是它只在声明它的代码块可见，也就是说它的作用域是声明它的代码块。我们把局部变量a用static修饰：
![art1](https://upload-images.jianshu.io/upload_images/5796542-a36d59e9e20bf39e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/535/format/webp)
看一下打印结果：
![art1](https://upload-images.jianshu.io/upload_images/5796542-6463855ad5a20d6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/460/format/webp)
当我们第一次调用test方法时，在静态存储区申请了一块内存吗，这块内存名字叫a，里面装着数字0，然后把数字0加1，变成了1，当第二次调用test方法时，会去静态存储区查找有没有一块内存叫a，如果有那就不用重新分配内存初始化，这里找到了这块叫a的内存，所以不用进行初始化，所以a里面还是装的1然后对1加1，得到了2.

#### static修饰全局变量
当全局变量没有使用static修饰符时，其存储在静态存储区，直到程序结束才销毁。也就是其作用域是整个源程序。我们可以使用extern关键字来引用这个全局变量。
Test.h:
![art1](https://upload-images.jianshu.io/upload_images/5796542-14de861ac69264f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/686/format/webp)
ViewController.m:
![art1](https://upload-images.jianshu.io/upload_images/5796542-a63d6b030ac5220f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/688/format/webp)
结果：
![art1](https://upload-images.jianshu.io/upload_images/5796542-03de4615b6603425.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400/format/webp)
当全局变量使用static修饰时，其生命周期没有变，依旧是在程序结束时才销毁。但是其作用域变了，以前是整个源程序，现在只限于申明它的这个文件才可见，即使用extern引用也不行，比如我们把上面的例子中globalVar前面用static修饰，那么程序就会报错：
![art1](https://upload-images.jianshu.io/upload_images/5796542-078743a17e4727b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/633/format/webp)

#### 总结：
*static修饰局部变量：*将局部变量的本来分配在栈区改为分配在静态存储区，也就改变了局部变量的生命周期。
*static修饰全局变量：*本来是在整个源程序的所有文件都可见，static修饰后，改为只在申明自己的文件可见，即修改了作用域。

### 二、const
const修饰变量主要强调变量是不可修改的。我们看一下下面代码段：
![art1](https://upload-images.jianshu.io/upload_images/5796542-086535a84235a37d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/705/format/webp)
从结果可以看到变量a的值修改成功，那么我们将变量a用const修饰试一试：
![art1](https://upload-images.jianshu.io/upload_images/5796542-46716470893f1036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/858/format/webp)
使用const修饰变量a之后，a的值就不能修改了，所以这里修改就不成功了。

> 需要注意的一点是，const修饰的是其右边的值，也就是const右边的这个整体的值不能改变。

下面例子是const修饰*str这个整体，所以这个整体不能改变，这个整体是str指向的内存中的值。
![art1](https://upload-images.jianshu.io/upload_images/5796542-1219ddc15ed8269a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/854/format/webp)

下面例子是const修饰str的，所以str指向的地址不能改变，因此这里的改变就产生了错误。
![art1](https://upload-images.jianshu.io/upload_images/5796542-1c00f6e37826b729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/839/format/webp)


一般联合使用static和const来定义一个只能在本文件中使用的，不能修改的变量：
![art1](https://upload-images.jianshu.io/upload_images/5796542-44758319cb382d84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/664/format/webp)

*这样定义相对于用#define来定义的话，优点就在于它指定了变量的类型，而#define是不能指定变量的类型的。*

### 三、extern
extern主要是用来引用全局变量，它的原理就是先在本文件中查找，本文件中查找不到再到其他文件中查找。
常把extern和const联合使用在项目中创建一个文件，这个文件文件中包含整个项目中都能访问的全局常量。比如我们在项目中创建的这个文件为LMConst.h,LMConst.m。
LMConst.h：
![art1](https://upload-images.jianshu.io/upload_images/5796542-45cecb3710c7770f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/715/format/webp)
LMConst.m:

pch文件：
![art1](https://upload-images.jianshu.io/upload_images/5796542-20b5e35b5cf0ab41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/519/format/webp)

然后我们就可以直接在其他文件中使用NOTIFICATION_NAME这个字符串常量了：
![art1](https://upload-images.jianshu.io/upload_images/5796542-e93d0dcff22eb5c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/570/format/webp)
