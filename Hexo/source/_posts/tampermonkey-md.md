---
title: 教你利用Tampermonkey，免VIP观看各大视频网站VIP视频
date: 2018-08-31 22:45:46
category: 有意思的小玩意儿
---
今天心血来潮想看某个电视剧，在家打开打开某视频网站，是这样的：
![shortcuts](/uploads/tampermonkey1.png)
我的内心是这样的：
![shortcuts](/uploads/tampermonkey2.jpg)
高贵的程序员怎么能被小小的VIP拦住自己前进的脚步！所以，今天就向大家推荐一个`chrome`超时空宇宙霹雳无敌轰天响地的插件 —— `Tampermonkey`。

### Tampermonkey是什么
> 不装扩展（Extensions)的chrome只能发挥它40%的能力。

`Tampermonkey`是`chrome`的一个扩展程序，是一个用户脚本管理器，适用于`Chrome`、`Microsoft Edge`，`Safari`，`Opera Next`、`Firfox`。功能之强大，号称第二个`chrome网上应用商店`，利用`Tampermonkey`，用户可以自由定制网页，实现你想要的各种功能，比如*去除广告*、*下载网盘文件*，*破解视频VIP限制*，它提供了脚本安装、自动更新检查、标签中的脚本运行状况速览等管理功能。比如你想要定制某一个功能，那么你只需往`Tampermonkey`里面添加对应的脚本就可以了，而不用再安装其它浏览器插件。相对于插件扩展，脚本更轻量级，不占用太多资源并且只在特定的站点生效。所以在技术人员的圈子了，`Tampermonkey`几乎是万能的。

### 如何安装Tampermonkey
这里已经为你准备了两种方法：
- 如何已经翻墙了，访问chrome网上应用商店，搜索Tampermonkey就可以在线安装了。什么，没有翻墙？*可以找我问服务器代理SS账号密码*，或者请看下一条方法.
- 百度网盘：https://pan.baidu.com/s/1J_ezaJL6Iq8i-mAQsjpZtw，提取码：9jmm，下载完成后，将文件拖到chrome浏览器里，然后松开鼠标，就可以完成安装了。

安装成功之后，在你的`chrome`右上角会出现一个小黑框框，如下图：
![shortcuts](/uploads/tampermonkey3.jpeg)
如果`chrome`提示：无法安装，请在`chrome`右上角，点那*三个点点*——>*更多工具*——>*扩展程序*，然后打开*开发者模式*，如下图：
![shortcuts](/uploads/tampermonkey4.jpeg)
![shortcuts](/uploads/tampermonkey5.png)
完成后，重新将扩展文件拖入`chrome`安装。

### 获取功能脚本
在文章的开头我们有说过，`Tampermonkey`是一个用户脚本管理器，想要去实现*破解VIP视频限制*的功能，我们还需要有一个*VIP视频破解脚本*。这个脚本从哪里来呢？这里就不得不提`Greasy Fork`网站。在`Greasy Fork`网站上还有很多很有用的插件，比如*屏蔽广告*、*下载网盘文件*、*vip视频解析*等。你可以根据个人需要，去网站上找一些实用的脚本。我们今天要用的是一款叫做：*vip视频破解脚本*，[脚本下载地址：https://greasyfork.org/zh-CN/scripts/26556-vip%E8%A7%86%E9%A2%91%E7%A0%B4%E8%A7%A3](https://greasyfork.org/zh-CN/scripts/26556-vip%E8%A7%86%E9%A2%91%E7%A0%B4%E8%A7%A3)
这款VIP视频破解脚本的功能主要有：
- 爱奇艺选集后获取正确解析地址
- 支持腾讯视频选集后自动破解
- 可设置是否自动解析
- 解析后自动停止原始视频播放
- 添加“能嵌则嵌”选项，如果选择了该项且非将http资源嵌入https页面则将接口视频嵌入页面，支持选集

![shortcuts](/uploads/tampermonkey6.jpeg)

安装成功后，点击右上角的`Tampermonkey` ——> 管理面板，可以看到如下界面：
![shortcuts](/uploads/tampermonkey7.jpeg)
![shortcuts](/uploads/tampermonkey8.jpeg)
*到此，恭喜你，安装成功啦，可以愉快的打开视频网站啦~*
![shortcuts](/uploads/tampermonkey9.jpg)
### 实验劳动成果
打开视频网站，比如说`腾讯视频`，打开`《如懿传》`
看到左上角的小问号没有？点它，然后：
- 勾选“能嵌就嵌”.
- 然后再列表下面点击第一个.
如图：
![shortcuts](/uploads/tampermonkey10.png)
好了，可以搬上凳子，端起瓜子愉快的看VIP电视剧啦~

### 内存问题？
有的人会担心插件内存，我们都知道插件是要占用内存的，我们绝对不允许chrome一打开，内存就蹭蹭的向小火箭一样往上升(尤其是Android开发者，Android Studio一开，Android虚拟机一开，内存直接爆炸，所以我现在为了节省内存，虚拟机都不开了，直接用真机)，现在的网页和浏览器本身，都非常占用内存，尤其是像这样一个万能的插件，会不会占用很多内存呢？答案时候，不会的。因为 Tampermonkey 基于代码脚本运行，内存占用非常小，而且每开一个网页，它只会运行相应的脚本。比如说你在看视频，那么只有视频相应的脚本开启。浏览器占内存的常识，在这里可能不能通用了。

### 最后
[Greasy Fory 是专门提供用户脚本的网站，感兴趣的可以上去看看有没有解决你痛点的脚本。链接地址：https://greasyfork.org/zh-CN](https://greasyfork.org/zh-CN)