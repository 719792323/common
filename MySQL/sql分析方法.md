### explain查看分析SQL的执行计划

一般来说，我们需要重点关注type、rows、filtered、extra、key。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1603e383426f4e389c0aa630f8d5ef72.png)
**type**
type表示连接类型，查看索引执行情况的一个重要指标。
以下性能从好到坏依次：system > const > eq_ref > ref > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

system：这种类型要求数据库表中只有一条数据，是const类型的一个特例，一般情况下是不会出现的。
**const**：通过一次索引就能找到数据，一般用于**主键或唯一索引作为条件，这类扫描效率极高**，速度非常快。
**eq_ref**：常用于**主键或唯一索引扫描，一般指使用主键的关联查询**
**ref** : 常用于**非主键和唯一索引扫描**。
ref_or_null：这种连接类型类似于ref，区别在于MySQL会额外搜索包含NULL值的行
index_merge：使用了索引合并优化方法，查询使用了两个以上的索引。
unique_subquery：类似于eq_ref，条件用了in子查询
index_subquery：区别于unique_subquery，用于非唯一索引，可以返回重复值。
**range**：常用于范围查询，比如：between … and ，大于小于或 In 等操作
**index：全索引扫描** 该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小；
**ALL：全表扫描**

**实际sql优化中，最后达到ref或range级别。**

**rows**
该列表示MySQL估算要找到我们所需的记录，**需要读取的行数**。**对于InnoDB表，此数字是估计值**，并非一定是个准确值。

**filtered**
该列是一个百分比的值，**表里符合条件的记录数的百分比**。简单点说，**这个字段表示存储引擎返回的数据在经过过滤后，剩下满足条件的记录数量的比例。**

**extra**
该字段包含有关MySQL如何解析查询的其他信息，它一般会出现这几个值：

**Using filesort**：表示按**文件排序**，一般是在指定的排序和索引排序不一致的情况才会出现。一般见于order by语句
**Using index** ：表示是否用了覆盖索引。
**Using temporary**: 表示是否使用了**临时表,性能特别差**，需要重点优化。一般多见于group by语句，或者union语句。
**Using where** : 表示使用了where条件过滤. WHERE子句用于限制哪一个行匹配下一个表或发送到客户。除非你专门从表中索取或检查所有行，如果Extra值不为Using where并且表联接类型为ALL或index，查询可能会有一些错误。**需要回表查询**
Using index condition：MySQL5.6之后新增的索引下推。在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据减少回表的数据。

**key**
该列表示实际用到的索引名称。一般配合**possible_keys**列一起看。

### profile分析执行耗时

explain只是看到SQL的预估执行计划，如果要了解SQL真正的执行线程状态及消耗的时间，需要使用profiling。开启profiling参数后，后续执行的SQL语句都会记录其资源开销，包括IO，上下文切换，CPU，内存等等，我们可以根据这些开销进一步分析当前慢SQL的瓶颈再进一步进行优化。

profiling默认是关闭，我们可以使用show variables like '%profil%'查看是否开启。



![在这里插入图片描述](https://img-blog.csdnimg.cn/bcfdbb491f3b43b8b7202c03a9848e37.png)
可以使用set profiling=ON开启。开启后，可以运行几条SQL，然后使用show profiles查看一下。

```sql
set profiling=ON
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/abf2859c9d9d4502898373190945e8f0.png)
show profiles会显示最近发给服务器的多条语句，条数由变量profiling_history_size定义，默认是15。
如果我们需要看单独某条SQL的分析，可以show profile查看最近一条SQL的分析，也可以使用show profile for query id（其中id就是show profiles中的QUERY_ID）查看具体一条的SQL语句分析。
除了查看profile ，还可以查看cpu和io。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a4f6625fc0b431fbcabf77107f1867f.png)

### Optimizer Trace分析详情

profile只能查看到SQL的执行耗时，但是无法看到SQL真正执行的过程信息，即不知道MySQL优化器是如何选择执行计划。这时候，我们可以使用Optimizer Trace，它可以跟踪执行语句的解析优化执行的全过程。

我们可以使用set optimizer_trace="enabled=on"打开开关，接着执行要跟踪的SQL，最后执行select * from information_schema.optimizer_trace跟踪，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/d06aab7b124d41ac80d11f505892d260.png)
大家可以查看分析其执行树，会包括三个阶段：

join_preparation：准备阶段
join_optimization：分析阶段
join_execution：执行阶段

查看json串
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9bbd915ba834b65bfcf9b038088e16b.png)

### 确定SQL问题并采用相应的方案

最后确认问题，就采取对应的措施。

多数慢SQL都跟索引有关，比如不加索引，索引不生效、不合理等，这时候，我们可以优化索引。
优化SQL语句，load额外的字段，比如一些in元素过多问题（分批），深分页问题（基于上一次数据过滤等），进行时间分段查询。
SQl没办法很好优化，可以改用ES的方式，或者数仓。
如果单表数据量过大导致慢查询，则可以考虑分库分表。
如果数据库在刷脏页导致慢查询，考虑是否可以优化一些参数，跟DBA讨论优化方案。
如果存量数据量太大，考虑是否可以让部分数据归档。