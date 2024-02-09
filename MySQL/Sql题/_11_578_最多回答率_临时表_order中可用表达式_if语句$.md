```
输入：
SurveyLog table:
+----+--------+-------------+-----------+-------+-----------+
| id | action | question_id | answer_id | q_num | timestamp |
+----+--------+-------------+-----------+-------+-----------+
| 5  | show   | 285         | null      | 1     | 123       |
| 5  | answer | 285         | 124124    | 1     | 124       |
| 5  | show   | 369         | null      | 2     | 125       |
| 5  | skip   | 369         | null      | 2     | 126       |
+----+--------+-------------+-----------+-------+-----------+
输出：
+------------+
| survey_log |
+------------+
| 285        |
+------------+
解释：
问题 285 显示 1 次、回答 1 次。回答率为 1.0 。
问题 369 显示 1 次、回答 0 次。回答率为 0.0 。
问题 285 回答率最高。
```



* 方案1：自连接

```sql
--- 使用临时表
with t as (
    SELECT 
    question_id,action,count(*) total
    FROM SurveyLog
    where action!='skip'
    GROUP BY question_id,action
)

--- 注意order语句执行在select之后，且可以用计算表达式
SELECT 
t1.question_id survey_log 
FROM t as t1 inner join t as t2 on t1.question_id=t2.question_id
where t1.action!=t2.action and t1.action='answer' and t2.action='show'
order by t1.total/t2.total desc,t1.question_id
limit 1
```

* 方案2：使用if语句

```sql
select question_id as survey_log
from SurveyLog 
group by question_id 
order by sum(if(action='answer', 1, 0)) / sum(if(action='show', 1, 0)) desc, question_id
limit 1
```

