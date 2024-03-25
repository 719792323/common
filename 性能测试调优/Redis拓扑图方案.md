# 参考资料

[限流算法](https://javaguide.cn/high-availability/limit-request.html)

> 有固定滑动窗口、动态滑动窗口、漏桶算法、令牌桶算法。**拓扑图采用动态滑动窗口算法**，需要看懂该算法如何作用于限流

[Redis实现限流算法窗口](https://blog.csdn.net/wangdamingll/article/details/108084646)

> 看懂如何使用redis的zset数据结构实现滑动窗口限流算法

[Zset相关命令](https://cloud.tencent.com/developer/article/2225821)

# 实验方案

1. 使用zset结构存储链路数据

   > zadd zsetname score value
   >
   > 其中score存储为链路数据对应的时间戳、value存储链路数据实体数据

   ```sh
   127.0.0.1:6379> zadd demo54 0 "a->b"
   (integer) 1
   127.0.0.1:6379> zadd demo54 1 "b->c"
   (integer) 1
   127.0.0.1:6379> zadd demo54 2 "c->d"
   (integer) 1
   127.0.0.1:6379> 
   ```

2. 根据时间范围获取链路数据

   > zrangebyscore zsetname start end
   >
   > 其中end是当前时间，start=end-窗口长度，通过该命令既可以获得当前窗口内数据，然后根据该窗口内数据构建拓扑图

   ```sh
   127.0.0.1:6379> ZRANGEBYSCORE demo54 1 2
   1) "b->c"
   2) "c->d"
   127.0.0.1:6379> 
   ```

3. 删除指定范围时间内数据

   > zremrangebyscore zsetanme start end
   >
   > 通过该命令可以删除过期的链路数据

   ```sh
   127.0.0.1:6379> ZRANGEBYSCORE demo54 -inf +inf
   1) "a->b"
   2) "b->c"
   3) "c->d"
   4) "d->c"
   127.0.0.1:6379> ZREMRANGEBYSCORE demo54 0 1
   (integer) 2
   127.0.0.1:6379> ZRANGEBYSCORE demo54 -inf +inf
   1) "c->d"
   2) "d->c"
   127.0.0.1:6379> 
   ```

   

