



## Lifecycle组件

​		在`Lifecycle`组件出现之前，或多或少都会在`Activity` / `Fragment`的生命周期方法中实现依赖组件的操作（比如：暂停播放，释放资源等）。如以下所示：

```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    public override fun onStart() {
        super.onStart()
        myLocationListener.start()
    }

    public override fun onStop() {
        super.onStop()
        myLocationListener.stop()
    }
}
```

以上做法存在2个问题:

* `Activity` / `Fragment`的生命周期方法会越来越臃肿。
* 无法实现生命周期 **管理的一致性**。
  * 当依赖组件中某些行为需要修改调用时机时，需要到使用该依赖组件的全部`Activity/Fragment` 中手动修改。（比如：原本在`onStart`中调用，现在改成在`onResume`中调用）
  * 修改时可能遗漏个别`Activity` / `Fragment`。

​		引入`Lifecycle`组件 之后，只需要在`MyLocationListener`中实现`LifecycleObserver`接口，并设置其对应的生命周期调度事件。

```kotlin
class MyLocationListener : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun start() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onStop() {
        ...
    }
}
```

最后只需要在`Activity` / `Fragment`中，将其作为生命周期观察者添加进去即可：

```kotlin
lifecycle.addObserver(myLocationListener)
```

> 总得来说，`Lifecycle`组件让其他组件具有脱离`Activity` / `Fragment`也可以处理生命周期变化的能力。

​		`Lifecycle`组件中存在3个关键角色：`LifecycleOwner` 、`Lifecycle` 和 `LifecycleObserver`，对这三者的关系以及其作用有一定了解后，才可以更好的了解`Lifecycle`组件：

### LifecycleOwner 

​		`LifecycleOwner` 单一方法接口，表示类具有 `Lifecycle` 。

​		实现 `LifecycleObserver` 的组件可与实现 `LifecycleOwner` 的组件无缝协同工作，因为`LifecycleOwner` 可以提供生命周期，而观察者可以注册以观察生命周期。

```java
public interface LifecycleOwner {
    @NonNull
    Lifecycle getLifecycle();
}
```

> 主要作用：
>
> ​	持有 `Lifecycle` 实例，以便发送生命周期事件（`Lifecycle.Event`）。
>
> ​	发送生命周期事件。

#### 注意

* **Support Library 26.1.0 及更高版本**中的 `Fragment` 和 `Activity` 已实现 `LifecycleOwner`接口。
* `LifecycleOwner`对象不等于`activity` / `fragment`。
  * `LifecycleOwner`对象 可以是 `activity` / `fragment` 这些真正具有 生命周期的对象。也可以是一个 普通的`LifecycleOwner` 实现类，内部无任何生命周期，依靠外界持有该 `LifecycleOwner` 实现类对象，进行 生命周期事件 的发送。(例如：`FragmentViewLifecycleOwner`)
* 在 `Fragment` 中调用 `LiveData#observe(LifecycleOwner,Observer)` 时，应传入`viewLifecycleOwner`（通过 `Fragment#getViewLifecycleOwner()` 获取），而非 `Fragment` 本身。
  * `Fragment` 会存在 `View` 被销毁，而 `Fragment` 本身没有被销毁的情况。这意味着 旧的`LivaData`观察者 在 `Fragment`的`View` 被重建后，没有被移除。
  * `viewLifecycleOwner`与 `View` 的生命周期绑定。(`Fragment#onCreateView`之前创建，生命周期为`onCreateView` 到 `onDestroyView` )

### Lifecycle 

​		`Lifecycle` 是一个类，用于存储有关组件（如 `Activity` 或 `Fragment`）的 生命周期状态 的信息，并允许其他对象观察此状态。

```java
public abstract class Lifecycle {

    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);

    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);
    
    @MainThread
    @NonNull
    public abstract State getCurrentState();
}
```

> 主要作用：
>
> ​	管理 存储`LifecycleObserver`的 `Map`
>
> ​	存储 生命周期源的生命周期状态
>
> ​	响应 生命周期事件（`Lifecycle.Event`） ,修改自身 `State`，并触发 `LifecycleObserver` 对象 对应的 生命周期事件 回调 。(简单说就是：分发生命周期事件)

`Lifecycle` 虽然作为抽象类存在，但官方已经为我们提供了它具体的实现类：`LifecycleRegistry`。

#### Lifecycle.Event 与 Lifecycle.State

​		`Lifecycle`中定义了两个枚举类：`Lifecycle.Event` 和 `Lifecycle.State`。分别表示 生命周期事件 和 生命周期状态。`Lifecycle.Event` 和 `Lifecycle.State` 关系图如下：

![](https://github.com/daqi17/AndroidBlog/blob/master/img/jetpack/Event%26State.png)

**我一直有一个疑惑，为什么 `Google` 即定义 `State` ，又定义 `Event` ？**

直到我看到以下答案：

> 对于第三方组件来说，event 是被动接收，state 是主动推送时 主动获取以判断（是否适合推送）
>
> event 是针对 第三方组件内部作为观察者，来观察 页面对组件的推送，
> 而 state 是针对 页面作为观察者，来观察 来自第三方组件内部对页面的推送，那么此时通过 state 的判断，我们可确保在页面处于非激活状态时不收到 基于 `LifeCycle` 的组件（比如 `LiveData`）的推送。

### LifecycleObserver

```java
public interface LifecycleObserver {}
```

> 主要作用：
>
> ​	不同 生命周期事件 发生时，开发者自定义对应的响应逻辑。此逻辑已无需在 `Activity / Frgament` 中进行声明。

对于 `LifecycleObserver` 的实现，官方提供两种实现：

* 实现 `LifecycleObserver` 接口，配合 `OnLifecycleEvent` 注解使用。

* 实现 `DefaultLifecycleObserver` 接口，重载对应的生命周期 **默认方法** 。

  *  `DefaultLifecycleObserver` 接口需要额外导入，在Gradle中添加以下依赖：

  ```groovy
  implementation "androidx.lifecycle:lifecycle-common-java8:2.2.0"
  ```

同时，官方在 `Lifecycle` 类注释中表明：

* 对基于 `Java 7` 进行编码的开发者，可以使用 `OnLifecycleEvent` 注解的方式来观察生命周期事件。一旦 `Java 8` 在 Android 上成为主流，注解的方式将被弃用。

> ```
> If you use Java 7 Language, Lifecycle events are observed using annotations.Once Java 8 Language becomes mainstream on Android, annotations will be deprecated, so between {@link DefaultLifecycleObserver} and annotations,you must always prefer {@code DefaultLifecycleObserver}.
> ```

* 使用 `OnLifecycleEvent` 注解也可以方便的获取对应的 `LifecycleOwner` (即在 生命周期事件回调中，获取到 生命周期事件源，不只是 `DefaultLifecycleObserver` 特有)  。无需自身手动存储 `LifecycleOwner` 变量。

  ```kotlin
  object :LifecycleObserver{
      @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
      fun onCreated(source: LifecycleOwner){}
      
      @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
      fun onAny(source:LifecycleOwner,event: Lifecycle.Event){}
  }
  ```


### Lifecycle源码浅析

> 本次依据 android 10 、androidx.lifecycle:lifecycle-common:2.2.0 和 androidx.fragment:fragment:1.2.4 源码分析

关于 `LifecycleOwner` 、`Lifecycle` 和 `LifecycleObserver` 怎么协同工作的整体流程如图所示：

![归纳图](https://github.com/daqi17/AndroidBlog/blob/master/img/jetpack/lifecycles.png)

* 通过 `Lifecycle#addObserver()`方法，将 `LifecycleObserver` 实例 添加到 `Lifecycle` 实例中。
  * 将 `LifecycleObserver` 实例 和 初始`State`( 初始`State`一般为 `Lifecycle.State.INITIALIZED`) 封装成 `ObserverWithState` 对象，添加到 `LifecycleRegistry`实例 的 `mObserverMap` 中。
  * 将新添加的 `ObserverWithState` 对象的`State`， 同步到与 `Lifecycle`实例存储的`State` 一致。

* 实现`LifecycleOwner` 接口的对象(如 : `activity`/`fragment`)，依据自身生命周期，通过 `Lifecycle` 实例发送 `Lifecycle.Event`。
  *  `Lifecycle` 实例 依据接收到的 `Lifecycle.Event`，修改自身的 `mState`。
  * 将 `mObserverMap` 中所有`ObserverWithState` 对象的 `State`， 同步到与 `Lifecycle` 实例存储的`State` 一致。

* 正常状态下， `mObserverMap` 中所有`ObserverWithState` 对象的 `State` 都与 `Lifecycle` 实例存储的`State`一致。
  * `State`同步不是一步到位的。即`INITIALIZED` 变化到 `RESUMED`，会从`INITIALIZED` -> `CREATED` -> `STARTED` -> `RESUMED`。
  * `State` 同步的同时，生命周期事件回调也是同步触发的。即`INITIALIZED` 变化到 `RESUMED`，`DefaultLifecycleObserver` 会按以下顺序触发 `onCreate()` 、`onStart()`、`onResume()`。

**注：**

* 枚举类中，越早声明的枚举属性 对应的 恒定序数 越小，依据此比较枚举属性的大小。

> 目前我还没get到 `LifecycleRegistry#mParentStates` 中的作用，如果有大佬清楚望告知。

#### Activity生命周期 与 Lifecycle.Event

​		通过 `FragmentActivity` 往上追溯，第一个实现了 `LifecycleOwner` 接口的父类为 `androidx.activity.ComponentActivity`。但你在这里找不到具体 `Lifecycle.Event` 发送的步骤，因为都交由`ReportFragment.injectIfNeededIn(this)`处理。

> API 29之前, 通过 `ReportFragment` 的 生命周期回调 来调度 生命周期事件。
>
> 在API 29+上，由 `ReportFragment.injectIfNeededIn` 中添加的 `ActivityLifecycleCallbacks` 处理。(实际已变成在 `Activity` 的生命周期回调中发送 生命周期事件)

```java
//androidx.lifecycle.ReportFragment.java
public static void injectIfNeededIn(Activity activity) {
    if (Build.VERSION.SDK_INT >= 29) {
        // On API 29+, we can register for the correct Lifecycle callbacks directly
        //Activity#registerActivityLifecycleCallbacks为API29 新添加的
        activity.registerActivityLifecycleCallbacks(
            new LifecycleCallbacks());
    }
	//设置一个fragment，依据fragment的生命周期回调来感知activity的生命周期
    android.app.FragmentManager manager = activity.getFragmentManager();
    if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
        manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
        manager.executePendingTransactions();
    }
}

static class LifecycleCallbacks implements Application.ActivityLifecycleCallbacks {
    //省略掉目前不分发事件的回调

    //Activity Create的最后一步。
    @Override
    public void onActivityPostCreated(@NonNull Activity activity,
                                      @Nullable Bundle savedInstanceState) {
        dispatch(activity, Lifecycle.Event.ON_CREATE);
    }

    //Activity Started的最后一步。
    @Override
    public void onActivityPostStarted(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_START);
    }

    //Activity Resumed的最后一步。
    @Override
    public void onActivityPostResumed(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_RESUME);
    }

    //Activity Paused的第一步
    @Override
    public void onActivityPrePaused(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_PAUSE);
    }

    //Activity Stopped的第一步
    @Override
    public void onActivityPreStopped(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_STOP);
    }

    //Activity Destroyed的第一步
    @Override
    public void onActivityPreDestroyed(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_DESTROY);
    }
}
```

​		这些 第一步 和 最后一步 表示`Activity` 在执行的 `performXXX()` 时，**最先** 和 **最后** 执行该方法。而执行`performXXX()`时，会执行对应的生命周期回调`onXXX()`，也可理解为在生命周期回调 **之前** 或 **之后** 执行。

```java
//android.app.Activity.java
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
    //第一步，内部会回调 ActivityLifecycleCallbacks.onActivityPreCreated。
    dispatchActivityPreCreated(icicle);
    //..
    //activity的onCreate回调
    //onCreate中会执行 ActivityLifecycleCallbacks.onActivityCreated 回调。
    if (persistentState != null) {
        onCreate(icicle, persistentState);
    } else {
        onCreate(icicle);
    }
    //..
    //最后一步,内部会回调 ActivityLifecycleCallbacks.onActivityPostCreated。
    dispatchActivityPostCreated(icicle);
}
```

#### Fragment生命周期 与 LifecycleEvent

​		`Fragment` 除了自身实现了 `LifecycleOwner`接口 外，还存在一个与 `View` 生命周期绑定的 `ViewLifecycleOwner` 。两者发送具体的生命周期事件如下：

![](https://github.com/daqi17/AndroidBlog/blob/master/img/jetpack/Fragment%26Lifecycle.png)

> 总的来说，无论是`Activity`/`Fragment`，还是`ViewLifecycleOwner`。`LifecycleObserver`的回调都会在 `onCreate`/`onCreateView`、`onStart` 和 `onResume` 之后，在 `onPause` 、`onStop` 和 `onDestroy` / `onDestroyView`之前。

### Lifecycle实战 —— 网络状态观察者

​		依据 `DefaultLifecycleObserver` 观察生命周期的变化 ，动态注册和注销网络`Callback`，将网络状态数据存储在 `LiveData` 中，再依托 `LiveData` 进行网络状态分发。外界无论想获取当前网络状态，还是监听网络状态变化，都可以依靠该 `LiveData` 实现。

```kotlin
//NetworkObserver.kt
/**
  * 网络状态观察者
  * @author daqi
  */
class NetworkObserver private constructor(private val context: Context?)
    :DefaultLifecycleObserver,ConnectivityManager.NetworkCallback() {

    //网络状态LiveData
    private val _NetworkState: MutableLiveData<NetState>

    val NetworkState: LiveData<NetState>
        get() = _NetworkState

    init {
        _NetworkState = MutableLiveData()
    }

    //在onResume注册网络状态回调意味着: 最开始的时候，你需要在Activity/Fragment走到onResume时才可知道网络状态。
    override fun onCreate(owner: LifecycleOwner) = registerNetworkReceiver()

    override fun onDestroy(owner: LifecycleOwner) = unRegisterNetworkReceiver()

    //注册网络监听
    private fun registerNetworkReceiver(){
        val connectivity = context?.getSystemService(Context.CONNECTIVITY_SERVICE) as? ConnectivityManager
        //设置网络回调
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            connectivity?.registerDefaultNetworkCallback(NetworkObserver@this)
        }else{
            val builder = NetworkRequest.Builder()
            //蜂窝
            builder.addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
            //wifi
            builder.addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
            connectivity?.registerNetworkCallback(builder.build(),NetworkObserver@this)
        }
    }

    //注销网路监听
    private fun unRegisterNetworkReceiver(){
        val connectivity = context?.getSystemService(Context.CONNECTIVITY_SERVICE) as? ConnectivityManager
        connectivity?.unregisterNetworkCallback(NetworkObserver@this)
    }


    //网络不可以回调
    override fun onLost(network: Network){
        super.onLost(network)
        _NetworkState.postValue(NetState.Disconnected())
    }

    //网络可用回调
    override fun onAvailable(network: Network){
        super.onAvailable(network)
        //获取对应的NetworkCapabilities对象，以便获取网络信息
        val connectivity = context?.getSystemService(Context.CONNECTIVITY_SERVICE) as? ConnectivityManager
        //
        val networkCapabilities = connectivity?.getNetworkCapabilities(network) ?: return
        //刷新网络状态
        refreshStatus(networkCapabilities)
    }

    override fun onCapabilitiesChanged(network: Network, networkCapabilities: NetworkCapabilities) {
        super.onCapabilitiesChanged(network, networkCapabilities)
        //刷新网络状态
        refreshStatus(networkCapabilities)
    }

    //链接属性改变时回调(可从中获取ip地址，网关,DNS之类的信息)
    override fun onLinkPropertiesChanged(network: Network, linkProperties: LinkProperties) {
        super.onLinkPropertiesChanged(network, linkProperties)
    }

    //刷新网络状态
    private fun refreshStatus(networkCapabilities: NetworkCapabilities){
        //获取当前的网络信息
        val currentNetState= _NetworkState.value ?: NetState.Disconnected()
        //获取当前的验证状态
        val currentValidatedState = networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)
        when{
            //WIFI网络
            networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> {
                //获取新的网络状态对象
                val netState = NetState.WifiNetWork(currentValidatedState)
                //再与当前比较，只有不一致时，才去刷新。
                if (netState != currentNetState)
                    _NetworkState.postValue(netState)
            }
            //蜂窝网络
            networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> {
                val netState = NetState.CellularNetWork(currentValidatedState)
                if (netState != currentNetState)
                    _NetworkState.postValue(netState)
            }
        }
    }

    companion object{
        //单例
        @Volatile
        private var mInstance: NetworkObserver? = null

        @JvmStatic
        fun getInstance(context: Context?) = mInstance ?: synchronized(this) {
            mInstance ?: NetworkObserver(context?.applicationContext).also {
                mInstance = it
            }
        }

        //直接获取当前网络状态是否已连接
        val isConnected:Boolean
            @JvmStatic
            get() = mInstance?._NetworkState?.value?.connectionState ?: false

        //直接获取当前网络状态是否可用
        val isAvailable:Boolean
            @JvmStatic
            get(){
                val isConnected = mInstance?._NetworkState?.value?.connectionState ?: false
                val isValidated = mInstance?._NetworkState?.value?.validatedState ?: false
                return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M){
                    isConnected && isValidated
                }else{
                    isConnected
                }
            }
    }
}

/**
  * 网络状态
  * @author daqi
  */
data class NetState(
    //连接状态
    val connectionState:Boolean,
    //网络类型
    val networkType:Int,
    //可用状态
    val validatedState:Boolean
){
    companion object{
        //蜂窝网络
        @JvmStatic
        fun CellularNetWork(validatedState:Boolean) =
            NetState(true, NetworkCapabilities.TRANSPORT_CELLULAR,validatedState)

        //WIFI网络
        @JvmStatic
        fun WifiNetWork(validatedState:Boolean) =
            NetState(true,NetworkCapabilities.TRANSPORT_WIFI,validatedState)

        //网络断开
        @JvmStatic
        fun Disconnected() = NetState(false,-1,false)
    }
}
```

> 更多使用可以查看 [秉心说 —— Jetpack 之 LifeCycle 使用篇](https://juejin.im/post/5df64c19518825121d6e2013#heading-3) 。

#### 小结

* 把 `LifecycleObserver` 实现抽离成一个类，而非 `activity` / `fragment`中的匿名内部类，才可做到 “一处修改、处处生效”。

*  `LifecycleObserver`实现类如何向外部（比如 `activity` / `fragment`） 传递数据呢 ？答案当然是 ：`LiveData` 。 
	* 这 `LiveData` 可以是 `ViewModel` 中的 `LiveData`。可以是 `LifecycleObserver`实现类 的属性。
  	* 对于普通的  `LifecycleObserver` 实现类，可以在 **构造方法** 中将 `activity` / `fragment` 作为**局部变量**传递进来，根据 `activity` / `fragment`获取 `ViewModle` ，从而修改`ViewModel` 中的 `LiveData` 的数据。( 考虑到 `ViewModel` 中的 `MutableLiveData`对象 应对外展示为 `LiveData`类型，所以建议选择触发 `ViewModel` 的方法对 `MutableLiveData` 对象进行修改 )。


### 参考文章

* [重学安卓：为你还原一个真实的 Jetpack Lifecycle](https://xiaozhuanlan.com/topic/3684721950)
* [Android 官网：Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle)
* [使用 Android Architecture Components 的五个常见误区](https://proandroiddev.com/5-common-mistakes-when-using-architecture-components-403e9899f4cb)

> 若文章有错误或不对的地方，欢迎大佬们指出。