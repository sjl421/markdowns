# mysql 生成xml

在登录mysql时使用 --xml 选项可以是查询所得到的数据xml格式化

```sql
mysql -u user -p --xml dbname
```

```xml
MariaDB [sun_test]> select * from favorite_food;
<?xml version="1.0"?>

<resultset statement="select * from favorite_food;" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <row>
	<field name="person_id">2</field>
	<field name="food">cookie</field>
  </row>

  <row>
	<field name="person_id">2</field>
	<field name="food">nachos</field>
  </row>

  <row>
	<field name="person_id">2</field>
	<field name="food">pizza</field>
  </row>
</resultset>
3 rows in set (0.00 sec)
```

