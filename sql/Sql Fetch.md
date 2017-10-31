# Sql Fetch

1. fetch last , 不支持

```
dbsun=> FETCH last from mycursor;
ERROR: backward scan is not supported in this version of Greenplum Database
```

2. FETCH FORWARD 0 gp不支持SCROLL 字段

```
dbsun=> FETCH FORWARD 0 FROM mycursor;
ERROR:  cursor can only scan forward
HINT:  Declare it with SCROLL option to enable backward scan.
```

3.  gp 不是支持SCROLL字段

```
ERROR:  cursor can only scan forward
HINT:  Declare it with SCROLL option to enable backward scan.	
```

```
A cursor cannot be used to retrieve rows in a nonsequential fashion. This is the default behavior in Greenplum Database, since scrollable cursors (SCROLL) are not supported.
需要在声明cursor的时候带上 SCROLL关键字,但是该关键字GP不支持;
```

4.  不支持

```
dbsun=> FETCH RELATIVE -1 FROM mycursor;
ERROR:  backward scan is not supported in this version of Greenplum Database
```

![fetch](G:\snap\fetch.PNG)



### Move 

move的功能和fetch相似,唯一的区别就是move只移动光标,不获取数据,fetch在移动光标的同时也获取数据;

1. move last 不支持;

```
dbsun=> MOVE last from mycursor;
ERROR:  backward scan is not supported in this version of Greenplum Database
```

2. MOVE FORWARD 0 正常工作;

```
dbsun=> MOVE FORWARD 0 from mycursor;
MOVE 0
```

3. MOVE RELATIVE 0 正常工作;

```
dbsun=> MOVE RELATIVE 0 from mycursor;
MOVE 0
```

4. MOVE RELATIVE -1 不支持;

```
dbsun=> MOVE RELATIVE -1 from mycursor;
ERROR:  backward scan is not supported in this version of Greenplum Database
```

