## 运行时数据区域

**JDK 1.7**：

![Java 运行时数据区域（JDK1.7）](https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.7.png)

**JDK 1.8**：

![Java 运行时数据区域（JDK1.8 ）](https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png)

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)

### 程序计数器

程序计数器是一块较小的内存空间，可以看作是**当前线程所执行的字节码的行号指示器**。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，**分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成**。

另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

从上面的介绍中我们知道了程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：**顺序执行、选择、循环、异常处理**。
- 在多线程的情况下，**程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了**。

⚠️ 注意：程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，**它的生命周期随着线程的创建而创建，随着线程的结束而死亡**。

### Java 虚拟机栈(VM Stack)

* 动态链接的作用就是为了将符号引用转换为调用方法的直接引用具体是怎么做的？
* StackOverFlowError和OutOfMemoryError的区别是什么？

与程序计数器一样，Java 虚拟机栈（后文简称栈）也是线程私有的，**它的生命周期和线程相同，随着线程的创建而创建，随着线程的死亡而死亡**。

栈绝对算的上是 JVM 运行时数据区域的一个核心，除了一些 Native 方法调用是通过本地方法栈实现的(后面会提到)，其他所有的 Java 方法调用都是通过栈来实现的（也需要和其他运行时数据区域比如程序计数器配合）。

方法调用的数据需要通过栈进行传递，**每一次方法调用都会有一个对应的栈帧被压入栈中，每一个方法调用结束后，都会有一个栈帧被弹出**。

栈由一个个栈帧组成，而每个栈帧中都拥有：**局部变量表、操作数栈、动态链接、方法返回地址**。和数据结构上的栈类似，两者都是先进后出的数据结构，只支持出栈和入栈两种操作。

![Java 虚拟机栈](https://oss.javaguide.cn/github/javaguide/java/jvm/stack-area.png)

***局部变量表*** 主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

![局部变量表](https://oss.javaguide.cn/github/javaguide/java/jvm/local-variables-table.png)

**操作数栈** 主要作为方法调用的中转站使用，用于存放方法执行过程中产生的中间计算结果。另外，计算过程中产生的临时变量也会放在操作数栈中。

***动态链接*** 主要服务一个方法需要调用其他方法的场景。Class 文件的常量池里保存有大量的符号引用比如方法引用的符号引用。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。**动态链接的作用就是为了将符号引用转换为调用方法的直接引用**，这个过程也被称为 **动态连接** 。

![img](https://oss.javaguide.cn/github/javaguide/jvmimage-20220331175738692.png)

栈空间虽然不是无限的，但一般正常调用的情况下是不会出现问题的。不过，如果函数调用陷入无限循环的话，就会导致栈中被压入太多栈帧而占用太多空间，导致栈空间过深。**那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。**

Java 方法有两种返回方式，一种是 return 语句正常返回，一种是抛出异常。不管哪种返回方式，都会导致栈帧被弹出。也就是说， **栈帧随着方法调用而创建，随着方法结束而销毁。无论方法正常完成还是异常完成都算作方法结束。**

**除了 `StackOverFlowError` 错误之外，栈还可能会出现`OutOfMemoryError`错误，这是因为如果栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。**

简单总结一下程序运行中栈可能会出现两种错误：

- **`StackOverFlowError`：** 若栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。
- **`OutOfMemoryError`：** 如果栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

![img](https://oss.javaguide.cn/github/javaguide/java/jvm/《深入理解虚拟机》第三版的第2章-虚拟机栈.png)

###  本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是：**虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误

### 堆

* 为什么GC基本只是对堆内存？

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的*唯一目的*就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 世界中“几乎”所有的对象都在堆中分配，但是，随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。**从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。**

在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永久代(Permanent Generation)

下图所示的 Eden 区、两个 Survivor 区 S0 和 S1 都属于新生代，中间一层属于老年代，最下面一层属于永久代。**JDK 8 版本之后 PermGen(永久代) 已被 Metaspace(元空间) 取代，元空间使用的是本地内存**

![堆内存结构](https://oss.javaguide.cn/github/javaguide/java/jvm/hotspot-heap-structure.png)

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 S0 或者 S1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置

### 堆常见错误

堆这里最容易出现的就是 `OutOfMemoryError` 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`**：当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发此错误。(和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx`参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap sizeopen in new window](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size))

### 堆外内存使用

* 为什么要使用堆外内存

  * 对垃圾回收停顿的改善（**堆内内存会经常收垃圾回收影响**）。由于堆外内存是直接受操作系统管理而不是 JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在 GC 时减少回收停顿对于应用的影响。

  * 提升程序 I/O 操作的性能。通常在 I/O 通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。

  `DirectByteBuffer` 是 Java 用于实现堆外内存的一个重要类，通常用在通信过程中做缓冲池，如在 Netty、MINA 等 NIO 框架中应用广泛。`DirectByteBuffer` 对于堆外内存的创建、使用、销毁等逻辑均由 Unsafe 提供的堆外内存 API 来实现。

  如下为 `DirectByteBuffer` 构造函数，创建 `DirectByteBuffer` 的时候，通过 `Unsafe.allocateMemory` 分配内存、`Unsafe.setMemory` 进行内存初始化，而后构建 `Cleaner` 对象用于跟踪 `DirectByteBuffer` 对象的垃圾回收，以实现当 `DirectByteBuffer` 被垃圾回收时，分配的堆外内存一起被释放。

  ```java
  DirectByteBuffer(int cap) {                   
      super(-1, 0, cap, cap);
      boolean pa = VM.isDirectMemoryPageAligned();
      int ps = Bits.pageSize();
      long size = Math.max(1L, (long)cap + (pa ? ps : 0));
      Bits.reserveMemory(size, cap);
  
      long base = 0;
      try {
          // 分配内存并返回基地址
          base = unsafe.allocateMemory(size);
      } catch (OutOfMemoryError x) {
          Bits.unreserveMemory(size, cap);
          throw x;
      }
      // 内存初始化
      unsafe.setMemory(base, size, (byte) 0);
      if (pa && (base % ps != 0)) {
          // Round up to page boundary
          address = base + ps - (base & (ps - 1));
      } else {
          address = base;
      }
      // 跟踪 DirectByteBuffer 对象的垃圾回收，以实现堆外内存释放
      cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
      att = null;
  }
  ```

### 方法区

* **方法区和永久代以及元空间是什么关系?**
* **为什么要用元空间替换永久代作为方法区的时间？**

方法区属于是 JVM 运行时数据区域的一块**逻辑区域**，是各个线程共享的内存区域。

《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，方法区到底要如何实现那就是虚拟机自己要考虑的事情了。也就是说，在不同的虚拟机实现上，方法区的实现是不同的。

方法区会存储已被虚拟机加载的 **类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**

**方法区和永久代以及元空间关系** 方法区和永久代以及元空间的关系很像 Java 中接口和类的关系，类实现了接口，这里的类就可以看作是永久代和元空间，接口可以看作是方法区，也就是说永久代以及元空间是 HotSpot 虚拟机对虚拟机规范中方法区的两种实现方式。并且，永久代是 JDK 1.8 之前的方法区实现，JDK 1.8 及以后方法区的实现变成了元空间。

![HotSpot 虚拟机方法区的两种实现](https://oss.javaguide.cn/github/javaguide/java/jvm/method-area-implementation.png)

**元空间替换永久代作为方法区实现原因**

1、整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是本地内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

> 当元空间溢出时会得到如下错误：`java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

2、元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。

3、在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

**方法区常用参数有哪些？**

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小。



```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。下面是一些常用参数：



```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

------

### 运行时常量池

* 为什么常量池也会存在OOM问题？

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译期生成的各种字面量（Literal）和符号引用（Symbolic Reference）的 **常量池表(Constant Pool Table)** 。

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数和字符串字面量。常见的符号引用包括类符号引用、字段符号引用、方法符号引用、接口方法符号。

**常量池表会在类加载后存放到方法区的运行时常量池中**，既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 `OutOfMemoryError` 错误。

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true

```

**JDK 1.7 为什么要将字符串常量池移动到堆中**

主要是因为永久代（方法区实现）的 GC 回收效率太低，只有在整堆收集 (Full GC)的时候才会被执行 GC。Java 程序中通常会有大量的被创建的字符串等待回收，将字符串常量池放到堆中，能够更高效及时地回收字符串内存



### 直接内存

直接内存是一种特殊的内存缓冲区，并不在 Java 堆或方法区中分配的，而是通过 JNI 的方式在本地内存上分配的。

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 错误出现。

**JDK1.4 中新加入的 NIO（Non-Blocking I/O，也被称为 New I/O），引入了一种基于通道（Channel）与缓存区（Buffer）的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据**。

直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

类似的概念还有 **堆外内存** 。在一些文章中将直接内存等价于堆外内存，个人觉得不是特别准确。

堆外内存就是把内存对象分配在堆（新生代+老年代+永久代）以外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响。



### 对象创建过程

* 类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是**否能在常量池中定位到这个类的符号引用**，并且检查这个符号引用代表的**类是否已被加载过、解析和初始化过**。如果没有，那必须先执行相应的类加载过程。

* 内存分配

  在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

  **内存分配的两种方式** ：

  - 指针碰撞： 
    - 适用场合：堆内存规整（即没有内存碎片）的情况下。
    - 原理：用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可。
    - 使用该分配方式的 GC 收集器：Serial, ParNew
  - 空闲列表： 
    - 适用场合：堆内存不规整的情况下。
    - 原理：虚拟机会维护一个列表，该列表中会记录哪些内存块是可用的，在分配的时候，找一块儿足够大的内存块儿来划分给对象实例，最后更新列表记录。
    - 使用该分配方式的 GC 收集器：CMS

  选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的。

  **内存分配并发问题（补充内容，需要掌握）**

  在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

  - **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
  - **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

* 初始化零值

  内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

* 设置对象头

  初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式

* 执行init方法（构造方法）

  在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的运行时数据**（**哈希码、GC 分代年龄、锁状态标志等等**），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

### 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有：**使用句柄**、**直接指针**。

* 句柄

如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与对象类型数据各自的具体地址信息。

![对象的访问定位-使用句柄](https://oss.javaguide.cn/github/javaguide/java/jvm/access-location-of-object-handle.png)

#### 直接指针

如果使用直接指针访问，reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](https://oss.javaguide.cn/github/javaguide/java/jvm/access-location-of-object-handle-direct-pointer.png)对象的访问定位-直接指针

这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

### 可能产生OOM的内存区域有

* 直接内存
* 运行时常量池
* 方法区
* 堆

### 内存分配和回收原则

* 对象优先分配在eden区

  对象在新生代中 Eden 区分配。当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。GC 期间虚拟机又发现 对象无法存入 Survivor 空间，会通过 **分配担保机制** 把新生代的对象提前转移到老年代中去，老年代上的空间如果足够存放，则不会出现 Full GC。执行 Minor GC 后，后面分配的对象如果能够存在 Eden 区的话，还是会在 Eden 区分配内存。

* 大对象直接进入老年代

  大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

  大对象直接进入老年代的行为是由虚拟机动态决定的，它与具体使用的垃圾回收器和相关参数有关。大对象直接进入老年代是一种优化策略，旨在避免将大对象放入新生代，从而减少新生代的垃圾回收频率和成本。

  - G1 垃圾回收器会根据 `-XX:G1HeapRegionSize` 参数设置的堆区域大小和 `-XX:G1MixedGCLiveThresholdPercent` 参数设置的阈值，来决定哪些对象会直接进入老年代。
  - Parallel Scavenge 垃圾回收器中，默认情况下，并没有一个固定的阈值(`XX:ThresholdTolerance`是动态调整的)来决定何时直接在老年代分配大对象。而是由虚拟机根据当前的堆内存情况和历史数据动态决定。

* 对象年龄成长过程

  对象都会首先在 Eden 区域分配。如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间（s0 或者 s1）中，并将对象年龄设为 1(Eden 区->Survivor 区后对象的初始年龄变为 1)。

  对象在 Survivor 中每熬过一次 MinorGC,年龄就增加 1 岁，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

  （Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的 50% 时（默认值是 50%，即survivor的中位年龄，可以通过 `-XX:TargetSurvivorRatio=percent` 来设置，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值）

* 主要gc区域

  针对 HotSpot VM 的实现，它里面的 GC 其实准确分类只有两大种：

  **部分收集** (Partial GC)：

  - 新生代收集（Minor GC / Young GC）：只对新生代进行垃圾收集；
  - 老年代收集（Major GC / Old GC）：只对老年代进行垃圾收集。需要注意的是 Major GC 在有的语境中也用于指代整堆收集；
  - 混合收集（Mixed GC）：对整个新生代和**部分**老年代进行垃圾收集。

  **整堆收集** (Full GC)：收集整个 Java 堆和方法区。

* 空间分配担保

  空间分配担保是为了确保在 Minor GC 之前老年代本身还有容纳新生代所有对象的剩余空间，JDK 6 Update 24 之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行 Minor GC，否则将进行 Full GC。

#### 对象死亡判断方法

1. **引用计数法**

给对象中添加一个引用计数器：

- 每当有一个地方引用它，计数器就加 1；
- 当引用失效，计数器就减 1；
- 任何时候计数器为 0 的对象就是不可能再被使用的。

**缺点：这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间循环引用的问题。**

![对象之间循环引用](https://oss.javaguide.cn/github/javaguide/java/jvm/object-circular-reference.png)

2. **引用可达法**

这个算法的基本思想就是通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的，需要被回收。

下图中的 `Object 6 ~ Object 10` 之间虽有引用关系，但它们到 GC Roots 不可达，因此为需要被回收的对象。

![可达性分析算法](https://oss.javaguide.cn/github/javaguide/java/jvm/jvm-gc-roots.png)

* 可以作为GC ROOTS的对象

  虚拟机栈(栈帧中的局部变量表)中引用的对象

  本地方法栈(Native 方法)中引用的对象

  方法区中类静态属性引用的对象(static)

  方法区中常量引用的对象(final)

  所有被同步锁持有的对象(sync、lock)

  JNI（Java Native Interface）引用的对象

* 对象可以被回收，就代表一定会被回收吗

  即使在可达性分析法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”**，要真正宣告一个对象死亡，至少要经历两次标记过程**；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 `finalize` 方法。当对象没有覆盖 `finalize` 方法，或 `finalize` 方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。

  **被判定为需要执行的对象将会被放在一个队列中进行第二次标记**，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。

  

### 引用类型分类

* 怎么判断对象是什么类型的引用？

引用分为强引用、软引用、弱引用、虚引用四种（引用强度逐渐减弱）

![Java 引用类型总结](https://oss.javaguide.cn/github/javaguide/java/jvm/java-reference-type.png)

**1．强引用（StrongReference没这个类）**

以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于**必不可少的生活用品**，**垃圾回收器绝不会回收它**。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

**2．软引用（SoftReference有这个类）**

如果一个对象只具有软引用，那就类似于可有可无的生活用品。**如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存**。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

**3．弱引用（WeakReference有这个类）**

如果一个对象只具有弱引用，那就类似于可有可无的生活用品。**弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，**一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存**。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

**4．虚引用（PhantomReference有这个类）**

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。**如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收**。

**虚引用主要用来跟踪对象被垃圾回收的活动**。

**虚引用与软引用和弱引用的一个区别在于：** 虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

特别注意，在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为**软引用可以加速 JVM 对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生**。

###  各类型引用的应用场景

* 软引用

  * 图片缓存
    在 Android 中，我们经常需要加载大量的图片，如果使用强引用来缓存这些图片，很容易导致内存溢出。因此，一种更好的做法是使用软引用来缓存图片对象。当系统内存不足时，垃圾回收器会回收软引用所指向的图片对象，从而释放内存。

  * 数据库缓存
    在一些需要频繁读写的数据库应用中，我们可以使用软引用来缓存最近访问的数据。当内存不足时，垃圾回收器会回收软引用所指向的数据对象，从而释放内存。

  * 线程池
    在使用线程池的应用中，我们可以使用软引用来缓存一些比较大的对象，如线程池的任务队列。当内存不足时，垃圾回收器会回收软引用所指向的对象，从而释放内存。

  如下使用软引用进行图片缓存的案例，使用了一个 `HashMap` 来保存软引用，实现了一个简单的图片缓存。当需要加载图片时，先从缓存中查找是否有软引用，如果有则直接获取软引用所指向的图片对象；否则从文件系统或网络加载图片，并将其放入缓存中。当内存不足时，垃圾回收器会回收软引用所指向的图片对象，从而释放内存

  ```java
  import java.lang.ref.SoftReference;
  import java.util.HashMap;
  import java.util.Map;
   
  public class ImageCache {
      private Map<String, SoftReference<Image>> cache = new HashMap<>();
   
      public Image getImage(String key) {
          SoftReference<Image> ref = cache.get(key);
          Image image = null;
          if (ref != null) {
              image = ref.get();
          }
          if (image == null) {
              image = loadImage(key);
              cache.put(key, new SoftReference<>(image));
          }
          return image;
      }
   
      private Image loadImage(String key) {
          // 从文件系统或网络加载图片
          return null;
      }
  }
  ```

* 弱引用

  弱引用是一种特殊的引用类型，在使用时不会增加对象的引用计数，也不会阻止对象被垃圾回收。当对象被垃圾回收后，弱引用会自动失效，不再指向任何对象

  * 弱引用可以用来实现缓存，当内存不足时，垃圾回收器会自动清理缓存中的对象。弱引用可以指向缓存中的对象，但不会阻止垃圾回收器回收它们。
  * 对象池是一种常见的设计模式，用于避免频繁地创建和销毁对象。使用弱引用可以让对象池中的对象在不再使用时自动被回收，从而避免内存泄漏
  * 在观察者模式中，观察者需要注册到主题对象上，并在主题对象状态变化时接收通知。使用弱引用可以避免观察者和主题对象之间形成循环引用，从而避免内存泄漏。

  使用案例1：

  ```java
  public static void main(String[] args) {
          Person person = new Person();
          WeakReference<Person> ref = new WeakReference<>(person);//给对象添加一个弱引用
          System.gc();//触发gc
          System.out.println(ref.get());//因为有强引用，所以对象不会失效，可以从弱引用中拿对象
          person = null;//删除对象强引用
          System.gc();
          System.out.println(ref.get());//发现弱引用已经失效
          ref = new WeakReference<>(new Person());//创建一个只有弱引用的对象
          System.out.println(ref.get());//gc频率较低，短时间不会被回收
          System.gc();
          System.out.println(ref.get());//null
      }
  ```

  使用案例2：和引用队列搭配

  ```java
  public class ReferenceTest {
   
      private static ReferenceQueue<VeryBig> rq = new ReferenceQueue<VeryBig>();
   
      public static void checkQueue() {
          Reference<? extends VeryBig> ref = null;
          while ((ref = rq.poll()) != null) {
              if (ref != null) {
                  System.out.println("In queue: "    + ((VeryBigWeakReference) (ref)).id);
              }
          }
      }
   
      public static void main(String args[]) {
          int size = 3;
          LinkedList<WeakReference<VeryBig>> weakList = new LinkedList<WeakReference<VeryBig>>();
          for (int i = 0; i < size; i++) {
              weakList.add(new VeryBigWeakReference(new VeryBig("Weak " + i), rq));
              System.out.println("Just created weak: " + weakList.getLast());
   
          }
   
          System.gc(); 
          try { // 下面休息几分钟，让上面的垃圾回收线程运行完成
              Thread.currentThread().sleep(6000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          checkQueue();
      }
  }
   
  class VeryBig {
      public String id;
      // 占用空间,让线程进行回收
      byte[] b = new byte[2 * 1024];
   
      public VeryBig(String id) {
          this.id = id;
      }
   
      protected void finalize() {
          System.out.println("Finalizing VeryBig " + id);
      }
  }
   
  class VeryBigWeakReference extends WeakReference<VeryBig> {
      public String id;
   
      public VeryBigWeakReference(VeryBig big, ReferenceQueue<VeryBig> rq) {
          super(big, rq);
          this.id = big.id;
      }
   
      protected void finalize() {
          System.out.println("Finalizing VeryBigWeakReference " + id);
      }
  }
  ```

  

#### 什么是废弃常量

运行时常量池主要回收的是废弃的常量。假如在字符串常量池中存在字符串 "abc"，如果当前没有任何 String 对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

注：

* JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代

* JDK1.7 字符串常量池被从方法区拿到了堆中（因为堆gc更加频繁，而字符串常量通常需要被回收的更频繁）, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是 hotspot 中的永久代 。

* JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)

#### 怎么判断一个类（class）已经无用

方法区主要回收的是无用的类，那么如何判断一个类是无用的类的呢？

判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面 3 个条件才能算是 **“无用的类”**：

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
- 加载该类的 `ClassLoader` 已经被回收。
- 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。（Class对象）

虚拟机可以对满足上述 3 个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

#### 内存回收算法

* 标记-清除算法

标记-清除（Mark-and-Sweep）算法分为“标记（Mark）”和“清除（Sweep）”阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。

它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. **效率问题**：标记和清除两个过程效率都不高。
2. **空间问题**：标记清除后会产生大量不连续的内存碎片。

![标记-清除算法](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-sweep-garbage-collection-algorithm.png)

* 复制算法（适合青年代）

  为了解决标记-清除算法的效率和内存碎片问题，复制（Copying）收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

  ![复制算法](https://oss.javaguide.cn/github/javaguide/java/jvm/copying-garbage-collection-algorithm.png)复制算法

  虽然改进了标记-清除算法，但依然存在下面这些问题：

  - **可用内存变小**：可用内存缩小为原来的一半。
  - **不适合老年代**：如果存活对象数量比较大，复制性能会变得很差。

* 标记-整理法（适合老年代）

  标记-整理（Mark-and-Compact）算法是根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。

  ![标记-整理算法](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-compact-garbage-collection-algorithm.png)

  由于多了整理这一步，因此效率也不高，**适合老年代这种垃圾回收频率不是很高的场景。**

* 分代收集算法（针对不同划分区使用不同算法）

  当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将 Java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

  比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。

  **延伸面试问题：** HotSpot 为什么要分为新生代和老年代？

  根据上面的对分代收集算法的介绍回答。

#### 垃圾收集器分类

JDK 默认垃圾收集器（使用 `java -XX:+PrintCommandLineFlags -version` 命令查看）：

- JDK 8：Parallel Scavenge（新生代）+ Parallel Old（老年代）
- **JDK 9 ~ JDK20: G1**

##### 具体一些垃圾分类器

* serial（新生代收集器）

  一个单线程收集器，它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ **"Stop The World"** ），直到它收集结束。**在新生代采用标记-复制算法，老年代采用标记-整理算法**。

  ![Serial 收集器](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

  Serial 收集器由于没有线程交互的开销，自然可以获得很高的单线程收集效率。Serial 收集器对于运行在 Client 模式下的虚拟机来说是个不错的选择

* ParNew（新生代收集器）

  **ParNew 收集器其实就是 Serial 收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和 Serial 收集器完全一样**。

  它是许多运行在 Server 模式下的虚拟机的首要选择，除了 Serial 收集器外，只有它能与 CMS 收集器（真正意义上的并发收集器，后面会介绍到）配合工作。

* Parallel Scavenge

  Parallel Scavenge 收集器也是新标记-复制，老标记整理算法的多线程收集器，它看上去几乎和 ParNew 都一样。 

  

  ```bash
  -XX:+UseParallelGC
  
      使用 Parallel 收集器+ 老年代串行
  
  -XX:+UseParallelOldGC
  
      使用 Parallel 收集器+ 老年代并行
  ```

  **Parallel Scavenge 收集器关注点是吞吐量**（高效率的利用 CPU）。**CMS 等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）**。所谓吞吐量就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值。 Parallel Scavenge 收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解，手工优化存在困难的时候，使用 Parallel Scavenge 收集器配合自适应调节策略，把内存管理优化交给虚拟机去完成也是一个不错的选择。

* serial old（老年代收集器）

  **Serial 收集器的老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途是作为 CMS 收集器的后备方案。

  ![Serial 收集器](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

* parallel old（老年代收集器）

  **Parallel Scavenge 收集器的老年代版本**。使用多线程和“标记-整理”算法。在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

  ![Parallel Old收集器运行示意图](https://oss.javaguide.cn/github/javaguide/java/jvm/parallel-scavenge-garbage-collector.png)

* cms

  **CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用。**

  **CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。**

  从名字中的**Mark Sweep**这两个词可以看出，CMS 收集器是一种 **“标记-清除”算法**实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

  - **初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
  - **并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
  - **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
  - **并发清除：** 开启用户线程，同时 GC 线程开始对未标记的区域做清扫。

  ![CMS 收集器](https://oss.javaguide.cn/github/javaguide/java/jvm/cms-garbage-collector.png)

  从它的名字就可以看出它是一款优秀的垃圾收集器，主要优点：**并发收集、低停顿**。但是它有下面三个明显的缺点：

  - **对 CPU 资源敏感；**
  - **无法处理浮动垃圾；**
  - **它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。**

* g1

  **G1 (Garbage-First) 是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.**

  被视为 JDK1.7 中 HotSpot 虚拟机的一个重要进化特征。它具备以下特点：

  - **并行与并发**：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
  - **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。
  - **空间整合**：与 CMS 的“标记-清除”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。
  - **可预测的停顿**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒。

  G1 收集器的运作大致分为以下几个步骤：

  - **初始标记**
  - **并发标记**
  - **最终标记**
  - **筛选回收**

  ![G1 收集器](https://oss.javaguide.cn/github/javaguide/java/jvm/g1-garbage-collector.png)G1 收集器

  **G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来)** 。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。

  **从 JDK9 开始，G1 垃圾收集器成为了默认的垃圾收集器。**

* ZGC

  与 CMS 中的 ParNew 和 G1 类似，ZGC 也采用标记-复制算法，不过 ZGC 对该算法做了重大改进。

  在 ZGC 中出现 Stop The World 的情况会更少！

  Java11 的时候 ，ZGC 还在试验阶段。经过多个版本的迭代，不断的完善和修复问题，ZGC 在 Java 15 已经可以正式使用了！

  不过，默认的垃圾回收器依然是 G1。你可以通过下面的参数启动 ZGC：

  

  ```bash
  java -XX:+UseZGC className
  ```

  关于 ZGC 收集器的详细介绍推荐阅读美团技术团队的 [新一代垃圾回收器 ZGC 的探索与实践open in new window](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html) 这篇文章

  
