# Sql更改表中一个字段的属性

添加一个字段：

```sql
alter table tablename add id int unsigned not Null auto_increment primary key
```

更改一个表的属性:

```sql
alter table multi_tenancy change ops_admin_role ops_admin_role boolean not null default 0;
```

MySQL添加字段的方法并不复杂，下面将为您详细介绍[MySQL](http://database.51cto.com/art/201011/232204.htm)添加字段和修改字段等操作的实现方法，希望对您学习MySQL添加字段方面会有所帮助。

1.登录数据库

> mysql -u root -p 数据库名称

2.查询所有数据表

> show tables;

3.查询表的字段信息

> desc 表名称;

4.1添加表字段

```sql
alter table table1 add transactor varchar(10) not Null;
alter table   table1 add id int unsigned not Null auto_increment primary key
```

4.2.修改某个表的字段类型及指定为空或非空

> alter table 表名称 change 字段名称 字段名称 字段类型 [是否允许非空];
>
> alter table 表名称 modify 字段名称 字段类型 [是否允许非空];

4.3.修改某个表的字段名称及指定为空或非空

> alter table 表名称 change 字段原名称 字段新名称 字段类型 [是否允许非空]

4.4如果要删除某一字段，可用命令：ALTER TABLE mytable DROP 字段名;