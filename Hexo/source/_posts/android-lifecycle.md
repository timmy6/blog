---
title: Android官方架构组件：Lifecycle介绍及原理分析
date: 2018-08-13 16:25:46
category: Android 进阶
---

#### 概述
在过去的谷歌IO大会上，Google官方向我们推出了 **[Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)**,其中谈到Android组件处理生命周期的问题，向我们介绍了 **[Handling Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle.html)**。同时，如何利用 `android.arch.lifecycle` 包提供的类来控制数据、监听器等的 lifecycle。同时，LiveData 与 ViewModel 的 lifecycle 也依赖于`Lifecycle`框架。不久前， Android Architecture Components 正式Release，`Lifecycle`也正式植入进了`SupportActivity（AppCompatActivity的基类）`和`Fragment`中。

#### Lifecycle简介及基础使用
##### 为什么要引进Lifecycle？
我们在处理Activity或者Fragment组件的生命周期相关时，不可避免会遇到这样的问题：我们在**Activity**的`onCreate()`中初始化某些成员（比如MVP架构中的Presenter，或者AudioManager、MediaPlayer等），然后在onStop中对这些成员进行对应处理，在onDestroy中释放这些资源，这样导致我们的代码也许会像这样：
```java
class MyPresenter{
    public MyPresenter() {
    }

    void create() {
        //do something
    }

    void destroy() {
        //do something
    }
}

class MyActivity extends AppCompatActivity {
    private MyPresenter presenter;

    public void onCreate(...) {
        presenter= new MyPresenter ();
        presenter.create();
    }

    public void onDestroy() {
        super.onDestroy();
        presenter.destory();
    }
}
```
代码没有问题，关键问题是，实际生产环境中 ，这样的代码会非常复杂，你最终会有太多的类似调用并且会导致`onCreate()` 和`onDestroy()`方法变的非常臃肿。

##### 如何集成LifeCycle
###### 1.Prestener继承LifecycleObserver接口
```java
public interface IPresenter extends LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    void onCreate(@NotNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    void onDestroy(@NotNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    void onLifecycleChanged(@NotNull LifecycleOwner owner,
                            @NotNull Lifecycle.Event event);
}

public class BasePresenter implements IPresenter {

    private static final String TAG = "BasePresenter";    

    @Override
    public void onLifecycleChanged(@NotNull LifecycleOwner owner, @NotNull Lifecycle.Event event) {

    }

    @Override
    public void onCreate(@NotNull LifecycleOwner owner) {
        Log.d("tag", "BasePresenter.onCreate" + this.getClass().toString());
    }

    @Override
    public void onDestroy(@NotNull LifecycleOwner owner) {
        Log.d("tag", "BasePresenter.onDestroy" + this.getClass().toString());
    }
}
```
这里我直接将我想要观察到`Presenter`的生命周期事件都列了出来，然后封装到`BasePresenter`中，这样每一个`BasePresenter` 的子类都能感知到Activity容器对应的生命周期事件，并在子类重写的方法中，对应相应行为。

###### 2.在Activity/Fragment容器中添加Observer
```java
public class MainActivity extends AppCompatActivity {
    private IPresenter mPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("tag", "onCreate" + this.getClass().toString());
        setContentView(R.layout.activity_main);
        mPresenter = new MainPresenter(this);
        getLifecycle().addObserver(mPresenter);//添加LifecycleObserver
    }

    @Override
    protected void onDestroy() {
        Log.d("tag", "onDestroy" + this.getClass().toString());
        super.onDestroy();
    }
}
```

如此，每当Activity发生了对应生命周期改变，Presenter就会执行对应事件注解的方法，只要通过注解进行声明，就能够使LifecycleObserver观察到对应的生命周期事件。
```java
public enum Event {
        /**
         * Constant for onCreate event of the {@link LifecycleOwner}.
         */
        ON_CREATE,
        /**
         * Constant for onStart event of the {@link LifecycleOwner}.
         */
        ON_START,
        /**
         * Constant for onResume event of the {@link LifecycleOwner}.
         */
        ON_RESUME,
        /**
         * Constant for onPause event of the {@link LifecycleOwner}.
         */
        ON_PAUSE,
        /**
         * Constant for onStop event of the {@link LifecycleOwner}.
         */
        ON_STOP,
        /**
         * Constant for onDestroy event of the {@link LifecycleOwner}.
         */
        ON_DESTROY,
        /**
         * An {@link Event Event} constant that can be used to match all events.
         */
        ON_ANY
    }
```

#### 原理分析：
借鉴Android架构组件(一)————Lifecycle的一张图进行简单的概括：
![shortcuts](/uploads/7293029-e8b3a15d2ed0a6ee.png)

我们先将这些重要的类挑选出来：
- **LifecycleObserver接口（Lifecycle观察者**：实现该接口的类，通过注解的方式，可以通过被`LifecycleOwner`类的AddObserver（LifecycleObserver o）方法注册，被注册后，LifeCycleObserver便可以观察到LifecycleOwner生命周期事件。

- **LifecycleOwner接口(Lifecycle持有者)**：实现该类接口的类持有生命周期（Lifecycle对象），该接口的生命周期(Lifecycle对象)的改变会被其他注册的观察者**LifecycleObserver**观察到并触发其对应的事件

- Lifecycle(生命周期)：和**LifecycleOwner**不同的是，**LifecycleOwner**本身持有`Lifecycle`对象，**LifecycleOwner**通过其`Lifecycle getLifecycle()`的接口获取背部`Lifecycle`对象。

- **State(当前生命周期所处状态)**：如图所示。
- **Event(生命周期改变对应的事件)**：如图所示，当`Lifecycle`发生改变，如进入`onCreate`，会自动发出`ON_CREATE`事件.

了解了这些类和接口的职责，接下来原理分析就简单很多了，我们以`Fragment`为例，来看下实际`Fragment`等类和上述类或接口的联系：

#####  Fragment：LifecycleOwner
- Fragment(Activity同理，我们 本文以Fragment为例，下同)：实现了**LifecycleOwner**接口，这意味着Fragment对象持有生命周期对象（Lifecycle），并可以通过**Lifecycle getLifecycle()**方法获取内部的Lifecycle对象：
```java
public class Fragment implements xxx, LifecycleOwner {

    //...省略其他

   LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}

public interface LifecycleOwner {
    @NonNull
    Lifecycle getLifecycle();
}
```
可以看到，实现的`getLifecycle()`方法，实际上返回的是**LifecycleRegistry**对象，**LifecycleRegistry**对象实际上继承了 `Lifecycle`。持有`Lifecycle`有什么作用呢？实际上在Fragment对应的生命周期内，都会发送对应的生命周期事件给内部的**LifecycleRegistry**对象处理：
```java
public class Fragment implements xxx, LifecycleOwner {
    //...
    void performCreate(Bundle savedInstanceState) {
        onCreate(savedInstanceState);  //1.先执行生命周期方法
        //...省略代码
        //2.生命周期事件分发
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
    }

    void performStart() {
        onStart();
        //...
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
    }

    void performResume() {
         onResume();
        //...
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
    }

    void performPause() {
        //3.注意，调用顺序变了
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
        //...
        onPause();
    }

    void performStop() {
       mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
        //...
        onStop();
    }

    void performDestroy() {
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY);
        //...
        onDestroy();
    }
}
```
随着`Fragment`不同走到不同的生命周期，除了暴露给我们的生命周期方法`onCreate/onStart/…./onDestroy`等，同时，Fragment内部的`Lifecycle`对象（就是`mLifecycleRegistry`）还将生命周期对应的事件作为参数传给了 `handleLifecycleEvent()` 方法。
** 同时，你会发现`Fragment`中`performCreate()`、`performStart()`、`performResume()`会先调用自身的`onXXX()`方法，然后再调用`LifecycleRegistry`的`handleLifecycleEvent()`方法；而在`performPause()`、`performStop()`、`performDestroy()`中会先`LifecycleRegistry`的`handleLifecycleEvent()`方法 ，然后调用自身的`onXXX()`方法。**

参考Android架构组件(一)——Lifecycle，文中的时序图：
![shortcuts](/uploads/7293029-a125ace9440970e6.png)
我们从图中可以看到，当`Fragment`将生命周期对应的事件交给其内部的`Lifecycle`处理后， `LifecycleObserver`（就是我们上文自定义的Presenter），就能够接收到对应的生命周期事件，这是如何实现的呢？

##### LifecycleRegistry：Lifecycle
首先确认一点，`LifecycleRegistry` 就是`Lifecycle`的子类：
```java
public class LifecycleRegistry extends Lifecycle {
}
```
我们再看一下Lifecycle类：
```java
public abstract class Lifecycle {

        //注册LifecycleObserver （比如Presenter）
        public abstract void addObserver(@NonNull LifecycleObserver observer);
        //移除LifecycleObserver 
        public abstract void removeObserver(@NonNull LifecycleObserver observer);
        //获取当前状态
        public abstract State getCurrentState();

        public enum Event {
            ON_CREATE,
            ON_START,
            ON_RESUME,
            ON_PAUSE,
            ON_STOP,
            ON_DESTROY,
            ON_ANY
        }

       public enum State {
            DESTROYED,
            INITIALIZED,
            CREATED,
            STARTED,
            RESUMED;

            public boolean isAtLeast(@NonNull State state) {
                return compareTo(state) >= 0;
            }
       }
}
```
`Lifecycle`没什么要讲的，几个抽象方法也能看懂，作为`Lifecycle`的子类，`LifecycleRegistry` 同样也能通过addObserver方法注册`LifecycleObserver` （就是Presenter），当`LifecycleRegistry`本身的生命周期改变后（可以想象，内部一定有一个成员变量State记录当前的生命周期），`LifecycleRegistry`就会逐个通知每一个注册的`LifecycleObserver` ，并执行对应生命周期的方法。
我们看一下`LifecycleRegistry` 的`handleLifecycleEvent()`方法：
```java
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }
```
看方法的名字我们就可以知道，`handleLifecycleEvent`方法会通过 `getStateAfter`获取当前应处的状态并修改`Lifecycle本身的State`值，紧接着遍历所`LifecycleObserver` 并同步且通知其状态发生变化，因此就能触发LifecycleObserver 对应的生命周期事件。

##### 尝试复用LifecycleRegistry
首先，`LifecycleRegistry` 本身就是一个成熟的 `Lifecycle` 实现类，它被实例化在`Activity`和`Fragment`中使用，如果我们需要自定义`LifecycleOwner`并实现接口需要返回一个Lifecycle实例，完全可以直接在自定义`LifecycleOwner`中new一个`LifecycleRegistry`成员并返回它（简而言之就是：直接拿来用即可）。

以下是Google官方文档：
> LifecycleRegistry: An implementation of Lifecycle that can handle multiple observers. It is used by Fragments and Support Library Activities. You can also directly use it if you have a custom LifecycleOwner.

##### 注解和DefaultLifecycleObserver的取舍
其次，Google的`Lifecycle`库中提供了一个 `DefaultLifecycleObserver `类,方便我们直接实现`LifecycleObserver`接口，相比较于文中demo所使用的注解方式，Google官方更推荐我们使用 `DefaultLifecycleObserver` 类，并声明
> 一旦Java 8成为Android的主流，注释将被弃用，所以介于DefaultLifecycleObserver和注解两者之间,更推荐使用 DefaultLifecycleObserver 。
官方原文：
```java
/*
 * If you use <b>Java 8 Language</b>, then observe events with {@link DefaultLifecycleObserver}.
 * To include it you should add {@code "android.arch.lifecycle:common-java8:<version>"} to your
 * build.gradle file.
 * <pre>
 * class TestObserver implements DefaultLifecycleObserver {
 *     {@literal @}Override
 *     public void onCreate(LifecycleOwner owner) {
 *         // your code
 *     }
 * }
 * </pre>
 * If you use <b>Java 7 Language</b>, Lifecycle events are observed using annotations.
 * Once Java 8 Language becomes mainstream on Android, annotations will be deprecated, so between
 * {@link DefaultLifecycleObserver} and annotations,
 * you must always prefer {@code DefaultLifecycleObserver}.
 */
```
##### Lifecycles 的最佳实践
- 保持 UI 控制器（Activity 和 Fragment）尽可能的精简。它们不应该试图去获取它们所需的数据；相反，要用 ViewModel来获取，并且观察 LiveData将数据变化反映到视图中。
- 尝试编写数据驱动（data-driven）的 UI，即 UI 控制器的责任是在数据改变时更新视图或者将用户的操作通知给 ViewModel。
- 将数据逻辑放到 ViewModel 类中。ViewModel 应该作为 UI 控制器和应用程序其它部分的连接服务。注意：不是由 ViewModel 负责获取数据（例如：从网络获取）。相反，ViewModel 调用相应的组件获取数据，然后将数据获取结果提供给 UI 控制器。
- 使用Data Binding来保持视图和 UI 控制器之间的接口干净。这样可以让视图更具声明性，并且尽可能减少在 Activity 和 Fragment 中编写更新代码。如果你喜欢在 Java 中执行该操作，请使用像Butter Knife 这样的库来避免使用样板代码并进行更好的抽象化。
- 如果 UI 很复杂，可以考虑创建一个 Presenter 类来处理 UI 的修改。虽然通常这样做不是必要的，但可能会让 UI 更容易测试。
- 不要在 ViewModel 中引用View或者 Activity的 context。因为如果ViewModel存活的比 Activity 时间长（在配置更改的情况下），Activity 将会被泄漏并且无法被正确的回收。

#### 总结：
总而言之，Lifecycle还是有可取之处的，相对于其它架构组件之间的配合，Lifecycle更简单且独立（实际上配合其他组件味道更佳）。