# alter table

1. 修改表中的一个字段

```sql
# 可以使用modiify 和 change，但是注意是用change 的时候change后面要跟两次colnname，如果这两个colname 是相同的，则表示是对这个字段修改，如果不是同名的，则是删掉之前的字段然后再添加一个新的字段
1. alter table tbname modify colname xxx xxx xxx；
2. alter table tbname change colname colname xxx xxx xxx;
```

2. 添加一个新字段

```sql
alter table tbname add colname xxx xxx xxx;
```

3. 删除一个字段

```sql
alter table tbnme drop colname;
```

