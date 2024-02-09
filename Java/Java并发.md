#### 线程模型

JDK 1.2 之前，Java 线程是基于绿色线程（Green Threads）实现的，这是一种**用户级线程（用户线程），也就是说 JVM 自己模拟了多线程的运行，而不依赖于操作系统**。由于绿色线程和原生线程比起来在使用时有一些限制（比如绿色线程不能直接使用操作系统提供的功能如异步 I/O、只能在一个内核线程上运行无法利用多核），在 JDK 1.2 及以后，Java 线程改为基于原生线程（Native Threads）实现，也就是说 JVM **直接使用操作系统原生的内核级线程（内核线程）来实现 Java 线程**，由操作系统内核进行线程的调度和管理。

我们上面提到了用户线程和内核线程，考虑到很多读者不太了解二者的区别，这里简单介绍一下：

- 用户线程：由用户空间程序管理和调度的线程，运行在用户空间（专门给应用程序使用）。
- 内核线程：由操作系统内核管理和调度的线程，运行在内核空间（只有内核程序可以访问）。

顺便简单总结一下用户线程和内核线程的区别和特点：用户线程创建和切换成本低，但不可以利用多核。内核态线程，创建和切换成本高，可以利用多核。

一句话概括 Java 线程和操作系统线程的关系：**现在的 Java 线程的本质其实就是操作系统的线程**。

线程模型是用户线程和内核线程之间的关联方式，常见的线程模型有这三种：

1. 一对一（一个用户线程对应一个内核线程）
2. 多对一（多个用户线程映射到一个内核线程）
3. 多对多（多个用户线程映射到多个内核线程）

![常见的三种线程模型](https://oss.javaguide.cn/github/javaguide/java/concurrent/three-types-of-thread-models.png)

#### 线程一些关键词

##### 并发与并行区别

- **并发**：两个及两个以上的作业在同一 **时间段** 内执行。
- **并行**：两个及两个以上的作业在同一 **时刻** 执行。

最关键的点是：是否是 **同时** 执行

##### 同步与异步区别

- **同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
- **异步**：调用在发出之后，不用等待返回结果，该调用直接返回。

##### 线程安全与不安全

* 线程安全指的是在多线程环境下，对于同一份数据，不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。

* 线程不安全则表示在多线程环境下，对于同一份数据，多个线程同时访问时可能会导致数据混乱、错误或者丢失。

#### Java创建线程的几种方式

一般来说，创建线程有很多种方式，例如继承`Thread`类、实现`Runnable`接口、实现`Callable`接口、使用线程池、使用**`CompletableFuture`**类、ThreadGroup、FutureTask、lambda表达式、Timer、Forkjoin。

[不同方式创建线程代码](https://mp.weixin.qq.com/s/NspUsyhEmKnJ-4OprRFp9g)

**本质都是：new Thread().start()**

#### Java线程的生命周期

* NEW: 初始状态，线程被**创建出来但没有被调用 `start()`** 。

* RUNNABLE: 运行状态，线程被**调用了 `start()`等待运行的状态**。

  > 在操作系统层面，线程有 READY 和 RUNNING 状态；而在 JVM 层面，只能看到 RUNNABLE 状态（图源：[HowToDoInJavaopen in new window](https://howtodoinJava.com/)：[Java Thread Life Cycle and Thread Statesopen in new window](https://howtodoinJava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/)），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。
  >
  > **为什么 JVM 没有区分这两种状态呢？** （摘自：[Java 线程运行怎么有第六种状态？ - Dawell 的回答open in new window](https://www.zhihu.com/question/56494969/answer/154053599) ） 现在的时分（time-sharing）多任务（multi-task）操作系统架构通常都是用所谓的“时间分片（time quantum or time slice）”方式进行抢占式（preemptive）轮转调度（round-robin 式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度。（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。

* BLOCKED：阻塞状态，需要等待锁释放（**没获取到锁**）。

* WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）（无限等待方法）。

* TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待（有限等待方法）。

* TERMINATED：终止状态，表示该线程已经运行完毕。

![Java 线程状态变迁图](https://oss.javaguide.cn/github/javaguide/java/concurrent/640.png)

#### 什么时候会触发线程上下文切换

线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()`、yeild等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或结束运行

#### 什么时候会发生死锁

死锁的四个必要条件：

1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件:若干线程之间形成一种头尾相接的循环等待资源关系。

#### 避免死锁的方案

1. **破坏循环等待**：一次性申请所有的资源（先把资源全部申请，再执行业务代码）
2. **破坏请求保持和不可剥夺条件**：限定时长持有锁（如Redission）

#### Wait和Sleep比较

**共同点**：两者都可以暂停线程的执行。

**区别**：

- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒，或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
- `sleep()` 是 `Thread` 类的静态本地方法，`wait()` 则是 `Object` 类的本地方法。为什么这样设计呢？下一个问题就会聊到。



#### volatile关键字

##### 保证可见性

在 Java 中，`volatile` 关键字可以保证变量的可见性，如果我们将变量声明为 **`volatile`** ，这就指示 JVM，这个变量是共享且不稳定的，**每次使用它都到主存中进行读取**。

`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。

##### 禁止指令重排

**在 Java 中，`volatile` 关键字除了可以保证变量的可见性，还有一个重要的作用就是防止 JVM 的指令重排序。** 

>  如果我们将变量声明为 **`volatile`** ，在对这个变量进行读写操作的时候，会通过插入特定的 **内存屏障** 的方式来禁止指令重排序。

例如：指令重排在单例时的应用

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}

```

`uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

##### 能否保证原子性

不能，**`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的**

如i++案例，因为虽然读取是从内存中读取，但是如果有多个并发读取时则会导致覆盖问题。

#### synchronized、volatile关键字与内存屏障的关系

![img](https://pic4.zhimg.com/80/v2-8d036daffacc112dc43d4e2696df7067_720w.webp)

* synchronized关键字包括的区域，当进入到临界区读变量信息时,保证读到的是最新的，这是因为在同步区内对变量的写入操作，在离开同步区时就将上前线程内的数据刷新到内存中，而对数据的读取也不能从缓存读取 只能从内存中读取保证了数据的读有效性这就是插入了**StoreStore**屏障
* 使用了volatile修饰变量,则对变量的写操作,会插入**StoreLoad**屏障
* 其余的操作,则需要通过Unsafe这个类来执行，UNSAFE.putOrderedObject类似这样的方法,会插入StoreStore内存屏障，Unsafe.putVolatiObiect 则是插入了StoreLoad屏障

#### 内存屏障模型

Java内存屏障主要有Load(读屏障）和Store (写屏障) 两类：

* 对Load Barrier来说，**在读指令前插入读屏障**（相当于清除缓存中的数据），可以让高速缓存中的数据失效，重新从主内存加载数据

  Load又可分为：

  * LoadLoad屏障

    序列: Load1，**LoadLoad**，Load2，确保Load1所要读入的数据能够在被Load2和后续的load指令访问前读入。通常能执行预加载指令或/和支持乱序处理的处理器中需要显式声明Loadload屏障，因为在这些处理器中正在等待的加载指令能够绕过正在等待存储的指令。 而对于总是能保证处理顺字的处理器上，设置该屏障相当于无操作。

  * LoadStore屏障

    序列: Load1，LoadStore，Store2
    确保Load1的数据在Store2和后续Store指令被刷新之前读取。在等待Store指令可以越过loads指令的乱序处理器上需要使用LoadStore屏障

* 对Store Barrier来说，**在写指令之后插入写屏障**（相当于刷新缓存中的数据），能让写入缓存的最新数据写回到主内存

  * StoreStore屏障

    序列: Store1，StoreStore，Store2，确保Store1的数据在Store2以及后续Store指令操作相关数据之前对其它处理器可见(例如向主存刷新数据)。通常情况下，如果处理器不能保证从写缓冲或/和缓存向其它处理器和主存中按顺序刷新数据，那么它需要使用StoreStore屏障。

  * StoreLoad屏障

    序列: Store1，StoreLoad，Load2
    确保Store1的数据在被Load2和后续的Load指令读取之前对其他处理器可见

#### 内存屏障的分类【可见性和有序性】

1. 按照可见性划分，也就是解决并发问题的可见性：
   * 加载屏障（Load Barrier）：相当于上面的LoadStoreBarrier，也对应JMM 8种基本操作中的load（载入）
   * 存储屏障（Store Barrier）：相当于上面的StoreLoadBarrier，也对应JMM 8种基本操作中的Store（存储）

2. 按照有序性划分，也就是解决了并发问题中的有序性：
   * 获取屏障（Acquire Barrier）：相当于上面的LoadLoadBarrier和LoadStoreBarrier组合
   * 释放屏障（Release Barrier）：相当于LoadStoreBarrier和StoreStoreBarrier组合

#### 锁分类

##### 悲观锁

悲观锁总是假设最坏的情况，认为共享资源每次被访问的时候就会出现问题(比如共享数据被修改)，所以每次在获取资源操作的时候都会上锁，这样其他线程想拿到这个资源就会阻塞直到锁被上一个持有者释放。也就是说，**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**。

像 Java 中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

##### 乐观锁

乐观锁总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源（也就是数据）是否被其它线程修改了（具体方法可以使用版本号机制或 CAS 算法）

##### 悲观锁和乐观锁适合什么场景

**悲观锁**通常多用于写比较多的情况（多写场景，**竞争激烈**），这样可以避免频繁失败和重试影响性能，悲观锁的开销是固定的。不过，如果乐观锁解决了频繁失败和重试这个问题的话（比如`LongAdder`），也是可以考虑使用乐观锁的，要视实际情况而定。

**乐观锁**通常多用于写比较少的情况（多读场景，**竞争较少**），这样可以避免频繁加锁影响性能。不过，乐观锁主要针对的对象是单个共享变量（参考`java.util.concurrent.atomic`包下面的原子变量类）。

##### 乐观锁存在哪些问题

1. ABA

   如果一个变量 V 初次读取的时候是 A 值，并且在准备赋值的时候检查到它仍然是 A 值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。这个问题被称为 CAS 操作的 **"ABA"问题。**

   ABA 问题的解决思路是在变量前面追加上**版本号或者时间戳**。JDK 1.5 以后的 `AtomicStampedReference` 类就是用来解决 ABA 问题的，其中的 `compareAndSet()` 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

2. 消耗大量CPU

   CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销。

3. 只能保证一个共享变量的原子操作

   CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5 开始，提供了`AtomicReference`类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作.所以我们可以使用锁或者利用`AtomicReference`类把多个共享变量合并成一个共享变量来操作

#### synchronized

主要解决的是多个线程之间**访问资源的同步性**，可以保证**被它修饰的方法或者代码块在任意时刻只能有一个线程执行**。

##### 1. 早期synchronized的问题

在 Java 早期版本中，`synchronized` 属于 **重量级锁**，效率低下。这是因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而**操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高**

##### 2. 后期改进

不过，在 Java 6 之后， `synchronized` 引入了大量的优化如**自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁**等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多。因此， `synchronized` 还是可以在实际项目中使用的，像 JDK 源码、很多开源框架都大量使用了 `synchronized` 。

关于偏向锁多补充一点：由于偏向锁增加了 JVM 的复杂性，同时也并没有为所有应用都带来性能提升。因此，在 JDK15 中，偏向锁被默认关闭（仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁），在 JDK18 中，偏向锁已经被彻底废弃（无法通过命令行打开）

##### 3. synchronized的修饰区域

`synchronized` 关键字的使用方式主要有下面 3 种：

1. 修饰实例方法（锁实例对象）
2. 修饰静态方法（锁该类的Class对象）
3. 修饰代码块（锁指定对象）

##### 4. synchronized是否可以锁构造方法

构造方法不能使用 synchronized 关键字修饰，构造方法本身就属于线程安全的，不存在同步的构造方法一说

##### 5.上锁过程

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

对应字节码

![synchronized关键字原理](https://oss.javaguide.cn/github/javaguide/java/concurrent/synchronized-principle.png)

**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

上面的字节码中包含一个 `monitorenter` 指令以及两个 `monitorexit` 指令，这是为了保证锁在同步代码块代码正常执行以及出现异常的这两种情况下都能被正确释放。

**在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1**。

![执行 monitorenter 获取锁](https://oss.javaguide.cn/github/javaguide/java/concurrent/synchronized-get-lock-code-block.png)

对象锁的的拥有者线程才可以执行 `monitorexit` 指令来释放锁。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。

![执行 monitorexit 释放锁](https://oss.javaguide.cn/github/javaguide/java/concurrent/synchronized-release-lock-block.png)

##### 6.锁升级过程

在 Java 6 之后， `synchronized` 引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多（JDK18 中，偏向锁已经被彻底废弃，前面已经提到过了）。

锁主要存在四种状态，依次是：**无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态**，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率

[锁升级过程分析](https://www.cnblogs.com/star95/p/17542850.html)



#### synchronized 和 volatile 有什么区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- `volatile` 关键字是线程同步的轻量级实现，所以 `volatile`性能肯定比`synchronized`关键字要好 。但是 `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。
- `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。
- `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。

