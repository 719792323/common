```
输入: 
Orders 表:
+--------------+-----------------+
| order_number | customer_number |
+--------------+-----------------+
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |
+--------------+-----------------+
输出: 
+-----------------+
| customer_number |
+-----------------+
| 3               |
+-----------------+
```

求分组后count(*)最大的那组对应的customer_number

* 自己的做法：先分组求count(*)，然后再子查询中limit 1

  ```sql
  select customer_number  from (
  select 
  customer_number,count(*) total 
  from Orders 
  group by customer_number 
  ) t1
  order by total  desc
  limit 1 
  ```

* 参考做法：order by直接使用count进行排序
  ```sql
  select customer_number 
  from Orders
  group by customer_number 
  order by count(*) desc
  limit 1;
  ```

  