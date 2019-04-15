### 1.Service 有几种启动方式？每种启动方式的生命周期是怎样的？

有两种启动模式：`startService()` 和 `bindService()`。

![生命周期](http://innovator-blogimage.test.upcdn.net/2019/03/151553687933_.pic_hd.jpg)

Service 的启动过程（摘自「Android 插件化指南 36 - 41 页」）：

#### 1️⃣ 在新进程启动 Service

我们先看 Service启动过程，假设要启动的 Service是在一个新的进程中，启动过程可分为 5 个阶段 :
1 ) App 向 AMS 发送一个启动 Service 的消息 。
2) AMS 检查启动 Service 的进程是否存在，如果不存在，先把 Service 信息存下来，
然后创建一个新的进程 。
3 )新进程启动后，通知 AMS ，“我可以啦” 。
4 ) AMS 把刚才保存的 Service 信息发送给新进程 。
 5)新进程启动 Service。
 
#### 2️⃣ 启动同一进程的 Service

如果是在当前进程启动这个 Service，那么上面的步骤就简化为: 
I) App 向 AMS 发送一个启动 Servic巳的消息。
2) AMS 例行检查，比如 Service 是否声明了，把 Service 在 AMS 这边注册 。 AMS 发 现要启动的 Service就是 App所在的 Service， 于是通知 App启动这个 Service。
3) App启动 Service。 我们看到，没有了启动新进程的过程 。

#### 3️⃣ 在同一进程绑定 Service

如果要在当前进程绑定这个 Service，可分为以下 5 个阶段:
1) App 向 AMS 发送一个绑定 Service 的消息。
2)AMS例行检查， 比如Service是否声明了，把Servic巳在AMS这边注册。 AMS发
现要启动的 Service 就是 App 所在的 Service，就先通知 App 启动这个 Service，然后再通知 App 对 Service 进行绑定操作 。
3) App 收到 AMS 第 l 个消息，启动 Service。
4) App 收到 AMS 第 2个消息，绑定 Service，并把一个 Binder对象传给 AMS。
5) AMS 把接收到的 Binder对象发送给 App。
你也许会问，都在一个进程， App 内部直接使用 Binder对象不就好了，其实，要考虑
不在一个进程的场景，代码又不能写两份，两 套 逻辑，所以就都放在一起了，即使在同一 个进程，也要绕着 AMS 走一圈。

### 2.AMS 是在何时被加载的？在一个 App 的生命周期中，AMS 起到什么样的作用？

参考 360 的插件化开源库 [RePlugin](https://github.com/Qihoo360/RePlugin/blob/dev/README_CN.md) 的源码。

### 3.每日一题：什么是 Android 的函数插桩？

参考以下文章：

[【Android】函数插桩（Gradle + ASM）](https://juejin.im/post/5c6eaa066fb9a049fc042048)

[会用就行了？你知道 AOP 框架的原理吗？](https://juejin.im/post/5c57b2d5e51d457ffd56ffbb)

### 4.每日一题：Android 事件分发机制，如何下发？如何上传？

参考文章 [Android事件分发机制详解：史上最全面、最易懂](https://www.jianshu.com/p/38015afcdb58)

### 5.每日一题：volley 的请求成功和失败的 HTTP 响应码有哪些？为什么说 volley 适用于频繁请求而每次请求数据量不会很大？

1、volley 除了 200-299 的响应认为是成功的响应，回调到 `onResponse()` 方法，成功拿不到具体的响应码；其余都处理成失败，回调到 `onErrorResponse()` 方法，失败可以拿到具体的响应码，所以自定义响应码不适用 volley。主要源码： **`com.android.volley.toolbox.BasicNetwork`**

2、volley 使用于频繁请求而每次请求数据量不会很大，是因为 volley 里面使用的 byte[]，请求频繁且大数量会占用过多的内存，导致 OOM 等问题。

volley 和 OKHttp 对比

![](http://innovator-blogimage.test.upcdn.net/2019/04/%E5%AF%B9%E6%AF%94.jpg)

### 6.每日一题：Parcelable 和 Serializable 的作用、效率、区别及选择？

- 作用：都是用来序列化的，就是将一个对象转换成可存储或可传输的状态。序列化后的对象可以在网络上进行传输，也可以存储到本地。

- 效率：Parcelable 比 Serializable 更好更快。Parcelable 是为了程序内不同组件传递和程序之间的传递数据设计的，所以在读写数据的时候，Parcelable 是在内存中直接进行读写,而 Serializable 是通过使用 IO 流的形式将数据读写入在硬盘上，所以 Parcelable 的效率比较高。同时 Parcelable 内存开销也比 Serializable 小。

- 区别：Serializable 使用方便，实现一下 Serializable 接口，定义一个 SerializableID 就可以了。Parcelable 是 Android 特有的序列化方式，实现起来麻烦一点，而且也要保证写入数据的顺序必须和读出数据的顺序一致。

- 选择：Serializable 的作用是为了保存对象的属性到本地文件、数据库、网络流、rmi 以方便数据传输，当然这种传输可以是程序内的也可以是两个程序间的

    Android 的 Parcelable的设计初衷是因为 Serializable 效率过慢，为了在程序内不同组件间以及不同 Android 程序间 (AIDL) 高效的传输数据而设计，这些数据仅在内存中存在，Parcelable 是通过 IBinder 通信的消息的载体。

    Parcelable 的性能比 Serializable 好，在内存开销方面较小，所以在内存间数据传输时推荐使用 Parcelable，如 activity 间传输数据，而 Serializable 可将数据持久化方便保存，所以在需要保存或网络传输数据时选择 Serializable，因为android 不同版本 Parcelable 可能不同，所以不推荐使用 Parcelable 进行数据持久化。
    
### 7.每日一题：说说对线程锁和线程阻塞的理解

Java 中的锁是一种线程同步机制，即在同一时刻，只能有一个线程访问临界资源，也称作同步互斥访问。在 Java 中，提供了两种方式来实现同步互斥访问：synchronized 和 Lock。

当某个线程调用该对象的 synchronized 方法或者访问 synchronized 代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，这时其他线程就处于阻塞的状态。只有等待这个线程执行完方法或者代码块，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

参考另一篇文章。[]()

### 8.每日一题：自定义Handler 时如何有效地避免内存泄漏问题？

Handler 造成内存泄漏的原因：在 java 中非静态内部类和匿名内部类都会隐式持有当前类的外部引用，由于 Handler 是非静态内部类所以其持有当前 Activity 的隐式引用，如果 Handler 没有被释放（延迟的消息会在被处理之前存在于主线程消息队列中， 而这个消息中又包含了 Handler 的引用，所以 Handler 并没有被释放），其所持有的外部引用也就是 Activity 也不可能被释放，当一个对象一句不需要再使用了，本来该被回收时，而有另外一个正在使用的对象持有它的引用从而导致它不能被回收，这导致本该被回收的对象不能被回收而停留在堆内存中，这就产生了内存泄漏 (上面的例子就是这个原因)。

**解决办法：使用静态内部类并继承 Handler 。**

```java
 /**
     * 创建静态内部类
     */
    private static class MyHandler extends Handler{
        //持有弱引用HandlerActivity,GC回收时会被回收掉.
        private final WeakReference<HandlerActivity> mActivty;
        public MyHandler(HandlerActivity activity){
            mActivty =new WeakReference<HandlerActivity>(activity);
        }
        @Override
        public void handleMessage(Message msg) {
            HandlerActivity activity=mActivty.get();
            super.handleMessage(msg);
            if(activity!=null){
                //执行业务逻辑
            }
        }
    }  
```

### 9.每日一题：LaunchMode 的应用场景？

#### 1.android:launchMode="standard"
特点：Standard 模式是系统默认的启动模式，可以存在多个实例，这是默认的启动模式，系统总是会在目标栈中创建新的activity实例。

用途：一般我们 app 中大部分页面都是由该模式的页面构成的，比较常见的场景是：社交应用中，点击查看用户A信息->查看用户A粉丝->在粉丝中挑选查看用户B信息->查看用户A粉丝... 这种情况下一般我们需要保留用户操作 Activity 栈的页面所有执行顺序。

#### 2.android:launchMode="singleTop"
特点：如果这个 activity 实例已经存在目标栈的栈顶，系统会调用这个 activity 中的 onNewIntent() 方法，并传递 intent，而不会创建新的 activity 实例；如果不存在这个 activity 实例或者 activity 实例不在栈顶，则 SingleTop 和 Standard 作用是一样的。

用途：SingleTop 模式一般常见于社交应用中的通知栏行为功能，例如：App 用户收到几条好友请求的推送消息，需要用户点击推送通知进入到请求者个人信息页，将信息页设置为 SingleTop 模式就可以增强复用性。

#### 3.android:launchMode="singleTask"
特点：不会存在多个实例，如果栈中不存在 activity 实例，系统会在新栈的根部创建一个新的 activity；如果这个 activity 实例已经存在，系统会调用这个 activity 的 onNewIntent() 方法而不会创建新的 activity 实例，同时会清空这个 activity 任务栈上面所有的 activity。

用途：SingleTask 模式一般用作应用的首页，例如浏览器主页，用户可能从多个应用启动浏览器，但主界面仅仅启动一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。

#### 4.android:launchMode="singleInstance"
特点：这种启动模式比较特殊，因为它会启用一个新的栈结构，将 Acitvity 放置于这个新的栈结构中，并保证不再有其他 Activity 实例进入，除此之外，SingleInstance 模式和 SingleTask 模式是一样的。

用途：SingleInstance 模式常应用于独立栈操作的应用，如闹钟的响铃页面。

你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素，因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。

### 10.每日一题：一个app中有几个context？分别由谁持有？它们之间有什么差别？

- 1.不考虑多进程的情况下，一个app中有
n = application + serviceCount + activityCount 个 context

- 2.
Application、Service和Activity都是ContextWrapper的直接或间接子类，它们有两种
获取 context 的方法

![](http://innovator-blogimage.test.upcdn.net/2019/04/15/15553392443415.jpg)

![](http://innovator-blogimage.test.upcdn.net/2019/04/15/15553392677957.jpg)

- 3.Application的context生命周期和Application一样；
Activity、Service的context生命周期和Activity、Service一样；
它们本质都是Context，只是生命周期不同，应用场景也不同。

参考以下文章：

[Android应用Context详解及源码解析](https://blog.csdn.net/yanbober/article/details/45967639)

[Android Context完全解析，你所不知道的Context的各种细节](https://blog.csdn.net/guolin_blog/article/details/47028975)



