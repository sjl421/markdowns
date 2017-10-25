# GreenPlum 数据的导入导出

```
\copy callrecord  TO '/tmp/a.csv' with csv;
```

导出数据库中某张表的数据到sql文件中

```
pg_dump -h 192.168.143.231 -Ugpadmin -t callrecord db1 > dtmall.sql
```

从sql文件中导入到数据库中的table中, 需要先创建好数据库

```
psql -h 192.168.143.87 -Ugpadmin  -d db1 -f dtmall.sql
```

```
\copy tablename  from '/tmp/a.csv' with csv;
```

