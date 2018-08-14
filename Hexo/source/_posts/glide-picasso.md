---
title: Glide 和 Picasso 对比.
date: 2017-05-29 10:16:47
category: Android 进阶
---
Glide 和 Picasso 可以说是目前 Android 上最流行的图片加载库了。大部分安卓应用开发人员都有使用过这两个库在他们的开发工作中。这两个库也都确实提供了大量图片加载的功能，而且也都经过了很多应用的检验，是可靠可信的。表面看上去似乎两者工作原理很相似，但是实际上是有着很大差别的，主要体现在下面几个方面：

- 下载图片的方式
- 图片的缓存机制
- 加载到内存的机制

本文主要会围绕这几个方面来深入研究和对比两个库的差异，从而给开发者们提供参考。
> 对比的版本是 Glide v3.7.0 和 Picasso v2.5.2 的版本。

### 导入库到项目中
Picasso 和 Glide 都在 Jcenter 上有建立库，所以只需要简单的在添加dependency 即可

#### Picasso
```java
dependencies {
    compile 'com.squareup.picasso:picasso:2.5.1'
}
```

#### Glide
```java
dependencies {
    compile 'com.github.bumptech.glide:glide:3.5.2'
}
```

### 库的大小和方法
对比两个.jar 库的大小，Glide 要比 Picasso 大很多，基本上是 Picasso 的3.5倍：
![glide](/uploads/glide1.png)
从库的大小，我们就可以预见，Glide 的方法必然是要大于Picasso 的，Picasso 的方法 总共有849个，而 Glide 的有2678个：
![glide](/uploads/glide2.png)

### 使用方式
如果只是简单的从一个 URL 中下载图片，然后显示到 imageView 中，那么两个库的使用方式基本相似，也都非常的简单。同时两个库也都支持动画和大小的剪切，也可以设置加载时候的预设图片等功能：

#### Picasso:
```java
Picasso.with(myFragment)
.load(url)
.centerCrop()
.placeholder(R.drawable.loading_spinner)
.into(myImageView);
```

#### Glide
```java
Glide.with(myFragment)
.load(url)
.centerCrop()
.placeholder(R.drawable.loading_spinner)
.crossFade()
.into(myImageView);
```
但是，Glide 这里有一个非常招人喜欢的地方，就是 Glide 在设计的时候，就有 Activity 和 Fragment 的生命周期。什么意思呢？ 就是说你可以传递 Activity 或者 Fragment 的 context 给 Glide.with()， 然后 Glide 就会非常智能的同 Activity 的生命周期集成， 比如 OnResume 或者 onPause():
![glide](/uploads/glide3.png)

### 缓存大小
两个库也都支持缓存图片，都通过下载图片后，缓存到本地。但是这里对于缓存本地的机制，两个库是完全不同的做法。

** Picasso **是下载图片然后缓存完整的大小到本地，比如说图片的大小是1080p的，之后如果我需要同一张图片，就会返回这张 full size 的，如果我需要resize，也是对这种 full size 的做 resize。

** Glide ** 则是完全不一样的做法。Glide 是会先下载图片，然后改变图片的大小，以适应 imageView 的要求，然后缓存到本地。 所以如果你是下载同一张图片，但是设定两个不一样大小的 imageView, 那么Glide 实际上是会缓存两份。
换个角度来看，这里不仅仅是缓存的问题，比如一个 ImageView 要改变它的大小，PIcasso 就只需要下载一次 full size 的图片，但是 Glide 实际上就不仅仅是下载一次了，它需要去单独下载然后改变大小适配 imageView，因为对于 Glide 来讲，需要缓存不同大小的同一张图片。

从这点来看，似乎 Glide 的这种设计很有问题了？当然不是，这种做法也会带来一定的好处，在下面的memory中就会展示。

### 内存使用
Glide 默认是用的 RGB_555 的设定，PIcasso 则是用的 ARGB _8888的设定。为了公平起见，我这里修改了 GlideModule，让 Glide 也使用 ARGB _8888的格式，做法也很简单，新建一个类然后继承 GlideModule

```java
<meta-data android:name="example.com.myanimation.GlideConfiguration" android:value="GlideModule"/>
```

```java
public class GlideConfiguration implements GlideModule {
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // Apply options to the builder here.
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
    }
    @Override
    public void registerComponents(Context context, Glide glide) {
        // register ModelLoaders here.
    }
}
```
#### 下面是两者加载的对比:
![glide](/uploads/glide4.png)

### 加载图片的时间
这里先说明下，当尝试加载一个图片的时候，两个库都会采用先从缓存中读取，如果缓存中没有，再去下载的做法。
实际试验中，Picasso 会比 Glide 快一点。猜测可能的原因还是因为之前讲到的缓存机制导致，因为Picasso 是直接把图加载到内存中，而 Glide 则需要改变图片大小再加载到内存中去。这个应该是会耗费一定的时间。

![glide](/uploads/glide5.gif)

但是，当加载图片从内存中的时候，Glide 则比 Picasso 要快。其原理还是因为缓存机制的区别。因为Picasso 从缓存中拿到的图片，还要先去 resize 后，然后设定给 imageView，但是 Glide 则不需要这样。

![glide](/uploads/glide6.gif)

### 其他功能的对比
- GIF 支持：Glide 支持 GIF。 对于加载 GIF 来说，Glide 只需要简单使用 Glide.with(...).load(...)。 但是 Picasso 是不支持的，因此如果你的应用中是需要加载 GIF 的话，那就只能用 Glide 了。

- 灵活性：Glide 提供了非常多的配置，你可以非常灵活的根据你的需求来客制化，从而缩减 Glide 库的大小等。

### 结论
正所谓人无完人，经过一番对比，Picasso 和 Glide 各有千秋，那么到底我们应该用哪个库呢？这个还是回到应用的需求来看，比如你想要你的 app 小一些，没有那么多的额外功能，那么 Picasso 是你的首选。反之，比如你的应用中需要加载 GIF，或者对于内存的大小比较在意，那么 Glide 应该是不错的选择。
实际上，就我个人来看，Glide 要略优于 Picasso. 特别是支持 GIF，这个杀手锏啊！