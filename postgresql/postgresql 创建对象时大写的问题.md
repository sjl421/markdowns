# postgresql 创建对象时大小写的问题

1. 创建database 支持大写, 如果数据库名包含有大写字符时,需要用""将数据库名包起来,否则会自动将大写字符转成小写;

```sql
create database Db3   // 最终创建出的数据库为:db3;
create database "Db3" // 最终创建出的数据库为:Db3;
\c "Db3" 和 \c Db3    都能成功够切换数据库;
```

2. 创建schema, table, view

创建schema, table, view  也支持大写字符, 同样再操作具有大写字符的这些对象时需要将对象名用""包起来;

schema:

```
db1=# create schema "S1";
CREATE SCHEMA
```

table:

```
db1=# create table "T1" (id int);
NOTICE:  Table doesn't have 'DISTRIBUTED BY' clause -- Using column named 'id' as the Greenplum Database data distribution key for this table.
HINT:  The 'DISTRIBUTED BY' clause determines the distribution of data. Make sure column(s) chosen are the optimal data distribution key to minimize skew.
CREATE TABLE
db1=# select * from "T1";
 id 
----
(0 rows)
```

view:

```
db1=# create view "V1" as select * from t1;
CREATE VIEW
db1=# select * from "V1";
 id 
----
(0 rows)
```

