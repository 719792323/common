```
输入：
Employee table:
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 1  | 2     | 30     |
| 2  | 2     | 30     |
| 3  | 2     | 40     |
| 1  | 3     | 40     |
| 3  | 3     | 60     |
| 1  | 4     | 60     |
| 3  | 4     | 70     |
| 1  | 7     | 90     |
| 1  | 8     | 90     |
+----+-------+--------+
输出：
+----+-------+--------+
| id | month | Salary |
+----+-------+--------+
| 1  | 7     | 90     |
| 1  | 4     | 130    |
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |
+----+-------+--------+
解释：
员工 “1” 有 5 条工资记录，不包括最近一个月的 “8”:
- 第 '7' 个月为 90。
- 第 '4' 个月为 60。
- 第 '3' 个月是 40。
- 第 '2' 个月为 30。
- 第 '1' 个月为 20。
因此，该员工的累计工资汇总为:
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 1  | 7     | 90     |  (90 + 0 + 0)
| 1  | 4     | 130    |  (60 + 40 + 30)
| 1  | 3     | 90     |  (40 + 30 + 20)
| 1  | 2     | 50     |  (30 + 20 + 0)
| 1  | 1     | 20     |  (20 + 0 + 0)
+----+-------+--------+
请注意，'7' 月的 3 个月的总和是 90，因为他们没有在 '6' 月或 '5' 月工作。

员工 '2' 只有一个工资记录('1' 月)，不包括最近的 '2' 月。
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 2  | 1     | 20     |  (20 + 0 + 0)
+----+-------+--------+

员工 '3' 有两个工资记录，不包括最近一个月的 '4' 月:
- 第 '3' 个月为 60 。
- 第 '2' 个月是 40。
因此，该员工的累计工资汇总为:
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 3  | 3     | 100    |  (60 + 40 + 0)
| 3  | 2     | 40     |  (40 + 0 + 0)
+----+-------+--------+
```

求分组排序内的某范围的累计和

* 方案一：窗口函数range

  ```sql
  SELECT id,month,sum(salary)over(partition by id ORDER BY month range 2 preceding) salary  FROM employee
  WHERE (id,month) not IN( SELECT id,max(month) month FROM employee GROUP BY id)
  ORDER BY id,month desc
  ```

* 方案二：自连接法（如果不能使用窗口时）

  * 先求当前月与前一个月的累计和得两个月累计和

    ```sql
    SELECT
        E1.id,
        E1.month,
        (IFNULL(E1.salary, 0) + IFNULL(E2.salary, 0) + IFNULL(E3.salary, 0)) AS Salary
    FROM
        Employee E1
            LEFT JOIN
        Employee E2 ON (E2.id = E1.id
            AND E2.month = E1.month - 1)
            LEFT JOIN
        Employee E3 ON (E3.id = E1.id
            AND E3.month = E1.month - 2)
    ORDER BY E1.id ASC , E1.month DESC
    ;
    ```

  * 再基于上述的中间过程求三个月的和

    ```sql
    SELECT
        E1.id,
        E1.month,
        (IFNULL(E1.salary, 0) + IFNULL(E2.salary, 0) + IFNULL(E3.salary, 0)) AS Salary
    FROM
        Employee E1
            LEFT JOIN
        Employee E2 ON (E2.id = E1.id
            AND E2.month = E1.month - 1)
            LEFT JOIN
        Employee E3 ON (E3.id = E1.id
            AND E3.month = E1.month - 2)
    ORDER BY E1.id ASC , E1.month DESC
    ;
    ```

  * 最后去掉最大月

    略