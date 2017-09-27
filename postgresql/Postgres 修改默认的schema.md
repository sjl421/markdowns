# Postgres 修改默认的schema

1. 查看当前的 search_path(schema)

```
db1=# show search_path ;
 search_path 
-------------
 s1
(1 row)
```

2. 设置当前会话的schema

```
db1=# set search_path = "$user", public,s1;
SET
db1=# \d
             List of relations
 Schema | Name | Type  |  Owner  | Storage 
--------+------+-------+---------+---------
 public | T1   | table | gpadmin | heap
 public | TT1  | table | gpadmin | heap
 public | t1   | table | gpadmin | heap
 public | t2   | table | gpadmin | heap
 public | t3   | table | role1   | heap
 public | tt1  | table | gpadmin | heap
 s1     | b1
```

3. 永久性的修改数据的默认的search_path

```
alter database mydb set search_path = s1;
db1=# show search_path ;
 search_path 
-------------
 s1
(1 row)
db1=# \d
             List of relations
 Schema | Name | Type  |  Owner  | Storage 
--------+------+-------+---------+---------
 s1     | b1   | table | gpadmin | heap
(1 row)
# 注: 如果你在当前数据库中执行sql,那你需要切换数据库后再重新连回来才能让新的schema生效;
```

4. 永久性的修改某个role的默认search_path

```

```





---

# 针对云上贵州readonly role的测试

1.  gpadmin 创建db1, 并将db1的owner指定为user1
2. gpamdin 将db1的默认的schema 设置为schema1,同同时授予user1 对 schema1 usege 和 create 权限;
3. user1 连接到db1时默认的schema是schema1;
4. user1 尝试在db1下创建table:

> 1. 只有create 权限,没有usage权限: 创建table和删除table的时候都需要执行schema;
> 2. 当同时授予create 和 usage 权限后 就可以不带schema 创建和删除table了;