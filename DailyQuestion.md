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

