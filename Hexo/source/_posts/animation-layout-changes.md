---
title: 使用动画改变Layout
date: 2016-03-19 10:19:49
category: Android 进阶
---
在Android 中，我们经常需要改变Layout布局，如，在某个ViewGroup中，add 一个View,或者 remove 一个View。如果我们直接添加或者删除View，会让用户界面改变的很生硬，这个时候，为了让我们的用户体验更顺畅，我们就需要通过动画来改变Layout。

## 一:通过配置属性使用系统默认动画.
一个Layout 动画，默认就有这个布局属性的，我们需要做的就是就是在布局里面，设置这个属性告诉Android系统，通过使用系统默认的动画改变这些布局。在Layout文件中，动画属性如下配置:
```java
<LinearLayout android:id="@+id/container"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:showDividers="middle"
            android:divider="?android:dividerHorizontal"
            android:animateLayoutChanges="true"
            android:paddingLeft="16dp"
            android:paddingRight="16dp" />
```
可以看到，上面代码中多了一个**android:animateLayoutChanges = “true”**属性，配置这个属性之后，就是告诉系统，如果这个ViewGroup 布局发生了改变，就使用系统默认动画。

## 二:通过LayoutTransition类使用自定义布局动画
**效果图如下:**
<img src="/uploads/layouchangean1.gif" width="50%" height="50%">
**Layout 布局:**
```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  xmlns:sa="http://schemas.android.com/apk/res-auto"
  android:id="@+id/layout_container"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  tools:context=".MainActivity">
 
  <android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    sa:popupTheme="@style/ThemeOverlay.AppCompat.Light">
 
    <Spinner
      android:id="@+id/language"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content" />
 
  </android.support.v7.widget.Toolbar>
 
  <android.support.v7.widget.CardView
    android:id="@+id/input_view"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"
    android:clipChildren="false">
 
    <RelativeLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:clipChildren="false"
      android:padding="@dimen/card_padding">
 
      <View
        android:id="@+id/focus_holder"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:focusableInTouchMode="true" />
 
      <EditText
        android:id="@+id/input"
        style="@style/Widget.TextView.Input"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:inputType="textMultiLine"
        android:imeOptions="flagNoFullscreen|actionDone"
        android:gravity="top"
        android:hint="@string/type_here" />
 
      <ImageView
        android:id="@+id/clear_input"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignTop="@id/input"
        android:layout_alignEnd="@id/input"
        android:layout_alignRight="@id/input"
        android:padding="8dp"
        android:src="@drawable/ic_clear"
        android:visibility="invisible"
        android:contentDescription="@string/clear_input" />
 
      <ImageView
        android:id="@+id/input_done"
        android:layout_width="32dip"
        android:layout_height="32dip"
        android:background="@drawable/done_background"
        android:src="@drawable/ic_arrow_forward"
        android:padding="2dp"
        android:layout_margin="8dp"
        tools:ignore="UnusedAttribute"
        android:elevation="4dp"
        android:visibility="invisible"
        android:layout_alignBottom="@id/input"
        android:layout_alignEnd="@id/input"
        android:layout_alignRight="@id/input"
        android:contentDescription="@string/done" />
 
    </RelativeLayout>
 
  </android.support.v7.widget.CardView>
 
  <FrameLayout
    android:id="@+id/translation_panel"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"
    android:padding="@dimen/translation_outer_margin">
 
    <android.support.v7.widget.CardView
      android:layout_width="match_parent"
      android:layout_height="wrap_content">
 
      <FrameLayout
        android:id="@+id/translation_copy"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:foreground="@drawable/click_foreground">
 
        <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:orientation="vertical"
          android:background="?attr/colorPrimary"
          tools:ignore="UselessParent">
 
          <FrameLayout
            android:id="@+id/translation_speak"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:foreground="@drawable/click_foreground"
            android:padding="@dimen/translation_inner_margin">
 
            <TextView
              android:id="@+id/translation_label"
              style="@style/Widget.TextView.Label"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:textAllCaps="true"
              android:drawableStart="@drawable/ic_tts"
              android:drawableLeft="@drawable/ic_tts"
              android:drawablePadding="4dip"
              android:text="@string/sample_language" />
          </FrameLayout>
 
          <TextView
            android:id="@+id/translation"
            style="@style/Widget.TextView.Translation"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="@dimen/translation_inner_margin"
            android:layout_marginStart="@dimen/translation_inner_margin"
            android:layout_marginRight="@dimen/translation_inner_margin"
            android:layout_marginEnd="@dimen/translation_inner_margin"
            android:layout_marginBottom="@dimen/translation_inner_margin"
            android:text="@string/sample_translation"/>
        </LinearLayout>
      </FrameLayout>
 
    </android.support.v7.widget.CardView>
 
  </FrameLayout>
 
</LinearLayout>
```
动画中，我们需要留意的关键控件是Toolbar,ID为input_view的CardView,ID为input_done的ImageView，以及ID为translation_panel的FrameLayout。其余的就只有一个focus_holder需要关心了，这是一个不可见的View，用于移除输入框的焦点。通过让焦点在EditText和focus_holder之间切换，实现输入模式的进入和退出。

动画的过程是这样的:在进入输入模式的时候，input_view向上移动知道覆盖Toolbar,input_done淡入，同时translation_panel淡出，而当用户输入退出输入模式的时候，则做相反的动画。可以在视频中看到这个过程.

让我们来看MainActivity:
```java
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        setTitle(R.string.sample_language);
 
        View input = findViewById(R.id.input);
        View inputDone = findViewById(R.id.input_done);
        final View focusHolder = findViewById(R.id.focus_holder);
 
        input.setOnFocusChangeListener(Part1TransitionController.newInstance(this));
        inputDone.setOnClickListener(
                new View.OnClickListener() {
                    @Override
                    public void onClick(@NonNull View v) {
                        focusHolder.requestFocus();
                    }
                });
    }
}
```
很简单，这里所做的事情不过是设置Toolbar,然后设置获取焦点的逻辑。而创建过度效果的逻辑则放在了Part1TransitionController中。我决定把这个逻辑放在抽象类中，以方便今后可以更容易替换成其他的实现方式。Part1TransitionController实际上继承于抽象基类TransitionController，里面有一些常用的逻辑:
```java
public abstract class TransitionController implements View.OnFocusChangeListener {
    private final WeakReference<Activity> activityWeakReference;
    private final AnimatorBuilder animatorBuilder;
 
    protected TransitionController(WeakReference<Activity> activityWeakReference, @NonNull AnimatorBuilder animatorBuilder) {
        this.activityWeakReference = activityWeakReference;
        this.animatorBuilder = animatorBuilder;
    }
 
    @Override
    public void onFocusChange(View v, boolean hasFocus) {
        Activity mainActivity = activityWeakReference.get();
        if (mainActivity != null) {
            if (hasFocus) {
                enterInputMode(mainActivity);
            } else {
                exitInputMode(mainActivity);
            }
        }
    }
 
    protected AnimatorBuilder getAnimatorBuilder() {
        return animatorBuilder;
    }
 
    protected abstract void enterInputMode(Activity mainActivity);
 
    protected abstract void exitInputMode(Activity mainActivity);
 
    protected void closeIme(View view) {
        Activity mainActivity = activityWeakReference.get();
        if (mainActivity != null) {
            InputMethodManager imm = (InputMethodManager) mainActivity.getSystemService(
                    Context.INPUT_METHOD_SERVICE);
            imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
        }
    }
 
    protected class ImeCloseListener extends AnimatorListenerAdapter {
        private final View view;
 
        public ImeCloseListener(View view) {
            this.view = view;
        }
 
        @Override
        public void onAnimationEnd(@NonNull Animator animation) {
            super.onAnimationEnd(animation);
            closeIme(view);
        }
 
    }
}
```
这类处理onFocusChanged()事件并且在进入和退出输入模式的时候调用了相应的抽象方法。这个类还包含了一个AnimatorListener类，用于确保在退出输入模式的时候，输入键盘可以隐藏起来。最后还有一个AnimatorBuild的实例，它用于构建动画原型，这些属性动画会被重复用到。那么，我们看一眼AnimatorBuilder类:
```java
public class AnimatorBuilder {
    private static final String TRANSLATION_Y = "translationY";
    private static final String ALPHA = "alpha";
 
    private final int duration;
 
    public static AnimatorBuilder newInstance(Context context) {
        int duration = context.getResources().getInteger(android.R.integer.config_mediumAnimTime);
        return new AnimatorBuilder(duration);
    }
 
    AnimatorBuilder(int duration) {
        this.duration = duration;
    }
 
    public Animator buildTranslationYAnimator(View view, int startY, int endY) {
        Animator animator = ObjectAnimator.ofFloat(view, TRANSLATION_Y, startY, endY);
        animator.setDuration(duration);
        return animator;
    }
 
    public Animator buildShowAnimator(View view) {
        return buildAlphaAnimator(view, 0f, 1f);
    }
 
    public Animator buildHideAnimator(View view) {
        return buildAlphaAnimator(view, 1f, 0f);
    }
 
    public Animator buildAlphaAnimator(View view, float startAlpha, float endAlpha) {
        Animator animator = ObjectAnimator.ofFloat(view, ALPHA, startAlpha, endAlpha);
        animator.setDuration(duration);
        return animator;
    }
}
```
这里构建了两个基本的animator:一个是修改view的translationY属性实现view上下移动的translation animator，另一个则是修改view的alpha属性从而改变view透明度的alpha animator。还有两个工具函数，用于封装alpha animator，提供了从不透明到全透明，以及从全透明到不透明的简便方法。

现在剩下的最后一件事，就是看看Part1TransitionController是如何将所有这些东西组合在一起的:
```java
public class Part1TransitionController extends TransitionController {
 
    public static TransitionController newInstance(Activity activity) {
        WeakReference<Activity> mainActivityWeakReference = new WeakReference<>(activity);
        AnimatorBuilder animatorBuilder = AnimatorBuilder.newInstance(activity);
        return new Part1TransitionController(mainActivityWeakReference, animatorBuilder);
    }
 
    Part1TransitionController(WeakReference<Activity> mainActivityWeakReference, AnimatorBuilder animatorBuilder) {
        super(mainActivityWeakReference, animatorBuilder);
    }
 
    @Override
    protected void enterInputMode(Activity activity) {
        View inputView = activity.findViewById(R.id.input_view);
        View inputDone = activity.findViewById(R.id.input_done);
        View translation = activity.findViewById(R.id.translation_panel);
        View toolbar = activity.findViewById(R.id.toolbar);
 
        inputDone.setVisibility(View.VISIBLE);
 
        AnimatorSet animatorSet = new AnimatorSet();
        AnimatorBuilder animatorBuilder = getAnimatorBuilder();
        Animator moveInputView = animatorBuilder.buildTranslationYAnimator(inputView, 0, -toolbar.getHeight());
        Animator showInputDone = animatorBuilder.buildShowAnimator(inputDone);
        Animator hideTranslation = animatorBuilder.buildHideAnimator(translation);
        animatorSet.playTogether(moveInputView, showInputDone, hideTranslation);
        animatorSet.start();
    }
 
    @Override
    protected void exitInputMode(Activity activity) {
        final View inputView = activity.findViewById(R.id.input_view);
        View inputDone = activity.findViewById(R.id.input_done);
        View translation = activity.findViewById(R.id.translation_panel);
        View toolbar = activity.findViewById(R.id.toolbar);
 
        AnimatorSet animatorSet = new AnimatorSet();
        AnimatorBuilder animatorBuilder = getAnimatorBuilder();
        Animator moveInputView = animatorBuilder.buildTranslationYAnimator(inputView, -toolbar.getHeight(), 0);
        Animator hideInputDone = animatorBuilder.buildHideAnimator(inputDone);
        Animator showTranslation = animatorBuilder.buildShowAnimator(translation);
        animatorSet.playTogether(moveInputView, hideInputDone, showTranslation);
        animatorSet.addListener(new ImeCloseListener(inputDone));
        animatorSet.start();
    }
}
```
我们重写了基类TransitionController的两个抽象方法。在两种情况下，我们分别从Activity找到了相应的view。在enterInputMode()中，我们构建了一个AnimatorSet，它包含了一个根据Toolbar的高度向上移动inputView属性动画，一个让inputDone从透明到不透明到动画，一个让translation(代表的是translation_panel)从不透明到透明的动画。而在exitInputMode()中我做的是相反的事情，不过还添加了animeCloseListener来实现在动画结束的时候隐藏输入键盘。

## 总结:
这两种方式都可以在layout改变的时候，使用动画。一个是是通过简单的在layout文件里面配置属性的方式，另一个是通过LayoutTransitions类，相对来说复杂一些，但也更灵活一些。

## 参考资料:
[Google Android 官网:http://developer.android.com/intl/zh-cn/training/animation/layout.html](http://developer.android.com/intl/zh-cn/training/animation/layout.html)