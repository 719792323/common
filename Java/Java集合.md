#### 集合继承关系图

![Java 集合框架概览](https://oss.javaguide.cn/github/javaguide/java/collection/java-collection-hierarchy.png)

![BlockingQueue 的实现类](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue-hierarchy.png)

![TreeMap 继承关系图](https://oss.javaguide.cn/github/javaguide/java/collection/treemap_hierarchy.png)

* `List`: 存储的元素是有序的、可重复的。
  * `ArrayList`：`Object[]` 数组。
  * **`Vector`：`Object[]` 数组。**
  * `LinkedList`：**双向链表**(JDK1.6 之前为循环链表，JDK1.7 取消了循环)
* `Set`: 存储的元素不可重复的。
  - `HashSet`(**无序**，唯一): 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素。
  - `LinkedHashSet`: `LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的，满足FIFIO使用特性。
  - `TreeSet`(**有序**，唯一): 红黑树(**自平衡的排序二叉树**)。
* `Queue/Deque`: `Queue`按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的，`Queue` 是单端队列，只能从一端插入元素，另一端删除元素，`Deque` 是双端队列，在队列的两端均可以插入或删除元素。。
  * `PriorityQueue`: `Object[]` 数组来实现的小顶堆
  * `DelayQueue`:基于`PriorityQueue`实现
  * `ArrayDeque`: 可扩容动态双向数组
* `Map`: 使用键值对（key-value）存储
  * `HashMap`：**JDK1.8 之前 `HashMap` 由数组+链表组成的**，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。**JDK1.8 及以后，当链表长度大于阈值（默认为 8）将链表转化为红黑树**，以减少搜索时间（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）
  * `LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，**增加了一条双向链表**（该数据结构=HashMap+双向链表），使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，**实现了访问顺序相关逻辑(支持遍历时会按照插入顺序有序进行迭代)**
  * `TreeMap`：**红黑树**（**该Map有排序性质**），即如果遍历entrySet时，遍历的entry是根据排序性质迭代的
* BlockingQueue：阻塞队列，继承自 `Queue`，`BlockingQueue`阻塞的原因是其支持当队列没有元素时一直阻塞，直到有元素；还支持如果队列已满，一直等到队列可以放入新元素时再放入。![BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue.png)
  * `ArrayBlockingQueue`：使用数组实现的有界阻塞队列。在创建时**需要指定容量大小**，并**支持公平和非公平两种方式的锁访问机制**
  * `LinkedBlockingQueue`：使用单向链表实现的可选有界阻塞队列。在创建时可以指定容量大小，如果不指定则默认为`Integer.MAX_VALUE`。和`ArrayBlockingQueue`不同的是， **它仅支持非公平的锁访问机制**。
  * `PriorityBlockingQueue`：支持优先级排序的无界阻塞队列。元素必须实现`Comparable`接口或者在构造函数中传入`Comparator`对象，并且不能插入 null 元素。
  * `SynchronousQueue`：同步队列，是一种不存储元素的阻塞队列。**每个插入操作都必须等待对应的删除操作，反之删除操作也必须等待插入操作**。**因此，`SynchronousQueue`通常用于线程之间的直接传递数据。**
  * `DelayQueue`：延迟队列，其中的元素只有到了其指定的延迟时间，才能够从队列中出队
* **无序性**：无序性不等于随机性 ，无序性是指**存储的数据在底层数组中并非按照数组索引的顺序添加** ，而是根据数据的哈希值决定的。
* **不可重复**：不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法

#### 集合常用功能

#####  Comparable 和 Comparator

* Compareable

  ```java
  public interface Comparable<T> {
      int compareTo(T var1);
  }
  
  ```

  说明：对于a.compareTo(b)，如果a>b，返回-1，a=b返回0，a<b，返回1

* Comparator

  ```java
  @FunctionalInterface
  public interface Comparator<T> {
      int compare(T var1, T var2);
      ...
  }
  ```

  说明：如果compare(a,b)时，希望a排序在b的前面，则返回-1，否则返回1

##### Collections常用功能

* reverse：反转集合

#### 集合对比

* ArrayList和数组

  - 静态与动态：`ArrayList`会根据实际存储的元素自动动态地扩容或缩容，而 `Array` 被创建之后就不能改变它的长度了。
  - 泛型：`ArrayList` 允许你使用泛型来确保类型安全，`Array` 则不可以。
  - 存储类型：`ArrayList` 中只能存储**对象**。对于**基本类型数据**，需要使用其对应的包装类（如 Integer、Double 等）。`Array` 可以直接存储基本类型数据，也可以存储对象
  - 操作：`ArrayList` 支持插入、删除、遍历等常见操作，并且提供了丰富的 API 操作方法，比如 `add()`、`remove()`等。`Array` 只是一个固定长度的数组，只能**按照下标访问其中的元素**，不具备动态添加、删除元素的能力。
  - 创建：`ArrayList`创建时不需要指定大小，而`Array`创建时必须指定大小。

* ArrayList和Vector

  `ArrayList` 是 `List` 的主要实现类，**底层使用 `Object[]`存储**，适用于频繁的查找工作，**线程不安全** ，`Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，**线程安全**（因为所有操作基本都用sync关键字修饰了，性能低下）。

* Vector与Stack

  Stack继承于Vector，两者都是线程安全的（都是使用 `synchronized` 关键字进行同步处理），Vector是列表，Stack是栈。
  
* HashSet、LinkedHashSet、TreeSet

  * 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的
  * `HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）、`LinkedHashSet` 的底层数据结构是基于LinkedHashMap实现、`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
  * **`HashSet` 用于不需要保证元素插入和取出顺序的场景，`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，`TreeSet` 用于支持对元素自定义排序规则的场景**

* ArrayDeque和LinkedList作为Deque的区别

  * `ArrayDeque` 是基于可变长的数组和双指针来实现，而 `LinkedList` 则通过链表来实现
  * **`ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持**
  * **从性能的角度上，选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好**，`ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

* ArrayBlockingQueue 、LinkedBlockingQueue

  * 数据结构：`ArrayBlockingQueue` 基于数组实现，而 `LinkedBlockingQueue` 基于链表实现

  * 是否有界：`ArrayBlockingQueue` 是有界队列，必须在创建时指定容量大小。`LinkedBlockingQueue` 创建时可以不指定容量大小，默认是`Integer.MAX_VALUE`，也就是无界的。但也可以指定队列大小，从而成为有界的

  * **锁机制**：`ArrayBlockingQueue`中的锁是没有分离的，即生产和消费用的是同一个锁；`LinkedBlockingQueue`中的锁是分离的，即生产用的是`putLock`，消费是`takeLock`，这样可以防止生产者和消费者线程之间的锁争夺。

  * 空间占用：`ArrayBlockingQueue` 需要提前分配数组内存，而 `LinkedBlockingQueue` 则是动态分配链表节点内存。这意味着，`ArrayBlockingQueue` 在创建时就会占用一定的内存空间，且往往申请的内存比实际所用的内存更大，而`LinkedBlockingQueue` 则是根据元素的增加而逐渐占用内存空间。

* HashMap、HashTable

  * 线程安全：`HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰

  * 性能： 因为线程安全的问题，`HashMap` 比`Hashtable` 效率高

  * nul值限制：**`HashMap` 可以存储 null 的 key 和 value，**但 null 作为键只能有一个，null 作为值可以有多个；**Hashtable 不允许有 null 键和 null 值**，否则会抛出 `NullPointerException`。

  * 初始容量：

    * 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。
    *  创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）

  * 数据结构：HashMap：数组+链表/红黑树，HashTable：数组+链表

* HashTable、ConcurrentHashmap

  * 数据结构：ConcurrentHashmap和HashMap数据结构完全一样（注意版本1.7前后），HashTable略
  * 线程安全：都是线程安全
  


#### ArrayList

##### 基本内容

* 是否可以添加null值：可以，`ArrayList` 中可以存储任何类型的对象，包括 `null` 值
* 插入和删除时间复杂度：头部和指定位置操作O(n)，尾部操作O(1)
* 是否线程安全：否
* 空间利用率：`ArrayList` 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间

#### LiskedList（双向链表）

##### 基本内容

* 是否可以添加null值：可以
* 插入和删除时间复杂度：头部和尾部操作O(1)，指定位置操作O(n)
* 是否线程安全：否
* 空间利用率： LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）

#### PriorityQueue

##### 基本内容

* 出队方式：`PriorityQueue` 是在 JDK1.5 中被引入的, 其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队
* 数据结构：利用了堆的数据结构来实现的，底层使用可变长的数组来存储数据
* 操作时间复杂度：插入了删除logn
* 是否线程安全：否

#### HashMap

##### 实现细节

* **数据集结构转换细节**

  ![jdk1.8之后的内部结构-HashMap](https://oss.javaguide.cn/github/javaguide/java/collection/jdk1.8_hashmap.png)

   JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间

  `putVal` 方法中执行链表转红黑树的判断逻辑。**

  链表的长度大于 8 的时候，就执行 `treeifyBin` （转换红黑树）的逻辑。

  ```java
  // 遍历链表
  for (int binCount = 0; ; ++binCount) {
      // 遍历到链表最后一个节点
      if ((e = p.next) == null) {
          p.next = newNode(hash, key, value, null);
          // 如果链表元素个数大于等于TREEIFY_THRESHOLD（8）
          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
              // 红黑树转换（并不会直接转换成红黑树）
              treeifyBin(tab, hash);
          break;
      }
      if (e.hash == hash &&
          ((k = e.key) == key || (key != null && key.equals(k))))
          break;
      p = e;
  }
  ```

  `treeifyBin` 方法中判断是否真的转换为红黑树。

  ```java
  final void treeifyBin(Node<K,V>[] tab, int hash) {
      int n, index; Node<K,V> e;
      // 判断当前数组的长度是否小于 64
      if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
          // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
          resize();
      else if ((e = tab[index = (n - 1) & hash]) != null) {
          // 否则才将列表转换为红黑树
  
          TreeNode<K,V> hd = null, tl = null;
          do {
              TreeNode<K,V> p = replacementTreeNode(e, null);
              if (tl == null)
                  hd = p;
              else {
                  p.prev = tl;
                  tl.next = p;
              }
              tl = p;
          } while ((e = e.next) != null);
          if ((tab[index] = hd) != null)
              hd.treeify(tab);
      }
  }
  ```

* 长度细节

  HashMap 的长度是 2 的幂次方的原因是为了计算key对应的数据索引更快，HashMap中计算数组下标的算法是：(n - 1) & hash，其中n是数组长度，hash是key对应的hash值，**取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方**

* 多线程问题

  * 死循环问题

    JDK1.7 及之前版本的 `HashMap` 在多线程环境下扩容操作可能存在死循环问题，这是由于当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，**头插法可能会导致链表中的节点指向错误的位置，从而形成一个环形链表**，进而使得查询元素的操作陷入死循环无法结束，JDK1.8 版本的 HashMap 采用了尾插法而不是头插法来避免链表倒置，使得插入的节点永远都是放在链表的末尾，避免了链表中的环形结构

  * 覆盖问题

    多线程下使用 `HashMap` 还是会存在数据覆盖的问题。并发环境下，推荐使用 `ConcurrentHashMap` 。

    * 案例1：两个线程 1,2 同时进行 put 操作，并且发生了哈希冲突（hash 函数计算出的插入下标是相同的），不同的线程可能在不同的时间片获得 CPU 执行的机会，当前线程 1 执行完哈希冲突判断后，由于时间片耗尽挂起。线程 2 先完成了插入操作。随后，线程 1 获得时间片，由于之前已经进行过 hash 碰撞的判断，所有此时会直接进行插入，这就导致线程 2 插入的数据被线程 1 覆盖了。

      ```java
      public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
      
      final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                         boolean evict) {
          // ...
          // 判断是否出现 hash 碰撞
          // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          // 桶中已经存在元素（处理hash冲突）
          else {
          // ...
      }
      
      ```

    * 案例2：两个线程同时 `put` 操作导致 `size` 的值不正确，进而导致数据覆盖的问题，线程 1 执行 `if(++size > threshold)` 判断时，假设获得 `size` 的值为 10，由于时间片耗尽挂起，线程 2 也执行 `if(++size > threshold)` 判断，获得 `size` 的值也为 10，并将元素插入到该桶位中，并将 `size` 的值更新为 11，随后，线程 1 获得时间片，它也将元素放入桶位中，并将 size 的值更新为 11，线程 1、2 都执行了一次 `put` 操作，但是 `size` 的值只增加了 1，也就导致实际上只有一个元素被添加到了 `HashMap` 中。

  
  #### ConcureentHashMap
  
  ##### 基本内容
  
  * 是否可以添加null值：key和value不能为null，`ConcurrentHashMap` 的 key 和 value 不能为 null 主要是为了避免二义性
  
  * 能保证复合操作的原子性：不能，复合操作是指由多个基本操作(如`put`、`get`、`remove`、`containsKey`等)组成的操作
  
  * 原子操作问题：
  
    复合操作是指由多个基本操作(如`put`、`get`、`remove`、`containsKey`等)组成的操作，例如先判断某个键是否存在`containsKey(key)`，然后根据结果进行插入或更新`put(key, value)`。这种操作在执行过程中可能会被其他线程打断，导致结果不符合预期。
  
    `ConcurrentHashMap` 提供了一些原子性的复合操作，如 `putIfAbsent`、`compute`、`computeIfAbsent` 、`computeIfPresent`、`merge`等。这些方法都可以接受一个函数作为参数，根据给定的 key 和 value 来计算一个新的 value，并且将其更新到 map 中。
  
  ##### 实现细节
  
  1. **JDK1.7之前**
  
     在 JDK1.7 的时候，`ConcurrentHashMap` 对整个桶数组进行了分割分段(`Segment`，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
  
     一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组，`Segment` 的个数一旦**初始化就不能改变**。 `Segment` 数组的大小默认是 16，也就是说默认可以同时支持 16 个线程并发写。
  
     `Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁。也就是说，对同一 `Segment` 的并发写入会被阻塞，不同 `Segment` 的写入是可以并发执行的。
  
     ![Java7 ConcurrentHashMap 存储结构](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)
  
  2. **JDK1.8之后**
  
     `ConcurrentHashMap` 已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 `synchronized` 锁做了很多优化） 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本。
  
     `ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 `Node + CAS + synchronized` 来保证并发安全。数据结构跟 `HashMap` 1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）。
  
     Java 8 中，锁粒度更细，`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。
  
     ![Java8 ConcurrentHashMap 存储结构](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

#### LinkedHashMap

`LinkedHashMap` 是 Java 集合框架中 `HashMap` 的一个子类，**它继承了 `HashMap`** 的所有属性和方法，并**且在 `HashMap` 的基础重写了 `afterNodeRemoval`、`afterNodeInsertion`、`afterNodeAccess` 方法。使之拥有顺序插入和访问有序的特性**。

##### 常见问题：

1. 如何按照插入顺序迭代元素

   `LinkedHashMap` 按照插入顺序迭代元素是它的默认行为。`LinkedHashMap` 内部维护了一个双向链表，用于记录元素的插入顺序。因此，当使用迭代器迭代元素时，元素的顺序与它们最初插入的顺序相同。

2. 如何按照访问顺序迭代元素

   `LinkedHashMap` 可以通过构造函数中的 `accessOrder` 参数指定按照访问顺序迭代元素。**当 `accessOrder` 为 true 时**，每次访问一个元素时，该元素会被移动到链表的末尾，因此下次访问该元素时，它就会成为链表中的最后一个元素，从而实现按照访问顺序迭代元素

3. 实现LRU

   将 `accessOrder` 设置为 true 并重写 `removeEldestEntry` 方法当链表大小超过容量时返回 true，使得每次访问一个元素时，该元素会被移动到链表的末尾。一旦插入操作让 `removeEldestEntry` 返回 true 时，视为缓存已满，`LinkedHashMap` 就会将链表首元素移除，由此我们就能实现一个 LRU 缓存

4. 与HashMap区别

   `LinkedHashMap` 和 `HashMap` 都是 Java 集合框架中的 Map 接口的实现类。它们的最大区别在于迭代元素的顺序。`HashMap` 迭代元素的顺序是不确定的，而 `LinkedHashMap` 提供了按照插入顺序或访问顺序迭代元素的功能。此外，`LinkedHashMap` 内部维护了一个双向链表，用于记录元素的插入顺序或访问顺序，而 `HashMap` 则没有这个链表。因此，`LinkedHashMap` 的插入性能可能会比 `HashMap` 略低，但它提供了更多的功能并且迭代效率相较于 `HashMap` 更加高效

   

#### CopyOnWriteArrayList

`CopyOnWriteArrayList` 中的**读取操作是完全无需加锁的**，**写入操作也不会阻塞读取操作，只有写写才会互斥**。

**存在问题：**

1. 内存占用：每次写操作都需要复制一份原始数据，会占用额外的内存空间，在数据量比较大的情况下，可能会导致内存资源不足。

2. 写操作开销：每一次写操作都需要复制一份原始数据，然后再进行修改和替换，所以写操作的开销相对较大，在写入比较频繁的场景下，性能可能会受到影响。

3. **数据一致性问题**：修改操作不会立即反映到最终结果中，还需要等待复制完成，这可能会导致一定的数据一致性问题。



#### BlockQueue

##### 基本内容

```java
public interface BlockingQueue<E> extends Queue<E> {

     //元素入队成功返回true，反之则会抛出异常IllegalStateException
    boolean add(E e);

     //元素入队成功返回true，反之返回false
    boolean offer(E e);

     //元素入队成功则直接返回，如果队列已满元素不可入队则将线程阻塞，因为阻塞期间可能会被打断，所以这里方法签名抛出了InterruptedException
    void put(E e) throws InterruptedException;

   //和上一个方法一样,只不过队列满时只会阻塞单位为unit，时间为timeout的时长，如果在等待时长内没有入队成功则直接返回false。
    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    //从队头取出一个元素，如果队列为空则阻塞等待，因为会阻塞线程的缘故，所以该方法可能会被打断，所以签名定义了InterruptedException
    E take() throws InterruptedException;

      //取出队头的元素并返回，如果当前队列为空则阻塞等待timeout且单位为unit的时长，如果这个时间段没有元素则直接返回null。
    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

      //获取队列剩余元素个数
    int remainingCapacity();

     //删除我们指定的对象，如果成功返回true，反之返回false。
    boolean remove(Object o);

    //判断队列中是否包含指定元素
    public boolean contains(Object o);

     //将队列中的元素全部存到指定的集合中
    int drainTo(Collection<? super E> c);

    //转移maxElements个元素到集合中
    int drainTo(Collection<? super E> c, int maxElements);
}

```

##### ArrayBlockingQueue

###### 基本特点

`ArrayBlockingQueue` 是 `BlockingQueue` 接口的**有界队列实现类**，常用于**多线程之间的数据共享**，底层采用数组实现。

1. `ArrayBlockingQueue` 的容量有限，一旦创建，容量不能改变。

2. 为了保证线程安全，`ArrayBlockingQueue` 的并发控制采用**可重入锁 `ReentrantLock` ，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。并且，它还支持公平和非公平两种方式的锁访问机制，默认是非公平锁。**

3. `ArrayBlockingQueue` 虽名为阻塞队列，但**也支持非阻塞获取和新增元素**（例如 `poll()` 和 `offer(E e)` 方法），只是队列满时添加元素会抛出异常，队列为空时获取的元素为 null，一般不会使用。

###### 实现原理

1. `ArrayBlockingQueue` 内部维护一个定长的数组用于存储元素。

2. 通过使用 `ReentrantLock` 锁对象对读写操作进行同步，即通过锁机制来实现线程安全。

3. 通过 `Condition` 实现线程间的等待和唤醒操作。

   唤醒详细过程：

   * 当队列已满时，生产者线程会调用 `notFull.await()` 方法让生产者进行等待，等待队列非满时插入（非满条件）。
   * 当队列为空时，消费者线程会调用 `notEmpty.await()`方法让消费者进行等待，等待队列非空时消费（非空条件）。
   * 当有新的元素被添加时，生产者线程会调用 `notEmpty.signal()`方法唤醒正在等待消费的消费者线程。
   * 当队列中有元素被取出时，消费者线程会调用 `notFull.signal()`方法唤醒正在等待插入元素的生产者线程。

###### 方法

| 方法                                      | 队列满时处理方式                                         | 方法返回值 |
| ----------------------------------------- | -------------------------------------------------------- | ---------- |
| `put(E e)`                                | 线程阻塞，直到中断或被唤醒                               | void       |
| `offer(E e)`                              | 直接返回 false                                           | boolean    |
| `offer(E e, long timeout, TimeUnit unit)` | 指定超时时间内阻塞，超过规定时间还未添加成功则返回 false | boolean    |
| `add(E e)`                                | 直接抛出 `IllegalStateException` 异常                    | boolean    |

获取/移除元素：

| 方法                                | 队列空时处理方式                                    | 方法返回值 |
| ----------------------------------- | --------------------------------------------------- | ---------- |
| `take()`                            | 线程阻塞，直到中断或被唤醒                          | E          |
| `poll()`                            | 返回 null                                           | E          |
| `poll(long timeout, TimeUnit unit)` | 指定超时时间内阻塞，超过规定时间还是空的则返回 null | E          |
| `peek()`                            | 返回 null                                           | E          |
| `remove()`                          | 直接抛出 `NoSuchElementException` 异常              | boolean    |

![img](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-get-add-element-methods.png)

##### LinkedBlockingQueue

`ArrayBlockingQueue` 和 `LinkedBlockingQueue` 是 Java 并发包中常用的两种阻塞队列实现，它们都是线程安全的。不过，不过它们之间也存在下面这些区别：

- 底层实现：`ArrayBlockingQueue` 基于数组实现，而 `LinkedBlockingQueue` 基于链表实现。
- 是否有界：`ArrayBlockingQueue` 是有界队列，必须在创建时指定容量大小。`LinkedBlockingQueue` 创建时可以不指定容量大小，默认是`Integer.MAX_VALUE`，也就是无界的。但也可以指定队列大小，从而成为有界的。
- **锁是否分离**： `ArrayBlockingQueue`中的锁是没有分离的，即生产和消费用的是同一个锁；`LinkedBlockingQueue`中的锁是分离的，即生产用的是`putLock`，消费是`takeLock`，这样可以防止生产者和消费者线程之间的锁争夺。
- 内存占用：`ArrayBlockingQueue` 需要提前分配数组内存，而 `LinkedBlockingQueue` 则是动态分配链表节点内存。这意味着，`ArrayBlockingQueue` 在创建时就会占用一定的内存空间，且往往申请的内存比实际所用的内存更大，而`LinkedBlockingQueue` 则是根据元素的增加而逐渐占用内存空间。

##### ConcurrentLinkedQueue

`ArrayBlockingQueue` 和 `ConcurrentLinkedQueue` 是 Java 并发包中常用的两种队列实现，它们都是线程安全的。不过，不过它们之间也存在下面这些区别：

- 底层实现：`ArrayBlockingQueue` 基于数组实现，而 `ConcurrentLinkedQueue` 基于链表实现。
- 是否有界：`ArrayBlockingQueue` 是有界队列，必须在创建时指定容量大小，而 `ConcurrentLinkedQueue` 是无界队列，可以动态地增加容量。
- 是否阻塞：`ArrayBlockingQueue` 支持阻塞和非阻塞两种获取和新增元素的方式（一般只会使用前者）， `ConcurrentLinkedQueue` 是无界的，仅支持非阻塞式获取和新增元素。

#### PriorityQueue

1. 使用堆结构的优势

* 假如我们使用排序的数组，那么入队操作则是 O(1),但出队操作确实得我们需要遍历一遍数组找到最小的哪一个,所以复杂度为 O(n)。

* 假如我们使用有序数组来实现，那么入队操作则因为需要排序变为 O(n),而出队操作则变为 O(1)。
  所以折中考虑，使用带有完全二叉树性质的二叉堆，使得入队和出队操作都是 O(log^n)最合适

2. **是否线程安全**

   不是线程安全

3. 底层实现数据结构，初始容量

   `PriorityQueue` 底层使用一个数组来表示优先队列，而这个数组实际上用到的 **小顶堆** 的思想来维持优先级的，初始化容量我们可以查看构造方法有一个`DEFAULT_INITIAL_CAPACITY,DEFAULT_INITIAL_CAPACITY` 用于初始化数组，查看其定义可以发现默认初始化容量大小为 11

4. 如何实现从大到小排序

   重写compartor方法即可

5. 使用数组构成堆而不是用链表原因

   **使用数组避免了为维护父子及左邻右舍等节点关系的内存空间占用** 

#### DelayQueue

1. 实现原理

   `DelayQueue` 底层是**使用优先队列 `PriorityQueue` 来存储元素**，而 `PriorityQueue` 采用二叉小顶堆的思想确保值小的元素排在最前面，这就使得 `DelayQueue` 对于延迟任务优先级的管理就变得十分方便了。**同时 `DelayQueue` 为了保证线程安全还用到了可重入锁 `ReentrantLock`,确保单位时间内只有一个线程可以操作延迟队列**。最后，**为了实现多线程之间等待和唤醒的交互效率，`DelayQueue` 还用到了 `Condition`，通过 `Condition` 的 `await` 和 `signal` 方法完成多线程之间的等待唤醒。**

2. 是否线程安全

   是，底层使用了ReentrantLock保证了线程安全

3. 应用场景

   `DelayQueue` 通常用于实现定时任务调度和缓存过期删除等场景。在定时任务调度中，需要将需要执行的任务封装成延迟任务对象，并将其添加到 `DelayQueue` 中，`DelayQueue` 会自动按照剩余延迟时间进行升序排序(默认情况)，以保证任务能够按照时间先后顺序执行。对于缓存过期这个场景而言，在数据被缓存到内存之后，我们可以将缓存的 key 封装成一个延迟的删除任务，并将其添加到 `DelayQueue` 中，当数据过期时，拿到这个任务的 key，将这个 key 从内存中移除。

4. Delayed接口作用

   `Delayed` 接口定义了元素的剩余延迟时间(`getDelay`)和元素之间的比较规则(该接口继承了 `Comparable` 接口)。若希望元素能够存放到 `DelayQueue` 中，就必须实现 `Delayed` 接口的 `getDelay()` 方法和 `compareTo()` 方法，否则 `DelayQueue` 无法得知当前任务剩余时长和任务优先级的比较

5. 和Timer/TimerTask区别

   `DelayQueue` 和 `Timer/TimerTask` 都可以用于实现定时任务调度，但是它们的实现方式不同。`DelayQueue` 是基于优先级队列和堆排序算法实现的，可以实现多个任务按照时间先后顺序执行；而 `Timer/TimerTask` 是基于单线程实现的，只能按照任务的执行顺序依次执行，如果某个任务执行时间过长，会影响其他任务的执行。另外，`DelayQueue` 还支持动态添加和移除任务，而 `Timer/TimerTask` 只能在创建时指定任务

   
