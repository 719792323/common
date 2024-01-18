```
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
输出：
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
```

要求：进行left join，把bonus=null和bonus<500的都输出

* 方案1：用is null来判断null

```sql
select e.name,b.bonus from Employee e left join Bonus b
on e.empId = b.empId 
where b.bonus  < 1000 or b.bonus is null
```

* 方案2：用IFNULL函数来统一处理null值（但这样会导致索引失效，如果Bonus有索引的话）

  ```sql
  select Employee.name, Bonus.bonus
  from Employee 
  left join Bonus using(empId) 
  where IFNULL(Bonus, 0) < 1000;
  ```

  