# SQL Update 语句

Update语句是针对数据库中表的操作

```sql
UPDATE [LOW_PRIORITY] [IGNORE] tbl_name
SET col_name1=expr1 [, col_name2=expr2 ...]
[WHERE where_definition]
[ORDER BY ...]
[LIMIT row_count]
```

1. 更新表中的某条记录

```sql
update  tablename  set column1='xxx', column2=xxx where xxxx
```

> 这里需要注意的地方是 update 后直接是表名，不用单独的跟table关键字；
>
> 非常重要的一点就是如果你不加 where条件限制，那么一条update语句会更新掉数据库中所有的表；

