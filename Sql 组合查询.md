# Sql 组合查询

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

