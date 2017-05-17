# GreenPlum 系统表

### gp_segment_configuration

```
 dbid | content | role | preferred_role | mode | status | port  |   hostname    |    address    | replication_port | san_mounts 
------+---------+------+----------------+------+--------+-------+---------------+---------------+------------------+------------
    1 |      -1 | p    | p              | s    | u      |  5432 | m1.adb.g1.com | m1.adb.g1.com |                  | 
    2 |       0 | p    | p              | s    | u      | 40000 | c1.adb.g1.com | c1.adb.g1.com |            41000 | 
   10 |       8 | p    | p              | s    | u      | 40000 | c2.adb.g1.com | c2.adb.g1.com |            41000 | 
    3 |       1 | p    | p              | s    | u      | 40001 | c1.adb.g1.com | c1.adb.g1.com |            41001 | 
   11 |       9 | p    | p              | s    | u      | 40001 | c2.adb.g1.com | c2.adb.g1.com |            41001 | 
    4 |       2 | p    | p              | s    | u      | 40002 | c1.adb.g1.com | c1.adb.g1.com |            41002 | 
   12 |      10 | p    | p              | s    | u      | 40002 | c2.adb.g1.com | c2.adb.g1.com |            41002 | 
    5 |       3 | p    | p              | s    | u      | 40003 | c1.adb.g1.com | c1.adb.g1.com |            41003 | 
   13 |      11 | p    | p              | s    | u      | 40003 | c2.adb.g1.com | c2.adb.g1.com |            41003 | 
    6 |       4 | p    | p              | s    | u      | 40004 | c1.adb.g1.com | c1.adb.g1.com |            41004 | 
   14 |      12 | p    | p              | s    | u      | 40004 | c2.adb.g1.com | c2.adb.g1.com |            41004 | 
    7 |       5 | p    | p              | s    | u      | 40005 | c1.adb.g1.com | c1.adb.g1.com |            41005 | 
   15 |      13 | p    | p              | s    | u      | 40005 | c2.adb.g1.com | c2.adb.g1.com |            41005 | 
    8 |       6 | p    | p              | s    | u      | 40006 | c1.adb.g1.com | c1.adb.g1.com |            41006 | 
   16 |      14 | p    | p              | s    | u      | 40006 | c2.adb.g1.com | c2.adb.g1.com |            41006 | 
    9 |       7 | p    | p              | s    | u      | 40007 | c1.adb.g1.com | c1.adb.g1.com |            41007 | 
   17 |      15 | p    | p              | s    | u      | 40007 | c2.adb.g1.com | c2.adb.g1.com |            41007 | 
   18 |       0 | m    | m              | s    | u      | 50000 | c2.adb.g1.com | c2.adb.g1.com |            51000 | 
   19 |       1 | m    | m              | s    | u      | 50001 | c2.adb.g1.com | c2.adb.g1.com |            51001 | 
   20 |       2 | m    | m              | s    | u      | 50002 | c2.adb.g1.com | c2.adb.g1.com |            51002 | 
   21 |       3 | m    | m              | s    | u      | 50003 | c2.adb.g1.com | c2.adb.g1.com |            51003 | 
   22 |       4 | m    | m              | s    | u      | 50004 | c2.adb.g1.com | c2.adb.g1.com |            51004 | 
   23 |       5 | m    | m              | s    | u      | 50005 | c2.adb.g1.com | c2.adb.g1.com |            51005 | 
   24 |       6 | m    | m              | s    | u      | 50006 | c2.adb.g1.com | c2.adb.g1.com |            51006 | 
   25 |       7 | m    | m              | s    | u      | 50007 | c2.adb.g1.com | c2.adb.g1.com |            51007 | 
   26 |       8 | m    | m              | s    | u      | 50000 | c1.adb.g1.com | c1.adb.g1.com |            51000 | 
   27 |       9 | m    | m              | s    | u      | 50001 | c1.adb.g1.com | c1.adb.g1.com |            51001 | 
   28 |      10 | m    | m              | s    | u      | 50002 | c1.adb.g1.com | c1.adb.g1.com |            51002 | 
   29 |      11 | m    | m              | s    | u      | 50003 | c1.adb.g1.com | c1.adb.g1.com |            51003 | 
   30 |      12 | m    | m              | s    | u      | 50004 | c1.adb.g1.com | c1.adb.g1.com |            51004 | 
   31 |      13 | m    | m              | s    | u      | 50005 | c1.adb.g1.com | c1.adb.g1.com |            51005 | 
   32 |      14 | m    | m              | s    | u      | 50006 | c1.adb.g1.com | c1.adb.g1.com |            51006 | 
   33 |      15 | m    | m              | s    | u      | 50007 | c1.adb.g1.com | c1.adb.g1.com |            51007 | 
   34 |      -1 | m    | m              | s    | u      |  5432 | m2.adb.g1.com | m2.adb.g1.com |                  | 
```

```
解释：
dbid: 当前segment 的i；
content: 可能是表示当前segment的类型和id，如果是控制节点，当前字段值为-1， 如果是计算节点，就是从0开始的一个整数，其中每一个数字都有两条记录，其中一条记录的role 为 p， 另一条为 m，即一个是primary segment，另一个是mirror segment；
role: 当前的segment是primary还是mirror；
preferred_role: 不清楚；
mode: 
status: u表示up的意思，表示运行正常， d 表示当前segment已经不可用了；
port:
hostname:
address:
replication_port:
san_mounts:
```

