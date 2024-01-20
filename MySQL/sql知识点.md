# sql执行顺序

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-sSnecEa8-1628578703350)(C:\Users\hbwhx\Desktop\学习\窗口函数\Windows Functions.assets\image-20210713143908533.png)]](https://img-blog.csdnimg.cn/796f5d52a21e488bb6f644758c7afb5c.png)

# 关键字

* join

  ![img](https://pic4.zhimg.com/80/v2-ffbd09f10b89934fd87e561122bc04ab_720w.webp)

  * 目前mysql不支持full join
    * 可以用左外和又外union实现full join

* group by

* having

  **在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用**

  Having语法（**先执行where，后group by，最后having**）

  ```sql
  SELECT column_name, aggregate_function(column_name)
  FROM table_name
  WHERE column_name operator value 
  GROUP BY column_name
  HAVING aggregate_function(column_name) operator value
  ```

  我们拥有下面这个 "Orders" 表：

  | O_Id | OrderDate  | OrderPrice | Customer |
  | :--- | :--------- | :--------- | :------- |
  | 1    | 2008/12/29 | 1000       | Bush     |
  | 2    | 2008/11/23 | 1600       | Carter   |
  | 3    | 2008/10/05 | 700        | Bush     |
  | 4    | 2008/09/28 | 300        | Bush     |
  | 5    | 2008/08/06 | 2000       | Adams    |
  | 6    | 2008/07/21 | 100        | Carter   |

  现在，我们希望查找订单总金额少于 2000 的客户。

  我们使用如下 SQL 语句：

  ```sql
  SELECT Customer,SUM(OrderPrice) FROM Orders
  GROUP BY Customer
  HAVING SUM(OrderPrice)<2000
  ```

  结果集类似：

  | Customer | SUM(OrderPrice) |
  | :------- | :-------------- |
  | Carter   | 1700            |

  现在我们希望查找客户 "Bush" 或 "Adams" 拥有超过 1500 的订单总金额。

  我们在 SQL 语句中增加了一个普通的 WHERE 子句：

  ```sql
  SELECT Customer,SUM(OrderPrice) FROM Orders
  WHERE Customer='Bush' OR Customer='Adams'
  GROUP BY Customer
  HAVING SUM(OrderPrice)>1500
  ```

  结果集：

  | Customer | SUM(OrderPrice) |
  | :------- | :-------------- |
  | Bush     | 2000            |
  | Adams    | 2000            |

  **注：**

  1. 当同时含有where子句、group by 子句 、having子句及聚集函数时，执行顺序如下：

     * **执行where子句查找符合条件的数据**；

     * **使用group by 子句对数据进行分组**；对group by 子句形成的组运行聚集函数计算每一组的值；

     * **最后用having 子句去掉不符合条件的组**。

  2. having 子句中的每一个元素也必须出现在select列表中

  3. having子句和where子句都可以用来设定限制条件以使查询结果满足一定的条件限制

  4. having子句限制的是组，而不是行。where子句中不能使用聚集函数，而having子句中可以

* distinct

  distinct这个关键字来过**滤掉多余的重复记录只保留一条**，但往往只用它来返回不重复记录的条数，而不是用它来返回不重记录的所有值。其原因是distinct只有用二重循环查询来解决，而这样对于一个数据量非常大的站来说，无疑是会直接影响到效率的。

  ```sql
  SELECT DISTINCT column1, column2, ...
  FROM table_name;
  ```

  table表

  字段1   字段2
    id    name
    1      a
    2      b
    3      c
    4      c
    5      b

  * distinct单列去重

    ```sql
    select distinct name from table
    得到的结果是:
    
     ----------
    
    name
       a
       b
       c
    ```

  * distinct多列去重

    ```sql
    select distinct name, id from table
    
    结果会是:
    
    ----------
    
    id name
       1 a
       2 b
       3 c
       4 c
       5 b
    distinct怎么没起作用？作用是起了的，不过他同时作用了两个字段，也就是必须得id与name都相同的才会被排除。。。。。。。
    ```

  * distinct必须放在开头

    ```sql
    select id, distinct name from table
    很遗憾，除了错误信息你什么也得不到，distinct必须放在开头。
    ```

    

* union

* union all

* count

  * count（*） 、count（1）：这两个的使用方法和结果是相同的。表示返回所有的行，经常使用在没有where条件的语句中，速度较快。

  * count（column）：返回字段在表中出现的次数，**不包括有null值**

  * count（distinct column） ：返回列中不包含指定字段为null的唯一行数

  * count（expression）：**返回不包含`NULL`值的行数**，expression 是表达式

  > count(*)、count(1)、count(column)区别是什么

  **注意：count如果不接窗口函数使用会只返回一条数据**

# 函数

* rank

* min/max/avg

* ifnull(column,default)

  如过column对应的为null，则在本次查询使用default作为替换值

* datediff(date1,date2)

  返回date2-date1相隔的天数，实际计算两个date类型相差天数时要用datediff(date2,date1)，而不是用date2-date1，因为date2-date1不是按天的精度来算，即如果两个date之差小于<24个小时，那么结果就不是1
  
* round(column,n)

  对column进行四舍五入保留到小数点后n位
  
* ceil

  向上取整

* floor

  向下取整

# 正则表达式

正则表达式语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column_name REGEXP 'pattern';
```

* 注意：如使用\s（空格）时，在'pattern'中要写成'\\\s'，进行转义



# 窗口函数

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-3ut14zoW-1628578703352)(C:\Users\hbwhx\Desktop\学习\窗口函数\Windows Functions.assets\image-20210713144138756.png)]](https://img-blog.csdnimg.cn/1456767a7ac84c0fbfc9e69d1436af92.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDg1MjA2Nw==,size_16,color_FFFFFF,t_70)