# FTS

进程创建流程:

```
main -> PostmasterMain -> ServerLoop -> do_reaper -> CommenceNormalOperations -> ftsprobe_start -> ftsMain -> FtsLoop()
```

