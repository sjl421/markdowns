# PostgreSQL 创建表

PostgreSQL的CREATE TABLE语句是用来在任何指定的的数据库中创建一个新表。 yiibai.com

## 语法

CREATE TABLE语句的基本语法如下：

```sql
CREATE TABLE table_name(
   column1 datatype,
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
   PRIMARY KEY( one or more columns )
);  
```

CREATE TABLE是告诉数据库系统关键字，创建一个新的表。独特的名称或标识如下表CREATE TABLE语句。当前数据库中的表最初是空的，并且将所拥有的用户发出的命令。

然后在括号内来定义每一列的列表，在表中是什么样的数据类型。其语法变得更清晰，下面的例子。

## 实例

下面是一个例子，它创建了一个公司ID作为主键的表和NOT NULL的约束显示这些字段不能为NULL，同时创建该表的记录： www.yiibai.com

```sql
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);  
```

让我们创建一个表，在随后的章节中，我们将在练习中使用：

```sql
CREATE TABLE DEPARTMENT(
   ID INT PRIMARY KEY      NOT NULL,
   DEPT           CHAR(50) NOT NULL,
   EMP_ID         INT      NOT NULL
);  
```

可以验证已成功创建使用\d命令，将用于列出了附加的数据库中的所有表。 www.yiibai.com

```
testdb-# \d  
```

以上PostgreSQL的表会产生以下结果：

```sql
           List of relations
 Schema |    Name    | Type  |  Owner
--------+------------+-------+----------
 public | company    | table | postgres
 public | department | table | postgres
(2 rows)
  
```

使用\d表名来描述每个表如下所示：

```
testdb-# \d company  
```

以上PostgreSQL的表会产生以下结果：

```sql
        Table "public.company"
  Column   |     Type      | Modifiers
-----------+---------------+-----------
 id        | integer       | not null
 name      | text          | not null
 age       | integer       | not null
 address   | character(50) |
 salary    | real          |
 join_date | date          |
Indexes:
    "company_pkey" PRIMARY KEY, btree (id) 
```
创建自增字段

mysql 中使用  auto_increment 来表示某个字段是自增的；

postgres 使用 serial 来表示某个字段是自增的

```sql
create table info (
id serial not null,
name varchar(32)
);
```

