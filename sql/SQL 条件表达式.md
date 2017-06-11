# SQL 条件表达式

在sql的过滤条件中很多时候都要使用条件表达式；



```
IS NULL; 
NOT;
AND;
OR;
```

相等条件

```
=
```

不等条件

```
!=
<>
```

范围表达式

```
<
>
<=
>=
between val1 and val2
```

成员条件

```
可以使用 or  构成成员条件（但是这种方法并不推荐）
比较推荐的是使用 in 关键字；
```

匹配条件

```
left(), right()
MariaDB [lsql]> select emp_id, fname, lname from employee where left(lname,1) = 'T';
+--------+--------+--------+
| emp_id | fname  | lname  |
+--------+--------+--------+
|      3 | Robert | Tyler  |
|      7 | Chris  | Tucker |
|     18 | Rick   | Tulman |
+--------+--------+--------+
3 rows in set (0.00 sec)
```

使用通配符

```sql
_ 匹配一个字符
% 匹配任意数目的字符（也包括0个）
注意 like 关键字；
MariaDB [lsql]> select lname from employee where lname like '_a%e%';
+-----------+
| lname     |
+-----------+
| Barker    |
| Hawthorne |
| Parker    |
| Jameson   |
+-----------+
4 rows in set (0.00 sec)
# 需要注意的是，sql在匹配的时候是忽略的大小写的；
```

使用正则表达式

```sql
MariaDB [lsql]> select emp_id, fname, lname from employee where lname regexp '^[FG]';
+--------+-------+----------+
| emp_id | fname | lname    |
+--------+-------+----------+
|      5 | John  | Gooding  |
|      6 | Helen | Fleming  |
|      9 | Jane  | Grossman |
|     17 | Beth  | Fowler   |
+--------+-------+----------+
4 rows in set (0.01 sec)
```

null

表示某个字段为空

```sql
MariaDB [lsql]> select emp_id, fname, lname, superior_emp_id from employee where superior_emp_id is null;
+--------+---------+-------+-----------------+
| emp_id | fname   | lname | superior_emp_id |
+--------+---------+-------+-----------------+
|      1 | Michael | Smith |            NULL |
+--------+---------+-------+-----------------+
1 row in set (0.00 sec)
```

在使用 null 做条件判断是不要使用 `=null` ，要使用`is null` 或者 `is not null`;

```sql
MariaDB [lsql]> select emp_id, fname, lname, superior_emp_id from employee where superior_emp_id = null;
Empty set (0.00 sec)
```

常见的会犯的错误：

```sql
MariaDB [lsql]> select emp_id, fname, lname, superior_emp_id from employee where superior_emp_id != 6;
+--------+----------+-----------+-----------------+
| emp_id | fname    | lname     | superior_emp_id |
+--------+----------+-----------+-----------------+
|      2 | Susan    | Barker    |               1 |
|      3 | Robert   | Tyler     |               1 |
|      4 | Susan    | Hawthorne |               3 |
|      5 | John     | Gooding   |               4 |
|      6 | Helen    | Fleming   |               4 |
|     10 | Paula    | Roberts   |               4 |
|     11 | Thomas   | Ziegler   |              10 |
|     12 | Samantha | Jameson   |              10 |
|     13 | John     | Blake     |               4 |
|     14 | Cindy    | Mason     |              13 |
|     15 | Frank    | Portman   |              13 |
|     16 | Theresa  | Markham   |               4 |
|     17 | Beth     | Fowler    |              16 |
|     18 | Rick     | Tulman    |              16 |
+--------+----------+-----------+-----------------+
14 rows in set (0.00 sec)

MariaDB [lsql]> select emp_id, fname, lname, superior_emp_id from employee where superior_emp_id != 6 or superior_emp_id is null;
+--------+----------+-----------+-----------------+
| emp_id | fname    | lname     | superior_emp_id |
+--------+----------+-----------+-----------------+
|      1 | Michael  | Smith     |            NULL |
|      2 | Susan    | Barker    |               1 |
|      3 | Robert   | Tyler     |               1 |
|      4 | Susan    | Hawthorne |               3 |
|      5 | John     | Gooding   |               4 |
|      6 | Helen    | Fleming   |               4 |
|     10 | Paula    | Roberts   |               4 |
|     11 | Thomas   | Ziegler   |              10 |
|     12 | Samantha | Jameson   |              10 |
|     13 | John     | Blake     |               4 |
|     14 | Cindy    | Mason     |              13 |
|     15 | Frank    | Portman   |              13 |
|     16 | Theresa  | Markham   |               4 |
|     17 | Beth     | Fowler    |              16 |
|     18 | Rick     | Tulman    |              16 |
+--------+----------+-----------+-----------------+
15 rows in set (0.00 sec)

第二个查询中多了一列，他的super_emp_id 为 null;
```