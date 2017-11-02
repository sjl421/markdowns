#GPDB syslogger

>The system logger (syslogger) appeared in Postgres 8.0. It catches all stderr output from the postmaster, backends, and other subprocesses by redirecting to a pipe, and writes it to a set of logfiles. It's possible to have size and age limits for the logfile configured in postgresql.conf. If these limits are reached or passed, the current logfile is closed and a new one is created (rotated).The logfiles are stored in a subdirectory (configurable inpostgresql.conf), using an internal naming scheme that mangles creation time and current postmaster pid.



##SysLoggerMain

stderr管道 重定向

```
int	fd = open(DEVNULL, O_WRONLY, 0);

/*
 * The closes might look redundant, but they are not: we want to be
 * darn sure the pipe gets closed even if the open failed.	We can
 * survive running with stderr pointing nowhere, but we can't afford
 * to have extra pipe input descriptors hanging around.
 */
close(fileno(stdout));
close(fileno(stderr));
if (fd != -1)
{
	dup2(fd, fileno(stdout));
	dup2(fd, fileno(stderr));
	close(fd);
}
```



WIN32 下启动lgger  进程

```
#ifdef WIN32
	/* Fire up separate data transfer thread */
	InitializeCriticalSection(&sysloggerSection);
	EnterCriticalSection(&sysloggerSection);

    threadHandle = (HANDLE) _beginthreadex(NULL, 0, pipeThread, NULL, 0, NULL);
    if (threadHandle == 0)
        elog(FATAL, "could not create syslogger data transfer thread: %m");
#endif   /* WIN32 */
```



调用关系图
PostmasterMain -> SysLogger_Start ->  fork_process()(创建线程) -> SysLoggerMain(0, NULL) (logger 线程工作的方法);



### 从管道中读取数据

```c
next_chunkloop:
	if (bytesRead < sizeof(PipeProtoHeader))
	{
		/*
		 * We always try to make sure that the buffer has at least sizeof(PipeProtoHeader)
		 * bytes if we have read several bytes in the previous read. This handles the case
		 * when a valid chunk has to be read in two read calls, and the first read only
		 * picks up less than sizeof(PipeProtoHeader) bytes.
		 *
		 * However, this read may force some 3rd party error messages (less than
		 * sizeof(PipeProtoHeader) bytes) to sits in the buffer until the next message
		 * comes in. Thus you may experience some delays for small 3rd party error messages
		 * showing up in the logfile. Hopefully, this is very rare.
		 */
		readPos = bytesRead;
		bytesRead = piperead(syslogPipe[0], (char *)chunk + readPos, PIPE_CHUNK_SIZE - readPos);
	}
	
	if (bytesRead == 0)
	{
		/*
		 * Zero bytes read when select() is saying read-ready means
		 * EOF on the pipe: that is, there are no longer any processes
		 * with the pipe write end open.  Therefore, the postmaster
		 * and all backends are shut down, and we are done.
		 */
		pipe_eof_seen = true;

		/* if there's any data left then force it out now */
		syslogger_flush_chunks();
	}
...
  //将日志pipe chunk中的数据写入到日志中
  syslogger_handle_chunk(chunk);
```

###  syslogger_handle_chunk(chunk);

```
    if(chunk->hdr.is_last == 't')
	{
		if (chunk->hdr.is_segv_msg == 't')
		{
			syslogger_log_segv_chunk(first);
		}
		else
		{
			syslogger_log_chunk_list(first);
		}
	}
```

### syslogger_log_chunk_list

用来写入具体的日志的

---

调用关系图
PostmasterMain -> SysLogger_Start ->  fork_process()(创建线程) -> SysLoggerMain(0, NULL) (logger 线程工作的方法);

###SysLoggerMain

1. 设置am_syslogger标志位

   ```
   //如果syslogger进程,am_syslogger位为true
   am_syslogger = true;
   ```

2. 设置进程在linux 系统中的显示信息, 可以用ps -ef 看到;

   ```
   if (Gp_entry_postmaster && Gp_role == GP_ROLE_DISPATCH)
   	init_ps_display("master logger process", "", "", "");
   else
   	init_ps_display("logger process", "", "", "");
   ```

3. /* main worker loop */

   ```
   1. 检查配置文件;
   2. 检查文件是否需要Rotation;
   3. 检查工作目录是否正确,如果不正确则创建目录;
   4.
   ```

4. ...

   ```
   syslogger_forkexec() -> postmaster_forkexec() -> internal_forkexec() -> CreateProcess()
   ```

   ​