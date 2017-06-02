# Mysql count() 函数的使用

count函数是用来统计表中或数组中记录的一个函数，下面我来介绍在mysql中count函数用法与性能比较吧。

1. count(\*) 它返回检索行的数目， 不论其是否包含 NULL值。SELECT 从一个表中检索，而不检索其它的列，并且没有 WHERE子句时， COUNT(*)被优化到最快的返回速度。

```sql
mysql> SELECT COUNT(*) FROM student;
```

```
COUNT(DISTINCT 字段)
返回不同的非NULL值数目。
若找不到匹配的项，则COUNT(DISTINCT)返回 0 。
```

```sql
CREATE TABLE `user` (
  `id` int(5) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  `password` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=latin1
测试数据为：
1 name1 123456
2 name2 123456
3 name3 123456
4 name4  NULL
分别输入一下命令：
1. select count(*) from `user`
2. select count(name) from `user`
3. select count(password) from `user`
对应的返回的结果是：4，4，3
```

