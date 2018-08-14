---
title: View 事件体系
date: 2016-04-03 22:42:19
category: Android 进阶
---
## View 基本知识
**(1)**view的层次结构: ViewGroup 也是View;

**(2)**view的位置参数: top、left、right、bottom，分别对应View的左上角和右下角相对于父容器的横纵坐标值。
从Android3.0开始,view增加了x、y、translationX、translationY四个参数，这几个参数也是相对于父容器的坐标。x和y是左上角的坐标，而translationX和translationY是view左上角相对于父容器的偏移量，默认值都是0。
```java
x = left + translationX
y = top + translationY
```
**(3)**MotionEvent是指手指接触屏幕后所产生的一系列事件,主要有ACTION_UP、ACTION_DOWN、ACTION_MOVE等。正常情况下，一次手指触屏会出发一系列点击事件，主要有下面两种类型:
1.点击屏幕后离开，事件序列是ACTION_DOWN —> ACTION_UP;
2.点击屏幕后滑动一会再离开，事件序列是ACTION_DOWN -> ACTION_MOVE ->ACTION_MOVE ->…->ACTION_UP;通过MOTIONEVENT可以得到点击事件发生的x和y坐标，其中getX和getY是相对于view左上角的x和y坐标，getRawX和getRawY是相对于手机屏幕左上角的x和y坐标。

**(4)**TouchSlope是系统能识别出的可以被认为是滑动的最小距离，获取方式是ViewConfiguration.get(context).getScaledTouchSlope();

**(5)**VelocityTracker用于追踪手指在滑动过程的速度，包括水平和垂直向上的速度。
速度计算公式：速度 = (终点位置 - 起点位置) / 时间段 速度可能为负值，例如当手指从屏幕右边往左边滑动的时候。此外，速度是单位时间内移动的像素数，单位时间不一定是1秒钟，可以使用方法 computeCurrentVelocity()指定单位时间是多少，单位是ms。例如通过computeCurrentVelocity(1000)来获取速度，手指在1s中滑动了100个像素，那么速度是100，及100(像素/1000ms)。如果computeCurrentVelocity(100)来获取速度，在100ms内手指只是滑动了10个像素，那么速度是10，即10(像素/100ms)。
**VelocityTracker的使用方式**
```java
//初始化
VelocityTracker mVelocityTracker = VelocityTracker.obtain();
//在onTouchEvent方法中
mVelocityTracker.addMovement(event);
//获取速度
mVelocityTracker.computeCurrentVelocity(1000);
float xVelocity = mVelocityTracker.getXVelocity();
//重置和回收
mVelocityTracker.clear();//一般在MotionEvent.ACTION_UP的时候调用
mVelocityTracker.recycle();//一般在onDetachedFromWindow中调用
```

**(6)**GestureDetector 用于辅助检测用户的单击、滑动、长按、双击等行为。GestureDetector的使用比较简单，主要也是辅助检测常见的触屏时间。作者建议：如果只是监听滑动相关的事件在onTouchEvent中实现；如果要监听双击这种行为的话，那么就使用GestureDetector。

**(7)**Scroller分析：参考[http://mrfu.me/2015/12/27/scroll_analyse/](http://mrfu.me/2015/12/27/scroll_analyse/)

## View的滑动
**(1)**常见的实现view的滑动的方式有三种
第一种是通过view本身提供的scrollTo和scrollBy方法：操作简单，适合对view内容的滑动；

第二种是通过动画给view施加平移效果来实现滑动：操作简单，适用于没有交互的view和实现复杂的动画效果；

第三种是通过改变view的LayoutParams使得view重新布局从而实现滑动：操作稍微复杂，适用于有交互的view。

**(2)**scrollTo和scrollBy方法只能改变view内容的位置而不能改变view在布局中的位置。 scrollBy是基于当前位置的相对滑动，而scrollTo是基于所传参数的绝对滑动。通过View的getScrollX和getScrollY方法可以得到滑动的距离。

**(3)**使用动画来移动view主要是操作view的translationX和translationY属性，既可以使用传统的view动画，也可以使用属性动画，使用后者需要考虑兼容性问题，如果要兼容Android 3.0以下版本系统的话推荐使用nineoldandroids。

**(4)**动画兼容库nineoldandroids中的ViewHelper类提供了很多的get/set方法来为属性动画服务，例如setTranslationX和setTranslationY方法，这些方法是没有版本要求的。

## 弹性滑动
**(1)**Scroller的工作原理：Scroller本身并不能实现view的滑动，它需要配合view的computeScroll方法才能完成弹性滑动的效果，它不断地让view重绘，而每一次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔Scroller就可以得出view的当前的滑动位置，知道了滑动位置就可以通过scrollTo方法来完成view的滑动。就这样，view的每一次重绘都会导致view进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，这就是Scroller的工作原理。

**(2)**使用延时策略来实现弹性滑动，它的核心思想是通过发送一系列延时消息从而达到一种渐进式的效果，具体来说可以使用Handler的sendEmptyMessageDelayed(xxx)或view的postDelayed方法，也可以使用线程的sleep方法。

## view的事件分发机制

**(1)**事件分发过程的三个重要方法:
**public boolear dispatchTouchvent(MotionEvent ev)** 用来进行事件的分发。如果事件能够传递给当前view那么此方法一定会被调用，返回结果受当前view的onTouchEvent和下级view的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

**public boolear onInterceptTouchEvent(MotionEvent event)**在dispatchTouchEvent 方法内部调用，用来判断是否拦截某个事件，如果当前view拦截了某个事件，那么在同一个事件序列当中，此方法不会再被调用。返回结果表示是否拦截当前事件。若返回True事件会传递到自己的onTouchEvent();若返回值为False传递到子view的dispatchTouchEvent()。

**public boolean onTouchEvent(MotionEvent event)**在dispatchTouchEvent方法内部调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前view无法再次接收到事件。若返回值为True，事件由自己处理，后续事件序列让其处理；若返回值为False，自己不消耗事件，向上返回让其他的父容器的onTouchEvent接受处理。

三个方法的关系可以用下面的伪代码表示:
```java
public boolean dispatchouchEvent(MotionEvent ev) {
	boolean consume = false;
	if(onInterceptTouchEvent(ev)) {
		consume = onTouchEvent(ev);
	} else {
		consume = child.dispatchTouchEvent(ev);
	}
	return consume;
}
```

**(2)onTouchListener的优先级别比onTouchvent要高**
如果给view设置了onTouchListener，那么onTouchListener中的onTouch方法会被回调。这时事件如何处理还要看onTouch的返回值，如果返回false，那么当前view的onTouchEvent方法会被回调；如果返回true,那么onTouchEvent方法将不会被调用。在onTouchEvent方法中，如果当前view设置了onClickListener，那么它的onClick方法会被调用，所以onClickListener的优先级最低。

**(3)当一个点击事件发生之后，传递过程遵循如下顺序呢:Activity -> Window -> View**
如果一个view的onTouchEvent方法返回false，那么它的父容器onTouchEvent方法将会被调用，依次类推，如果所有的元素都不处理这个事件，那么这个事件将会最终传递给Activity处理，调用Activity的onTouchEvent方法。

**(4)**正常情况下，一个事件序列只能被一个view拦截并消耗，因为一旦某个元素拦截了某个事件，那么同一个事件序列内的所有事件都会直接交给它处理，并且该元素的onInterceptTouchEvent方法不会再被调用了。

**(5)**某个view一旦开始处理事件，如果它不消耗ACTION_DOWN事件，那么同一事件序列的其他事件都不会再交给它来处理，并且事件将重新交给它的父容器去处理(调用父容器的onTouchEvent方法)；如果它消耗ACTION_DOWN事件，但是不消耗其他类型事件，那么这个点击事件会消失，父容器的onTouchEvent方法不会被调用，当前view依然可以收到后续的事件，但是这些事件最后都会传递给Activity处理。

**(6)**ViewGroup默认不拦截任何事件，因为它的onInterceptTouchEvent方法默认返回false。view没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。

**(7)**View的onTouchEvent默认都会消耗事件(返回true)，除非它是不可点击的(clickable和longClickable都为false)。view的longClickable默认是false的，clickable则不一定，Button默认是true，而TextView默认是false。

**(8)**View的enable属性不影响onTouchEvent的默认返回值。哪怕一个view是disable状态，只要它的clickable或者longClickable有一个是true，那么它的onTouchEvent就会返回true。

**(9)**事件传递过程总是先传递给父元素，然后再由父元素分发给子view，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外，即当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件。

(10)以上结论均可以在书中的源码解析部分得到解释。Window的实现类为PhoneWindow，获取Activity的contentView的方法
```java
((ViewGroup)getWindow().getDecorView().findViewById(android.R.id.content)).getChildAt(0);
```

## view的滑动冲突
**(1)常见的滑动冲突的场景：**
1.外部滑动方向和内部滑动方向不一致，例如viewpager中包含listview；

2.外部滑动方向和内部滑动方向一致，例如viewpager的单页中存在可以滑动的bannerview；

3.上面两种情况的嵌套，例如viewpager的单个页面中包含了bannerview和listview。

**(2)滑动冲突处理规则**
可以根据滑动距离和水平方向形成的夹角；或者根据水平和竖直方向滑动的距离差；或者两个方向上的速度差等。

**(3)解决方式**
(1)外部拦截法：点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要就不拦截。该方法需要重写父容器的onInterceptTouchEvent方法，在内部做相应的拦截即可，其他均不需要做修改。
伪代码如下:
```java
public boolean onInterceptTouchEvent(MotionEvent event) {
    boolean intercepted = false;
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN: {
        intercepted = false;
        break;
    }
    case MotionEvent.ACTION_MOVE: {
        int deltaX = x - mLastXIntercept;
        int deltaY = y - mLastYIntercept;
        if (父容器需要拦截当前点击事件的条件，例如：Math.abs(deltaX) > Math.abs(deltaY)) {
            intercepted = true;
        } else {
            intercepted = false;
        }
        break;
    }
    case MotionEvent.ACTION_UP: {
        intercepted = false;
        break;
    }
    default:
        break;
    }

    mLastXIntercept = x;
    mLastYIntercept = y;

    return intercepted;
}
```
**(2)内部拦截法：父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器来处理。这种方法和Android中的事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作。**
```java
public boolean dispatchTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN: {]
        getParent().requestDisallowInterceptTouchEvent(true);
        break;
    }
    case MotionEvent.ACTION_MOVE: {
        int deltaX = x - mLastX;
        int deltaY = y - mLastY;
        if (当前view需要拦截当前点击事件的条件，例如：Math.abs(deltaX) > Math.abs(deltaY)) {
            getParent().requestDisallowInterceptTouchEvent(false);
        }
        break;
    }
    case MotionEvent.ACTION_UP: {
        break;
    }
    default:
        break;
    }

    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(event);
}
```

## 总结
本文参考书籍《Android 开发艺术探索》任志刚. 