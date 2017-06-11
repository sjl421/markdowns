# Sql 子查询

```sql
mysql> select account_id, product_cd, cust_id, avail_balance from account where account_id = (select max(account_id) from account);
+------------+------------+---------+---------------+
| account_id | product_cd | cust_id | avail_balance |
+------------+------------+---------+---------------+
|         29 | SBL        |      13 |      50000.00 |
+------------+------------+---------+---------------+
1 row in set (0.01 sec)
```



### 非关联子查询

子语句可以单独的执行，并不需要引用包含语句中的任何内容；

```sql
select account_id, product_cd, cust_id, avail_balance from account where open_emp_id <> (select e.emp_id from employee e inner join branch b on e.assigned_branch_id = b.branch_id where e.title = 'Head Teller' and b.city = 'Woburn');
+------------+------------+---------+---------------+
| account_id | product_cd | cust_id | avail_balance |
+------------+------------+---------+---------------+
|          7 | CHK        |       3 |       1057.75 |
|          8 | MM         |       3 |       2212.50 |
|         10 | CHK        |       4 |        534.12 |
|         11 | SAV        |       4 |        767.77 |
|         12 | MM         |       4 |       5487.09 |
|         13 | CHK        |       5 |       2237.97 |
|         14 | CHK        |       6 |        122.37 |
|         15 | CD         |       6 |      10000.00 |
|         18 | CHK        |       8 |       3487.19 |
|         19 | SAV        |       8 |        387.99 |
|         21 | CHK        |       9 |        125.67 |
|         22 | MM         |       9 |       9345.55 |
|         23 | CD         |       9 |       1500.00 |
|         24 | CHK        |      10 |      23575.12 |
|         25 | BUS        |      10 |          0.00 |
|         28 | CHK        |      12 |      38552.05 |
|         29 | SBL        |      13 |      50000.00 |
+------------+------------+---------+---------------+
17 rows in set (0.00 sec)
```



### 多行多列的子查询

使用 in 和 not in 结合使用返回多行多列的子查询；

```sql lite
mysql> select branch_id, name, city from branch where name in ('Headquarters', 'Quincy Branch');
+-----------+---------------+---------+
| branch_id | name          | city    |
+-----------+---------------+---------+
|         1 | Headquarters  | Waltham |
|         3 | Quincy Branch | Quincy  |
+-----------+---------------+---------+
2 rows in set (0.00 sec)
```

all 运算符，用于将某个单值与集合中的每个值进行比较，在使用all运算符构建过滤条件时需要将其中一个比较运算符（=，<>,<,>等） 与all运算符配合使用；使用all 运算符时，只有与值集中的所有成员比较都成立时条件才为真。

```sql
mysql> select emp_id, fname, lname, title from employee where emp_id <> all(select superior_emp_id from employee where superior_emp_id is not null);
+--------+----------+----------+----------------+
| emp_id | fname    | lname    | title          |
+--------+----------+----------+----------------+
|      2 | Susan    | Barker   | Vice President |
|      5 | John     | Gooding  | Loan Manager   |
|      7 | Chris    | Tucker   | Teller         |
|      8 | Sarah    | Parker   | Teller         |
|      9 | Jane     | Grossman | Teller         |
|     11 | Thomas   | Ziegler  | Teller         |
|     12 | Samantha | Jameson  | Teller         |
|     14 | Cindy    | Mason    | Teller         |
|     15 | Frank    | Portman  | Teller         |
|     17 | Beth     | Fowler   | Teller         |
|     18 | Rick     | Tulman   | Teller         |
+--------+----------+----------+----------------+
11 rows in set (0.00 sec)
```

>当使用not in 或者 <>运算符比较一个值和一个值集时，需要确保值集中不包含null值，因为任何一个将值与null进行比较的企图都讲产生未知的结果。



### any 运算符

any运算符会将一个值与值集中的每个成员相比较，并且只要有一个比较成立，则条件为真；

```sql
select account_id, cust_id, product_cd, avail_balance 
from account
where avail_balance > any ( select a.avail_balance
  from account a inner join individual i
    on a.cust_id = i.cust_id
  where i.fname = 'Frank' and i.lname = 'Tucker');
+------------+---------+------------+---------------+
| account_id | cust_id | product_cd | avail_balance |
+------------+---------+------------+---------------+
|          3 |       1 | CD         |       3000.00 |
|          4 |       2 | CHK        |       2258.02 |
|          8 |       3 | MM         |       2212.50 |
|         12 |       4 | MM         |       5487.09 |
|         13 |       5 | CHK        |       2237.97 |
|         15 |       6 | CD         |      10000.00 |
|         17 |       7 | CD         |       5000.00 |
|         18 |       8 | CHK        |       3487.19 |
|         22 |       9 | MM         |       9345.55 |
|         23 |       9 | CD         |       1500.00 |
|         24 |      10 | CHK        |      23575.12 |
|         27 |      11 | BUS        |       9345.55 |
|         28 |      12 | CHK        |      38552.05 |
|         29 |      13 | SBL        |      50000.00 |
+------------+---------+------------+---------------+
14 rows in set (0.00 sec)
```

### 多列子查询

过滤条件中包含多余一条的信息，需要用（）将所包含的信息包起来

```sql
select account_id, product_cd, cust_id from account 
where (open_branch_id, open_emp_id) in
  (select b.branch_id, e.emp_id
    from branch b inner join employee e
      on b.branch_id = e.assigned_branch_id
    where b.name = 'Woburn Branch'
      and (e.title = 'Teller' or e.title = 'Head Teller'));
+------------+------------+---------+
| account_id | product_cd | cust_id |
+------------+------------+---------+
|          1 | CHK        |       1 |
|          2 | SAV        |       1 |
|          3 | CD         |       1 |
|          4 | CHK        |       2 |
|          5 | SAV        |       2 |
|         17 | CD         |       7 |
|         27 | BUS        |      11 |
+------------+------------+---------+
7 rows in set (0.00 sec)
```

### 关联子查询

非关联子查询是通过将子查询中的语句执行完之后再执行包含查询语句；

关联子查询不是再包含语句执行前一次执行完毕，而是为每一个候选行执行一次；

```sql
select c.cust_id, c.cust_type_cd, c.city
from customer c
where 2 = (select count(*)
  from account a
  where a.cust_id = c.cust_id);
+---------+--------------+---------+
| cust_id | cust_type_cd | city    |
+---------+--------------+---------+
|       2 | I            | Woburn  |
|       3 | I            | Quincy  |
|       6 | I            | Waltham |
|       8 | I            | Salem   |
|      10 | B            | Salem   |
+---------+--------------+---------+
5 rows in set (0.00 sec)
```

这个的执行过程是：

子查询最后引用了c.cust_id, 使之具有关联性，这样他的执行必须依赖于包含查询提供的c.cust_id。在这种情况下，先从customer表中检索出13行客户记录，接着为每个客户执行一次子查询，每次执行包含查询都要想子查询传递客户ID，若子查询返回值2，则过滤条件满足，该行将被添加到结构集；



### exist 运算符

exist运算符检查子查询至少返回一行的结果；

select 1 的作用

```sql
mysql> select 1 from employee where assigned_branch_id  = 1;
+---+
| 1 |
+---+
| 1 |
| 1 |
| 1 |
| 1 |
| 1 |
| 1 |
| 1 |
| 1 |
| 1 |
+---+
9 rows in set (0.00 sec)
```

`select 1`的子查询语句会返回一列的`1`，这是因为有的查询语句只需要知道子查询返回的结果是多少行，而与结果的确切内容无关。

实际上读者可以让子查询语句返回任意的内容，但是通常来说会使用`select 1`或者 `select *`

使用 not exists

```sql
select a.account_id, a.product_cd, a.cust_id
from account a
where not exists (select 1
  from business b
  where b.cust_id = a.cust_id);
+------------+------------+---------+
| account_id | product_cd | cust_id |
+------------+------------+---------+
|          1 | CHK        |       1 |
|          2 | SAV        |       1 |
|          3 | CD         |       1 |
|          4 | CHK        |       2 |
|          5 | SAV        |       2 |
|          7 | CHK        |       3 |
|          8 | MM         |       3 |
|         10 | CHK        |       4 |
|         11 | SAV        |       4 |
|         12 | MM         |       4 |
|         13 | CHK        |       5 |
|         14 | CHK        |       6 |
|         15 | CD         |       6 |
|         17 | CD         |       7 |
|         18 | CHK        |       8 |
|         19 | SAV        |       8 |
|         21 | CHK        |       9 |
|         22 | MM         |       9 |
|         23 | CD         |       9 |
+------------+------------+---------+
19 rows in set (0.01 sec)  
```



### 何时使用子查询

1. 使用子查询作为数据源

```sql
select d.dept_id, d.name, e_cnt.how_many num_employees
from department d inner join
  (select dept_id, count(*) how_many
  from employee
  group by dept_id) e_cnt
on d.dept_id = e_cnt.dept_id;
+---------+----------------+---------------+
| dept_id | name           | num_employees |
+---------+----------------+---------------+
|       1 | Operations     |            14 |
|       2 | Loans          |             1 |
|       3 | Administration |             3 |
+---------+----------------+---------------+
3 rows in set (0.00 sec)
```

使用子查询可以构建超越基础表的数据源，具有非常大的灵活性；

2. 数据加工

```sql
select 'small fry' name, 0 low_limit, 4999.99 high_limit
union all
select 'average joes' name, 5000 low_limit, 9999.99 high_limit
union all
select 'heavy hitters' name, 10000 low_limit, 9999999.99 high_limit;
+---------------+-----------+------------+
| name          | low_limit | high_limit |
+---------------+-----------+------------+
| small fry     |         0 |    4999.99 |
| average joes  |      5000 |    9999.99 |
| heavy hitters |     10000 | 9999999.99 |
+---------------+-----------+------------+
3 rows in set (0.01 sec)
```

```sql
select groups.name, count(*) num_customers
from
  (select sum(a.avail_balance) cust_balance
  from account a inner join product p
    on a.product_cd = p.product_cd
  where p.product_type_cd = 'ACCOUNT'
  group by a.cust_id) cust_rollup
  inner join
  (select 'small fry' name, 0 low_limit, 4999.99 high_limit
	union all
	select 'average joes' name, 5000 low_limit, 9999.99 high_limit
	union all
	select 'heavy hitters' name, 10000 low_limit, 9999999.99 high_limit) groups
  on cust_rollup.cust_balance
    between groups.low_limit and groups.high_limit
group by groups.name;
+---------------+---------------+
| name          | num_customers |
+---------------+---------------+
| average joes  |             2 |
| heavy hitters |             4 |
| small fry     |             5 |
+---------------+---------------+
3 rows in set (0.00 sec)
```

### 面向任务的子查询

```sq
select p.name product, b.name branch,
  concat(e.fname, ' ', e.lname) name,
  sum(a.avail_balance) tot_deposits
from account a inner join employee e
  on a.open_emp_id = e.emp_id
  inner join branch b
  on a.open_branch_id = b.branch_id
  inner join product p
  on a.product_cd = p.product_cd
where p.product_type_cd = 'ACCOUNT'
group by p.name, b.name, e.fname, e.lname
order by 1, 2;
+------------------------+---------------+-----------------+--------------+
| product                | branch        | name            | tot_deposits |
+------------------------+---------------+-----------------+--------------+
| certificate of deposit | Headquarters  | Michael Smith   |     11500.00 |
| certificate of deposit | Woburn Branch | Paula Roberts   |      8000.00 |
| checking account       | Headquarters  | Michael Smith   |       782.16 |
| checking account       | Quincy Branch | John Blake      |      1057.75 |
| checking account       | So. NH Branch | Theresa Markham |     67852.33 |
| checking account       | Woburn Branch | Paula Roberts   |      3315.77 |
| money market account   | Headquarters  | Michael Smith   |     14832.64 |
| money market account   | Quincy Branch | John Blake      |      2212.50 |
| savings account        | Headquarters  | Michael Smith   |       767.77 |
| savings account        | So. NH Branch | Theresa Markham |       387.99 |
| savings account        | Woburn Branch | Paula Roberts   |       700.00 |
+------------------------+---------------+-----------------+--------------+
11 rows in set (0.04 sec)
```

2. ​

```sql
select product_cd, open_branch_id branch_id, open_emp_id emp_id, sum(avail_balance) tot_deposits
from account
group by product_cd, open_branch_id, open_emp_id;
+------------+-----------+--------+--------------+
| product_cd | branch_id | emp_id | tot_deposits |
+------------+-----------+--------+--------------+
| BUS        |         2 |     10 |      9345.55 |
| BUS        |         4 |     16 |         0.00 |
| CD         |         1 |      1 |     11500.00 |
| CD         |         2 |     10 |      8000.00 |
| CHK        |         1 |      1 |       782.16 |
| CHK        |         2 |     10 |      3315.77 |
| CHK        |         3 |     13 |      1057.75 |
| CHK        |         4 |     16 |     67852.33 |
| MM         |         1 |      1 |     14832.64 |
| MM         |         3 |     13 |      2212.50 |
| SAV        |         1 |      1 |       767.77 |
| SAV        |         2 |     10 |       700.00 |
| SAV        |         4 |     16 |       387.99 |
| SBL        |         3 |     13 |     50000.00 |
+------------+-----------+--------+--------------+
14 rows in set (0.00 sec)
```

3. ​

```sql
select p.name product, b.name branch, concat(e.fname, ' ', e.lname) name, account_groups.tot_deposits
from 
  (select product_cd, open_branch_id branch_id, open_emp_id emp_id, sum(avail_balance) tot_deposits
   from account
   group by product_cd, open_branch_id, open_emp_id) account_groups
   inner join employee e on e.emp_id = account_groups.emp_id
   inner join branch b on b.branch_id = account_groups.branch_id
   inner join product p on p.product_cd = account_groups.product_cd
where p.product_type_cd = 'ACCOUNT';
+------------------------+---------------+-----------------+--------------+
| product                | branch        | name            | tot_deposits |
+------------------------+---------------+-----------------+--------------+
| certificate of deposit | Headquarters  | Michael Smith   |     11500.00 |
| checking account       | Headquarters  | Michael Smith   |       782.16 |
| money market account   | Headquarters  | Michael Smith   |     14832.64 |
| savings account        | Headquarters  | Michael Smith   |       767.77 |
| certificate of deposit | Woburn Branch | Paula Roberts   |      8000.00 |
| checking account       | Woburn Branch | Paula Roberts   |      3315.77 |
| savings account        | Woburn Branch | Paula Roberts   |       700.00 |
| checking account       | Quincy Branch | John Blake      |      1057.75 |
| money market account   | Quincy Branch | John Blake      |      2212.50 |
| checking account       | So. NH Branch | Theresa Markham |     67852.33 |
| savings account        | So. NH Branch | Theresa Markham |       387.99 |
+------------------------+---------------+-----------------+--------------+
11 rows in set (0.00 sec)
```

> 话说第三个版本的执行起来更快，分组实现不是基于可能很长的字符串类型（branch.name, product.name, employee.fname, employee.lname), 而是基于更小而数字型的外键列(product_cd, open_branch_id, opem_emp_id).

### 过滤条件中的子查询

```sql
select open_emp_id, count(*) how_many
from account
group by open_emp_id
having count(*) = (select max(emp_cnt.how_many)
  from (select count(*) how_many
    from account group by open_emp_id) emp_cnt);  
+-------------+----------+
| open_emp_id | how_many |
+-------------+----------+
|           1 |        8 |
+-------------+----------+
1 row in set (0.01 sec)
```

### 子查询作为表达式生成器

```sql
select 
(select p.name from product p
  where p.product_cd = a.product_cd and p.product_type_cd = 'ACCOUNT') product,
(select b.name from branch b
  where b.branch_id = a.open_branch_id) branch,
(select concat(e.fname, ' ', e.lname) from employee e
  where e.emp_id = a.open_emp_id) name,
sum(a.avail_balance) tot_deposits
from account a
group by a.product_cd, a.open_branch_id, a.open_emp_id
order by 1,2;
+------------------------+---------------+-----------------+--------------+
| product                | branch        | name            | tot_deposits |
+------------------------+---------------+-----------------+--------------+
| NULL                   | Quincy Branch | John Blake      |     50000.00 |
| NULL                   | So. NH Branch | Theresa Markham |         0.00 |
| NULL                   | Woburn Branch | Paula Roberts   |      9345.55 |
| certificate of deposit | Headquarters  | Michael Smith   |     11500.00 |
| certificate of deposit | Woburn Branch | Paula Roberts   |      8000.00 |
| checking account       | Headquarters  | Michael Smith   |       782.16 |
| checking account       | Quincy Branch | John Blake      |      1057.75 |
| checking account       | So. NH Branch | Theresa Markham |     67852.33 |
| checking account       | Woburn Branch | Paula Roberts   |      3315.77 |
| money market account   | Headquarters  | Michael Smith   |     14832.64 |
| money market account   | Quincy Branch | John Blake      |      2212.50 |
| savings account        | Headquarters  | Michael Smith   |       767.77 |
| savings account        | So. NH Branch | Theresa Markham |       387.99 |
| savings account        | Woburn Branch | Paula Roberts   |       700.00 |
+------------------------+---------------+-----------------+--------------+
14 rows in set (0.00 sec)
```

