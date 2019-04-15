### 一、什么是 java 的锁？为什么要设置这个东西？

在多线程编程中，线程安全问题是一个最为关键的问题，其核心概念就在于当多个线程访问某一共享、可变数据时，始都不会导致数据破坏以及其他不该出现的结果。而所有的并发模式在解决这个问题时，采用的方案都是序列化访问临界资源 ，即在同一时刻，只能有一个线程访问临界资源，也称作 同步互斥访问。

Java 中的锁是一种线程同步机制，在 Java 中，提供了两种方式来实现同步互斥访问：synchronized 和 Lock。

### 二、synchronized


在 Java 中，可以使用 synchronized 关键字来标记一个方法或者代码块，当某个线程调用该对象的 synchronized 方法或者访问 synchronized 代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

#### 1.synchronized 修饰方法

注意：**如果一个线程 A 需要访问对象 object1 的 synchronized 方法 fun1，另外一个线程 B 需要访问对象 object2 的 synchronized 方法 fun1，即使 object1 和 object2 是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。**


#### 2.synchronized 同步块

synchronized 同步代码块会比 synchronized 方法颗粒度更小，更容易控制想要锁住的范围。synchronized 同步代码块的一般形式如下：

```java
synchronized (lock){
    //访问共享可变资源
    ...
}
```

其中这个 lock 可以是**当前对象，当前对象内部的某个成员变量对象或者 Class 对象。** synchronized 修饰方法也可以理解成在方法的开始，对当前对象加锁。如下两种形式的效果是一样的：

```java
public synchronized void insert(String name){
        for(int i=0;i<100;i++){
            Log.i("GameProps",name+"在插入数据 "+i);
            swords.add(i);
        }
    }
```

```java
public void insert(String name){
        synchronized (this){
            for(int i=0;i<100;i++){
                Log.i("GameProps",name+"在插入数据 "+i);
                swords.add(i);
            }
        }
    }
```

当 lock 是 Class 对象时，情况就不一样啦！！！

**每个类也会有一个锁，静态的 synchronized 方法就是以 Class 对象作为锁，它是用来控制对 static 数据成员 （static 数据成员不专属于任何一个对象，是类成员） 的并发访问。synchronized（XXX.class）一个时刻只能有一条线程访问这个方法。**

有一点要注意：**对于 synchronized（lock）锁的是对象，是多个线程在竞争对象锁时的那个对象，即使某个线程在获得对象锁后，将引用指向其他对象，仍然不会改变开始时别的线程竞争的对象锁。**

**对于 synchronized 方法 或者 synchronized 代码块，当出现异常时，JVM 会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。**

关键字 synchronized 主要包含两个特征：

**互斥性：保证在同一时刻，只有一个线程可以执行某一个方法或某一个代码块；**

**可见性：保证线程工作内存中的变量与公共内存中的变量同步，使多线程读取共享变量时可以获得最新值的使用。**

#### 3.synchronized原理

JVM 基于进入和退出 Monitor 对象来实现代码块同步和方法同步，两者实现细节不同。

- **代码块同步：** 在编译后通过将 monitorenter 指令插入到同步代码块的开始处，将 monitorexit 指令插入到方法结束处和异常处（所以出现异常会释放锁），通过反编译字节码可以观察到。任何一个对象都有一个 monitor 与之关联，线程执行 monitorenter 指令时，会尝试获取对象对应的 monitor 的所有权，即尝试获得对象的锁。

- **方法同步：** synchronized 方法在 method_info 结构有 ACC_synchronized 标记，线程执行时会识别该标记，获取对应的锁，实现方法同步。


**两者虽然实现细节不同，但本质上都是对一个对象的监视器（monitor）的获取。任意一个对象都拥有自己的监视器，当同步代码块或同步方法时，执行方法的线程必须先获得该对象的监视器才能进入同步块或同步方法，没有获取到监视器的线程将会被阻塞，并进入同步队列，状态变为BLOCKED。当成功获取监视器的线程释放了锁后，会唤醒阻塞在同步队列的线程，使其重新尝试对监视器的获取。**

![](http://innovator-blogimage.test.upcdn.net/2019/04/monitor.png)

#### 4.synchronized 和 volatile 对比

volatile,它能够使变量在值发生改变时能尽快地让其他线程知道。

1）volatile 本质是在告诉 jvm 当前变量在寄存器中的值是不确定的,需要从主存中读取；synchronized 则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住。

2）volatile 仅能使用在变量级别,synchronized 则可以使用在变量,方法。

3）volatile 仅能实现变量的修改可见性,而 synchronized 则可以保证变量的修改可见性和原子性。


### 三、Lock

当一个线程执行了被 synchronized 修饰的代码块时，以下 3 种情况才会自动释放锁：

- 占有锁的线程执行完了该代码块，然后释放对锁的占有；

- 占有锁线程执行发生异常，此时 JVM 会让线程自动释放锁；

- 占有锁线程进入 WAITING 状态从而释放锁，例如在该线程中调用 wait() 方法等。

如果当某个线程占有了锁，但是一直不释放，导致其他线程一直在等待锁，这个时候如果有等待超时的机制就好了，这样其他线程在等待了一段时间没有获得锁就能去干其他事了。

还真的有，这个就是 Lock 接口提供的功能啦。Lock 接口比 synchronized 更灵活，颗粒度更小。

注意啦！！！

- **synchronized 是 Java 的关键字，因此是 Java 的内置特性，是基于 JVM 层面实现的，其经过编译之后，会在同步块的前后分别形成 monitorenter 和 monitorexit 两个字节码指令（ monitorenter 指令执行时会让对象的锁计数加 1，而 monitorexit 指令执行时会让对象的锁计数减 1），如果有异常发生，线程自动释放锁；而 Lock 是一个 Java 接口，是基于 JDK 层面实现的，通过这个接口可以实现同步访问；**

- **采用 synchronized 方式不需要用户去手动释放锁，当 synchronized 方法或者 synchronized 代码块执行完之后，系统会自动让线程释放对锁的占用；而 Lock 则必须要用户去手动释放锁 (发生异常时，不会自动释放锁)，如果没有主动释放锁，就有可能导致死锁现象。**

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;  // 可以响应中断
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  // 可以响应中断
    void unlock();
    Condition newCondition();
}
```

### 四、ReentrantLock 可重入锁

**ReentrantLock 是唯一实现了 Lock 接口的类，并且 ReentrantLock 提供了更多的方法。**

#### 1.lock()

`lock()` 方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。**使用 Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。**

因此，一般来说，使用 Lock 必须在 `try…catch…` 块中进行，并且将释放锁的操作放在 `finally` 块中进行，以保证锁一定被被释放，防止死锁的发生。

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```

#### 2. tryLock() & tryLock(long time, TimeUnit unit)

**`tryLock()` 方法用来尝试获取锁，如果获取成功，则返回 true；如果获取失败（即锁已被其他线程获取），则返回 false，也就是说，这个方法无论如何都会立即返回（在拿不到锁时不会一直在那等待）。**

**`tryLock(long time, TimeUnit unit)` 方法和 `tryLock()` 方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回 false，同时可以响应中断。如果一开始拿到锁或者在等待期间内拿到了锁，则返回 true。**

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

#### 3.lockInterruptibly()

**lockInterruptibly() 方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态，抛出 InterruptedException。**

例如，当两个线程同时通过 `lock.lockInterruptibly()` 想获取某个锁时，假若此时线程 A 获取到了锁，而线程 B 只有在等待，那么对线程 B 调用 `threadB.interrupt()` 方法,B  线程在轮询到中断标志位 true 的时候，就会抛出 InterruptedException ,让上层调用者去处理线程 B。

**由于 `lockInterruptibly()` 的声明中抛出了异常，所以 lock.lockInterruptibly() 必须放在 try 块中或者在调用 lockInterruptibly() 的方法外声明抛出 InterruptedException。推荐使用后者。**

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

注意！！！

ReentrantLock 的加锁方法 `lock()` 提供了无条件地轮询获取锁的方式，`lockInterruptibly()` 提供了可中断的锁获取方式。这两个方法的区别在于 `lock()` 方法默认处理了中断请求，一旦监测到中断状态，忽略中断请求，继续获取锁直到成功；而 `lockInterruptibly()` 则直接抛出中断异常，由上层调用者区去处理中断。

**当一个线程获取了锁之后，是不会被 `interrupt()` 方法中断的。因为 `interrupt()` 方法只能中断阻塞过程中的线程而不能中断正在运行过程中的线程。** 因此，当通过 `lockInterruptibly()` 方法获取某个锁时，如果不能获取到，那么只有进行等待的情况下，才可以响应中断的。与 synchronized 相比，使用 synchronized 的话， 当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

**在使用 Lock 时，无论以哪种方式获取锁，习惯上最好一律将获取锁的代码放到 `try…catch…` 之前，因为我们一般将锁的 `unlock()` 操作放到 finally 子句中，如果线程运行出现异常，在执行 finally 子句时，就会执行 `unlock()` 操作，从而保证了锁必须释放，否则会导致其他线程一直阻塞。**


### 五、ReadWriteLock

ReadWriteLock 也是一个接口，在它里面只定义了两个方法：

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```

一个用来获取读锁，一个用来获取写锁。也就是说，将对临界资源的读写操作分成两个锁来分配给线程，从而使得多个线程可以同时进行读操作。

### 六、ReentrantReadWriteLock

**ReentrantReadWriteLock 实现了 ReadWriteLock 接口 其包含两个很重要的方法：readLock() 和 writeLock() 分别用来获取读锁和写锁，并且这两个锁实现了 Lock 接口。**

注意！！！

**如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁（防止脏读）。如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程也会一直等待释放写锁（防止脏写或者脏读）。**


### 七、总结一下

总的来说，Lock 和 Synchronized 有以下几点不同：

　　**(1). Lock 是一个接口，是 JDK 层面的实现；而 synchronized 是 Java 中的关键字，是 Java 的内置特性，是 JVM 层面的实现。Lock 和 synchronized 锁的都是对象；**

　　**(2). synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而 Lock 在发生异常时，如果没有主动通过 `unLock()` 去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁；**

　　**(3). Lock 可以让等待锁的线程响应中断(使用 tryLock(time,timeUnint) 或者 lockInterruptily() )，而使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断；**

　**(4). 通过 Lock 可以知道有没有成功获取锁(使用 tryLock())，而 synchronized 却无法办到；**

　**(5). Lock 可以提高多个线程进行读操作的效率。**

　　在性能上来说，如果竞争资源不激烈，两者的性能是差不多的。而当竞争资源非常激烈时（即有大量线程同时竞争），此时 Lock 的性能要远远优于 synchronized。所以说，在具体使用时要根据适当情况选择。

### 八、锁的相关概念介绍

#### 1.可重入锁

**如果锁具备可重入性，则称作为可重入锁。像 synchronized 和 ReentrantLock 都是可重入锁，可重入性实际上表明了 锁的分配粒度：基于线程的分配，而不是基于方法调用的分配。可重入锁最大的作用是避免死锁。**

　
举个简单的例子，当一个线程执行到某个 synchronized 方法时，比如说 method1，而在 method1 中会调用另外一个 synchronized 方法 method2，此时线程不必重新去申请锁，而是可以直接执行方法 method2。

```java
class MyClass {
    public synchronized void method1() {
        method2();
    }

    public synchronized void method2() {

    }
}
```

上述代码中的两个方法 method1 和 method2 都用 synchronized 修饰了。假如某一时刻，线程 A 执行到了 method1，此时线程 A 获取了这个对象的锁，而由于 method2 也是 synchronized 方法，假如 synchronized 不具备可重入性，此时线程 A 需要重新申请锁。但是，这就会造成死锁，因为线程 A 已经持有了该对象的锁，而又在申请获取该对象的锁，这样就会线程 A 一直等待永远不会获取到的锁。而由于 synchronized 和 Lock 都具备可重入性，所以不会发生上述现象。

#### 2.可中断锁

**可中断锁就是可以响应中断的锁。在 Java 中，synchronized 就不是可中断锁，而 Lock 是可中断锁。**


如果某一线程 A 正在执行锁中的代码，另一线程 B 正在等待获取该锁，可能由于等待时间过长，线程 B 不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。使用 tryLock(long time, TimeUnit unit) 和 lockInterruptibly() 就可以响应中断请求。

#### 3.公平锁

**公平锁即尽量以请求锁的顺序来获取锁。而非公平锁就是抢占式地获取锁。**

比如，同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。而非公平锁则无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。

在 Java 中，synchronized 就是非公平锁），它无法保证等待的线程获取锁的顺序。而对于 ReentrantLock 和 ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以构造方法中设置为公平锁（协同式线程调度）`ReentrantLock lock = new ReentrantLock(true);`。

在 ReentrantLock 类中定义了很多方法，举几个例子：

- `isFair()` // 判断锁是否是公平锁

- `isLocked()` // 判断锁是否被任何线程获取了

- `isHeldByCurrentThread()` // 判断锁是否被当前线程获取了

- `hasQueuedThreads()` // 判断是否有线程在等待该锁

- `getHoldCount()` // 查询当前线程占有 lock 锁的次数

- `getQueueLength()` // 获取正在等待此锁的线程数

- `getWaitQueueLength(Condition condition)` // 获取正在等待此锁相关条件 condition 的线程数

#### 3.读写锁

读写锁将对临界资源的访问分成了两个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。ReadWriteLock 就是读写锁，它是一个接口，ReentrantReadWriteLock 实现了这个接口。可以通过 `readLock()` 获取读锁，通过 `writeLock()` 获取写锁。

#### 4.乐观锁

乐观锁与悲观锁是一种广义上的概念，体现了看待线程同步的不同角度。乐观锁是指在对同一个数据并发操作时，当前线程在使用数据时不会有别的线程修改数据，所以不需要添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。

乐观锁在Java中是通过使用无锁编程来实现，最常采用的是 CAS 算法，Java 原子类中的递增操作就通过 CAS 自旋实现的。

CAS 算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

![乐观锁和悲观锁](http://innovator-blogimage.test.upcdn.net/2019/04/leguan.png)

#### 5.悲观锁

悲观锁是指对于同一个数据的并发操作，在当前线程使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized 关键字和 Lock 的实现类都是悲观锁。

**悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。**

### 九、死锁

当线程A持有独占锁a，并尝试去获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a的情况下，就会发生AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁。

造成死锁必须达成的4个条件（原因）：

- 互斥条件：一个资源每次只能被一个线程使用。（独占锁的特点之一。）
- 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。（独占锁的特点之一。）
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。（独占锁的特点之一。）
- 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

**避免死锁的办法：在并发程序中，避免了逻辑中出现复数个线程互相持有对方线程所需要的独占锁的的情况，就可以避免死锁。**

### 十、Java 并发编程

java 中线程之间的同步技术。**wait() 与 notify()/notifyAll() ，与 synchronized 搭配使用。**

- wait():

    wait(): 释放占有的对象锁，线程进入等待池，释放 cpu, 而其他正在等待的线程即可抢占此锁，获得锁的线程即可运行程序。而 sleep() 不同的是，线程调用此方法后，会休眠一段时间，休眠期间，会暂时释放 cpu，但并不释放对象锁。也就是说，在休眠期间，其他线程依然无法进入此代码内部。休眠结束，线程重新获得 cpu, 执行代码。wait() 和 sleep() 最大的不同在于 wait() 会释放对象锁，而 sleep() 不会！
    
    wait() 总是在一个循环中被调用，挂起当前线程来等待一个条件的成立。

- notify(): 

    该方法会唤醒因为调用对象的 wait() 而等待的线程，其实就是对对象锁的唤醒，从而使得 wait() 的线程可以有机会获取对象锁。调用 notify() 后，并不会立即释放锁，而是继续执行当前代码，直到 synchronized 中的代码全部执行完毕，才会释放对象锁。JVM 则会在等待的线程中调度一个线程去获得对象锁，执行代码。需要注意的是，wait() 和 notify() 必须在 synchronized 代码块中调用。

- notifyAll():
    
    则是唤醒所有等待的线程。

正确的用法示例如下：

```java
/ 线程 A 的代码
synchronized(obj_A){

    while(!condition){ 
        obj_A.wait();
    }
    // do something 
}

// 线程 B 的代码
synchronized(obj_A){

    if(!condition){ 
        // do something ...
        condition = true;
        obj_A.notify();
    }
}
```


