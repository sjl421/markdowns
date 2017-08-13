# psql 的工具模式



```
PGOPTIONS='-c gp_session_role=utility' psql -h192.168.181.44 -p40000 -d zly
```





```
PGOPTIONS='-c gp_session_role=utility' psql -d postgres -c 'select * from gp_segment_configuration'
```

