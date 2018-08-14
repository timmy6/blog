---
title: ListView 滑动冲突
date: 2016-04-24 23:25:22
category: Android 进阶
---

在Android中，有时候会遇到子控件和父控件都要滑动的情况，尤其是当子控件为listview的时候。这种情况较常见，典型的launcher，每个屏幕上放上listview就会出现这种情况。
有两点需要注意：
**1**.一般来说，view的onTouchEvent返回true，即消耗点击事件，viewgroup的onInterceptTouchEvent返回false，即不拦截点击事件，这一点从android源码中可以看出来。但是listview的父类AbsListView重写了onInterceptTouchEvent，返回了true，注意这里不是一定返回true，但是我觉得这一点可以先忽略。

**2**.onTouchEvent和onInterceptTouchEvent的调用顺序。点击事件从父控件向子控件传递，如果父控件不拦截，则交由子控件拦截，如果父控件拦截了，则交由父控件的onTouchEvent处理，如果最终处理点击事件的控件的onTouchEvent返回了false，则将会直接调用其父控件的onTouchEvent，如此向上类推。

其实解决方法也很简单：重写父控件的onInterceptTouchEvent函数，在move的时候根据需要返回true，比如左右滑动返回true，其他情况均返回false。这样，当左右滑动的时候，由于onInterceptTouchEvent返回了true，父控件就能处理，其他情况，事件将传递到listview中，listview自身可以处理上下滑动。

```java
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) 
{
Log.d(TAG, "onInterceptTouchEvent-slop:"+mTouchSlop);

final int action = ev.getAction();
if ((action == MotionEvent.ACTION_MOVE) && (mTouchState != TOUCH_STATE_REST))
{
return true;
}

final float x = ev.getX();
final float y = ev.getY();

switch (action)
{
case MotionEvent.ACTION_MOVE:
final int xDiff = (int)Math.abs(mLastMotionX-x);
if (xDiff>mTouchSlop)
{
mTouchState = TOUCH_STATE_SCROLLING;
}
break;

case MotionEvent.ACTION_DOWN:
mLastMotionX = x;
mLastMotionY = y;
mTouchState = mScroller.isFinished()? TOUCH_STATE_REST : TOUCH_STATE_SCROLLING;
break;

case MotionEvent.ACTION_CANCEL:
case MotionEvent.ACTION_UP:
mTouchState = TOUCH_STATE_REST;
break;
}

return mTouchState != TOUCH_STATE_REST;
}
```