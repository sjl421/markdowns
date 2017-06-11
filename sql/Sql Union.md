# Sql Union

当两个数据集合执行union操作时，必须需要满足一下的规范

1. 两个数据结合必须具有同样数目的列;
2. 两个数据集中对应列的数据类型必须是一样的（或者服务器能够将其中一种转换为另一种）;


union 和 Union all 的区别

* Union 会将结果集中重复的元素删除掉；
* union all 不会删除结果集中重复的元素；



### intersect操作符

intersect 操作符执行集合的交集操作；





### except操作符

except操作符执行集合的差操作；



### 对复合查询结果排序

在order by 子语句中指定要排序的列时，需要从复合查询的第一个查询中选择列名。通常情况下，符合查询中两个查询中对应的列名是相同的，但是为了避免错误的反正，最好的办法是在符合查询中定义别名

```sql
select emp_id, assigned_branch_id
from employee
where title = 'Teller'
union
select open_emp_id, open_branch_id
from account
where product_cd = 'SAV'
order by emp_id;
+--------+--------------------+
| emp_id | assigned_branch_id |
+--------+--------------------+
|      1 |                  1 |
|      7 |                  1 |
|      8 |                  1 |
|      9 |                  1 |
|     10 |                  2 |
|     11 |                  2 |
|     12 |                  2 |
|     14 |                  3 |
|     15 |                  3 |
|     16 |                  4 |
|     17 |                  4 |
|     18 |                  4 |
+--------+--------------------+
12 rows in set (0.00 sec)
```

```sql
select emp_id, assigned_branch_id
from employee
where title = 'Teller'
union
select open_emp_id, open_branch_id
from account
where product_cd = 'SAV'
order by open_emp_id;
ERROR 1054 (42S22) at line 1 in file: 's4.sql': Unknown column 'open_emp_id' in 'order clause'
```

### 复合查询中语句的执行顺序

复合查询中，sql的执行顺序从上而下执行的，所以此时使用union 和 union all 的位置不同，会导致最终复合查询得到的结果集的不同；