#### 内存分布图

![内存区域常见配置参数](https://javaguide.cn/assets/%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E5%B8%B8%E8%A7%81%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0-YVcjKi3R.png)

#### 堆空间调整

* 堆内存

  与**性能有关的最常见实践**之一是根据应用程序要求初始化堆内存。如果我们需要指定最小和最大堆大小（推荐显示指定大小），以下参数可以帮助你实现：

  

  ```bash
  -Xms<heap size>[unit]
  -Xmx<heap size>[unit]
  ```

  - **heap size** 表示要初始化内存的具体大小。
  - **unit** 表示要初始化内存的单位。单位为 g (GB)、“m（MB）、k（KB）。

* 堆中young区域内存

  在堆总可用内存配置完成之后，第二大影响因素是为 `Young Generation` 在堆内存所占的比例。默认情况下，YG 的最小大小为 1310 *MB*，最大大小为*无限制*。

  一共有两种指定 新生代内存(Young Generation)大小的方法：

  **1.通过`-XX:NewSize`和`-XX:MaxNewSize`指定**

  

  ```bash
  -XX:NewSize=<young size>[unit]
  -XX:MaxNewSize=<young size>[unit]
  ```

  ------

  **2.通过`-Xmn<young size>[unit]`指定**

  如果我们要为 新生代分配 256m 的内存（NewSize 与 MaxNewSize 设为一致），我们的参数应该这样来写：

  

  ```bash
  -Xmn256
  ```

* **调优策略（扩大young大小或者比例）**

  将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况

  另外，你还可以通过 **`-XX:NewRatio=<int>`** 来设置老年代与新生代内存的比值。

  比如下面的参数就是设置老年代与新生代内存的比值为 1。也就是说老年代和新生代所占比值为 1：1，新生代占整个堆栈的 1/2。

  

  ```plain
  -XX:NewRatio=1
  ```

#### 永久代/元空间大小调整

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小



```bash
-XX:PermSize=N #方法区 (永久代) 初始大小
-XX:MaxPermSize=N #方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

```bash
-XX:MetaspaceSize=N #设置 Metaspace 的初始大小（是一个常见的误区，后面会解释）
-XX:MaxMetaspaceSize=N #设置 Metaspace 的最
```

注：

1. Metaspace 的初始容量并不是 `-XX:MetaspaceSize` 设置，无论 `-XX:MetaspaceSize` 配置什么值，对于 64 位 JVM 来说，Metaspace 的初始容量都是 21807104（约 20.8m）。

2. Metaspace 由于使用不断扩容到`-XX:MetaspaceSize`参数指定的量，就会发生 FGC，且之后每次 Metaspace 扩容都会发生 Full GC。

   也就是说，MetaspaceSize 表示 Metaspace 使用过程中触发 Full GC 的阈值，只对触发起作用。

#### 垃圾收集配置

1. 垃圾回收器指定

   JVM 具有四种类型的 GC 实现：

   - 串行垃圾收集器
   - 并行垃圾收集器
   - CMS 垃圾收集器
   - G1 垃圾收集器

   可以使用以下参数声明这些实现：

   

   ```bash
   -XX:+UseSerialGC
   -XX:+UseParallelGC
   -XX:+UseParNewGC
   -XX:+UseG1GC
   ```

2. 输出GC日志

   ```text
   # 必选
   # 打印基本 GC 信息
   -XX:+PrintGCDetails
   -XX:+PrintGCDateStamps
   # 打印对象分布
   -XX:+PrintTenuringDistribution
   # 打印堆数据
   -XX:+PrintHeapAtGC
   # 打印Reference处理信息
   # 强引用/弱引用/软引用/虚引用/finalize 相关的方法
   -XX:+PrintReferenceGC
   # 打印STW时间
   -XX:+PrintGCApplicationStoppedTime
   
   # 可选
   # 打印safepoint信息，进入 STW 阶段之前，需要要找到一个合适的 safepoint
   -XX:+PrintSafepointStatistics
   -XX:PrintSafepointStatisticsCount=1
   
   # GC日志输出的文件路径
   -Xloggc:/path/to/gc-%t.log
   # 开启日志文件分割
   -XX:+UseGCLogFileRotation
   # 最多分割几个文件，超过之后从头文件开始写
   -XX:NumberOfGCLogFiles=14
   # 每个文件上限大小，超过就触发分割
   -XX:GCLogFileSize=50M
   
   ```



#### OOM参数

JVM 提供了一些参数，这些参数将堆内存转储到一个物理文件中，以后可以用来查找泄漏:

```text
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./java_pid<pid>.hprof
-XX:OnOutOfMemoryError="< cmd args >;< cmd args >"
-XX:+UseGCOverheadLimit
```

**HeapDumpOnOutOfMemoryError** 指示 JVM 在遇到 **OutOfMemoryError** 错误时将 heap 转储到物理文件中。

**HeapDumpPath** 表示要写入文件的路径; 可以给出任何文件名; 但是，如果 JVM 在名称中找到一个 `<pid>` 标记，则当前进程的进程 id 将附加到文件名中，并使用`.hprof`格式

**OnOutOfMemoryError** 用于发出紧急命令，以便在内存不足的情况下执行; 应该在 `cmd args` 空间中使用适当的命令。例如，如果我们想在内存不足时重启服务器，我们可以设置参数: `-XX:OnOutOfMemoryError="shutdown -r"` 。

**UseGCOverheadLimit** 是一种策略，它限制在抛出 OutOfMemory 错误之前在 GC 中花费的 VM 时间的比例



