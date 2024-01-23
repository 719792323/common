#### synchronized、[volatile](https://link.zhihu.com/?target=https%3A//so.csdn.net/so/search%3Fq%3Dvolatile%26spm%3D1001.2101.3001.7020)关键字与内存屏障的关系

![img](https://pic4.zhimg.com/80/v2-8d036daffacc112dc43d4e2696df7067_720w.webp)

* synchronized关键字包的码区域，当程进入到临界区读变量信息时,保证读到的是最新的，这是因为在同步区内对变量的写入操作，在离开同步区时就将上前线程内的数据刷新到内存中,而对数据的读取也不能从缓存读取 只能从内存中读取保证了数据的读有效性这就是插入了**StoreStore**屏障
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