# Sql Select

## 1. Select Fundamental

查询中的自语句构成：

```sql
select
from
where
group by
having
order by
```

1. 对结果集中的数据经过“一定”处理后显示

```sql
select person_id, 'ACTIVE', person_id*2, food from favorite_food;

+-----------+--------+-------------+--------+
| person_id | ACTIVE | person_id*2 | food   |
+-----------+--------+-------------+--------+
|         2 | ACTIVE |           4 | cookie |
|         2 | ACTIVE |           4 | nachos |
|         2 | ACTIVE |           4 | pizza  |
+-----------+--------+-------------+--------+
3 rows in set (0.00 sec)

select 'ACTIVE' 这种格式的会在结果集中的每一行中都添加一个 ACTIVE的字段，字段的value也是'ACTIVE'
同时select语句中也对person_id 字段进行了乘2的处理后增加了一列；
```

2. 使用内建函数

```sql
MariaDB [sun_test]> select version(),user(), database();
+----------------+----------------+------------+
| version()      | user()         | database() |
+----------------+----------------+------------+
| 5.5.44-MariaDB | root@localhost | sun_test   |
+----------------+----------------+------------+
1 row in set (0.01 sec)
```

3. 列的别名

sql 语句中能够对每一列的key值设置一个别名，这样就不用非得按照数据库中列名或者表达式的名字来展示了，同时这也对在java或其他编程语言中更友好的读取查询结果；

使用的格式就是

```sql
select value1 alias1, value2 alias2,...
为了可读性更好，可以是用as 关键字
select value1 as alias1, values2 as alias2,...
```

可以有多组，每组之间用',隔开，一组中的 values 和alias 用空格隔开，其中value也可以是别的表达式的返回结果；

### 1. select a b 

```sql
select '1' b;
select 'hello' a, 'sun' b;
```

* 第一条sql是支持这种格式的语句的，做法就是将a的结果赋给b；第二条sql语句是将‘hello'赋给a，‘sun’赋给b
* 这里a 可以是任何东西，可以是一个数字，一个字符串，还可以是另外的一条sql语句(此时需要把sql语句用（）包起来)， 当是一条sql语句时就是把这条sql语句的执行结果用b来保存，eg:

```sql
select (select ount(*) from gp_segment_configuration where content=-1) num;

# (...) 中的sql语句查询的结果赋给了 num 变量，如果在java中通过executeQuery执行该查询语句的话就可以从返回的ResultSet中共 rs.getString("num")来获得查询的结果;
# 这里非常需要注意的一点就是要保证()中的sql语句的返回结果是一个值, 不能是多个值，也不能是一条记录；
```

于是就有了下面的这样的在java中执行的sql语句：

```sql
select 
(select count(*) from gp_segment_configuration where content=-1) ctrlHosts," +
"(select count(distinct hostname) from gp_segment_configuration where content != -1) calculateHosts," +
"(select count(*) count from pg_database where datname not in ('template0', 'template1', 'postgres')) dbCount;
# 相当于一次执行了三条的查询语句;
```

4. 去除重复的行

使用distinct 关键字;

```sql
MariaDB [sun_test]> select  person_id from favorite_food;
+-----------+
| person_id |
+-----------+
|         2 |
|         2 |
|         2 |
+-----------+
3 rows in set (0.00 sec)

MariaDB [sun_test]> select distinct person_id from favorite_food;
+-----------+
| person_id |
+-----------+
|         2 |
+-----------+
```



### From 子语句

1. 使用子查询获取临时表，然后再从临时表中查询；

```sql
select e.emp_id, e.fname, e.lname from (select emp_id, fname, lname, start_date from employee) e;
```

### 使用视图

```sql
create view employee_vw as select emp_id, fname, lname from employee;

mysql> select emp_id, fname, lname from employee_vw;
+--------+-------+----------+
| emp_id | fname | lname    |
+--------+-------+----------+
|      1 | sun   | jian     |
|      2 | lin   | bingqian |
+--------+-------+----------+
2 rows in set (0.00 sec)
```

当视图被创建出来后，并没有产生或存储任何数据，服务器只是简单的保留该查询以供后续使用

