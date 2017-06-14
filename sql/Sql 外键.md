# Sql 外键

外链接用 outer join 来表示，前面还有 left/right 来修饰，用来表示用哪边的表示左边/右边的数据将作为主要的数据维度来展示结果集，在另一边对的表中的信息将主要用来显示匹配列的额外数据；

以以下表为例：

account 表作为主维度，account中所有的数据都将显示，同样这样也就决定了结果集中的行数，business表仅用来显示一些额外的数据，如果有匹配的数据就显示，如果没有匹配的数据就显示NULL；

```sql
mysql> select a.account_id, a.cust_id, b.name from account a left outer join business b on a.cust_id = b.cust_id;
+------------+---------+------------------------+
| account_id | cust_id | name                   |
+------------+---------+------------------------+
|         24 |      10 | Chilton Engineering    |
|         25 |      10 | Chilton Engineering    |
|         27 |      11 | Northeast Cooling Inc. |
|         28 |      12 | Superior Auto Body     |
|         29 |      13 | AAA Insurance Inc.     |
|          1 |       1 | NULL                   |
|          2 |       1 | NULL                   |
|          3 |       1 | NULL                   |
|          4 |       2 | NULL                   |
|          5 |       2 | NULL                   |
|          7 |       3 | NULL                   |
|          8 |       3 | NULL                   |
|         10 |       4 | NULL                   |
|         11 |       4 | NULL                   |
|         12 |       4 | NULL                   |
|         13 |       5 | NULL                   |
|         14 |       6 | NULL                   |
|         15 |       6 | NULL                   |
|         17 |       7 | NULL                   |
|         18 |       8 | NULL                   |
|         19 |       8 | NULL                   |
|         21 |       9 | NULL                   |
|         22 |       9 | NULL                   |
|         23 |       9 | NULL                   |
+------------+---------+------------------------+
24 rows in set (0.08 sec)
```

三表外联

```sql
mysql> 
select a.account_id, a.product_cd, concat(i.fname, ' ', i.lname) person_name, b.name business_name 
from account a left outer join individual i 
on a.cust_id = i.cust_id 
left outer join business b 
on a.cust_id = b.cust_id;
+------------+------------+-----------------+------------------------+
| account_id | product_cd | person_name     | business_name          |
+------------+------------+-----------------+------------------------+
|         24 | CHK        | NULL            | Chilton Engineering    |
|         25 | BUS        | NULL            | Chilton Engineering    |
|         27 | BUS        | NULL            | Northeast Cooling Inc. |
|         28 | CHK        | NULL            | Superior Auto Body     |
|         29 | SBL        | NULL            | AAA Insurance Inc.     |
|          1 | CHK        | James Hadley    | NULL                   |
|          2 | SAV        | James Hadley    | NULL                   |
|          3 | CD         | James Hadley    | NULL                   |
|          4 | CHK        | Susan Tingley   | NULL                   |
|          5 | SAV        | Susan Tingley   | NULL                   |
|          7 | CHK        | Frank Tucker    | NULL                   |
|          8 | MM         | Frank Tucker    | NULL                   |
|         10 | CHK        | John Hayward    | NULL                   |
|         11 | SAV        | John Hayward    | NULL                   |
|         12 | MM         | John Hayward    | NULL                   |
|         13 | CHK        | Charles Frasier | NULL                   |
|         14 | CHK        | John Spencer    | NULL                   |
|         15 | CD         | John Spencer    | NULL                   |
|         17 | CD         | Margaret Young  | NULL                   |
|         18 | CHK        | George Blake    | NULL                   |
|         19 | SAV        | George Blake    | NULL                   |
|         21 | CHK        | Richard Farley  | NULL                   |
|         22 | MM         | Richard Farley  | NULL                   |
|         23 | CD         | Richard Farley  | NULL                   |
+------------+------------+-----------------+------------------------+
24 rows in set (0.00 sec)
```

### 外链接和子查询的结合

如果不知道一个数据库对外链接到同一个表的其他表的数目是否有限制，我们可以使用子查询来限制查询中连接的数目。

```sql
select account_ind.account_id, account_ind.product_cd, account_ind.person_name, b.name business_name
from 
  (select a.account_id, a.product_cd, a.cust_id, concat(i.fname, ' ', i.lname) person_name      from account a left outer join individual i
     on a.cust_id = i.cust_id
  ) account_ind 
   left outer join business b
   on account_ind.cust_id = b.cust_id;
```

```sql
+------------+------------+-----------------+------------------------+
| account_id | product_cd | person_name     | business_name          |
+------------+------------+-----------------+------------------------+
|         24 | CHK        | NULL            | Chilton Engineering    |
|         25 | BUS        | NULL            | Chilton Engineering    |
|         27 | BUS        | NULL            | Northeast Cooling Inc. |
|         28 | CHK        | NULL            | Superior Auto Body     |
|         29 | SBL        | NULL            | AAA Insurance Inc.     |
|          1 | CHK        | James Hadley    | NULL                   |
|          2 | SAV        | James Hadley    | NULL                   |
|          3 | CD         | James Hadley    | NULL                   |
|          4 | CHK        | Susan Tingley   | NULL                   |
|          5 | SAV        | Susan Tingley   | NULL                   |
|          7 | CHK        | Frank Tucker    | NULL                   |
|          8 | MM         | Frank Tucker    | NULL                   |
|         10 | CHK        | John Hayward    | NULL                   |
|         11 | SAV        | John Hayward    | NULL                   |
|         12 | MM         | John Hayward    | NULL                   |
|         13 | CHK        | Charles Frasier | NULL                   |
|         14 | CHK        | John Spencer    | NULL                   |
|         15 | CD         | John Spencer    | NULL                   |
|         17 | CD         | Margaret Young  | NULL                   |
|         18 | CHK        | George Blake    | NULL                   |
|         19 | SAV        | George Blake    | NULL                   |
|         21 | CHK        | Richard Farley  | NULL                   |
|         22 | MM         | Richard Farley  | NULL                   |
|         23 | CD         | Richard Farley  | NULL                   |
+------------+------------+-----------------+------------------------+
24 rows in set (0.00 sec)
```

### 自外连接

```sql
select e.fname, e.lname, e_mgr.fname mgr_fname, e_mgr.lname mgr_lname 
from employee e left outer join employee e_mgr
on e.superior_emp_id = e_mgr.emp_id;
+----------+-----------+-----------+-----------+
| fname    | lname     | mgr_fname | mgr_lname |
+----------+-----------+-----------+-----------+
| Michael  | Smith     | NULL      | NULL      |
| Susan    | Barker    | Michael   | Smith     |
| Robert   | Tyler     | Michael   | Smith     |
| Susan    | Hawthorne | Robert    | Tyler     |
| John     | Gooding   | Susan     | Hawthorne |
| Helen    | Fleming   | Susan     | Hawthorne |
| Chris    | Tucker    | Helen     | Fleming   |
| Sarah    | Parker    | Helen     | Fleming   |
| Jane     | Grossman  | Helen     | Fleming   |
| Paula    | Roberts   | Susan     | Hawthorne |
| Thomas   | Ziegler   | Paula     | Roberts   |
| Samantha | Jameson   | Paula     | Roberts   |
| John     | Blake     | Susan     | Hawthorne |
| Cindy    | Mason     | John      | Blake     |
| Frank    | Portman   | John      | Blake     |
| Theresa  | Markham   | Susan     | Hawthorne |
| Beth     | Fowler    | Theresa   | Markham   |
| Rick     | Tulman    | Theresa   | Markham   |
+----------+-----------+-----------+-----------+
18 rows in set (0.01 sec)
```

看一下换成内连接后的结果：

```sql
mysql> select e.fname, e.lname, e_mgr.fname mgr_fname, e_mgr.lname mgr_lname  from employee e inner join employee e_mgr on e.superior_emp_id = e_mgr.emp_id;
+----------+-----------+-----------+-----------+
| fname    | lname     | mgr_fname | mgr_lname |
+----------+-----------+-----------+-----------+
| Susan    | Barker    | Michael   | Smith     |
| Robert   | Tyler     | Michael   | Smith     |
| Susan    | Hawthorne | Robert    | Tyler     |
| John     | Gooding   | Susan     | Hawthorne |
| Helen    | Fleming   | Susan     | Hawthorne |
| Chris    | Tucker    | Helen     | Fleming   |
| Sarah    | Parker    | Helen     | Fleming   |
| Jane     | Grossman  | Helen     | Fleming   |
| Paula    | Roberts   | Susan     | Hawthorne |
| Thomas   | Ziegler   | Paula     | Roberts   |
| Samantha | Jameson   | Paula     | Roberts   |
| John     | Blake     | Susan     | Hawthorne |
| Cindy    | Mason     | John      | Blake     |
| Frank    | Portman   | John      | Blake     |
| Theresa  | Markham   | Susan     | Hawthorne |
| Beth     | Fowler    | Theresa   | Markham   |
| Rick     | Tulman    | Theresa   | Markham   |
+----------+-----------+-----------+-----------+
17 rows in set (0.00 sec)
#没有主管的员工将被忽略；
```

换成right outer join

```sql
mysql> select e.fname, e.lname, e_mgr.fname mgr_fname, e_mgr.lname mgr_lname  from employee e right outer join employee 
+----------+-----------+-----------+-----------+
| fname    | lname     | mgr_fname | mgr_lname |
+----------+-----------+-----------+-----------+
| Susan    | Barker    | Michael   | Smith     |
| Robert   | Tyler     | Michael   | Smith     |
| NULL     | NULL      | Susan     | Barker    |
| Susan    | Hawthorne | Robert    | Tyler     |
| John     | Gooding   | Susan     | Hawthorne |
| Helen    | Fleming   | Susan     | Hawthorne |
| Paula    | Roberts   | Susan     | Hawthorne |
| John     | Blake     | Susan     | Hawthorne |
| Theresa  | Markham   | Susan     | Hawthorne |
| NULL     | NULL      | John      | Gooding   |
| Chris    | Tucker    | Helen     | Fleming   |
| Sarah    | Parker    | Helen     | Fleming   |
| Jane     | Grossman  | Helen     | Fleming   |
| NULL     | NULL      | Chris     | Tucker    |
| NULL     | NULL      | Sarah     | Parker    |
| NULL     | NULL      | Jane      | Grossman  |
| Thomas   | Ziegler   | Paula     | Roberts   |
| Samantha | Jameson   | Paula     | Roberts   |
| NULL     | NULL      | Thomas    | Ziegler   |
| NULL     | NULL      | Samantha  | Jameson   |
| Cindy    | Mason     | John      | Blake     |
| Frank    | Portman   | John      | Blake     |
| NULL     | NULL      | Cindy     | Mason     |
| NULL     | NULL      | Frank     | Portman   |
| Beth     | Fowler    | Theresa   | Markham   |
| Rick     | Tulman    | Theresa   | Markham   |
| NULL     | NULL      | Beth      | Fowler    |
| NULL     | NULL      | Rick      | Tulman    |
+----------+-----------+-----------+-----------+
28 rows in set (0.00 sec)
#换成右链接后将变成显示所有的主管及主管名下的员工；
```

## 外链接

外链接用来产生笛卡尔积，一般很少使用

```sql
mysql> select pt.name, p.product_cd, p.name from product p cross join product_type pt;
+-------------------------------+------------+-------------------------+
| name                          | product_cd | name                    |
+-------------------------------+------------+-------------------------+
| Customer Accounts             | AUT        | auto loan               |
| Insurance Offerings           | AUT        | auto loan               |
| Individual and Business Loans | AUT        | auto loan               |
| Customer Accounts             | BUS        | business line of credit |
| Insurance Offerings           | BUS        | business line of credit |
| Individual and Business Loans | BUS        | business line of credit |
| Customer Accounts             | CD         | certificate of deposit  |
| Insurance Offerings           | CD         | certificate of deposit  |
| Individual and Business Loans | CD         | certificate of deposit  |
| Customer Accounts             | CHK        | checking account        |
| Insurance Offerings           | CHK        | checking account        |
| Individual and Business Loans | CHK        | checking account        |
| Customer Accounts             | MM         | money market account    |
| Insurance Offerings           | MM         | money market account    |
| Individual and Business Loans | MM         | money market account    |
| Customer Accounts             | MRT        | home mortgage           |
| Insurance Offerings           | MRT        | home mortgage           |
| Individual and Business Loans | MRT        | home mortgage           |
| Customer Accounts             | SAV        | savings account         |
| Insurance Offerings           | SAV        | savings account         |
| Individual and Business Loans | SAV        | savings account         |
| Customer Accounts             | SBL        | small business loan     |
| Insurance Offerings           | SBL        | small business loan     |
| Individual and Business Loans | SBL        | small business loan     |
+-------------------------------+------------+-------------------------+
24 rows in set (0.00 sec)
# 8 x 3 = 24
```

## 自然链接

自然链接是在依赖夺标交叉时的相同列名来推断正确的连接条件

```sql
mysql> select a.account_id, a.cust_id, c.cust_type_cd, c.fed_id from account a natural join customer c;
+------------+---------+--------------+-------------+
| account_id | cust_id | cust_type_cd | fed_id      |
+------------+---------+--------------+-------------+
|          1 |       1 | I            | 111-11-1111 |
|          2 |       1 | I            | 111-11-1111 |
|          3 |       1 | I            | 111-11-1111 |
|          4 |       2 | I            | 222-22-2222 |
|          5 |       2 | I            | 222-22-2222 |
|          7 |       3 | I            | 333-33-3333 |
|          8 |       3 | I            | 333-33-3333 |
|         10 |       4 | I            | 444-44-4444 |
|         11 |       4 | I            | 444-44-4444 |
|         12 |       4 | I            | 444-44-4444 |
|         13 |       5 | I            | 555-55-5555 |
|         14 |       6 | I            | 666-66-6666 |
|         15 |       6 | I            | 666-66-6666 |
|         17 |       7 | I            | 777-77-7777 |
|         18 |       8 | I            | 888-88-8888 |
|         19 |       8 | I            | 888-88-8888 |
|         21 |       9 | I            | 999-99-9999 |
|         22 |       9 | I            | 999-99-9999 |
|         23 |       9 | I            | 999-99-9999 |
|         24 |      10 | B            | 04-1111111  |
|         25 |      10 | B            | 04-1111111  |
|         27 |      11 | B            | 04-2222222  |
|         28 |      12 | B            | 04-3333333  |
|         29 |      13 | B            | 04-4444444  |
+------------+---------+--------------+-------------+
24 rows in set (0.00 sec)
```



