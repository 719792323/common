### 1、隐式转换

字段为字符串类型。
![在这里插入图片描述](https://img-blog.csdnimg.cn/08f6cbd714b9445eae0317c92560a928.png)
建立了B+Tree索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fef8319e49c74ffa8302f31bfcd954e2.png)
使用数值格式，没有走索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7bd65d338ea04859b17da54b98fc00fc.png)
使用正确的格式，正常走索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/68a41e21b7b0422a87563566781bd843.png)

字段为字串类型，是B+树的普通索引，如果查询条件传了一个数字过去，就会导致索引失效。

> 为什么第一条语句未加单引号就不走索引了呢？这是因为不加单引号时，是字符串跟数字的比较，它们**类型不匹配，MySQL会做隐式的类型转换，把它们转换为浮点数再做比较。隐式的类型转换，索引会失效。**

### 2、最左匹配原则

MySQl建立联合索引时，会遵循最左前缀匹配的原则，即最左优先。如果你建立一个（a,b,c）的联合索引，相当于建立了(A)、(A,B)、(A,C)、(A,B,C) 四个索引。

建立联合索引（A、B、C）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/be4f0a98bc9941bf954bfee2866deff5.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f0f53db202654372b204c8283b68456c.png)
**注意**
B/C不是中的第一个列，不满足最左匹配原则，所以索引不生效。
在联合索引中，只有查询条件满足最左匹配原则时，索引才正常生效。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b3151d35eee24c23b780ae73069c0a6e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5daa5c8d2def4b0aaeb223cfc2ec8ed0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/17b76e8a3d38483fb8ce3d0702a13695.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8ee2adea92c74f2bb2b0c04ce6e42d54.png)

### 3、深分页问题

limit深分页问题，会导致慢查询。

batch_code字段建立索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/785b72b0c7d745f9a9d53f30e387abab.png)

```sql
select a.id,a.sp_no,a.sp_name,a.sp_result,a.sp_status from t_approve_record a 
where a.batch_code = '20221029133216' limit 100000,10;
```

这个SQL的执行流程：

> 通过普通二级索引树idx_batch_code，过滤batch_code条件，找到满足条件的主键id。
> 通过主键id，回到id主键索引树，找到满足记录的行，然后取出需要展示的列（回表过程）。
> 扫描满足条件的100010行，然后扔掉前100000行，返回。

如何优化深分页问题?

我们可以通过减少回表次数来优化。一般有**标签记录法和延迟关联法**。

**标签记录法**

就是标记一下上次查询到哪一条了，下次再来查的时候，从该条开始往下扫描。就好像看书一样，上次看到哪里了，你就折叠一下或者夹个书签，下次来看的时候，直接就翻到啦。

假设上一次记录到100000，则SQL可以修改为：

```sql
select a.id,a.sp_no,a.sp_name,a.sp_result,a.sp_status from t_approve_record a 
where a.id > 100000 limit 10;
```

这样的话，后面无论翻多少页，性能都会不错的，因为命中了id索引。
但是这种方式有局限性：需要一种类似连续自增的字段。

**延迟关联法**

延迟关联法，就是把条件转移到主键索引树，然后减少回表。

```sql
select a.id,a.sp_no,a.sp_name,a.sp_result,a.sp_status from t_approve_record a  
inner join
(select a.id from t_approve_record a WHERE a.batch_code = '20221029133216' limit 100000, 10)  b 
on a.id= b.id;
```

优化思路就是，先通过idx_batch_code二级索引树查询到满足条件的主键ID，再与原表通过主键ID内连接，这样后面直接走了主键索引了，同时也减少了回表。

explain看一下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/93c7efa3978746f0b33b6ee924c82b66.png)

### 4、in元素过多

如果使用了in，即使后面的条件加了索引，还是要注意in后面的元素不要过多哈。in元素一般建议不要超过200个，如果超过了，建议分组，每次200一组进行哈。
oracle数据库有的版本还有数量限制，只能用exist替换或者临时表、in() or in()等方案。
Oracle 9i 中个数不能超过256,Oracle 10g个数不能超过1000个。

in查询为什么慢呢？

> 这是因为in查询在MySQL底层是通过**n\*m**的方式去搜索，类似**union**。
> in查询在进行cost代价计算时（代价 = 元组数 * > IO平均值），是通过将in包含的数值，一条条去查询获取元组数的，因此这个计算过程会比较的慢，所以MySQL设置了个**临界值**(eq_range_index_dive_limit)，5.6之后超过这个临界值后该列的cost就不参与计算了。因此会导致执行计划选择不准确。默认是200，即in条件超过了200个数据，会导致in的代价计算存在问题，可能会导致Mysql选择的索引不准确。

### 5、order by 走文件排序

如果order by 使用到文件排序，则会可能会产生慢查询。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d8897adb559040d4be42e3b1e084ca85.png)

order by文件排序效率为什么较低？
![在这里插入图片描述](https://img-blog.csdnimg.cn/621a0260a17041d697860c3a2e8bb3cf.png)

> **order by排序，分为全字段排序和rowid排序**。
> 它是拿max_length_for_sort_data和结果行数据长度对比，如果结果行数据长度超过max_length_for_sort_data这个值，就会走rowid排序，相反，则走全字段排序。

#### 5.1 rowid排序

**rowid排序，一般需要回表去找满足条件的数据**，所以效率会慢一点。以下这个SQL，使用rowid排序，执行过程是这样：

```sql
explain
select a.sp_no,a.batch_code,a.sp_order from t_approve_record a  
where a.batch_code = '20221029133216' 
order by a.sp_order 
limit 10;
```

1. MySQL为对应的线程初始化sort_buffer，放入需要排序的sp_order字段，以及主键id；
2. 从索引树idx_batch_code， 找到第一个满足batch_code='20221029133216’条件的主键id,假设id为X；
3. 到主键id索引树拿到id=X的这一行数据， 取sp_order和主键id的值，存到sort_buffer；
4. 从索引树idx_batch_code拿到下一个记录的主键id，假设id=Y；
5. 重复步骤 3、4 直到batch_code的值不等于’20221029133216’为止；
6. 前面5步已经查找到了所有batch_code='20221029133216’的数据，在sort_buffer中，将所有数据根据sp_order进行排序；遍历排序结果，取前10行，并按照id的值回到原表中，取出sp_no,batch_code,sp_order几个字段返回给客户端。

#### 5.2 全字段排序

同样的SQL，如果是走全字段排序是这样的：

1. MySQL 为对应的线程初始化sort_buffer，放入需要查询的sp_no,batch_code,sp_order字段；
2. 从索引树idx_batch_code， 找到第一个满足 batch_code='20221029133216’条件的主键 id，假设找到id=X；
3. 到主键id索引树拿到id=X的这一行数据， 取sp_no,batch_code,sp_order三个字段的值，存到sort_buffer；
4. 从索引树idx_batch_code拿到下一个记录的主键id，假设id=Y；
5. 重复步骤 3、4 直到batch_code的值不等于’20221029133216’为止；
6. 前面5步已经查找到了所有batch_code='20221029133216’的数据，在sort_buffer中，将所有数据根据sp_order进行排序；
7. 按照排序结果取前10行返回给客户端。

sort_buffer的大小是由一个参数控制的：**sort_buffer_size**。

> 如果要排序的数据小于sort_buffer_size，排序在sort_buffer内存中完成
> 如果要排序的数据大于sort_buffer_size，则借助磁盘文件来进行排序。
> 借助磁盘文件排序的话，效率就更慢一点。因为先把数据放入sort_buffer，当快要满时。会排一下序，然后把sort_buffer中的数据，放到临时磁盘文件，等到所有满足条件数据都查完排完，再用归并算法把磁盘的临时排好序的小文件，合并成一个有序的大文件。

#### 5.3 如何优化order by的文件排序？

因为数据是无序的，所以就需要排序。如果数据本身是有序的，那就不会再用到文件排序啦。而索引数据本身是有序的，我们通过**建立索引来优化order by语句**。
我们还可以通过**调整max_length_for_sort_data、sort_buffer_size等参数优化**；

### 6、索引字段上使用（！= 或者 < >），索引可能失效

虽然age加了索引，但是使用了 **！= 或者<>，not in** 这些时，索引如同虚设。

![在这里插入图片描述](https://img-blog.csdnimg.cn/cb476ab890104c66927a7e93f29efc33.png)
其实这个也是跟mySQL优化器有关，如果优化器觉得即使走了索引，还是需要扫描很多很多行的哈，它觉得不划算，不如直接不走索引。平时我们用！= 或者< >，not in的时候注意下。

### 7、索引字段上使用is null， is not null，索引可能失效

is null 不走索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e1e8cb44b224d54a494e4c390e898fc.png)

is not null 不走索引，key=All。![在这里插入图片描述](https://img-blog.csdnimg.cn/f0a23227d3c14f79ae3b2a57a8436b9f.png)
and或者or连接起来，索引照样失效。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fb90e75891e54095973f3a4883ced38c.png)
很多时候，也是因为数据量问题，导致了MySQL优化器放弃走索引。同时，平时我们用explain分析SQL的时候，如果type=range,要注意一下哈，因为这个可能因为数据量问题，导致索引无效。

### 8、左右连接，关联的字段的编码格式不一样

2张表建立索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c365deb208a7436a926d7d6a55458dcd.png)

表1的**字段**是utf8编码，表2的字段是utf8mb4编码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2afc869f76ff428a9e8c59826a64ba36.png)

连接查询，走了全表扫描
![在这里插入图片描述](https://img-blog.csdnimg.cn/74bfc21e9a9147918431fd65d7e0b278.png)

换回相同的编码后
![在这里插入图片描述](https://img-blog.csdnimg.cn/ca76937de0ee496ca87d5ccfe2cb726b.png)

如果把它们的template_id字段改为编码一致，相同的SQL，还是会走索引。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5441a94658e74cba924a37915363a061.png)

### 9、group by使用临时表

sp_name字段未建索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a8882afb5614cd5a434cbd72599ee66.png)
Extra 这个字段的Using temporary表示在执行分组的时候使用了**临时表**。
Extra 这个字段的Using filesort表示使用了**文件排序**

原因与group by的执行过程有关。

可以有这些优化方案：

**group by 后面的字段加索引**。
order by null 不用排序。
尽量只使用**内存临时表**。
使用SQL_BIG_RESULT。

建立索引后的执行计划：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0dd08754642439c80ca034baf892f53.png)

### 10、delete + in子查询不走索引

查看执行计划，发现不走索引。

![在这里插入图片描述](https://img-blog.csdnimg.cn/86e8d22919814f50a20c695455417ba2.png)
把delete换成select，就会走索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce699e922c6e48808729c4d13fa600eb.png)
查看最终执行的sql：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7e9df2e63be4244bab26c941ec33acf.png)

```sql
 select `qw`.`t_approve_record`.`id` AS `id`,`qw`.`t_approve_record`.`template_id` AS `template_id`,`qw`.`t_approve_record`.`sp_no` AS `sp_no`,`qw`.`t_approve_record`.`sp_name` AS `sp_name`,`qw`.`t_approve_record`.`sp_result` AS `sp_result`,`qw`.`t_approve_record`.`apply_time` AS `apply_time`,`qw`.`t_approve_record`.`applyer_userid` AS `applyer_userid`,`qw`.`t_approve_record`.`approver_userid` AS `approver_userid`,`qw`.`t_approve_record`.`sp_status` AS `sp_status`,`qw`.`t_approve_record`.`sp_time` AS `sp_time`,`qw`.`t_approve_record`.`sp_use_time` AS `sp_use_time`,`qw`.`t_approve_record`.`is_overtime` AS `is_overtime`,`qw`.`t_approve_record`.`last_update` AS `last_update`,`qw`.`t_approve_record`.`delete_time` AS `delete_time`,`qw`.`t_approve_record`.`create_time` AS `create_time`,`qw`.`t_approve_record`.`batch_code` AS `batch_code`,`qw`.`t_approve_record`.`sp_order` AS `sp_order` from `qw`.`t_approve_template` join `qw`.`t_approve_record` where (`qw`.`t_approve_record`.`template_id` = `qw`.`t_approve_template`.`template_id`)
```

> 可以发现，实际执行的时候，MySQL对select in子查询做了优化，把子查询改成join的方式，所以可以走索引。
> 但是很遗憾，对于delete in子查询，MySQL却没有对它做这个优化。

### 11、其它索引失效场景，如InnoDB下like '%xxx’索引失效

![在这里插入图片描述](https://img-blog.csdnimg.cn/57b48efec87048afa1e2190ca21a8196.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b1fde4a99ea647daa58edab5e87f038b.png)
索引失效
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c4a6bae821f4599a47c6dc503d7a459.png)



# limit a,b

https://blog.csdn.net/chihaihai/article/details/106256874



