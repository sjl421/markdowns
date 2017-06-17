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

---

## 有时候配置了远程连接发现连接不上

问题出在 mysql.user 表中的配置信息：

通过：`selelct user, host, password from user;` 查看user表中的信息；

```
selelct user, host, password from user;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost |                                           |
| root | 127.0.0.1 |                                           |
| root | ::1       |                                           |
| root | %         | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
+------+-----------+-------------------------------------------+
```

可以看到user表中配置了比较多的连接规则，mysql需要确定使用哪条规则，statckoverflow上有一个比较不错的回答(https://stackoverflow.com/questions/10299148/mysql-error-1045-28000-access-denied-for-user-billlocalhost-using-passw)

>Hence, such an anonymous user would "mask" any other user like `'[any_username]'@'%'` when connecting from `localhost`.
>
>`'bill'@'localhost'` does match `'bill'@'%'`, but would match (e.g.) `''@'localhost'` beforehands.
>
>The recommended solution is to drop this anonymous user (this is usually a good thing to do anyways).
>
>------
>
>*Below edits are mostly irrelevant to the main question. These are only meant to answer some questions raised in other comments within this thread.*
>
>**Edit 1**
>
>Authenticating as `'bill'@'%'` through a socket.
>
>```
>root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql -ubill -ppass --socket=/tmp/mysql-5.5.sock
>    Welcome to the MySQL monitor (...)
>
>    mysql> SELECT user, host FROM mysql.user;
>    +------+-----------+
>    | user | host      |
>    +------+-----------+
>    | bill | %         |
>    | root | 127.0.0.1 |
>    | root | ::1       |
>    | root | localhost |
>    +------+-----------+
>    4 rows in set (0.00 sec)
>
>    mysql> SELECT USER(), CURRENT_USER();
>    +----------------+----------------+
>    | USER()         | CURRENT_USER() |
>    +----------------+----------------+
>    | bill@localhost | bill@%         |
>    +----------------+----------------+
>    1 row in set (0.02 sec)
>
>    mysql> SHOW VARIABLES LIKE 'skip_networking';
>    +-----------------+-------+
>    | Variable_name   | Value |
>    +-----------------+-------+
>    | skip_networking | ON    |
>    +-----------------+-------+
>    1 row in set (0.00 sec)
>```
>
>**Edit 2**
>
>Exact same setup, except I re-activated networking, and I now create an anonymous user `''@'localhost'`.
>
>```
>    root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql
>    Welcome to the MySQL monitor (...)
>
>    mysql> CREATE USER ''@'localhost' IDENTIFIED BY 'anotherpass';
>    Query OK, 0 rows affected (0.00 sec)
>
>    mysql> Bye
>
>    root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql -ubill -ppass \
>        --socket=/tmp/mysql-5.5.sock
>    ERROR 1045 (28000): Access denied for user 'bill'@'localhost' (using password: YES)
>    root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql -ubill -ppass \
>        -h127.0.0.1 --protocol=TCP
>    ERROR 1045 (28000): Access denied for user 'bill'@'localhost' (using password: YES)
>    root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql -ubill -ppass \
>        -hlocalhost --protocol=TCP
>    ERROR 1045 (28000): Access denied for user 'bill'@'localhost' (using password: YES)
>
>
>```
>
>**Edit 3**
>
>Same situation as in edit 2, now providing the anonymous user's password.
>
>```
>    root@myhost:/home/mysql-5.5.16-linux2.6-x86_64# ./mysql -ubill -panotherpass -hlocalhost
>    Welcome to the MySQL monitor (...)
>
>    mysql> SELECT USER(), CURRENT_USER();
>    +----------------+----------------+
>    | USER()         | CURRENT_USER() |
>    +----------------+----------------+
>    | bill@localhost | @localhost     |
>    +----------------+----------------+
>    1 row in set (0.01 sec)
>
>
>```
>
>Conclusion 1, from edit 1: One can authenticate as `'bill'@'%'`through a socket.
>
>Conclusion 2, from edit 2: Whether one connects through TCP or through a socket has no impact on the authentication process (except one cannot connect as anyone else but `'something'@'localhost'` through a socket, obviously).
>
>Conclusion 3, from edit 3: Although I specified `-ubill`, I have been granted access as an anonymous user. This is because of the "sorting rules" advised above. Notice that in most default installations, [a no-password, anonymous user exists](http://dev.mysql.com/doc/refman/5.6/en/default-privileges.html) (and should be secured/removed).

