# Sql 分组与聚集

group by 和 Count（*）的结合使用

```sql
*MariaDB [lsql]> select open_emp_id, count(*)  from account group by open_emp_id;
+-------------+----------+
| open_emp_id | count(*) |
+-------------+----------+
|           1 |        8 |
|          10 |        7 |
|          13 |        3 |
|          16 |        6 |
+-------------+----------+
4 rows in set (0.00 sec)
```

不可用的一种情况：

> 当对数据分组时，或许还需要在结果集中过滤掉不想要的数据，并且过滤条件是针对分组数据而不是原始数据。由于group by 子句实在where子句被评估之后运行，因此无法被where子句增加过滤条件

```sql
MariaDB [lsql]> select opem_emp_id , count(*) how_many from account where count(*) > 4 group by opem_emp_id;
ERROR 1054 (42S22): Unknown column 'opem_emp_id' in 'field list'
```

可以使用having 子句进行过滤

```sql
MariaDB [lsql]> select open_emp_id, count(*) how_many from account group by open_emp_id having count(*) > 4;
+-------------+----------+
| open_emp_id | how_many |
+-------------+----------+
|           1 |        8 |
|          10 |        7 |
|          16 |        6 |
+-------------+----------+
3 rows in set (0.00 sec)
```

常用的聚集函数：

```sql
Max(),Min(),Avg(), Sum(), Count();
```



### 隐式的分组

```sql
MariaDB [lsql]> select max(avail_balance) max_blance, min(avail_balance) min_balance, avg(avail_balance) avg_balance, sum(avail_balance) tot_balance, count(*) num_accounts from account where product_cd = 'CHK';
+------------+-------------+-------------+-------------+--------------+
| max_blance | min_balance | avg_balance | tot_balance | num_accounts |
+------------+-------------+-------------+-------------+--------------+
|   38552.05 |      122.37 | 7300.800985 |    73008.01 |           10 |
+------------+-------------+-------------+-------------+--------------+
1 row in set (0.00 sec)
```

虽然这里并没有使用group by 语句，但是where的过滤条件已经构建了一个隐式的分组；

失败的案例

```sql
...
```



```sql
MariaDB [lsql]> select product_cd, max(avail_balance) max_balance, min(avail_balance) min_balance, avg(avail_balance) avg_balance, sum(avail_balance),count(*) num_accounts from account group by product_cd;
+------------+-------------+-------------+--------------+--------------------+--------------+
| product_cd | max_balance | min_balance | avg_balance  | sum(avail_balance) | num_accounts |
+------------+-------------+-------------+--------------+--------------------+--------------+
| BUS        |     9345.55 |        0.00 |  4672.774902 |            9345.55 |            2 |
| CD         |    10000.00 |     1500.00 |  4875.000000 |           19500.00 |            4 |
| CHK        |    38552.05 |      122.37 |  7300.800985 |           73008.01 |           10 |
| MM         |     9345.55 |     2212.50 |  5681.713216 |           17045.14 |            3 |
| SAV        |      767.77 |      200.00 |   463.940002 |            1855.76 |            4 |
| SBL        |    50000.00 |    50000.00 | 50000.000000 |           50000.00 |            1 |
+------------+-------------+-------------+--------------+--------------------+--------------+
6 rows in set (0.00 sec)
```



### 对独立值技术

```sql
MariaDB [lsql]> select count(account_id) from account;
+-------------------+
| count(account_id) |
+-------------------+
|                24 |
+-------------------+
1 row in set (0.00 sec)
```

```sql
MariaDB [lsql]> select count(distinct open_emp_id) from account;
+-----------------------------+
| count(distinct open_emp_id) |
+-----------------------------+
|                           4 |
+-----------------------------+
1 row in set (0.00 sec)
```

```sql
MariaDB [lsql]> select open_emp_id,count(open_emp_id) from account group by open_emp_id;
+-------------+--------------------+
| open_emp_id | count(open_emp_id) |
+-------------+--------------------+
|           1 |                  8 |
|          10 |                  7 |
|          13 |                  3 |
|          16 |                  6 |
+-------------+--------------------+
```

### 使用表达式

```sql
MariaDB [lsql]> select max(pending_balance - avail_balance) max_uncleared from account;
+---------------+
| max_uncleared |
+---------------+
|        660.00 |
+---------------+
1 row in set (0.00 sec)
```

### 对null的处理

在对于count(*) ,他是对所有的行进行计数；

在使用count（colname）的时候会对null的列忽略，因为count(colname)是对colname中包含的值进行计数；



### 产生分组

使用 group by 来进行分组

```sql
MariaDB [lsql]> select product_cd, sum(avail_balance) prod_balance from account group by product_cd;
+------------+--------------+
| product_cd | prod_balance |
+------------+--------------+
| BUS        |      9345.55 |
| CD         |     19500.00 |
| CHK        |     73008.01 |
| MM         |     17045.14 |
| SAV        |      1855.76 |
| SBL        |     50000.00 |
+------------+--------------+
6 rows in set (0.00 sec)
```

### 对多列进行分组

```sql
MariaDB [lsql]> select product_cd, open_branch_id, sum(avail_balance) from account group by product_cd, open_branch_id;
+------------+----------------+--------------------+
| product_cd | open_branch_id | sum(avail_balance) |
+------------+----------------+--------------------+
| BUS        |              2 |            9345.55 |
| BUS        |              4 |               0.00 |
| CD         |              1 |           11500.00 |
| CD         |              2 |            8000.00 |
| CHK        |              1 |             782.16 |
| CHK        |              2 |            3315.77 |
| CHK        |              3 |            1057.75 |
| CHK        |              4 |           67852.33 |
| MM         |              1 |           14832.64 |
| MM         |              3 |            2212.50 |
| SAV        |              1 |             767.77 |
| SAV        |              2 |             700.00 |
| SAV        |              4 |             387.99 |
| SBL        |              3 |           50000.00 |
+------------+----------------+--------------------+
14 rows in set (0.00 sec)
```

```sql
MariaDB [lsql]> select product_cd, open_branch_id, sum(avail_balance) from account group by product_cd, open_branch_id having product_cd ='CD';
+------------+----------------+--------------------+
| product_cd | open_branch_id | sum(avail_balance) |
+------------+----------------+--------------------+
| CD         |              1 |           11500.00 |
| CD         |              2 |            8000.00 |
+------------+----------------+--------------------+
2 rows in set (0.00 sec)
```

分组的条件也可以是表达式

### 产生合计数

```sql
MariaDB [lsql]> select product_cd, open_branch_id, sum(avail_balance) tot_balance from account group by product_cd, open_branch_id;
+------------+----------------+-------------+
| product_cd | open_branch_id | tot_balance |
+------------+----------------+-------------+
| BUS        |              2 |     9345.55 |
| BUS        |              4 |        0.00 |
| CD         |              1 |    11500.00 |
| CD         |              2 |     8000.00 |
| CHK        |              1 |      782.16 |
| CHK        |              2 |     3315.77 |
| CHK        |              3 |     1057.75 |
| CHK        |              4 |    67852.33 |
| MM         |              1 |    14832.64 |
| MM         |              3 |     2212.50 |
| SAV        |              1 |      767.77 |
| SAV        |              2 |      700.00 |
| SAV        |              4 |      387.99 |
| SBL        |              3 |    50000.00 |
+------------+----------------+-------------+
14 rows in set (0.00 sec)
```

加上 rollup 选项：

```sql
MariaDB [lsql]> select product_cd, open_branch_id, sum(avail_balance) tot_balance from account group by product_cd, open_branch_id with rollup;
+------------+----------------+-------------+
| product_cd | open_branch_id | tot_balance |
+------------+----------------+-------------+
| BUS        |              2 |     9345.55 |
| BUS        |              4 |        0.00 |
| BUS        |           NULL |     9345.55 |
| CD         |              1 |    11500.00 |
| CD         |              2 |     8000.00 |
| CD         |           NULL |    19500.00 |
| CHK        |              1 |      782.16 |
| CHK        |              2 |     3315.77 |
| CHK        |              3 |     1057.75 |
| CHK        |              4 |    67852.33 |
| CHK        |           NULL |    73008.01 |
| MM         |              1 |    14832.64 |
| MM         |              3 |     2212.50 |
| MM         |           NULL |    17045.14 |
| SAV        |              1 |      767.77 |
| SAV        |              2 |      700.00 |
| SAV        |              4 |      387.99 |
| SAV        |           NULL |     1855.76 |
| SBL        |              3 |    50000.00 |
| SBL        |           NULL |    50000.00 |
| NULL       |           NULL |   170754.46 |
+------------+----------------+-------------+
21 rows in set (0.00 sec)
```

### 分组过滤条件

having 子语句是用来对分组设置过滤条件的关键字

```sql
MariaDB [lsql]> select product_cd, sum(avail_balance) prod_balance from account where status = 'ACTIVE' group by product_cd having sum(avail_balance) >= 10000;
+------------+--------------+
| product_cd | prod_balance |
+------------+--------------+
| CD         |     19500.00 |
| CHK        |     73008.01 |
| MM         |     17045.14 |
| SBL        |     50000.00 |
+------------+--------------+
4 rows in set (0.00 sec)
```

```sql
MariaDB [lsql]> select product_cd, sum(avail_balance) prod_balance from account where status = 'ACTIVE' and sum(avail_balance) > 1000 group by product_cd;
ERROR 1111 (HY000): Invalid use of group function
#失败的原因是where子语句不能使用聚集函数
```

在对包含group by 子句的查询中增加过滤条件时，需要仔细考虑过滤条件是针对原始数据还是分组数据的，原始数据放到where子句中，针对分组数据的使用having子句；

### 在having子句中包含未在select语句中出现的聚集函数

```sql
MariaDB [lsql]> select product_cd, sum(avail_balance) prod_balance from account where status = 'ACTIVE' group by product_cd having min(avail_balance) >= 1000 and max(avail_balance) <= 10000;
+------------+--------------+
| product_cd | prod_balance |
+------------+--------------+
| CD         |     19500.00 |
| MM         |     17045.14 |
+------------+--------------+
2 rows in set (0.00 sec)
```

