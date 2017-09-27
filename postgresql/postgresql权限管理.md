# postgresql权限管理

测试数据库 db2, owner=user2, 库中存在表 t1;

数据库中其他用户: user3, user4, 都拥有db2的connect 权限;

其中user4拥有t1的select 权限;

当通过gpadmin用户执行:

```
REVOKE ALL ON SCHEMA public FROM PUBLIC;
```

后,所有普通用户包括数据库的owner都无法看到public schema下的信息;



数据库的owner没有对public schema grant/revoke 操作的权限, 必须得用新创建的schema,  此时schema的owner 



---

针对通过api来创建的用户revoke掉从public 继承而来的权限,然后再通过单独的grant来授予相应的权限;

需要做的改动:

1. 将PUBLIC role中对public schema的create 权限去掉,这个改动会影响到整个租户;

```
REVOKE CREATE ON SCHEMA PUBLIC FROM PUBLIC;
```

2. 在创建db的时候通gpadmin 来创建,然后将owner指定给特定人,假如说user1,同时需要用gpamdin执行:

   ```
   GRANT ALL ON SCHEMA public TO  user1 with grant option; 
   #目前在创建db的时候会默认创建publc schema,并且public schema的owner是gpamdin
   #该授权需要gpadmin用户才能执行
   ```

    否则的话将没有用户可以在db 的public schema下操作,也没有办法将public schema的权限授给其他人;

3. 目前dms都是通过登录人来创建的,只有拥有创建数据库权限的用户才能创建,由于目前ops创建的用户是通过从PUBLIC (role)那里集成而来的对public schema的usage 和 create 权限,如果针对云上贵州采用第2点改动,那么就会造成dms的用户无法获取对public schema的usage 和 create 权限,同时也无法以gpadmin用户来是执行下面sql,造成所有出了gpadmin用户以外,没有用户可以在public schema下进行操作;

```
GRANT ALL ON SCHEMA public TO  user1 with grant option; 
```