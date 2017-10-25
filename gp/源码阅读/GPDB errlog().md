# GPDB errlog()

###errstart()

	bool
	errstart(int elevel, const char *filename, int lineno,
			 const char *funcname, const char *domain)

```
/*
 * errstart --- begin an error-reporting cycle
 *
 * Create a stack entry and store the given parameters in it.  Subsequently,
 * errmsg() and perhaps other routines will be called to further populate
 * the stack entry.  Finally, errfinish() will be called to actually process
 * the error report.
 *
 * Returns TRUE in normal case.  Returns FALSE to short-circuit the error
 * report (if it's a warning or lower and not to be reported anywhere).
 */
```



2. 判断是否需要处理error report
   1. 如果是error level 是warning 或者更低并且禁止了logging, errstart()函数直接返回false, 并且后续并不启动任何的错误日志的处理机制;
3. 判断是否开启了server log output

```
	/* Determine whether message is enabled for server log output */
	if (IsPostmasterEnvironment)
		output_to_server = is_log_level_output(elevel, log_min_messages);
	else
	{
		/* In bootstrap/standalone case, do not sort LOG out-of-order */
		output_to_server = (elevel >= log_min_messages);
	}
```

 如果是在postmaster 进程及postmaster子进程中 `IsPostmasterEnvironment` 为true,

在standalone process (bootstrap or standalone backend) 中为false;



#errfinish()

 * errfinish --- end an error-reporting cycle
 * Produce the appropriate error report(s) and pop the error stack.
    If elevel is ERROR or worse, control does not return to the caller.
 * See elog.h for the error level definitions.



##ereport

```
#define ereport_domain(elevel, domain, rest)	\
	(errstart(elevel, __FILE__, __LINE__, PG_FUNCNAME_MACRO, domain) ? \
	 (errfinish rest) : (void) 0)

#define ereport(elevel, rest)	\
	ereport_domain(elevel, TEXTDOMAIN, rest)
```

errstart()开启一个日志的周期, errfinish()结束一个日志周期;

日志的处理逻辑是先将日志信息放到错误日志堆栈中:

```
static ErrorData errordata[ERRORDATA_STACK_SIZE];
```



-> 如果错误堆栈已经满了,里面的数据将会丢失,系统并打印出PANIC报告;

```
	if (++errordata_stack_depth >= ERRORDATA_STACK_SIZE)
	{
		/*
		 * Wups, stack not big enough.	We treat this as a PANIC condition
		 * because it suggests an infinite loop of errors during error
		 * recovery.
		 */
		errordata_stack_depth = -1;		/* make room on stack */
		ereport(PANIC, (errmsg_internal("ERRORDATA_STACK_SIZE exceeded")));
	}
```



# CdbProgramErrorHandler

/* CDB: Signal handler for program errors */



##pgsignal(int signo, pgsigfunc func)

注册信号处理函数

postMain()中通过pgsignal()方法来注册信号处理函数

```
#ifndef _WIN32
#ifdef SIGILL
		pqsignal(SIGILL, CdbProgramErrorHandler);
#endif
#ifdef SIGSEGV
		pqsignal(SIGSEGV, CdbProgramErrorHandler);
#endif
#ifdef SIGBUS
		pqsignal(SIGBUS, CdbProgramErrorHandler);
#endif
```



##write_message_to_server_log

 * Write error report to server's log.
 * This is an equivalent function as send_message_to_server_log, but will write
 * the error report in the format of GpPipeProtoHeader, followed by a serialized
 * format of GpErrorData. The error report is sent over to the syslogger process
 * through the pipe.
 * This function is thread-safe. Here, we assume that sprintf is thread-safe.
    void

```c
void
write_message_to_server_log(int elevel,
							int sqlerrcode,
							const char *message,
							const char *detail,
							const char *hint,
							const char *query_text,
							int cursorpos,
							int internalpos,
							const char *internalquery,
							const char *context,
							const char *funcname,
							bool show_funcname,
							const char *filename,
							int lineno,
							int stacktracesize,
							bool omit_location,
							bool send_alert,
							void* const *stacktracearray,
							bool printstack)
```

##gp_write_pipe_chunk

```
/*
 * Send the data through the pipe.
 */
static inline void
gp_write_pipe_chunk(const char *buffer, int len)
```

##append_string_to_pipe_chunk

```c
/*
 * Append a string (termniated by '\0') to the GpPipeProtoChunk.
 *
 * If GpPipeProtoChunk does not have space for the given string,
 * this function appends enough data to fill the buffer, and
 * sends out the buffer. After that, the payload session of
 * GpPipeProtoChunk is reset and the rest of the given string
 * is appended. If the given string is pretty large, this function
 * may send out multiple chunks.
 */
static inline void
append_string_to_pipe_chunk(PipeProtoChunk *buffer, const char* input)
```

##write_stderr

```
/*
 * Write errors to stderr (or by equal means when stderr is
 * not available). Used before ereport/elog can be used
 * safely (memory context, GUC load etc)
 */
extern void
write_stderr(const char *fmt,...)
```

##EmitErrorReport(void)

> Actual output of the top-of-stack error message In the ereport(ERROR) case this is called from PostgresMain (or not at all,if the error is caught by somebody).  For all other severity levels this is called by errfinish.

代码实现:

```
	/* Send to server log, if enabled */
	if (edata->output_to_server)
		send_message_to_server_log(edata);

	/* Send to client, if enabled */
	if (edata->output_to_client)
		send_message_to_frontend(edata);
```