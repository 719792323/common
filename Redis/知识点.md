# 缓存概念

### 1.缓存作用

缓存用于提高系统性能减少数据重新获取所需的时间，缓存的设计理念是空间换时间，将数据存储到更高速的物理介质上降低再获取耗时。

### 2.本地缓存

本地缓存位于应用内部，其最大的优点是应用存在于同一个进程内部，请求本地缓存的速度非常快，不存在额外的网络开销。本地缓存有：高效（没有网络开销）、低依赖、轻量、简单、成本低等特点，但是对分布式架构缺乏支持且受部署机器内存容量影响明显。

常见的本地缓存：

* ConcurrentHashMap

  只提供简单的存储获取功能，没有过期、淘汰、命中率统计等功能

* guava cache、caffeine

  提供了缓存基本功能上，还要过期、淘汰、命中率统计、加载等等功能。

### 3.分布式缓存

分布式缓存脱离于应用独立存在，多个应用可直接的共同使用同一个分布式缓存服务。

### 4.多级缓存

* [redis+caffeine使用](https://mp.weixin.qq.com/s/_ysKYrzyRGebtotGyzQUIw)

好处是提升性能和提高资源利用率，缺点是增加了缓存一致性维护的难度

# Redis数据类型

* Bitmap、HyperLogLog、GEO是什么数据类型

Redis 内置了多种数据类型实现（比如 String、Hash、Sorted Set、Bitmap、HyperLogLog、GEO）。并且，Redis 还支持**事务**、**发布订阅**、持久化、Lua 脚本、多种开箱即用的集群方案（Redis Sentinel、Redis Cluster）。

![Redis 数据类型概览](https://oss.javaguide.cn/github/javaguide/database/redis/redis-overview-of-data-types-2023-09-28.jpg)

### 1.string类型

* 常见应用

  * 常规数据（比如 Session、Token、序列化后的对象、图片的路径）的缓存；

  * 计数比如用户单位时间的请求数（简单限流可以用到）、页面单位时间的访问数（incr、decr）

  * 分布式锁(利用 `SETNX key value` 命令可以实现一个最简易的分布式锁)；

  | 命令                            | 介绍                             |
  | ------------------------------- | -------------------------------- |
  | SET key value                   | 设置指定 key 的值                |
  | **SETNX key value**             | 只有在 key 不存在时设置 key 的值 |
  | GET key                         | 获取指定 key 的值                |
  | MSET key1 value1 key2 value2 …… | 设置一个或多个指定 key 的值      |
  | MGET key1 key2 ...              | 获取一个或多个指定 key 的值      |
  | STRLEN key                      | 返回 key 所储存的字符串值的长度  |
  | **INCR key**                    | 将 key 中储存的数字值增一        |
  | **DECR key**                    | 将 key 中储存的数字值减一        |
  | EXISTS key                      | 判断指定 key 是否存在            |
  | DEL key（通用）                 | 删除指定的 key                   |
  | EXPIRE key seconds（通用）      | 给指定 key 设置过期时间          |

* 数据结构特点

  ```c
  len：字符串的长度也就是已经使用的字节数
  alloc：总共可用的字符空间大小，alloc-len 就是 SDS 剩余的空间大小
  buf[]：实际存储字符串的数组
  flags：低三位保存类型标志
  ```

  SDS 相比于 C 语言中的字符串有如下提升：

  1. **可以避免缓冲区溢出**：C 语言中的字符串被修改（比如拼接）时，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。SDS 被修改时，**会先根据 len 属性检查空间大小是否满足要求，如果不满足，则先扩展至所需大小再进行修改操作**。
  2. **获取字符串长度的复杂度较低**：C 语言中的字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。**SDS 的长度获取直接读取 len 属性即可，时间复杂度为 O(1)**。
  3. **减少内存分配次数**：为了避免修改（增加/减少）字符串时，每次都需要重新分配内存（C 语言的字符串是这样的），**SDS 实现了空间预分配和惰性空间释放两种优化策略**。当 SDS 需要增加字符串时，Redis 会为 SDS 分配好内存，并且根据特定的算法分配多余的内存，这样可以减少连续执行字符串增长操作所需的内存重分配次数。当 SDS 需要减少字符串时，这部分内存不会立即被回收，会被记录下来，等待后续使用（支持手动释放，有对应的 API）。
  4. **二进制安全**：**C 语言中的字符串以空字符 `\0` 作为字符串结束的标识**，这存在一些问题，像一些二进制文件（比如图片、视频、音频）就可能包括空字符，C 字符串无法正确保存。SDS 使用 len 属性判断字符串是否结束，不存在这个问题。

### 2.sorted set

数据结构：ZipList（键值对数量少于 128 个、且每个元素的长度小于 64 字节）、SkipList（当超过ziplist的阈值时转换为skiplist）

* 常见应用

  * 排行榜：Redis 中有一个叫做 `Sorted Set` （有序集合）的数据类型经常被用在各种排行榜的场景，比如直播间送礼物的排行榜、朋友圈的微信步数排行榜、王者荣耀中的段位排行榜、话题热度排行榜等等。
  
  | 命令                                           | 介绍                                                         |
  | ---------------------------------------------- | ------------------------------------------------------------ |
  | ZADD **key score1 member1 score2** member2 ... | 向指定有序集合添加一个或多个元素（**需要设置分数**）         |
  | ZCARD KEY                                      | 获取指定有序集合的元素数量                                   |
  | ZSCORE key member                              | 获取指定有序集合中指定元素的 score 值                        |
  | ZINTERSTORE destination numkeys key1 key2 ...  | 将给定所有有序集合的**交集**存储在 destination 中，对相同元素对应的 score 值进行 SUM 聚合操作，numkeys 为集合数量 |
  | ZUNIONSTORE destination numkeys key1 key2 ...  | 求**并集**，其它和 ZINTERSTORE 类似                          |
  | ZDIFFSTORE destination numkeys key1 key2 ...   | 求**差集**，其它和 ZINTERSTORE 类似                          |
  | ZRANGE key start end                           | **获取指定有序集合 start 和 end 之间的元素（score 从低到高），start和end指定的是score** |
  | ZREVRANGE key start end                        | **获取指定有序集合 start 和 end 之间的元素（score 从高到底）** |
  | ZREVRANK key member                            | **获取指定有序集合中指定元素的排名(score 从大到小排序)**，排名就是元素在这个集合中的排名顺序（从0开始） |



### 3.跳表分析

对于sorted set来说在元素小于 64 字节且个数小于 128 的时候，会使用 ziplist，而这个阈值的默认值的设置就来自下面这两个配置项

```sh
zset-max-ziplist-value 64
zset-max-ziplist-entries 128
```

一旦有序集合中的某个元素超出这两个其中的一个阈值它就会转为 **skiplist**（实际是 dict+skiplist，还会借用字典来提高获取指定元素的效率）



* 跳表和一些常见的数据结构比较

  * 平衡树 vs 跳表：平衡树的插入、删除和查询的时间复杂度和跳表一样都是 **O(log n)**。对于范围查询来说，**平衡树也可以通过中序遍历的方式达到和跳表一样的效果**。但是它的每一次插入或者删除操作都需要保证整颗树左右节点的绝对平衡，只要不平衡就要通过旋转操作来保持平衡，这个过程是比较耗时的。跳表诞生的初衷就是为了克服平衡树的一些缺点。跳表使用概率平衡而不是严格强制的平衡，因此，跳表中的插入和删除算法比平衡树的等效算法简单得多，速度也快得多。
  * 红黑树 vs 跳表：相比较于红黑树来说，跳表的实现也更简单一些，不需要通过旋转和染色（红黑变换）来保证黑平衡。并且，按照区间来查找数据这个操作，红黑树的效率没有跳表高，红黑树的插入、删除和查询的时间复杂度和跳表一样都是 **O(log n)**。
  * B+树 vs 跳表：B+树更适合作为数据库和文件系统中常用的索引结构之一，它的核心思想是通过可能少的 IO 定位到尽可能多的索引来获得查询数据。对于 Redis 这种内存数据库来说，它对这些并不感冒，因为 Redis 作为内存数据库它不可能存储大量的数据，所以对于索引不需要通过 B+树这种方式进行维护，只需按照概率进行随机维护即可，节约内存。而且使用跳表实现 zset 时相较前者来说更简单一些，在进行插入时只需通过索引将数据插入到链表中合适的位置再随机维护一定高度的索引即可，也不需要像 B+树那样插入时发现失衡时还需要对节点分裂与合并（**skiplist更节省内存**）

* 跳表结构分析

  https://javaguide.cn/database/redis/redis-skiplist.html

### 4.set

数据结构：Dict、Intset

* 常见应用

  * 存放的数据**不能重复的场景**：网站 UV 统计（数据量巨大的场景还是 `HyperLogLog`更适合一些）、文章点赞、动态点赞等等。

  * 需要获取多个数据源**交集、并集和差集**的场景：共同好友(交集)、共同粉丝(交集)、共同关注(交集)、好友推荐（差集）、音乐推荐（差集）、订阅号推荐（差集+交集） 等等。

    > 并集：SINTER key1 key2 ...
    >
    > 交集：SUNION key1 key2 ...
    >
    > 差集：SDIFF key1 key2 ...

  * 需要**随机获取数据源中的元素**的场景：抽奖系统、随机点名等等

    > SADD key member1 member2 ...：向指定集合添加一个或多个元素。
    > SPOP key count：随机移除并获取指定集合中一个或多个元素，适合不允许重复中奖的场景。
    > SRANDMEMBER key count : 随机获取指定集合中指定数量的元素，适合允许重复中奖的场景。

  * 命令

    | 命令                                  | 介绍                                           |
    | ------------------------------------- | ---------------------------------------------- |
    | SADD key member1 member2 ...          | 向指定集合**添加**一个或多个元素               |
    | SMEMBERS key                          | **获取**指定集合中的**所有**元素               |
    | SCARD key                             | **获取**指定集合的元素数量                     |
    | SISMEMBER key member                  | 判断指定元素是否**存在**指定集合中             |
    | SINTER key1 key2 ...                  | 获取给定所有集合的**交集**                     |
    | SINTERSTORE destination key1 key2 ... | 将给定所有集合的**交集存储在 destination** 中  |
    | SUNION key1 key2 ...                  | 获取给定所有集合的**并集**                     |
    | SUNIONSTORE destination key1 key2 ... | 将给定所有集合的**并集存储在 destination** 中  |
    | SDIFF key1 key2 ...                   | 获取给定所有集合的**差集**                     |
    | SDIFFSTORE destination key1 key2 ...  | 将给定所有集合的**差集存储在 destination** 中  |
    | SPOP key count                        | **随机移除**并获取指定集合中**一个或多个元素** |
    | SRANDMEMBER key count                 | **随机获取**指定集合中**指定数量**的元素       |

### 5.bitmap

Bitmap 存储的是连续的二进制数字（0 和 1），通过 Bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 Bitmap 本身会极大的节省储存空间，可以将 Bitmap 看作是一个存储二进制数字（0 和 1）的数组，数组中每个元素的下标叫做 offset（偏移量），**offset从0开始。Bitmap 不是 Redis 中的实际数据类型，而是在 String 类型上定义的一组面向位的操作，将其视为位向量。由于字符串是二进制安全的块，且最大长度为 512 MB，它们适合用于设置最多 2^32 (42亿左右)个不同的位**（2^32/2^3=2^29B=512MB）

------

著作权归JavaGuide(javaguide.cn)所有 基于MIT协议 原文链接：https://javaguide.cn/database/redis/redis-data-structures-02.html

| 命令                                  | 介绍                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| SETBIT key offset value               | 设置指定 offset 位置的值                                     |
| GETBIT key offset                     | 获取指定 offset 位置的值                                     |
| BITCOUNT key start end                | **获取 start 和 end 之前值为 1 的元素个数**                  |
| BITOP operation destkey key1 key2 ... | **对一个或多个 Bitmap 进行运算，可用运算符有 AND, OR, XOR 以及 NOT** |

* 常见应用

  * 活跃用户统计

    如果想要使用 Bitmap 统计活跃用户的话，可以使用日期（精确到天）作为 key，然后用户 ID 为 offset，如果当日活跃过就设置为 1。

    ```bash
    # 初始化
    SETBIT 20210308 1 1
    (integer) 0
    SETBIT 20210308 2 1
    (integer) 0
    SETBIT 20210309 1 1
    (integer) 0
    # 统计20210308~20210309 总活跃用户数
    BITOP and desk1 20210308 20210309
    (integer) 1
    BITCOUNT desk1
    (integer) 1
    
    # SETBIT 会返回之前位的值（默认是 0）这里会生成 7 个位
    > SETBIT mykey 7 1
    (integer) 0
    > SETBIT mykey 7 0
    (integer) 1
    > GETBIT mykey 7
    (integer) 0
    > SETBIT mykey 6 1
    (integer) 0
    > SETBIT mykey 8 1
    (integer) 0
    # 通过 bitcount 统计被被设置为 1 的位的数量。
    > BITCOUNT mykey
    (integer) 2
    ```

 ### 6.List

List的数据结构为LinkedList（双向链表）/ZipList（压缩表）/QuickList，Redis 3.2 之前，List 底层实现是 LinkedList 或者 ZipList。 Redis 3.2 之后，引入了 LinkedList 和 ZipList 的结合 QuickList，List 的底层实现变为 QuickList。从 Redis 7.0 开始， ZipList 被 ListPack 取代。

| 命令                        | 介绍                                       |
| --------------------------- | ------------------------------------------ |
| RPUSH key value1 value2 ... | 在指定列表的尾部（右边）添加一个或多个元素 |
| LPUSH key value1 value2 ... | 在指定列表的头部（左边）添加一个或多个元素 |
| LSET key index value        | 将指定列表索引 index 位置的值设置为 value  |
| LPOP key                    | 移除并获取指定列表的第一个元素(最左边)     |
| RPOP key                    | 移除并获取指定列表的最后一个元素(最右边)   |
| LLEN key                    | 获取列表元素数量                           |
| LRANGE key start end        | 获取列表 start 和 end 之间 的元素          |

* 常见应用

  * 队列

    ```sh
    RPUSH myList value1
    (integer) 1
    RPUSH myList value2 value3
    (integer) 3
    LPOP myList
    "value1"
    LRANGE myList 0 1
    1) "value2"
    2) "value3"
    LRANGE myList 0 -1
    1) "value2"
    2) "value3"
    ```

  * 栈

    ```sh
    RPUSH myList2 value1 value2 value3
    (integer) 3
    RPOP myList2 # 将 list的最右边的元素取出
    "value3"
    ```

### 7.hash

数据结果：Dict、Intset

Redis 中的 Hash 是一个 String 类型的 field-value（键值对） 的映射表，特别适合用于存储对象，后续操作的时候，你可以直接修改这个对象中的某些字段的值。Hash 类似于 JDK1.8 前的 `HashMap`，内部实现也差不多(数组 + 链表)。不过，Redis 的 Hash 做了更多优化。

| 命令                                      | 介绍                                                     |
| ----------------------------------------- | -------------------------------------------------------- |
| HSET key field value                      | 设置指定哈希表中指定字段的值                             |
| **HSETNX key field value**                | 只有指定字段不存在时设置指定字段的值                     |
| HMSET key field1 value1 field2 value2 ... | 同时将一个或多个 field-value (域-值)对设置到指定哈希表中 |
| HGET key field                            | 获取指定哈希表中指定字段的值                             |
| HMGET key field1 field2 ...               | 获取指定哈希表中一个或者多个指定字段的值                 |
| HGETALL key                               | 获取指定哈希表中所有的键值对                             |
| **HEXISTS key field**                     | 查看指定哈希表中指定的字段是否存在                       |
| HDEL key field1 field2 ...                | 删除一个或多个哈希表字段                                 |
| HLEN key                                  | 获取指定哈希表中字段的数量                               |
| **HINCRBY key field increment**           | 对指定哈希中的指定字段做运算操作（正数为加，负数为减）   |

* 常见应用
  - 举例：用户信息、商品信息、文章信息、购物车信息。
  - 相关命令：`HSET` （设置单个字段的值）、`HMSET`（设置多个字段的值）、`HGET`（获取单个字段的值）、`HMGET`（获取多个字段的值）

### 8.Hyperlog

https://juejin.cn/post/6844903785744056333

`HyperLogLog`，下面简称为`HLL`，它是 `LogLog` 算法的升级版，作用是能够提供**不精确的去重计数**。存在以下的特点：

- 代码实现较难。
- 能够使用极少的内存来统计巨量的数据，在 `Redis` 中实现的 `HyperLogLog`，**只需要`12K`内存就能统计`2^64`个数据**。
- 计数存在**一定的误差，误差率整体较低。标准误差为 0.81%** 。
- 误差可以被设置`辅助计算因子`进行降低。

| 命令                                      | 介绍                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| PFADD key element1 element2 ...           | 添加一个或多个元素到 HyperLogLog 中                          |
| PFCOUNT key1 key2                         | 获取一个或者多个 HyperLogLog 的唯一计数。                    |
| PFMERGE destkey sourcekey1 sourcekey2 ... | 将多个 HyperLogLog 合并到 destkey 中，destkey 会结合多个源，算出对应的唯一计数。 |

* 常见应用场景

  * 超大数量计数非严格精确计数:热门网站每日/每周/每月访问 ip 数统计、热门帖子 uv 统计

  ```sh
  PFADD hll foo bar zap
  (integer) 1
  PFADD hll zap zap zap
  (integer) 0
  PFADD hll foo bar
  (integer) 0
  PFCOUNT hll
  (integer) 3
  PFADD some-other-hll 1 2 3
  (integer) 1
  PFCOUNT hll some-other-hll
  (integer) 6
  PFMERGE desthll hll some-other-hll
  "OK"
  PFCOUNT desthll
  (integer) 6
  ```

  


# Redis 为什么这么快？

Redis 内部做了非常多的性能优化，比较重要的有下面 3 点：

1. Redis **基于内存**，内存的访问速度是磁盘的上千倍；
2. Redis 基于 Reactor 模式设计开发了一套高效的事件处理模型，主要是**单线程事件循环和 IO 多路复用**（Redis 线程模式后面会详细介绍到）；
3. Redis 内置了**多种优化过后的数据类型/**结构实现，性能非常高。

> ![why-redis-so-fast](https://javaguide.cn/assets/why-redis-so-fast-E21l9uI2.png)

# 说一下 Redis 和 Memcached 的区别和共同点

现在公司一般都是用 Redis 来实现缓存，而且 Redis 自身也越来越强大了！不过，了解 Redis 和 Memcached 的区别和共同点，有助于我们在做相应的技术选型的时候，能够做到有理有据！

**共同点**：

1. 都是基于内存的数据库，一般都用来当做缓存使用。
2. 都有过期策略。
3. 两者的性能都非常高。

**区别**：

1. **Redis 支持更丰富的数据类型（支持更复杂的应用场景）**。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。
2. **Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而 Memcached 把数据全部存在内存之中。**
3. **Redis 有灾难恢复机制。** 因为可以把缓存中的数据持久化到磁盘上。
4. **Redis 在服务器内存使用完之后，可以将不用的数据放到磁盘上。但是，Memcached 在服务器内存使用完之后，就会直接报异常。**
5. **Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的。**
6. **Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型。** （Redis 6.0 针对网络数据的读写引入了多线程）
7. **Redis 支持发布订阅模型、Lua 脚本、事务等功能，而 Memcached 不支持。并且，Redis 支持更多的编程语言。**
8. **Memcached 过期数据的删除策略只用了惰性删除，而 Redis 同时使用了惰性删除与定期删除。**

# 缓存读写策略有哪些？

#### Cache Aside Pattern (旁路缓存模式)

是我们平时使用比较多的一个缓存读写模式**，比较适合读请求比较多的场景。**

Cache Aside Pattern 中服务端需要同时维系 db 和 cache，并且是以 db 的结果为准。

下面我们来看一下这个策略模式下的缓存读写步骤。

**写**：

- 先更新 db
- 然后直接删除 cache 。

**读** :

- 从 cache 中读取数据，读取到就直接返回
- cache 中读取不到的话，就从 db 中读取数据返回
- 再把数据放到 cache 中。

**额外**：

**在写数据的过程中，可以先删除 cache ，后更新 db 么？**

答案：不建议 

* 数据不一致性问题：因为这样可能会造成 **数据库（db）和缓存（Cache）数据不一致**的问题。不过该方案本来就有一致性问题（**请求 1 先读数据 A，请求 2 随后写数据 A，并且数据 A 在请求 1 请求之前不在缓存中的话，也有可能产生数据不一致性的问题**），只不过改成先删除后写导致不一致的概率大很多
* 资源浪费问题：更新cache需要成本，如果该缓存不会被频繁使用会导致资源浪费

**缺陷：**

* 缺陷 1：首次请求数据一定不在 cache 的问题

解决办法：可以将热点数据可以提前放入 cache 中。

* 缺陷 2：写操作比较频繁的话导致 cache 中的数据会被频繁被删除，这样会影响缓存命中率 。

解决办法：

- 数据库和缓存数据强一致场景：更新 db 的时候同时更新 cache，不过我们需要加一个锁/分布式锁来保证更新 cache 的时候不存在线程安全问题。
- 可以短暂地允许数据库和缓存数据不一致的场景：更新 db 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小

#### Read/Write Through Pattern（读写穿透)

服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。cache 服务负责将此数据读取和写入 db，从而减轻了应用程序的职责。

**写（Write Through）：**

- 先查 cache，cache 中不存在，直接更新 db。
- cache 中存在，则先更新 cache，然后 cache 服务自己更新 db（**同步更新 cache 和 db)**

**读(Read Through)：**

- 从 cache 中读取数据，读取到就直接返回 。
- 读取不到的话，先从 db 加载，写入到 cache 后返回响应。



#### Write Behind Pattern（异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，**两者都是由 cache 服务来负责 cache 和 db 的读写**

很明显，这种方式对数据一致性带来了更大的挑战，比如 cache 数据可能还没异步更新 db 的话，cache 服务可能就就挂掉了。

这种策略在我们平时开发过程中也非常非常少见，但是不代表它的应用场景少，比如消息队列中消息的异步写入磁盘、MySQL 的 Innodb Buffer Pool 机制都用到了这种策略。

**Write Behind Pattern 下 db 的写性能非常高，非常适合一些数据经常变化又对数据一致性要求没那么高的场景，比如浏览量、点赞**

# Redis的应用

### 1.分布式锁

通过 Redis 来做分布式锁是一种比较常见的方式。通常情况下，我们都是**基于 Redisson 来实现分布式锁**。关于 Redis 实现分布式锁的详细介绍，可以看我写的这篇文章，原理是使用setnx来实现

### 2.限流

一般是通过 Redis + Lua 脚本的方式来实现限流（采用计时器+过期时间的策略）。

实现限流方案：https://mp.weixin.qq.com/s/kyFAWH3mVNJvurQDt4vchA

### 3.消息队列

原理是使用List数据结构，但是不建议使用

#### 3.1 Redis2.0之前队列

**Redis 2.0 之前，如果想要使用 Redis 来做消息队列的话，只能通过 List 来实现。**

通过 `RPUSH/LPOP` 或者 `LPUSH/RPOP`即可实现简易版消息队列：

```bash
# 生产者生产消息
RPUSH myList msg1 msg2
(integer) 2
RPUSH myList msg3
(integer) 3
# 消费者消费消息
LPOP myList
"msg1"
```

不过**，通过 `RPUSH/LPOP` 或者 `LPUSH/RPOP`这样的方式存在性能问题，我们需要不断轮询去调用 `RPOP` 或 `LPOP` 来消费消息。当 List 为空时，大部分的轮询的请求都是无效请求，这种方式大量浪费了系统资源**。

因此，Redis 还提供了 **`BLPOP`、`BRPOP`** 这种阻塞式读取的命令（带 B-Blocking 的都是阻塞式），并且还支持一个超时参数。如果 List 为空，Redis 服务端不会立刻返回结果，它会等待 List 中有新数据后在返回或者是等待最多一个超时时间后返回空。如果将超时时间设置为 0 时，即可无限等待，直到弹出消息

```bash
# 超时时间为 10s
# 如果有数据立刻返回，否则最多等待10秒
BRPOP myList 10
null
```

**List 实现消息队列功能太简单，像消息确认机制等功能还需要我们自己实现，最要命的是没有广播机制，消息也只能被消费一次。**

#### 3.2 Redis2.0及之后的pub/sub功能

**Redis 2.0 引入了发布订阅 (pub/sub) 功能，解决了 List 实现消息队列没有广播机制的问题。**

![Redis 发布订阅 (pub/sub) 功能](https://oss.javaguide.cn/github/javaguide/database/redis/redis-pub-sub.png)Redis 发布订阅 (pub/sub) 功能

pub/sub 中引入了一个概念叫 **channel（频道）**，发布订阅机制的实现就是基于这个 channel 来做的。

pub/sub 涉及发布者（Publisher）和订阅者（Subscriber，也叫消费者）两个角色：

- 发布者通过 `PUBLISH` 投递消息给指定 channel。
- 订阅者通过`SUBSCRIBE`订阅它关心的 channel。并且，订阅者可以订阅一个或者多个 channel。

pub/sub 既能单播又能广播，还支持 channel 的简单正则匹配。不过，消息丢失（客户端断开连接或者 Redis 宕机都会导致消息丢失）、消息堆积（发布者发布消息的时候不会管消费者的具体消费能力如何）等问题依然没有一个比较好的解决办法。

#### 3.4 Redis5.0后stream功能

- 发布 / 订阅模式
- 按照消费者组进行消费（借鉴了 Kafka 消费者组的概念）
- 消息持久化（ RDB 和 AOF）
- ACK 机制（通过确认机制来告知已经成功处理了消息）
- 阻塞式获取消息

### 4.延时队列

Redisson 内置了延时队列（基于 Sorted Set 实现的）

### 5.分布式session

利用 String 或者 Hash 数据类型保存 Session 数据，所有的服务器都可以访问

### 6.搜索引擎

Redis 是可以实现全文搜索引擎功能的，需要借助 **RediSearch** ，这是一个基于 Redis 的搜索引擎模块。

RediSearch 支持中文分词、聚合统计、停用词、同义词、拼写检查、标签查询、向量相似度查询、多关键词搜索、分页搜索等功能，算是一个功能比较完善的全文搜索引擎了。

小规模搜索引擎可以使用，但是大规模的不建议使用，大规模建议使用es，分析见https://javaguide.cn/database/redis/redis-questions-01.html#redis-%E5%8F%AF%E4%BB%A5%E5%81%9A%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E4%B9%88

### 7.排行榜

使用sorted set实现



# Redis线程模型

**对于读写命令来说，Redis 一直是单线程模型**。不过，在 **Redis 4.0** 版本之后引入了**多线程来执行一些大键值对的异步删除操作**， **Redis 6.0** 版本之后引入了**多线程来处理网络请求**（提高网络 IO 读写性能）

**Redis 基于 Reactor 模式设计开发了一套高效的事件处理模型** （Netty 的线程模型也基于 Reactor 模式，Reactor 模式不愧是高性能 IO 的基石），这套事件处理模型对应的是 Redis 中的文件事件处理器（file event handler即redis中的reactor模型）。由于文件事件处理器（file event handler）是单线程方式运行的，所以我们一般都说 Redis 是单线程模型。

**核心任务选择单线程处理任务的原因：**

- **单线程编程容易并且更容易维护；**
- **Redis 的性能瓶颈不在 CPU ，主要在内存和网络；**
- **多线程就会存在死锁、线程上下文切换等问题，甚至会影响性能。**

**网络引入多线程的原因：**

**Redis6.0 引入多线程主要是为了提高网络 IO 读写性能**，因为这个算是 Redis 中的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络）。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了，执行命令仍然是单线程顺序执行。因此，你也不需要担心线程安全问题。

**总结**：命令执行（读写）一直是单线程，开发简单也保证了线程安全，但是网络模型是reactor模型，多线程接受任务然后投递给消息队列给等待核心线程取执行。

### 1.redis的reactor模型

Redis 基于 Reactor 模式开发了自己的网络事件处理器：这个处理器被称为文件事件处理器**（file event handler）**。

- 文件事件处理器使用 **I/O 多路复用（multiplexing）程序来同时监听多个套接字**，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接应答**（accept）、读取（read）、写入（write）、关 闭（close）**等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件

文件事件处理器（file event handler）主要是包含 4 个部分：

- 多个 socket（客户端连接）
- IO 多路复用程序（支持多个客户端连接的关键）
- 文件事件分派器（将 socket 关联到相应的事件处理器）
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

![文件事件处理器（file event handler）](https://oss.javaguide.cn/github/javaguide/database/redis/redis-event-handler.png)

### 2.redis后台线程

https://juejin.cn/post/7102780434739626014

我们虽然经常说 Redis 是单线程模型（主要逻辑是单线程完成的），但实际还有一些后台线程用于执行一些比较耗时的操作：

- 通过 `bio_close_file` 后台线程来释放 AOF / RDB 等过程中产生的临时文件资源。
- 通过 `bio_aof_fsync` 后台线程调用 `fsync` 函数将系统内核缓冲区还未同步到到磁盘的数据强制刷到磁盘（ AOF 文件）。
- 通过 `bio_lazy_free`后台线程释放大对象（已删除）占用的内存空间.

# Redis内存管理

### 1.设置过期时间有什么用

* **过期时间除了有助于缓解内存的消耗**
* 业务上有有效期概念，可以用过期时间实现

### 2.过期数据删除策略

* **惰性删除**：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。

* **定期删除**：每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

Redis 采用的是 **定期删除+惰性/懒汉式删除**的**内存淘汰机制**

1. **volatile-lru（least recently used）**：**从已设置过期时间**的数据集（`server.db[i].expires`）中挑选最近最少使用的数据淘汰。
2. **volatile-ttl**：**从已设置过期时间**的数据集（`server.db[i].expires`）中挑选将要过期的数据淘汰。
3. **volatile-random**：**从已设置过期时间**的数据集（`server.db[i].expires`）中任意选择数据淘汰。
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。
5. **allkeys-random**：从数据集（`server.db[i].dict`）中任意选择数据淘汰。
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

**4.0 版本后增加以下两种**：（注意这是lfu和lru有区别）

1. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选最不经常使用的数据淘汰。
2. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key



# Redis事务

**Redis 事务提供了一种将多个命令请求打包的功能。然后，再按顺序执行打包的所有命令，并且不会被中途打断**

### 1.redis事务指令

Redis 可以通过 **`MULTI`，`EXEC`，`DISCARD` 和 `WATCH`** 等命令来实现事务(Transaction)功能

MULTI命令后可以输入多个命令，Redis 不会立即执行这些命令，而是将它们放到队列，当调用了 exec 命令后，再执行所有的命令

```sh
MULTI
OK
SET PROJECT "JavaGuide"
QUEUED
GET PROJECT
QUEUED
EXEC
1) OK
2) "JavaGuide"
```

discard可以取消存入队列的指令

```sh
MULTI
OK
SET PROJECT "JavaGuide"
QUEUED
GET PROJECT
QUEUED
DISCARD
OK
```

通过watch命令监听指定的 Key，当调用 `EXEC` 命令执行事务时，如果一个被 `WATCH` 命令监视的 Key 被 **其他客户端/Session** 修改的话，整个事务都不会被执行。不过，如果 **WATCH** 与 **事务** 在同一个 Session 里，并且被 **WATCH** 监视的 Key 被修改的操作发生在事务内部，这个事务是可以被执行成功的

```sh
# 客户端 1
SET PROJECT "RustGuide"
OK
WATCH PROJECT
OK
MULTI
OK
SET PROJECT "JavaGuide"
QUEUED

# 客户端 2
# 在客户端 1 执行 EXEC 命令提交事务之前修改 PROJECT 的值
SET PROJECT "GoGuide"

# 客户端 1
# 修改失败，因为 PROJECT 的值被客户端2修改了
EXEC
(nil)
GET PROJECT
"GoGuide"

```

### 2.redis事务对acid的支持

* **不满足原子性和隔离性**

Redis 事务在运行错误的情况下，除了执行过程中出现错误的命令外，其他命令都能正常执行。并且，Redis 事务是不支持回滚（roll back）操作的。因此，Redis 事务其实是不满足原子性的

* **持久性（可以通过设置来保证，但是通常下为了保证性能不满足）**

  Redis 不同于 Memcached 的很重要一点就是，Redis 支持持久化，而且支持 3 种持久化方式:

  - 快照（snapshotting，RDB）
  - 只追加文件（append-only file, AOF）
  - RDB 和 AOF 的混合持久化(Redis 4.0 新增)

  与 RDB 持久化相比，AOF 持久化的实时性更好。在 Redis 的配置文件中存在三种不同的 AOF 持久化方式（ `fsync`策略），它们分别是：

  ```sh
  appendfsync always    #每次有数据修改发生时都会调用fsync函数同步AOF文件,fsync完成后线程返回,这样会严重降低Redis的速度
  appendfsync everysec  #每秒钟调用fsync函数同步一次AOF文件
  appendfsync no        #让操作系统决定何时进行同步，一般为30秒一次
  ```

  **AOF 持久化的`fsync`策略为 no、everysec 时都会存在数据丢失的情况 。always 下可以基本是可以满足持久性要求的，但性能太差，实际开发过程中不会使用**。因此，Redis 事务的持久性也是没办法保证的


# Redis持久化

### 1.RDB

Redis 可以通过创建快照来获得存储在内存里面的数据在 **某个时间点** 上的副本。Redis 创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构，主要用来提高 Re               dis 性能），还可以将快照留在原地以便重启服务器的时候使用（**主从复制和服务重启时快速恢复使用**

**RDB配置：**

```sh
save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发bgsave命令创建快照。

save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发bgsave命令创建快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发bgsave命令创建快照。
```

**阻塞问题：**

Redis 提供了两个命令来生成 RDB 快照文件：

- `save` : 同步保存操作，会阻塞 Redis 主线程；

- `bgsave` : fork 出一个子进程，子进程执行，不会阻塞 Redis 主线程，默认选项。

  注：

  > 主进程创建子进程，会调用操作系统提供的 fork 函数。
  >
  > 而 fork 在执行过程中，**主进程需要拷贝自己的内存页表给子进程**，如果这个实例很大，那么这个拷贝的过程也会比较耗时。
  >
  > 而且这个 fork 过程会消耗大量的 CPU 资源，在完成 fork 之前，整个 Redis 实例会被阻塞住，无法处理任何客户端请求。
  >
  > 可以在 Redis 上执行 INFO 命令，查看 latest_fork_usec (上一次fork耗时)，单位微秒

### 2.AOF

与快照持久化相比，AOF 持久化的实时性更好。默认情况下 Redis 没有开启 AOF（append only file）方式的持久化（Redis 6.0 之后已经默认是开启了），可以通过 `appendonly` 参数开启：

```sh
appendonly yes
```

开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入到 AOF 缓冲区 `server.aof_buf` 中，然后再写入到 AOF 文件中（此时还在系统内核缓存区未同步到磁盘），最后再根据持久化方式（ `fsync`策略）的配置来决定何时将系统内核缓存区的数据同步到硬盘中的。

只有同步到磁盘中才算持久化保存了，否则依然存在数据丢失的风险，比如说：系统内核缓存区的数据还未同步，磁盘机器就宕机了，那这部分数据就算丢失了。

* **AOF 持久化功能**：

1. **命令追加（append）**：所有的写命令会追加到 AOF 缓冲区中。

2. **文件写入（write）**：将 AOF 缓冲区的数据写入到 AOF 文件中。这一步需要调用`write`函数（系统调用），`write`将数据写入到了系统内核缓冲区之后直接返回了（延迟写）。注意！！！此时并没有同步到磁盘。

   > `write`：写入系统内核缓冲区之后直接返回（仅仅是写到缓冲区），不会立即同步到硬盘。虽然提高了效率，但也带来了数据丢失的风险。同步硬盘操作通常依赖于系统调度机制，Linux 内核通常为 30s 同步一次，具体值取决于写出的数据量和 I/O 缓冲区的状态。

3. **文件同步（fsync）**：AOF 缓冲区根据对应的持久化方式（ `fsync` 策略）向硬盘做同步操作。这一步需要调用 `fsync` 函数（系统调用）， `fsync` 针对单个文件操作，对其进行强制硬盘同步，`fsync` 将阻塞直到写入磁盘完成后返回，保证了数据持久化。

   > `fsync`：`fsync`用于强制刷新系统内核缓冲区（同步到到磁盘），确保写磁盘操作结束才会返回

4. **文件重写（rewrite）**：随着 AOF 文件越来越大，需要定期对 AOF 文件进行重写，达到压缩的目的。

5. **重启加载（load）**：当 Redis 重启时，可以加载 AOF 文件进行数据恢复。

* **AOF配置：**

  > * appendfsync always：主线程调用 write 执行写操作后，后台线程（ aof_fsync 线程）立即会调用 fsync 函数同步 AOF 文件（刷盘），fsync 完成后线程返回，这样会严重降低 Redis 的性能（write + fsync）。
  > * appendfsync everysec：主线程调用 write 执行写操作后立即返回，由后台线程（ aof_fsync 线程）每秒钟调用 fsync 函数（系统调用）同步一次 AOF 文件（write+fsync，fsync间隔为 1 秒）
  > * appendfsync no：主线程调用 write 执行写操作后立即返回，让操作系统决定何时进行同步，Linux 下一般为 30 秒一次（write但不fsync，fsync 的时机由操作系统决定）

* **AOF的执行时间**

   Redis AOF 持久化机制是在执行完命令之后再记录日志

  * 原因：
    * 避免额外的检查开销，AOF 记录日志不会对命令进行语法检查
    * 在命令执行完之后再记录，不会阻塞当前的命令执行
  * 问题
    * 如果刚执行完命令 Redis 就宕机会导致对应的修改丢失
    * 可能会阻塞后续其他命令的执行（AOF 记录日志是在 Redis 主线程中进行的）

* **AOF重写**

  当 AOF 变得太大时，Redis 能够在后台自动重写 AOF 产生一个新的 AOF 文件，这个新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小

  相关配置：

  > `auto-aof-rewrite-min-size`：如果 AOF 文件大小小于该值，则不会触发 AOF 重写。默认值为 64 MB;
  >
  > `auto-aof-rewrite-percentage`：执行 AOF 重写时，当前 AOF 大小（aof_current_size）和上一次重写时 AOF 大小（aof_base_size）的比值。如果当前 AOF 文件大小增加了这个百分比值，将触发 AOF 重写。将此值设置为 0 将禁用自动 AOF 重写。默认值为 100

* **AOF校验**

  AOF 校验机制是 Redis 在启动时对 AOF 文件进行检查，以**判断文件是否完整，是否有损坏或者丢失的数据**。这个机制的原理其实非常简单，就是通过使用一种叫做 **校验和（checksum）** 的数字来验证 AOF 文件。

### 3.RDB与AOF混合

由于 RDB 和 AOF 各有优势，于是，Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 `aof-use-rdb-preamble` 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的， AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差

### 4.AOF与RDB比较

**RDB 比 AOF 优秀的地方**：

- 体积小：RDB 文件存储的内容是经过压缩的二进制数据， 保存着某个时间点的数据集，文件很小，适合做数据的备份，灾难恢复。AOF 文件存储的是每一次写命令，类似于 MySQL 的 binlog 日志，通常会比 RDB 文件大很多。当 AOF 变得太大时，Redis 能够在后台自动重写 AOF。新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小。不过， Redis 7.0 版本之前，如果在重写期间有写入命令，AOF 可能会使用大量内存，重写期间到达的所有写入命令都会写入磁盘两次。
- 恢复速度快：使用 RDB 文件恢复数据，直接解析还原数据即可，不需要一条一条地执行命令，速度非常快。而 AOF 则需要依次执行每个写命令，速度非常慢。也就是说，与 AOF 相比，恢复大数据集的时候，RDB 速度更快。

**AOF 比 RDB 优秀的地方**：

- 安全性更高（顺序IO每条都存）：RDB 的数据安全性不如 AOF，没有办法实时或者秒级持久化数据。生成 RDB 文件的过程是比较繁重的， 虽然 BGSAVE 子进程写入 RDB 文件的工作不会阻塞主线程，但会对机器的 CPU 资源和内存资源产生影响，严重的情况下甚至会直接把 Redis 服务干宕机。AOF 支持秒级数据丢失（取决 fsync 策略，如果是 everysec，最多丢失 1 秒的数据），仅仅是追加命令到 AOF 文件，操作轻量。
- 兼容性更好：RDB 文件是以特定的二进制格式保存的，并且在 Redis 版本演进中有多个版本的 RDB，所以存在老版本的 Redis 服务不兼容新版本的 RDB 格式的问题。
- 易解析：AOF 以一种易于理解和解析的格式包含所有操作的日志。你可以轻松地导出 AOF 文件进行分析，你也可以直接操作 AOF 文件来解决一些问题。比如，如果执行`FLUSHALL`命令意外地刷新了所有内容后，只要 AOF 文件没有被重写，删除最新命令并重启即可恢复之前的状态。

# Redis集群

### 1.Redis主从

- 作用

  * 实现读写分离，提高并发量

  * 实现故障恢复，保证服务可用性

- 实现

  ```sh
  # 在指定 redis 节点上执行 replicaof 命令就可以让其成为 master 的 slave
  # ip port 是 master 的
  replicaof ip port
  ```

- 常见问题

  * 过期数据问题

    > 主从复制下会读取到过期的数据吗?
    > 主从复制下如何避免读取到过期的数据?
    
    可能会读到过期数据如果对主库写使用的expire或者pexpire
    
    > Redis 3.2 版本之前，客户端读从库并不会判断数据是否过期，有可能返回过期数据。Redis 3.2 版本及之后，客户端读从库会先判断数据是否过期，如果过期的话，就会删除对应的数据并返回空值。
    > 采用 EXPIRE或者 PEXPIRE 设置过期时间的话，表示的是从执行这个命今开始往后TTL 时间过期。如果从节点同步执行命令因为网络等原因延迟的话，客户端就可能会读取到过期的数据可以考虑使用 EXPIREAT 和 PEXPIREAT这两个命令语义和效果和 EXPIRE 或者 PEXPIRE 似，但不是指定TTL (生存时间)的秒/毫秒，而是使用绝对的Unix 时间戳(自 1970 年1月 1日以来的秒数)。由于设置的是时间点，主从节点的时钟需要保持一致
  
  * 主从之间如何数据同步
  
    * redis2.8之前（SYNC流程）
  
      > 1.slave 向 master 发送 **SYNC** 命令请求启动复制流程
      >
      > 2.master 收到 SYNC 命今之后执行 BGSAVE命令(子线程执行，不会阻塞主线程) 生成 RDB 文件(dump.rdb)
      > 3.master 将生成的 RDB 文件发送给 slave
      > 4.slave 收到 RDB 文件之后就开始加载解析 RDB 同步更新本地数据
      >
      > 5.更新完成之后，slave 的状态相当于是 master 执行 BGSAVE 命令时的状态。master 会将 BGSAVE 命令之后接受的写命令缓存起来，因为这部分写命令 slave 还未同步(**master为每一个slave 单独开辟一块 replication buffer (复制缓存区) 来记录 RDB 文件生成后 master 收到**
      > **的所有写命令**)
      > 6.master 将自己缓存的这些写命令发送给 slave，slave 执行这些写命令同步 master 的最新状态
      > 7.slave 到这人时候已经完成全量复制，后续会通过和 master 维护的长连接来进行命今传播，同步最新的写命令。
      >
      > **注意：可以通过配置来控制replication buffer大小，如果命令缓存超过这个大小则会断开和slave的连接**
    
    存在的问题
    
    1. bgsave非常消耗资源
    
       > * 写入 RDB 的过程中，为了避免影响写操作，主线程修改的内存数据会被复制一份副本，BGSAVE 子进程把这副本数据写入 RDB 文件。如果修改内存数据的请求比较多的话，生成内存数据副本产生的内存消耗是非常大的。这个过程称为 Copy On Write (写时复制，COW) ，操作系统层面提供的机制。
       > * 大量的写时复制会产生大量的分页错误(也叫缺页中断、页缺失) ，消耗大量的 CPU 资源.
    
    2. 同步过程中slave无法提供服务，且同步失败后需要完全重新开始同步
    
       > * slave 加载 RDB 的过程中不能对外提供读服务
       > * slave 和 master 断开连接之后，slave 重新连上 master 需要重新进行全量同步
    
    * Redis2.8 PSYNC
    
      主要优化了每次**重连**都需要重新全量复制的问题
    
      https://www.yuque.com/snailclimb/mf2z3k/ks9olb19hc9wse5k
    
    * Redis4.0 PSYNC2
    
      优化了主备切换都需要重新全量复制问题、
    
  * 主从之间还存的问题
  
    手动指定新master和修改客户端对应的地址： master 宕机，需要从 slave 中手动选择一个新的 master，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预。人工干预大大增加了问题的处理时间以及出错的可能性。
  

### 2.Redis Sentinel

> 1.什么是 Sentinel? 有什么用?
> 2.Sentinel 如何检测节点是否下线? 主观下线与客观下线的区别?
> 3.Sentinel 是如何实现故障转移的?
> 4.为什么建议部署多个 sentinel 节点(哨兵集群) ?
> 5.Sentinel 如何选择出新的 master (选举机制)?
> 6.如何从 Sentinel 集群中选择出 Leader ?
> 7.Sentinel 可以防止脑裂吗?

* 主要功能

  解决Redis主从master切换(自动故障转移)，以及客户端地址改变问题

  > * 监控: 监控所有 redis 节点 (包括 sentinel 节点自身)的状态是否正常。
  > * 故障转移: 如果一个 master 出现故障，Sentinel 会帮助我们实现故障转移，自动将某一台 slave 升级为 master，确保整个 Redis 系统的可用性
  > * 通知: 通知 slave 新的 master 连接信息，让它们执行 replicaof 成为新的 master 的 slave。
  > * 配置提供:客户端连接 sentinel 请求 master 的地址，如果发生故障转移，sentinel 会通知新的master 链接信息给客户端

### 3.RedisCluster

> 1. 为什么需要 Redis Cluster? 解决了什么问题? 有什么优势?
> 2. Redis Cluster 是如何分片的?
> 3. 为什么 Redis Cluster 的哈希槽是 16384 个?
> 4. 如何确定给定 key 的应该分布到哪个哈希槽中?
> 5. Redis Cluster 支持重新分配哈希槽吗?
> 6. Redis Cluster 扩容缩容期间可以提供服务吗?
> 7. Redis Cluster 中的节点是怎么进行通信的?

* 主要功能

  * 解决单机缓存容量有限问题
  * 解决单机并发量有限的问题（读压力和写压力都缓解了）

* 架构

  为了保证高可用，Redis Cluster 至少需要 3 个 master 以及3个 slave，也就是说**每个 master 必须有 1个 slave（可以有多个但是至少要有一个）**。master 和 slave 之间做主从复制，slave 会实时同步 master 上的数据，不同于普通的 Redis 主从架构，**这里的 slave 不对外提供读服务**，主要用来保障 master 的高可用，当 master 出现故障的时候替代它。

  > 如果宕机的 master 无 slave 的话，为了保障集群的完整性，保证所有的哈希槽都指派给了可用的 master ，整个集群将不可用。这种情况下，还是想让集群保持可用的话，可以将 cluster-require-full-coverage 这个参数设置成 no,cluster-require-full-coverage表示需要 16384个 slot 都常被分配的时候 Redis Cluster 才可以对外提供服务

* 重定向方案

  * ASK 重定向:可以看做是临时重定向，**后续查询仍然发送到旧节点**
  * MOVED 重定向:可以看做是永久重定向，**后续查询发送到新节点**

# Redis性能优化

https://mp.weixin.qq.com/s/nNEuYw0NlYGhuKKKKoWfcQ

常见阻塞原因：https://javaguide.cn/database/redis/redis-common-blocking-problems-summary.html#save-%E5%88%9B%E5%BB%BA-rdb-%E5%BF%AB%E7%85%A7

### 1. 批量操作

一个 Redis 命令的执行可以简化为以下 4 步：

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

其中，第 1 步和第 4 步耗费时间之和称为 **Round Trip Time (RTT,往返时间)** ，也就是数据在网络上传输的时间。

使用批量操作可以减少网络传输次数，进而有效减小网络开销，大幅减少 RTT

* **一些批量操作命令**

  > `MGET`(获取一个或多个指定 key 的值)、`MSET`(设置一个或多个指定 key 的值)、
  >
  > `HMGET`(获取指定哈希表中一个或者多个指定字段的值)、`HMSET`(同时将一个或多个 field-value 对设置到指定哈希表中)、
  >
  > `SADD`（向指定集合添加一个或多个元素）
  >
  > ...

注：Redis 官方提供的分片集群解决方案 Redis Cluster 下，使用这些原生批量操作命令可能会存在一些小问题需要解决。就比如说 `MGET` 无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上，`MGET`可能还是需要多次网络传输，原子操作也无法保证了。不过，相较于非批量操作，还是可以节省不少网络传输次数。

* pipeline

  对于不支持批量操作的命令，我们可以利用 **pipeline（流水线)** 将一批 Redis 命令封装成一组，这些 Redis 命令会被一次性提交到 Redis 服务器，只需要一次网络传输。不过，需要注意控制一次批量操作的 **元素个数**(例如 500 以内，实际也和元素字节数有关)，避免网络传输的数据量过大。

  与`MGET`、`MSET`等原生批量操作命令一样，pipeline 同样在 Redis Cluster 上使用会存在一些小问题。原因类似，无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上。如果想要使用的话，客户端需要自己维护 key 与 slot 的关系。

  **原生批量操作命令和 pipeline 的是有区别的**，使用的时候需要注意：

  - 原生批量操作命令是原子操作，pipeline 是非原子操作。
  - pipeline 可以打包不同的命令，原生批量操作命令不可以。
  - 原生批量操作命令是 Redis 服务端支持实现的，而 pipeline 需要服务端和客户端的共同实现。

  **pipeline 和 Redis 事务的对比**：

  - 事务是原子操作，**pipeline 是非原子操作**。两个不同的事务不会同时运行，**而 pipeline 可以同时以交错方式执行**。
  - Redis 事务中每个命令都需要发送到服务端，而 Pipeline 只需要发送一次，请求次数更少。

  **pipeline与Lua区别**

  * lua是原子的
  * pipeline 不适用于执行顺序有依赖关系的一批命令


* lua脚本

  Lua 脚本同样支持批量操作多条命令。一段 Lua 脚本可以视作一条命令执行，可以看作是 **原子操作** 。也就是说，一段 Lua 脚本执行过程中不会有其他脚本或 Redis 命令同时执行，保证了操作不会被其他指令插入或打扰，这是 pipeline 所不具备的。

   Lua 脚本依然存在下面这些缺陷：

  - 如果 Lua 脚本运行时出错并中途结束，之后的操作不会进行，但是之前已经发生的写操作不会撤销，所以即使使用了 Lua 脚本，也不能实现类似数据库回滚的原子性。
  - Redis Cluster 下 Lua 脚本的原子操作也无法保证了，原因同样是无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上

### 2.大量key集中过期优化

对于过期 key，Redis 采用的是 **定期删除+惰性/懒汉式删除** 策略。定期删除执行过程中，**如果突然遇到大量过期 key 的话，客户端请求必须等待定期清理过期 key 任务线程执行完成，因为这个这个定期任务线程是在 Redis 主线程中执行的。这就导致客户端请求没办法被及时处理，响应速度会比较慢**

通常使用如下方案解决：

* 给 key 设置随机过期时间。
* **开启 lazy-free（惰性删除/延迟释放）** 。lazy-free 特性是 Redis 4.0 开始引入的，指的是让 Redis 采用异步方式延迟释放 key 使用的内存，将该操作交给单独的子线程处理，避免阻塞主线程。

### 3.大key问题

简单来说，如果一个 key 对应的 value 所占用的内存比较大，那这个 key 就可以看作是 bigkey。具体多大才算大呢？有一个不是特别精确的参考标准：

- String 类型的 value 超过 1MB
- 复合类型（List、Hash、Set、Sorted Set 等）的 value 包含的元素超过 5000 个（不过，对于复合类型的 value 来说，不一定包含的元素越多，占用的内存就越多）。

#### **1.可能会产生Key的情况：**

* 程序设计不当，比如直接使用 String 类型存储较大的文件对应的二进制数据。

* 对于业务的数据规模考虑不周到，比如使用集合类型的时候没有考虑到数据量的快速增长。（如使用List作为消息队列，但是没有及时消费）

* 未及时清理垃圾数据，比如哈希中冗余了大量的无用键值对。

#### **2.大Key会导致的问题**

* 占用大量存储空间
* 客户端超时阻塞：由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
* 网络阻塞：每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
* 工作线程阻塞：如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令

#### **3.怎么查找大key**

* **使用 Redis 自带的 `--bigkeys` 参数来查找**

  这个命令**会扫描(Scan) Redis 中的所有 key** ，会对 Redis 的性能有一点影响。并且，**这种方式只能找出每种数据结构 top 1 bigkey（占用内存最大的 String 数据类型和包含元素最多的复合数据类型）**。然而，一个 key 的元素多并不代表占用内存也多，需要我们根据具体的业务情况来进一步判断。

  注：在线上执行该命令时，为了降低对 Redis 的影响，需要指定 `-i` 参数控制扫描的频率。`redis-cli -p 6379 --bigkeys -i 3` 表示扫描过程中每次扫描后休息的时间间隔为 3 秒

  ```sh
  # redis-cli -p 6379 --bigkeys
  
  # Scanning the entire keyspace to find biggest keys as well as
  # average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
  # per 100 SCAN commands (not usually needed).
  
  [00.00%] Biggest string found so far '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' with 4437 bytes
  [00.00%] Biggest list   found so far '"my-list"' with 17 items
  
  -------- summary -------
  
  Sampled 5 keys in the keyspace!
  Total key length in bytes is 264 (avg len 52.80)
  
  Biggest   list found '"my-list"' has 17 items
  Biggest string found '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' has 4437 bytes
  
  1 lists with 17 items (20.00% of keys, avg size 17.00)
  0 hashs with 0 fields (00.00% of keys, avg size 0.00)
  4 strings with 4831 bytes (80.00% of keys, avg size 1207.75)
  0 streams with 0 entries (00.00% of keys, avg size 0.00)
  0 sets with 0 members (00.00% of keys, avg size 0.00)
  0 zsets with 0 members (00.00% of keys, avg size 0.00
  ```

* 使用scan命令

  `SCAN` 命令可以按照一定的模式和数量返回匹配的 key。获取了 key 之后，可以利用 `STRLEN`、`HLEN`、`LLEN`等命令返回其长度或成员数量。

  | 数据结构   | 命令   | 复杂度 | 结果（对应 key）   |
  | ---------- | ------ | ------ | ------------------ |
  | String     | STRLEN | O(1)   | 字符串值的长度     |
  | Hash       | HLEN   | O(1)   | 哈希表中字段的数量 |
  | List       | LLEN   | O(1)   | 列表元素数量       |
  | Set        | SCARD  | O(1)   | 集合元素数量       |
  | Sorted Set | ZCARD  | O(1)   | 有序集合的元素数量 |

  对于集合类型还可以使用 `MEMORY USAGE` 命令（Redis 4.0+），这个命令会返回键值对占用的内存空间

* 借助RDB分析工具

  通过分析 RDB 文件来找出 big key。这种方案的前提是你的 Redis 采用的是 RDB 持久化。

  网上有现成的代码/工具可以直接拿来使用：

  - [redis-rdb-toolsopen in new window](https://github.com/sripathikrishnan/redis-rdb-tools)：Python 语言写的用来分析 Redis 的 RDB 快照文件用的工具
  - [rdb_bigkeysopen in new window](https://github.com/weiyanwei412/rdb_bigkeys) : Go 语言写的用来分析 Redis 的 RDB 快照文件用的工具，性能更好

#### 4.怎么处理大key

**分割 bigkey**：将一个 bigkey 分割为多个小 key。例如，将一个含有上万字段数量的 Hash 按照一定策略（比如二次哈希）拆分为多个 Hash。

**手动清理**：Redis 4.0+ 可以使用 `UNLINK` 命令来异步删除一个或多个指定的 key。Redis 4.0 以下可以考虑使用 `SCAN` 命令结合 `DEL` 命令来分批次删除。

**采用合适的数据结构**：例如，文件二进制数据不使用 String 保存、使用 HyperLogLog 统计页面 UV、Bitmap 保存状态信息（0/1）。

**开启 lazy-free（惰性删除/延迟释放）** ：lazy-free 特性是 Redis 4.0 开始引入的，指的是让 Redis 采用异步方式延迟释放 key 使用的内存，将该操作交给单独的子线程处理，避免阻塞主线程。

### 4.热key问题

#### 1.如何发现热key

* **使用 Redis 自带的 `--hotkeys` 参数来查找**

  Redis 4.0.3 版本中新增了 `hotkeys` 参数，**该参数能够返回所有 key 的被访问次数**。使用该方案的前提条件是 Redis Server 的 `maxmemory-policy` 参数设置为 LFU 算法

  注意：hotkeys` 参数命令也会增加 Redis 实例的 CPU 和内存消耗（全局扫描），因此需要谨慎使用。

* 使用Monitor命令

  `MONITOR` 命令是 Redis 提供的一种实时查看 Redis 的所有操作的方式，可以用于临时监控 Redis 实例的操作情况，包括读写、删除等操作。

  由于该命令对 Redis 性能的影响比较大，因此禁止长时间开启 `MONITOR`（生产环境中建议谨慎使用该命令）。

* 使用开源项目

  京东零售的 [hotkey](https://gitee.com/jd-platform-opensource/hotkey) 这个项目不光支持 hotkey 的发现，还支持 hotkey 的处理

* 根据业务进行预估

* 业务代码中记录分析

  在业务代码中添加相应的逻辑对 key 的访问情况进行记录分析。不过，这种方式会让业务代码的复杂性增加，一般也不会采用

* 云服务中提供的hotkey分析

#### 2.解决热key方案

**读写分离**：主节点处理写请求，从节点处理读请求。

**使用 Redis Cluster**：将热点数据分散存储在多个 Redis 节点上。

**二级缓存**：hotkey 采用二级缓存的方式进行处理，将 hotkey 存放一份到 JVM 本地内存中（可以用 Caffeine）

### 5.redis慢查询

#### 1.哪些是慢查询命令

Redis 中的大部分命令都是 O(1)时间复杂度，但也有少部分 O(n) 时间复杂度的命令，例如：

- `KEYS *`：会返回所有符合规则的 key。
- `HGETALL`：会返回一个 Hash 中所有的键值对。
- `LRANGE`：会返回 List 中指定范围内的元素。
- `SMEMBERS`：返回 Set 中的所有元素。
- `SINTER`/`SUNION`/`SDIFF`：计算多个 Set 的交集/并集/差集。
- ……

由于这些命令时间复杂度是 O(n)，有时候也会全表扫描，随着 n 的增大，执行耗时也会越长。不过， 这些命令并不是一定不能使用，但是需要明确 N 的值。另外，有遍历的需求可以使用 `HSCAN`、`SSCAN`、`ZSCAN` 代替。

除了这些 O(n)时间复杂度的命令可能会导致慢查询之外， 还有一些时间复杂度可能在 O(N) 以上的命令，例如：

- `ZRANGE`/`ZREVRANGE`：返回指定 Sorted Set 中指定排名范围内的所有元素。时间复杂度为 O(log(n)+m)，n 为所有元素的数量， m 为返回的元素数量，当 m 和 n 相当大时，O(n) 的时间复杂度更小。
- `ZREMRANGEBYRANK`/`ZREMRANGEBYSCORE`：移除 Sorted Set 中指定排名范围/指定 score 范围内的所有元素。时间复杂度为 O(log(n)+m)，n 为所有元素的数量， m 被删除元素的数量，当 m 和 n 相当大时，O(n) 的时间复杂度更小。

#### 2.开启redis慢查询记录

在 `redis.conf` 文件中，我们可以使用 `slowlog-log-slower-than` 参数设置耗时命令的阈值，并使用 `slowlog-max-len` 参数设置耗时命令的最大记录条数。

当 Redis 服务器检测到执行时间超过 `slowlog-log-slower-than`阈值的命令时，就会将该命令记录在慢查询日志(slow log) 中，这点和 MySQL 记录慢查询语句类似。当慢查询日志超过设定的最大记录条数之后，Redis 会把最早的执行命令依次舍弃。

注意：由于慢查询日志会占用一定内存空间，如果设置最大记录条数过大，可能会导致内存占用过高的问题

`SLOWLOG GET` 命令默认返回最近 10 条的的慢查询命令，你也自己可以指定返回的慢查询命令的数量 `SLOWLOG GET N`。

慢查询日志中的每个条目都由以下六个值组成：

1. 唯一渐进的日志标识符。
2. 处理记录命令的 Unix 时间戳。
3. 执行所需的时间量，以微秒为单位。
4. 组成命令参数的数组。
5. 客户端 IP 地址和端口。
6. 客户端名称。

```sh
127.0.0.1:6379> SLOWLOG GET #慢日志查询
 1) 1) (integer) 5
   2) (integer) 1684326682
   3) (integer) 12000
   4) 1) "KEYS"
      2) "*"
   5) "172.17.0.1:61152"
   6) ""
  // ...
```

# Redis常见生成问题

### 1.缓存穿透

缓存穿透说简单点就是大量请求的 key 是不合理的，**根本不存在于缓存中，也不存在于数据库中** 。这就导致这些请求直接到了数据库上，根本没有经过缓存这一层，对数据库造成了巨大的压力，可能直接就被这么多请求弄宕机了。

解决方案：

* 做好前置校验（必选）

  一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等

* 缓存无效key

  给无效key进行缓存并设置过期时间，这种方式可以解决请求的 key 变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key。

* 布隆过滤器

  把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走缓存和数据库流程。

  注意：布隆过滤器存在无法删除以存入的key的问题

* 接口限流

  根据用户或者 IP 对接口进行限流，对于异常频繁的访问行为，还可以采取黑名单机制，例如将异常 IP 列入黑名单

### 2.缓存击穿

缓存击穿中，请求的 key 对应的是 **热点数据** ，该数据 **存在于数据库中，但不存在于缓存中（通常是因为缓存中的那份数据已经过期）** 。这就可能会导致瞬时大量的请求直接打到了数据库上，对数据库造成了巨大的压力，可能直接就被这么多请求弄宕机了。

解决方案：

* 设置热点数据永不过期或者过期时间比较长。

* 针对热点数据提前预热，将其存入缓存中并设置合理的过期时间比如秒杀场景下的数据在秒杀结束之前不过期。

* 请求数据库写数据到缓存之前，先获取互斥锁，保证只有一个请求会落到数据库上，减少数据库的压力。（发现缓存中没有该数据，需要查数据库时先获取互斥锁）

### 3.缓存雪崩

**缓存在同一时间大面积的失效，导致大量的请求都直接落到了数据库上，对数据库造成了巨大的压力。**或者 **缓存服务宕机也会导致缓存雪崩现象，导致所有的请求都落到了数据库上**

解决方案：

* 可用性方案

  1. 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用。

  2. 限流，避免同时处理大量的请求。

  3. 多级缓存，例如本地缓存+Redis 缓存的组合，当 Redis 缓存出现问题时，还可以从本地缓存中获取到部分数据

* 缓存集中失效

  1. 设置不同的失效时间比如随机设置缓存的失效时间（推荐）。
  2. 缓存永不失效（不太推荐，实用性太差）。
  3. 缓存预热，也就是在程序启动后或运行过程中，主动将热点数据加载到缓存中

### 4. 缓存一致性解决方案

[常用的缓存数据读写方案](https://mp.weixin.qq.com/s?__biz=MzIyOTYxNDI5OA==&mid=2247487312&idx=1&sn=fa19566f5729d6598155b5c676eee62d&chksm=e8beb8e5dfc931f3e35655da9da0b61c79f2843101c130cf38996446975014f958a6481aacf1&scene=178&cur_album_id=1699766580538032128#rd)

* **更新数据库 + 更新缓存方案**，在「并发」场景下无法保证缓存和数据一致性，且存在**「缓存资源浪费」和「机器性能浪费」**的情况发生（因为不能保证两个操作都成功，所以会导致不一致）
* **更新数据库 + 删除缓存**的方案中，「先删除缓存，再更新数据库」在「并发」场景下依旧有数据不一致问题，解决方案是「延迟双删」，但这个延迟时间很难评估，所以推荐用「先更新数据库，再删除缓存」的方案
* 在「**先更新数据库，再删除缓存**」方案下，为了**保证两步都成功执行**，需配合**「消息队列」或「订阅变更日志**」的方案来做，本质是通过「重试」的方式保证数据一致性
* 在**「先更新数据库，再删除缓存」**方案下，「**读写分离 + 主从库延迟」也会导致缓存和数据库不一致**，缓解此问题的方案是「延迟双删」，凭借经验发送「延迟消息」到队列中，延迟删除缓存，同时也要控制主从库延迟，尽可能降低不一致发生的概率



# Redis测试方案

https://mp.weixin.qq.com/s/nNEuYw0NlYGhuKKKKoWfcQ

> 1、获取 Redis 实例在当前环境下的基线性能。
>
> 2、是否用了慢查询命令？如果是的话，就使用其他命令替代慢查询命令，或者把聚合计算命令放在客户端做。
>
> 3、是否对过期 key 设置了相同的过期时间？对于批量删除的 key，可以在每个 key 的过期时间上加一个随机数，避免同时删除。
>
> 4、是否存在 bigkey？对于 bigkey 的删除操作，如果你的 Redis 是 4.0 及以上的版本，可以直接利用异步线程机制减少主线程阻塞；如果是 Redis 4.0 以前的版本，可以使用 SCAN 命令迭代删除；对于 bigkey 的集合查询和聚合操作，可以使用 SCAN 命令在客户端完成。
>
> 5、Redis AOF 配置级别是什么？业务层面是否的确需要这一可靠性级别？如果我们需要高性能，同时也允许数据丢失，可以将配置项 no-appendfsync-on-rewrite 设置为 yes，避免 AOF 重写和 fsync 竞争磁盘 IO 资源，导致 Redis 延迟增加。当然， 如果既需要高性能又需要高可靠性，最好使用高速固态盘作为 AOF 日志的写入盘。
>
> 6、Redis 实例的内存使用是否过大？发生 swap 了吗？如果是的话，就增加机器内存，或者是使用 Redis 集群，分摊单机 Redis 的键值对数量和内存压力。同时，要避免出现 Redis 和其他内存需求大的应用共享机器的情况。
>
> 7、在 Redis 实例的运行环境中，是否启用了透明大页机制？如果是的话，直接关闭内存大页机制就行了。
>
> 8、是否运行了 Redis 主从集群？如果是的话，把主库实例的数据量大小控制在 2~4GB，以免主从复制时，从库因加载大的 RDB 文件而阻塞。
>
> 9、是否使用了多核 CPU 或 NUMA 架构的机器运行 Redis 实例？使用多核 CPU 时，可以给 Redis 实例绑定物理核；使用 NUMA 架构时，注意把 Redis 实例和网络中断处理程序运行在同一个 CPU Socket 上。

### 1.测试Redis响应延迟

为了避免业务服务器到 Redis 服务器之间的网络延迟，**需要直接在 Redis 服务器上测试实例的响应延迟情况**

* 测试 60 秒内的最大响应延迟

  ```sh
  # 从输出结果可以看到，这 60 秒内的最大响应延迟为 119 微秒（0.119毫秒）
  ./redis-cli --intrinsic-latency 120
  Max latency so far: 17 microseconds.
  Max latency so far: 44 microseconds.
  Max latency so far: 94 microseconds.
  Max latency so far: 110 microseconds.
  Max latency so far: 119 microseconds.
  ```

* 查看一段时间内 Redis 的最小、最大、平均访问延迟

  **如果观察到的 Redis 运行时延迟是其基线性能的 2 倍及以上，就可以认定 Redis 变慢了**

  ```sh
  $ redis-cli -h 127.0.0.1 -p 6379 --latency-history -i 1
  min: 0, max: 1, avg: 0.13 (100 samples) -- 1.01 seconds range
  min: 0, max: 1, avg: 0.12 (99 samples) -- 1.01 seconds range
  min: 0, max: 1, avg: 0.13 (99 samples) -- 1.01 seconds range
  min: 0, max: 1, avg: 0.10 (99 samples) -- 1.01 seconds range
  min: 0, max: 1, avg: 0.13 (98 samples) -- 1.00 seconds range
  min: 0, max: 1, avg: 0.08 (99 samples) -- 1.01 seconds range
  ```

### 2.通过慢日志分析

查看 Redis 慢日志之前，你需要设置慢日志的阈值。例如，设置慢日志的阈值为 5 毫秒，并且保留最近 500 条慢日志记录

```sh
# 命令执行耗时超过 5 毫秒，记录慢日志
CONFIG SET slowlog-log-slower-than 5000
# 只保留最近 500 条慢日志
CONFIG SET slowlog-max-len 500
```

* 如果发现经常使用 O(N) 以上复杂度的命令，例如 SORT、SUNION、ZUNIONSTORE 聚合类命令

  > Redis 在操作内存数据时，时间复杂度过高，**要花费更多的 CPU 资源**

* 使用 O(N) 复杂度的命令，但 N 的值非常大

  > Redis 一次需要返回给客户端的数据过多，更多时间花费在数据协议的组装和**网络传输过程中**

* 如果你查询慢日志发现，并不是复杂度过高的命令导致的，而都是 SET / DEL 这种简单命令出现在慢日志中，那么你就要怀疑你的实例否写入了 bigkey（通过redis的一些bigkey查询命令）

  > 1.对线上实例进行 bigkey 扫描时，Redis 的 OPS 会突增，为了降低扫描过程中对 Redis 的影响，最好控制一下扫描的频率，指定 -i 参数即可，它表示扫描过程中每次扫描后休息的时间间隔，单位是秒
  >
  > 2.扫描结果中，对于容器类型（List、Hash、Set、ZSet）的 key，只能扫描出元素最多的 key。但一个 key 的元素多，不一定表示占用内存也多，你还需要根据业务情况，进一步评估内存占用情况。

### 3.通过监控分析

* cpu使用率很高

  如果你的应用程序操作 Redis 的 OPS 不是很大，但 Redis 实例的 **CPU 使用率却很高**，那么很有可能是使用了复杂度过高的命令导致的

* 集中过期问题

  如果你发现，平时在操作 Redis 时，并没有延迟很大的情况发生，但在某个时间点突然出现一波延时，其现象表现为：**变慢的时间点很有规律，例如某个整点，或者每间隔多久就会发生一波延迟。**如果是出现这种情况，那么你需要排查一下，业务代码中是否存在设置大量 key 集中过期的情况。如果有大量的 key 在某个固定时间点集中过期，在这个时间点访问 Redis 时，就有可能导致延时变大。

  > 如果在执行主动过期的过程中，出现了需要大量删除过期 key 的情况，那么此时应用程序在访问 Redis 时，必须要等待这个过期任务执行结束，Redis 才可以服务这个客户端请求。
  >
  > 如果此时需要过期删除的是一个 bigkey，那么这个耗时会更久。而且，**这个操作延迟的命令并不会记录在慢日志中。**
  >
  > 因为慢日志中**只记录一个命令真正操作内存数据的耗时**，而 Redis 主动删除过期 key 的逻辑，是在命令真正执行之前执行的。

* 内存使用率很高

  当 Redis 内存达到 maxmemory 后，每次写入新的数据之前，**Redis 必须先从实例中踢出一部分数据，让整个实例的内存维持在 maxmemory 之下**，然后才能把新数据写进来。

  > 需要注意的是，Redis 的淘汰数据的逻辑与删除过期 key 的一样，也是在命令真正执行之前执行的，也就是说它也会增加我们操作 Redis 的延迟，而且，写 OPS 越高，延迟也会越明显。
  >
  > 如果Redis 实例中还存储了 bigkey，那么**在淘汰删除 bigkey 释放内存时，也会耗时比较久。**

* 监控fork耗时

  当 Redis **开启了后台 RDB 和 AOF rewrite 后**，在执行时，它们都需要主进程创建出一个子进程进行数据的持久化。主进程创建子进程，会调用操作系统提供的 fork 函数。而 fork 在执行过程中，**主进程需要拷贝自己的内存页表给子进程**，如果这个实例很大，那么这个拷贝的过程也会比较耗时。而且这个 **fork 过程会消耗大量的 CPU 资源，在完成 fork 之前，整个 Redis 实例会被阻塞住**，无法处理任何客户端请求。如果此时你的 CPU 资源本来就很紧张，那么 fork 的耗时会更长，甚至达到秒级，这会严重影响 Redis 的性能。

  ```sh
  # 执行INFO命令
  # 上一次 fork 耗时，单位微秒
  latest_fork_usec:59477查看，latest_fork_usec 
  ```

* 监控**expired_keys**

  运维层面，你需要把 Redis 的各项运行状态数据监控起来，在 Redis 上执行 INFO 命令就可以拿到这个实例所有的运行状态数据。

  在这里我们需要重点关注 expired_keys 这一项，它代表整个实例到目前为止，累计删除过期 key 的数量。这个指标监控起来，**当这个指标在很短时间内出现了突增，需要及时报警出来，然后与业务应用报慢的时间点进行对比分析，确认时间是否一致，如果一致，则可以确认确实是因为集中过期 key 导致的延迟变大。**

* 监控是否发生Swap

  如果你发现 Redis 突然变得非常慢，**每次的操作耗时都达到了几百毫秒甚至秒级**，那此时你就需要检查 Redis 是否使用到了 Swap，在这种情况下 Redis 基本上已经无法提供高性能的服务了，通过如下命令查看是否发生swap

  ```sh
  # 先找到 Redis 的进程 ID
  $ ps -aux | grep redis-server
  # 查看 Redis Swap 使用情况
  $ cat /proc/$pid/smaps | egrep '^(Swap|Size)'
  Size:               1256 kB
  Swap:                  0 kB
  Size:                  4 kB
  Swap:                  0 kB
  Size:                132 kB
  Swap:                  0 kB
  Size:              63488 kB
  Swap:                  0 kB
  Size:                132 kB
  Swap:                  0 kB
  Size:              65404 kB
  Swap:                  0 kB
  Size:            1921024 kB
  Swap:                  0 kB
  ```

  Size 下面的 Swap 就表示这块 Size 大小的内存，有多少数据已经被换到磁盘上了，如果这两个值相等，说明这块内存的数据都已经完全被换到磁盘上了。如果只是少量数据被换到磁盘上，例如每一块 Swap 占对应 Size 的比例很小，那影响并不是很大。**如果是几百兆甚至上 GB 的内存被换到了磁盘上**，那么你就需要警惕了，这种情况 Redis 的性能肯定会急剧下降。

* 内存碎片率

  Redis 的数据都存储在内存中，当我们的应用程序频繁修改 Redis 中的数据时，就有可能会导致 Redis 产生内存碎片。

  内存碎片会降低 Redis 的内存使用率，我们可以通过执行 INFO 命令，得到这个实例的内存碎片率：

  ```sh
  
  # Memory
  used_memory:5709194824
  used_memory_human:5.32G
  used_memory_rss:8264855552
  used_memory_rss_human:7.70G
  ...
  mem_fragmentation_ratio:1.45
  ```

  内存碎片率=mem_fragmentation_ratio = used_memory_rss / used_memory。

  其中 used_memory 表示 Redis 存储数据的内存大小，而 used_memory_rss 表示操作系统实际分配给 Redis 进程的大小。

  如果 mem_fragmentation_ratio > 1.5，说明内存碎片率已经超过了 50%，这时我们就需要采取一些措施来降低内存碎片了。

  1.如果你使用的是 Redis 4.0 以下版本，只能通过重启实例来解决

  2.如果你使用的是 Redis 4.0 版本，它正好提供了自动碎片整理的功能，可以通过配置开启碎片自动整理。

  **但是，开启内存碎片整理，它也有可能会导致 Redis 性能下降。**

  原因在于，Redis 的碎片整理工作是也在**主线程**中执行的，当其进行碎片整理时，必然会消耗 CPU 资源，产生更多的耗时，从而影响到客户端的请求。

  所以，当你需要开启这个功能时，最好提前测试评估它对 Redis 的影响。

  Redis 碎片整理的参数配置如下：

  ```sh
  
  # 开启自动内存碎片整理（总开关）
  activedefrag yes
  
  # 内存使用 100MB 以下，不进行碎片整理
  active-defrag-ignore-bytes 100mb
  
  # 内存碎片率超过 10%，开始碎片整理
  active-defrag-threshold-lower 10
  # 内存碎片率超过 100%，尽最大努力碎片整理
  active-defrag-threshold-upper 100
  
  # 内存碎片整理占用 CPU 资源最小百分比
  active-defrag-cycle-min 1
  # 内存碎片整理占用 CPU 资源最大百分比
  active-defrag-cycle-max 25
  
  # 碎片整理期间，对于 List/Set/Hash/ZSet 类型元素一次 Scan 的数量
  active-defrag-max-scan-fields 1000
  ```

  

