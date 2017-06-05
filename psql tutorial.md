# psql tutorial

postgreSql 可以在shell终端系下执行一些常用的命令，如创建或者删除user，role，database，连接某个数据库等；

1. create role

create role 中 role可以是一个单个的user也可以使一个group，如果是一个group，可以把某些用户放到该group下，该group也可是另外某个group的成员；

```sql
CREATE ROLE users;	# 创建一个用户组
GRANT users to sun, test;	#将用户sun和test加入该用户组
```

2. 创建或者删除数据库

```shell
createdb dbname	#创建数据库
dropdb dbname	#删除数据库
psql -l			# 列出所有数据库
```

