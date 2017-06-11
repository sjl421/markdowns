# SQL 查询入门

```sql
MariaDB [lsql]> select emp_id, 'ACTIVE', emp_id * 3.14159, UPPER(lname) from employee;
+--------+--------+------------------+--------------+
| emp_id | ACTIVE | emp_id * 3.14159 | UPPER(lname) |
+--------+--------+------------------+--------------+
|      1 | ACTIVE |          3.14159 | SMITH        |
|      2 | ACTIVE |          6.28318 | BARKER       |
|      3 | ACTIVE |          9.42477 | TYLER        |
|      4 | ACTIVE |         12.56636 | HAWTHORNE    |
|      5 | ACTIVE |         15.70795 | GOODING      |
|      6 | ACTIVE |         18.84954 | FLEMING      |
|      7 | ACTIVE |         21.99113 | TUCKER       |
|      8 | ACTIVE |         25.13272 | PARKER       |
|      9 | ACTIVE |         28.27431 | GROSSMAN     |
|     10 | ACTIVE |         31.41590 | ROBERTS      |
|     11 | ACTIVE |         34.55749 | ZIEGLER      |
|     12 | ACTIVE |         37.69908 | JAMESON      |
|     13 | ACTIVE |         40.84067 | BLAKE        |
|     14 | ACTIVE |         43.98226 | MASON        |
|     15 | ACTIVE |         47.12385 | PORTMAN      |
|     16 | ACTIVE |         50.26544 | MARKHAM      |
|     17 | ACTIVE |         53.40703 | FOWLER       |
|     18 | ACTIVE |         56.54862 | TULMAN       |
+--------+--------+------------------+--------------+
18 rows in set (0.00 sec)
```



## from 子语句

```sql
MariaDB [lsql]> select e.emp_id, e.fname, e.lname from (select emp_id, fname, lname, start_date, title from employee) e;
+--------+----------+-----------+
| emp_id | fname    | lname     |
+--------+----------+-----------+
|      1 | Michael  | Smith     |
|      2 | Susan    | Barker    |
|      3 | Robert   | Tyler     |
|      4 | Susan    | Hawthorne |
|      5 | John     | Gooding   |
|      6 | Helen    | Fleming   |
|      7 | Chris    | Tucker    |
|      8 | Sarah    | Parker    |
|      9 | Jane     | Grossman  |
|     10 | Paula    | Roberts   |
|     11 | Thomas   | Ziegler   |
|     12 | Samantha | Jameson   |
|     13 | John     | Blake     |
|     14 | Cindy    | Mason     |
|     15 | Frank    | Portman   |
|     16 | Theresa  | Markham   |
|     17 | Beth     | Fowler    |
|     18 | Rick     | Tulman    |
+--------+----------+-----------+
18 rows in set (0.00 sec)
```

## 表连接

```sql
MariaDB [lsql]> select employee.emp_id, employee.fname, employee.lname, department.name  dept_name from employee join department on employee.dept_id = department.dept_id;
+--------+----------+-----------+----------------+
| emp_id | fname    | lname     | dept_name      |
+--------+----------+-----------+----------------+
|      4 | Susan    | Hawthorne | Operations     |
|      6 | Helen    | Fleming   | Operations     |
|      7 | Chris    | Tucker    | Operations     |
|      8 | Sarah    | Parker    | Operations     |
|      9 | Jane     | Grossman  | Operations     |
|     10 | Paula    | Roberts   | Operations     |
|     11 | Thomas   | Ziegler   | Operations     |
|     12 | Samantha | Jameson   | Operations     |
|     13 | John     | Blake     | Operations     |
|     14 | Cindy    | Mason     | Operations     |
|     15 | Frank    | Portman   | Operations     |
|     16 | Theresa  | Markham   | Operations     |
|     17 | Beth     | Fowler    | Operations     |
|     18 | Rick     | Tulman    | Operations     |
|      5 | John     | Gooding   | Loans          |
|      1 | Michael  | Smith     | Administration |
|      2 | Susan    | Barker    | Administration |
|      3 | Robert   | Tyler     | Administration |
+--------+----------+-----------+----------------+
18 rows in set (0.00 sec)
```



**出问题的情况**

```sql
MariaDB [lsql]> select e.emp_id, d.name from employee e join department d;
+--------+----------------+
| emp_id | name           |
+--------+----------------+
|      1 | Operations     |
|      1 | Loans          |
|      1 | Administration |
|      2 | Operations     |
|      2 | Loans          |
|      2 | Administration |
|      3 | Operations     |
|      3 | Loans          |
|      3 | Administration |
|      4 | Operations     |
|      4 | Loans          |
|      4 | Administration |
|      5 | Operations     |
|      5 | Loans          |
|      5 | Administration |
|      6 | Operations     |
|      6 | Loans          |
|      6 | Administration |
|      7 | Operations     |
|      7 | Loans          |
|      7 | Administration |
|      8 | Operations     |
|      8 | Loans          |
|      8 | Administration |
|      9 | Operations     |
|      9 | Loans          |
|      9 | Administration |
|     10 | Operations     |
|     10 | Loans          |
|     10 | Administration |
|     11 | Operations     |
|     11 | Loans          |
|     11 | Administration |
|     12 | Operations     |
|     12 | Loans          |
|     12 | Administration |
|     13 | Operations     |
|     13 | Loans          |
|     13 | Administration |
|     14 | Operations     |
|     14 | Loans          |
|     14 | Administration |
|     15 | Operations     |
|     15 | Loans          |
|     15 | Administration |
|     16 | Operations     |
|     16 | Loans          |
|     16 | Administration |
|     17 | Operations     |
|     17 | Loans          |
|     17 | Administration |
|     18 | Operations     |
|     18 | Loans          |
|     18 | Administration |
+--------+----------------+
54 rows in set (0.00 sec)
```

这里的emp_id列都有三行

```sql
MariaDB [lsql]> select emp_id from employee;
+--------+
| emp_id |
+--------+
|      1 |
|      2 |
|      3 |
|      4 |
|      5 |
|      6 |
|     10 |
|     13 |
|     16 |
|      7 |
|      8 |
|      9 |
|     11 |
|     12 |
|     14 |
|     15 |
|     17 |
|     18 |
+--------+
18 rows in set (0.00 sec)
```

```sql
MariaDB [lsql]> select name from department;
+----------------+
| name           |
+----------------+
| Operations     |
| Loans          |
| Administration |
+----------------+
```

这里的这种情况叫做笛卡尔积，18（查询2的结果集） * 3（查询三的结果集） = 54（查询一的结果集）

```sql
MariaDB [lsql]> select emp_id, fname, lname, start_date from employee where title='Head Teller' and start_date > '2006-01-01';
+--------+-------+---------+------------+
| emp_id | fname | lname   | start_date |
+--------+-------+---------+------------+
|      6 | Helen | Fleming | 2008-03-17 |
|     10 | Paula | Roberts | 2006-07-27 |
+--------+-------+---------+------------+
2 rows in set (0.00 sec)
```

```sql
MariaDB [lsql]> select emp_id, fname, lname, start_date, title from employee where (title='Head Teller' and start_date > '2006-01-01') or (title='Teller' and start_date > '2007-01-01');
+--------+----------+---------+------------+-------------+
| emp_id | fname    | lname   | start_date | title       |
+--------+----------+---------+------------+-------------+
|      6 | Helen    | Fleming | 2008-03-17 | Head Teller |
|      7 | Chris    | Tucker  | 2008-09-15 | Teller      |
|     10 | Paula    | Roberts | 2006-07-27 | Head Teller |
|     12 | Samantha | Jameson | 2007-01-08 | Teller      |
|     15 | Frank    | Portman | 2007-04-01 | Teller      |
+--------+----------+---------+------------+-------------+
5 rows in set (0.00 sec)
#在指定复杂的过滤条件时需要用（）将不同的条件区分开；
```



### group by 和 having 语句

1. ​

```sql
MariaDB [lsql]> select  d.name, count(e.emp_id) num_employee from department d inner join employee e on d.dept_id = e.dept_id  group by d.name;
+----------------+--------------+
| name           | num_employee |
+----------------+--------------+
| Administration |            3 |
| Loans          |            1 |
| Operations     |           14 |
+----------------+--------------+
```

2. ​

```sql
MariaDB [lsql]> select  d.name, count(e.emp_id) num_employee from department d inner join employee e on d.dept_id = e.dept_id  group by d.name having count(e.emp_id) > 2;
+----------------+--------------+
| name           | num_employee |
+----------------+--------------+
| Administration |            3 |
| Operations     |           14 |
+----------------+--------------+
2 rows in set (0.00 sec)
# 使用having 子语句在group by 的基础上进一步的过滤
```

### order by

order by col1, col2 [asc/desc] ,

order by 后面可以是 一列或者多列，也可以是内建函数；

默认是升序 asc， 使用desc 关键字后是降序的排列；

```sql
MariaDB [lsql]> select open_emp_id, product_cd from account order by open_emp_id;
+-------------+------------+
| open_emp_id | product_cd |
+-------------+------------+
|           1 | CD         |
|           1 | MM         |
|           1 | SAV        |
|           1 | CHK        |
|           1 | CHK        |
|           1 | CHK        |
|           1 | MM         |
|           1 | CD         |
|          10 | CHK        |
|          10 | BUS        |
|          10 | CD         |
|          10 | CHK        |
|          10 | SAV        |
|          10 | CD         |
|          10 | SAV        |
|          13 | SBL        |
|          13 | CHK        |
|          13 | MM         |
|          16 | SAV        |
|          16 | CHK        |
|          16 | CHK        |
|          16 | BUS        |
|          16 | CHK        |
|          16 | CHK        |
+-------------+------------+
24 rows in set (0.00 sec)
```

```sql
MariaDB [lsql]> select open_emp_id, product_cd from account order by open_emp_id, product_cd;
+-------------+------------+
| open_emp_id | product_cd |
+-------------+------------+
|           1 | CD         |
|           1 | CD         |
|           1 | CHK        |
|           1 | CHK        |
|           1 | CHK        |
|           1 | MM         |
|           1 | MM         |
|           1 | SAV        |
|          10 | BUS        |
|          10 | CD         |
|          10 | CD         |
|          10 | CHK        |
|          10 | CHK        |
|          10 | SAV        |
|          10 | SAV        |
|          13 | CHK        |
|          13 | MM         |
|          13 | SBL        |
|          16 | BUS        |
|          16 | CHK        |
|          16 | CHK        |
|          16 | CHK        |
|          16 | CHK        |
|          16 | SAV        |
+-------------+------------+
24 rows in set (0.00 sec)
```

### 根据表达式来排序

```sql
MariaDB [lsql]> select cust_id, cust_type_cd, city, state, fed_id from customer order by right(fed_id,3);
+---------+--------------+------------+-------+-------------+
| cust_id | cust_type_cd | city       | state | fed_id      |
+---------+--------------+------------+-------+-------------+
|       1 | I            | Lynnfield  | MA    | 111-11-1111 |
|      10 | B            | Salem      | NH    | 04-1111111  |
|       2 | I            | Woburn     | MA    | 222-22-2222 |
|      11 | B            | Wilmington | MA    | 04-2222222  |
|       3 | I            | Quincy     | MA    | 333-33-3333 |
|      12 | B            | Salem      | NH    | 04-3333333  |
|      13 | B            | Quincy     | MA    | 04-4444444  |
|       4 | I            | Waltham    | MA    | 444-44-4444 |
|       5 | I            | Salem      | NH    | 555-55-5555 |
|       6 | I            | Waltham    | MA    | 666-66-6666 |
|       7 | I            | Wilmington | MA    | 777-77-7777 |
|       8 | I            | Salem      | NH    | 888-88-8888 |
|       9 | I            | Newton     | MA    | 999-99-9999 |
+---------+--------------+------------+-------+-------------+
# right() 内建函数是根据的给定字符串的最后的三个字符来排序；
```

### 根据占位符来排序

```sql
MariaDB [lsql]> select emp_id, title, start_date, fname, lname from employee order by 2,5;
+--------+--------------------+------------+----------+-----------+
| emp_id | title              | start_date | fname    | lname     |
+--------+--------------------+------------+----------+-----------+
|     13 | Head Teller        | 2004-05-11 | John     | Blake     |
|      6 | Head Teller        | 2008-03-17 | Helen    | Fleming   |
|     16 | Head Teller        | 2005-03-15 | Theresa  | Markham   |
|     10 | Head Teller        | 2006-07-27 | Paula    | Roberts   |
|      5 | Loan Manager       | 2007-11-14 | John     | Gooding   |
|      4 | Operations Manager | 2006-04-24 | Susan    | Hawthorne |
|      1 | President          | 2005-06-22 | Michael  | Smith     |
|     17 | Teller             | 2006-06-29 | Beth     | Fowler    |
|      9 | Teller             | 2006-05-03 | Jane     | Grossman  |
|     12 | Teller             | 2007-01-08 | Samantha | Jameson   |
|     14 | Teller             | 2006-08-09 | Cindy    | Mason     |
|      8 | Teller             | 2006-12-02 | Sarah    | Parker    |
|     15 | Teller             | 2007-04-01 | Frank    | Portman   |
|      7 | Teller             | 2008-09-15 | Chris    | Tucker    |
|     18 | Teller             | 2006-12-12 | Rick     | Tulman    |
|     11 | Teller             | 2004-10-23 | Thomas   | Ziegler   |
|      3 | Treasurer          | 2005-02-09 | Robert   | Tyler     |
|      2 | Vice President     | 2006-09-12 | Susan    | Barker    |
+--------+--------------------+------------+----------+-----------+
18 rows in set (0.00 sec)
# 根据占位符来排序和使用使用指定的字段名来排序是一样的，这里只是把字段名改成了占位符而已；
```

