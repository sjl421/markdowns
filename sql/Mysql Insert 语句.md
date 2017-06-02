# Mysql Insert 语句

### 1用法

在标准的SQL语句中，一次插入一条记录的INSERT语句只有一种形式。

```sql
INSERT INTO tablename(列名…) VALUES(列值);
```

而在MySQL中还有另外一种形式。

```sql
INSERT INTO tablename SET column_name1 = value1, column_name2 = value2，…;
```

第一种方法将列名和列值分开了，在使用时，列名必须和列值的数一致。如下面的语句向users表中插入了一条记录：

```sql
INSERT INTO users(id, name, age) VALUES(123, '姚明', 25);
```

第二种方法允许列名和列值成对出现和使用，如下面的语句将产生中样的效果。

```sql
INSERT INTO users SET id = 123, name = '姚明', age = 25;
```

### 2不同点

1.  如果使用了SET方式，必须至少为一列赋值。如果某一个字段使用了省缺值（如默认或自增值），这两种方法都可以省略这些字段。如id字段上使用了自增值，上面两条语句可以写成如下形式：

```sql
INSERT INTO users (name, age) VALUES('姚明',25);
```

```sql
INSERT INTO uses SET name = '姚明', age = 25;
```

2. MySQL在VALUES上也做了些变化。

如果**VALUES**中什么都不写，那MySQL将使用表中每一列的默认值来插入新记录。

```sql
INSERT INTO users () VALUES();
```

如果**表名**后什么都不写，就表示向表中所有的字段赋值。使用这种方式，不仅在VALUES中的值要和列数一致，而且顺序不能颠倒。 INSERT INTO users VALUES(123, '姚明', 25);

如果将INSERT语句写成如下形式MySQL将会报错。如：

```sql
INSERT INTO users VALUES('姚明',25)
```

3. 标准的INSERT语句允许一次插入多条数据，set不行

```sql
INSERT INTO users (name, age) VALUES('姚明',25),('麦蒂',25)
```

