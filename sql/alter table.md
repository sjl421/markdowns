# alter table

1. 修改表中的一个字段

```sql
# 可以使用modiify 和 change，
# 使用change 需要输入两个字段名，意思是将第一个字段的信息修改为第二个字段的（也可以为来理解为删除第一个字段，然后再添加一个新的字段），如果两个字段名是相同的，则相当于 修改 的作用；
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

4. 添加外键约束

```sql
alter table favorite_food add constraint fk_fav_food_person_id foreign key (person_id) references person(person_id);
```

其中 fk_fav_food_person_id  为外间约束名称；

添加外键约束时若没有指定外键约束的名称，则系统会自动添加外键约束名：表名_ibfk_n(表示第n个外键约束)

5. 删除外键

根据外键约束的名字来删除外键

```sql
alter table favorite_food drop foreign key 外键约束名称；
```

