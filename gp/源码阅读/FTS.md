# FTS

进程创建流程:

```
main -> PostmasterMain -> ServerLoop -> do_reaper -> CommenceNormalOperations -> ftsprobe_start -> ftsMain -> FtsLoop()
```



```
static char failover_strategy='n'; 
? 见过 f， 还有哪些？
```



FTS 每60秒执行一次检查