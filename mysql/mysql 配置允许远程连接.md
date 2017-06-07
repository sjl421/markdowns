# mysql 配置允许远程连接

mysql中管理用户密码及允许连接的host的配置实在mysql.user表中；

mysql默认root用户没有密码，输入mysql –u root 进入mysql

**1、初始化root密码**

进入mysql数据库

```sql
mysql>update user set password=PASSWORD(‘123456’) where user='root';
```

**2、允许mysql远程访问,可以使用以下三种方式:**

**a、改表**

```sql
mysql -u root –p
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```

**b、授权**

例如，你想root使用123456从任何主机连接到mysql服务器。

```sql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

如果你想允许用户jack从ip为10.10.50.127的主机连接到mysql服务器，并使用654321作为密码

```sql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@’10.10.50.127’ IDENTIFIED BY '654321' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;		#使刚刚的配置生效，
```

可参考：https://dev.mysql.com/doc/refman/5.7/en/privilege-changes.html

**c、在安装mysql的机器上运行：**

```sql
//进入MySQL服务器
mysql -h localhost -u root
//赋予任何主机访问数据的权限
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
//使修改生效
mysql>FLUSH PRIVILEGES;
//退出MySQL服务器
mysql>EXIT
```

