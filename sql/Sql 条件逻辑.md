# Sql 条件逻辑

条件逻辑就是程序执行时从多个路径中选取其一的能力。例如，查询客户信息时，读者可能希望根据客户类型的不同从不同的地方检测出客户的信息；

```sql
MariaDB [lsql]> select c.cust_id, c.fed_id, c.cust_type_cd, concat(i.fname, ' ', i.lname) indiv_name, b.name business_name from customer c left join individual i on c.cust_id = i.cust_id left join business b on c.cust_id = b.cust_id;
+---------+-------------+--------------+-----------------+------------------------+
| cust_id | fed_id      | cust_type_cd | indiv_name      | business_name          |
+---------+-------------+--------------+-----------------+------------------------+
|       1 | 111-11-1111 | I            | James Hadley    | NULL                   |
|       2 | 222-22-2222 | I            | Susan Tingley   | NULL                   |
|       3 | 333-33-3333 | I            | Frank Tucker    | NULL                   |
|       4 | 444-44-4444 | I            | John Hayward    | NULL                   |
|       5 | 555-55-5555 | I            | Charles Frasier | NULL                   |
|       6 | 666-66-6666 | I            | John Spencer    | NULL                   |
|       7 | 777-77-7777 | I            | Margaret Young  | NULL                   |
|       8 | 888-88-8888 | I            | George Blake    | NULL                   |
|       9 | 999-99-9999 | I            | Richard Farley  | NULL                   |
|      10 | 04-1111111  | B            | NULL            | Chilton Engineering    |
|      11 | 04-2222222  | B            | NULL            | Northeast Cooling Inc. |
|      12 | 04-3333333  | B            | NULL            | Superior Auto Body     |
|      13 | 04-4444444  | B            | NULL            | AAA Insurance Inc.     |
+---------+-------------+--------------+-----------------+------------------------+
```

一种方法就是读者先看cust_type_cd列的值,然后距诶的那个是使用indiv_name还是business_name列的值.

还有一种做法就是可以使用case 表达式使用条件逻辑决定客户类型.

```sql
select c.cust_id, c.fed_id, 
  case 
    when c.cust_type_cd = 'I'
      then concat(i.fname, ' ', i.lname)
    when c.cust_type_cd = 'B'
      then b.name
    else 'Unknown'
  end name
from customer c left outer join individual i
  on c.cust_id = i.cust_id
  left outer join  business b
  on c.cust_id = b.cust_id;
+---------+-------------+------------------------+
| cust_id | fed_id      | name                   |
+---------+-------------+------------------------+
|       1 | 111-11-1111 | James Hadley           |
|       2 | 222-22-2222 | Susan Tingley          |
|       3 | 333-33-3333 | Frank Tucker           |
|       4 | 444-44-4444 | John Hayward           |
|       5 | 555-55-5555 | Charles Frasier        |
|       6 | 666-66-6666 | John Spencer           |
|       7 | 777-77-7777 | Margaret Young         |
|       8 | 888-88-8888 | George Blake           |
|       9 | 999-99-9999 | Richard Farley         |
|      10 | 04-1111111  | Chilton Engineering    |
|      11 | 04-2222222  | Northeast Cooling Inc. |
|      12 | 04-3333333  | Superior Auto Body     |
|      13 | 04-4444444  | AAA Insurance Inc.     |
+---------+-------------+------------------------+
13 rows in set (0.00 sec)
#可以见case子表达式的角色也是一个字段而已,可以看成和普通的表中的字段是一样的角色,只不过这个角色的生成是需要通过某个逻辑进行处理的;
#同时需要注意的是case表达式前面和其他已经存在的字段是需要用',' 分割卡开的;
```

## 两种不同类型的case表达式

### 1. 查找型case表达式

```sql
case
  when c1 then e1
  when c2 then e2
  ...
  when cn then en
  else ed
end
c1,c2,c3...都是表征的条件,e1,e2,e3...都是表征的返回表达式,剩下的就是 else 和 ed
```

```sql
select c.cust_id, c.fed_id,
  case 
    when c.cust_type_cd = 'I' then
      (select concat(i.fname, ' ', i.lname)
       from individual i
       where i.cust_id = c.cust_id)
    when c.cust_type_cd = 'B' then
      (select b.name
       from business b
       where b.cust_id = c.cust_id)
    else 'Unknown'
  end name
from customer c;
+---------+-------------+------------------------+
| cust_id | fed_id      | name                   |
+---------+-------------+------------------------+
|       1 | 111-11-1111 | James Hadley           |
|       2 | 222-22-2222 | Susan Tingley          |
|       3 | 333-33-3333 | Frank Tucker           |
|       4 | 444-44-4444 | John Hayward           |
|       5 | 555-55-5555 | Charles Frasier        |
|       6 | 666-66-6666 | John Spencer           |
|       7 | 777-77-7777 | Margaret Young         |
|       8 | 888-88-8888 | George Blake           |
|       9 | 999-99-9999 | Richard Farley         |
|      10 | 04-1111111  | Chilton Engineering    |
|      11 | 04-2222222  | Northeast Cooling Inc. |
|      12 | 04-3333333  | Superior Auto Body     |
|      13 | 04-4444444  | AAA Insurance Inc.     |
+---------+-------------+------------------------+
```

### 简单case表达式

```sql
case v0
 when v1 then e1
 when v2 then e2
 ...
 when vn then en
 else ed
end 
```

v0代表的一个值,符号v1,v2,...vn 代表要和v0比较的值,符号e1,e2,...en代表case表达式要返回的表达式, ed 代表前面所有的条件表达式不满足时返回的结果;

```sql
case customer.cust_type_cd
  when 'I' then
    (select concat(i.fname, ' ', i.lname)
     from individual i
     where i.cust_id = customer.cust_id)
  when 'B' then
    (select b.name 
     from business b
     where b.cust_id = customer.cust_id)
  else 'Unknown Customer Type'
end
```

简单case表达式理解起来比较简单,但同时也却分灵活性;

一般来说简单case表达式是可以用前面所讲到的查询case表达式来达到相同的结果的;

```sql
case 
  when customer.cust_type_cd = 'I' then
    (select concat(i.fname, ' ', i.lname)
     from individual i
     where i.cust_id = customer.cust_id)
  when customer.cust_type_cd = 'B' then 
    (select b.name 
     from business b
     where b.cust_id = customer.cust_id)
  else 'Unknown Customer Type'
end
```

利用查找型case表达式可以构建范围条件,不等式条件以及基于and/or/not这些运算符的符合条件;

### case表达式范例

1. 结果集变更

```sql
select year(open_date) year, count(*) how_many
from account
where open_date > '1999-12-31'
  and open_date < '2006-01-01'
group by year(open_date);
+------+----------+
| year | how_many |
+------+----------+
| 2000 |        3 |
| 2001 |        4 |
| 2002 |        5 |
| 2003 |        3 |
| 2004 |        9 |
+------+----------+
5 rows in set (0.00 sec)
```

将结果变更为列形式的:

```sql
select
  sum(case
        when extract(year from open_date) = 2000 then 1
        else 0
      end) year_2000,
  sum(case
        when extract(year from open_date) = 2001 then 1
        else 0
      end) year_2001,
  sum(case
        when extract(year from open_date) = 2002 then 1
        else 0
      end) year_2002,
  sum(case
        when extract(year from open_date) = 2003 then 1
        else 0
      end) year_2003,
  sum(case
        when extract(year from open_date) = 2004 then 1
        else 0
      end) year_2004,
  sum(case
        when extract(year from open_date) = 2005 then 1
        else 0
      end) year_2005
from account
where open_date > '1999-12-31' and open_date < '2006-01-01';
+-----------+-----------+-----------+-----------+-----------+-----------+
| year_2000 | year_2001 | year_2002 | year_2003 | year_2004 | year_2005 |
+-----------+-----------+-----------+-----------+-----------+-----------+
|         3 |         4 |         5 |         3 |         9 |         0 |
+-----------+-----------+-----------+-----------+-----------+-----------+
1 row in set (0.00 sec)
#其实这种也可以
select 
  (select expr1) name1, 
  (select expr2) namne2,
  (select expr3) name3,
....
```

2. 选择性聚合

```sql
select concat('ALERT! : Account #', a.account_id, ' Has Incorrect Balance!')
from account a
where (a.avail_balance, a.pending_balance) <>
  (select
    sum(case
          when t.funds_avail_date > CURRENT_TIMESTAMP()
            then 0
          when t.txn_type_cd = 'DBT'
            then t.amount * -1
          else t.amount
        end),
     sum(case
           when t.txn_type_cd = 'DBT'
             then t.amount * -1
           else t.amount
         end)
   from transaction t
   where t.account_id = a.account_id);
+-----------------------------------------------------------------------+
| concat('ALERT! : Account #', a.account_id, ' Has Incorrect Balance!') |
+-----------------------------------------------------------------------+
| ALERT! : Account #1 Has Incorrect Balance!                            |
| ALERT! : Account #2 Has Incorrect Balance!                            |
| ALERT! : Account #3 Has Incorrect Balance!                            |
| ALERT! : Account #4 Has Incorrect Balance!                            |
| ALERT! : Account #5 Has Incorrect Balance!                            |
| ALERT! : Account #7 Has Incorrect Balance!                            |
| ALERT! : Account #8 Has Incorrect Balance!                            |
| ALERT! : Account #10 Has Incorrect Balance!                           |
| ALERT! : Account #11 Has Incorrect Balance!                           |
| ALERT! : Account #12 Has Incorrect Balance!                           |
| ALERT! : Account #13 Has Incorrect Balance!                           |
| ALERT! : Account #14 Has Incorrect Balance!                           |
| ALERT! : Account #15 Has Incorrect Balance!                           |
| ALERT! : Account #17 Has Incorrect Balance!                           |
| ALERT! : Account #18 Has Incorrect Balance!                           |
| ALERT! : Account #19 Has Incorrect Balance!                           |
| ALERT! : Account #21 Has Incorrect Balance!                           |
| ALERT! : Account #22 Has Incorrect Balance!                           |
| ALERT! : Account #23 Has Incorrect Balance!                           |
| ALERT! : Account #24 Has Incorrect Balance!                           |
| ALERT! : Account #28 Has Incorrect Balance!                           |
+-----------------------------------------------------------------------+
21 rows in set (0.00 sec)
```

3.  存在性检查

```sql
select c.cust_id, c.fed_id, c.cust_type_cd,
  case
    when exists (
      select 1 from account a 
      where a.cust_id = c.cust_id and a.product_cd = 'CHK')
    then 'Y'
    else 'N'
  end has_checking,
  case
    when exists(
      select 1 from account a 
      where a.cust_id = c.cust_id and a.product_cd = 'SAV')
    then 'Y'
    else 'N'
  end has_saving
from customer c;
+---------+-------------+--------------+--------------+------------+
| cust_id | fed_id      | cust_type_cd | has_checking | has_saving |
+---------+-------------+--------------+--------------+------------+
|       1 | 111-11-1111 | I            | Y            | Y          |
|       2 | 222-22-2222 | I            | Y            | Y          |
|       3 | 333-33-3333 | I            | Y            | N          |
|       4 | 444-44-4444 | I            | Y            | Y          |
|       5 | 555-55-5555 | I            | Y            | N          |
|       6 | 666-66-6666 | I            | Y            | N          |
|       7 | 777-77-7777 | I            | N            | N          |
|       8 | 888-88-8888 | I            | Y            | Y          |
|       9 | 999-99-9999 | I            | Y            | N          |
|      10 | 04-1111111  | B            | Y            | N          |
|      11 | 04-2222222  | B            | N            | N          |
|      12 | 04-3333333  | B            | Y            | N          |
|      13 | 04-4444444  | B            | N            | N          |
+---------+-------------+--------------+--------------+------------+
13 rows in set (0.00 sec)
```

```sql
select c.cust_id, c.fed_id, c.cust_type_cd, 
  case (
    select count(*) from account a
    where a.cust_id = c.cust_id)
  when 0 then 'None'
  when 1 then '1'
  when 2 then '2'
  else '3+'
  end num_accounts
from customer c;  

+---------+-------------+--------------+--------------+
| cust_id | fed_id      | cust_type_cd | num_accounts |
+---------+-------------+--------------+--------------+
|       1 | 111-11-1111 | I            | 3+           |
|       2 | 222-22-2222 | I            | 2            |
|       3 | 333-33-3333 | I            | 2            |
|       4 | 444-44-4444 | I            | 3+           |
|       5 | 555-55-5555 | I            | 1            |
|       6 | 666-66-6666 | I            | 2            |
|       7 | 777-77-7777 | I            | 1            |
|       8 | 888-88-8888 | I            | 2            |
|       9 | 999-99-9999 | I            | 3+           |
|      10 | 04-1111111  | B            | 2            |
|      11 | 04-2222222  | B            | 1            |
|      12 | 04-3333333  | B            | 1            |
|      13 | 04-4444444  | B            | 1            |
+---------+-------------+--------------+--------------+
13 rows in set (0.00 sec)
```

除0错误

在mysql中做除法运算,如果分母为0, mysql 的返回结果为NULL

```sql
MariaDB [lsql]> select 100 / 0;
+---------+
| 100 / 0 |
+---------+
|    NULL |
+---------+
1 row in set (0.00 sec)
```

为了避免这种情况的发生,最好是将分母放到case语句中做判断;

```sql
select a.cust_id, a.product_cd, a.avail_balance / 
  case
    when prod_tots.tot_balance = 0 then 1
    else prod_tots.tot_balance
  end precent_of_tatal
from account a inner join 
  (select a.product_cd, sum(a.avail_balance) tot_balance
    from account a 
    group by a.product_cd) prod_tots
    on a.product_cd = prod_tots.product_cd;
+---------+------------+------------------+
| cust_id | product_cd | precent_of_tatal |
+---------+------------+------------------+
|       1 | CHK        |         0.014488 |
|       1 | SAV        |         0.269431 |
|       1 | CD         |         0.153846 |
|       2 | CHK        |         0.030928 |
|       2 | SAV        |         0.107773 |
|       3 | CHK        |         0.014488 |
|       3 | MM         |         0.129802 |
|       4 | CHK        |         0.007316 |
|       4 | SAV        |         0.413723 |
|       4 | MM         |         0.321915 |
|       5 | CHK        |         0.030654 |
|       6 | CHK        |         0.001676 |
|       6 | CD         |         0.512821 |
|       7 | CD         |         0.256410 |
|       8 | CHK        |         0.047764 |
|       8 | SAV        |         0.209073 |
|       9 | CHK        |         0.001721 |
|       9 | MM         |         0.548282 |
|       9 | CD         |         0.076923 |
|      10 | CHK        |         0.322911 |
|      10 | BUS        |         0.000000 |
|      11 | BUS        |         1.000000 |
|      12 | CHK        |         0.528052 |
|      13 | SBL        |         1.000000 |
+---------+------------+------------------+
24 rows in set (0.00 sec)
```

有条件的更新

```sql
update account set
  last_activity_date = CURRENT_TIMESTAMP(),
  pending_balance = pending_balance + 
    (select t.amount * 
      case t.txn_type_cd when 'DBT' then -1 else 1 end
     from transaction t
     where t.txn_id = 999),
   avail_balance = avail_balance + 
     (select 
       case
         when t.funds_avail_date > current_timestamp() then 0
         else t.amount * 
           case t.txn_type_cd when 'DBT' then -1 else 1 end
       end
      from transaction t
      where t.txn_id = 999)
  where account.account_id =
    (select t.account_id 
      from transaction t
      where t.txn_id = 999);
```

null 值的处理

```sql
select emp_id, fname, lname,
  case
    when tile is null then 'Unknown'
    else title
  end
from employee
```

