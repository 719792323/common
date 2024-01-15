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

