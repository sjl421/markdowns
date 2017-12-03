# PostgreSQL 角色与用户管理介绍

> 参考资料：http://www.jb51.net/article/40300.htm



## 1. 创建用户

```sql
CREATE ROLE 语法
CREATE ROLE name [ [ WITH ] option [ ... ] ]
where option can be:
      SUPERUSER | NOSUPERUSER
    | CREATEDB | NOCREATEDB
    | CREATEROLE | NOCREATEROLE
    | CREATEUSER | NOCREATEUSER
    | INHERIT | NOINHERIT
    | LOGIN | NOLOGIN
    | REPLICATION | NOREPLICATION
    | CONNECTION LIMIT connlimit
    | [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password'
    | VALID UNTIL 'timestamp'
    | IN ROLE role_name [, ...]
    | IN GROUP role_name [, ...]
    | ROLE role_name [, ...]
    | ADMIN role_name [, ...]
    | USER role_name [, ...]
    | SYSID uid
```

（注意：在上面的命令中创建角色时就赋予琪一定的属性时的 WITH 关键字可以有也可以忽略）

实例：

```sql
postgres=# CREATE ROLE david;　　//默认不带LOGIN属性
CREATE ROLE
postgres=# CREATE USER sandy;　　//默认具有LOGIN属性
CREATE ROLE
postgres=# \du					//列出当前数据库中所有的用户
```



创建Role并赋予其登录权限；

```
CREATE ROLR rolename LOGIN;
```



有时候想用刚创建的用户登录并连接某个数据库：此时可能会提示“没有 CONNECT 权限”，这是因为某个用户（不是超级用户）要连接某个数据库时需要有对这个数据库的CONNECT权限；

```sql
GRANT CONNECT ON DATABASE databasename TO username
```

然后就可以让这个用户连接制定的数据库了；



创建角色并赋予CREATEDB 和密码登录的权限：

```
postgres=# CREATE ROLE renee CREATEDB PASSWORD 'abc123' LOGIN; 
```

> 如果此时用renee 用户登录数据库，发现不需要输入密码既可登录，不符合实际情况。
>
> 原因：在角色属性中关于password的说明，在登录时要求指定密码时才会起作用，比如md5或者password模式，跟客户端的连接认证方式有关。
>
> 此时需要修改 pg_hba.conf文件，查看所对应的登录METHOD 是不是 trust，trust的登录模式是不需要输入密码的，![img](http://files.jb51.net/file_images/article/201308/201308050917082.jpg)
>
> 将local 的METHOD更改为 password，然后保存并重启postgresql即可；



## 2. 查看当前连接的用户

```sql
select current_user;
```

## 3. 授权登录权限

```sql
ALTER ROLE username LOGIN;
```

注意：这里使用的是 ALTER ROLE 而不是 GRANT

## 4. 查看角色信息

psql 终端可以用\du 或\du+ 查看，也可以查看系统表 select * from pg_roles;

角色信息：

| 属性          | 说明                                       |
| ----------- | ---------------------------------------- |
| login       | 只有具有 LOGIN 属性的角色可以用做数据库连接的初始角色名。         |
| superuser   | 数据库超级用户                                  |
| createdb    | 创建数据库权限                                  |
| createrole  | 允许其创建或删除其他普通的用户角色(超级用户除外)                |
| replication | 做流复制的时候用到的一个用户属性，一般单独设定。                 |
| password    | 在登录时要求指定密码时才会起作用，比如md5或者password模式，跟客户端的连接认证方式有关 |
| inherit     | 用户组对组员的一个继承标志，成员可以继承用户组的权限特性             |
| ...         | ...                                      |



## 5. 给已存在的用户赋予各种权限

使用ALTER ROLE 命令。

```sql
ALTER ROLE 语法：

ALTER ROLE name [ [ WITH ] option [ ... ] ]

where option can be:

      SUPERUSER | NOSUPERUSER

    | CREATEDB | NOCREATEDB

    | CREATEROLE | NOCREATEROLE

    | CREATEUSER | NOCREATEUSER

    | INHERIT | NOINHERIT

    | LOGIN | NOLOGIN

    | REPLICATION | NOREPLICATION

    | CONNECTION LIMIT connlimit

    | [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password'

    | VALID UNTIL 'timestamp'

ALTER ROLE name RENAME TO new_name

ALTER ROLE name [ IN DATABASE database_name ] SET configuration_parameter { TO | = } { value | DEFAULT }

ALTER ROLE name [ IN DATABASE database_name ] SET configuration_parameter FROM CURRENT

ALTER ROLE name [ IN DATABASE database_name ] RESET configuration_parameter

ALTER ROLE name [ IN DATABASE database_name ] RESET ALL

```

eg:

```sql
5.3 赋予david 带密码登录权限
postgres=# ALTER ROLE david WITH PASSWORD 'ufo456';
ALTER ROLE
postgres=#
5.4 设置sandy 角色的有效期
postgres=# ALTER ROLE sandy VALID UNTIL '2014-04-24';
ALTER ROLE
```



## 6.角色赋权/角色成员

```sql
六、角色赋权/角色成员
在系统的角色管理中，通常会把多个角色赋予一个组，这样在设置权限时只需给该组设置即可，撤销权限时也是从该组撤销。在PostgreSQL中，首先需要创建一个代表组的角色，之后再将该角色的membership 权限赋给独立的角色即可。
6.1 创建组角色
postgres=# CREATE ROLE father login nosuperuser nocreatedb nocreaterole noinherit encrypted password 'abc123';
CREATE ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of 
-----------+------------------------------------------------+-----------
 bella     | Create DB                                      | {}
 david     |                                                | {}
 father    | No inheritance                                 | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}
 renee     | Create role, Create DB                         | {}
 sandy     |                                                | {}
postgres=#
6.2 给father 角色赋予数据库test 连接权限和相关表的查询权限。
postgres=# GRANT CONNECT ON DATABASE test to father;
GRANT
postgres=# \c test renee
You are now connected to database "test" as user "renee".
test=> \dt
No relations found.
test=> CREATE TABLE emp (
test(> id serial,
test(> name text);
NOTICE:  CREATE TABLE will create implicit sequence "emp_id_seq" for serial column "emp.id"
CREATE TABLE
test=> INSERT INTO emp (name) VALUES ('david');  
INSERT 0 1
test=> INSERT INTO emp (name) VALUES ('sandy');
INSERT 0 1
test=> SELECT * from emp;
 id | name  
----+-------
  1 | david
  2 | sandy
(2 rows)
test=> \dt
       List of relations
 Schema | Name | Type  | Owner 
--------+------+-------+-------
 public | emp  | table | renee
(1 row)
test=> GRANT USAGE ON SCHEMA public to father;
WARNING:  no privileges were granted for "public"
GRANT
test=> GRANT SELECT on public.emp to father;
GRANT
test=> 
6.3 创建成员角色
test=> \c postgres postgres
You are now connected to database "postgres" as user "postgres".
postgres=# CREATE ROLE son1 login nosuperuser nocreatedb nocreaterole inherit encrypted password 'abc123';
CREATE ROLE
postgres=# 
这里创建了son1 角色，并开启inherit 属性。PostgreSQL 里的角色赋权是通过角色继承（INHERIT）的方式实现的。
6.4 将father 角色赋给son1
postgres=# GRANT father to son1;
GRANT ROLE
postgres=# 
还有另一种方法，就是在创建用户的时候赋予角色权限。
postgres=# CREATE ROLE son2 login nosuperuser nocreatedb nocreaterole inherit encrypted password 'abc123' in role father;
CREATE ROLE
postgres=# 
6.5 测试son1 角色
postgres=# \c test son1
You are now connected to database "test" as user "son1".
test=> \dt
       List of relations
 Schema | Name | Type  | Owner 
--------+------+-------+-------
 public | emp  | table | renee
(1 row)
test=> SELECT * from emp;
 id | name  
----+-------
  1 | david
  2 | sandy
(2 rows)
test=> 
用renee 角色新创建一张表，再次测试
test=> \c test renee
You are now connected to database "test" as user "renee".
test=> CREATE TABLE dept (
test(> deptid integer,
test(> deptname text);
CREATE TABLE
test=> INSERT INTO dept (deptid, deptname) values(1, 'ts');
INSERT 0 1
test=> \c test son1
You are now connected to database "test" as user "son1".
test=> SELECT * from dept ;
ERROR:  permission denied for relation dept
test=> 
son1 角色只能查询emp 表的数据，而不能查询dept 表的数据，测试成功。
6.6 查询角色组信息
test=> \c postgres postgres
You are now connected to database "postgres" as user "postgres".
postgres=# 
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of 
-----------+------------------------------------------------+-----------
 bella     | Create DB                                      | {}
 david     |                                                | {}
 father    | No inheritance                                 | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}
 renee     | Create role, Create DB                         | {}
 sandy     |                                                | {}
 son1      |                                                | {father}
 son2      |                                                | {father}
postgres=# 
“ Member of ” 项表示son1 和son2 角色属于father 角色组。
```

