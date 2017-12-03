# PG共享内存

共享内存是Linux系统中的概念

共享内存就是允许两个不相关的进程访问同一个逻辑内存。共享内存是在两个正在运行的进程之间共享和传递数据的一种非常有效的方式。不同进程之间共享的内存通常安排为同一段物理内存。进程可以将同一段共享内存连接到它们自己的地址空间中，所有进程都可以访问共享内存中的地址；

共享内存不提供同步机制；



```
main -> PostmasterMain -> ServerLoop -> BackendStartup -> BackendRun -> PostgresMain

PostgresMain -> exec_simple_query -> PortalRun
```

LWLock.h

存储在共享内存中的LWLock数组，子进程会通过fock()来获取继承这个指针；

```
/*
 * This points to the array of LWLocks in shared memory.  Backends inherit
 * the pointer by fork from the postmaster (except in the EXEC_BACKEND case,
 * where we have special measures to pass it down).
 */
NON_EXEC_STATIC LWLockPadded *LWLockArray = NULL;
```



postmaster.c

```
BackendParameters->LWLockArray;
```

**restore_backend_variables(BackendParameters *param, Port *port)**

对lwlock.h 中的LWLockArray指针赋值：

```
LWLockArray = param->LWLockArray;
```

**save_backend_variables(BackendParameters *param, Port *port,  HANDLE childProcess, pid_t childPid)**		

/* Save critical backend variables into the BackendParameters struct */		 

```
param->LWLockArray = LWLockArray;
```





---

###PG中创建共享内存：

main-> PostmasterMain -> reset_shared() -> CreateSharedMemoryAndSemaphores(makePrivate=false) -> PGSharedMemoryCreate  -> InternalIpcMemoryCreate();

reset_shared():

​	初始化共享内存， 主要是调用CreateSharedMemoryAndSemaphores 函数来创建共享内存；

CreateSharedMemoryAndSemaphores

​	创建全局的共享内存块和信号量，并对各种用途的共享内存初始化;

1. 先计算出整个GP所需要的共享内存块的大小
2. 调用`PGSharedMemoryCreate`整体创建共享内存；
3. 针对各种用途的共享内存调用其初始化函数进行初始化，（包括对LWLocks的初始化）

PGSharedMemoryCreate：

​	调用InternalIpcMemoryCreate() 执行具体共享内存的创建工作；

InternalIpcMemoryCreate()

1. 用来创建新的共享内存段（shared memory segment）， 如果创建成功，将该新建的共享内存attach到创建它的线程上；
2. 同时在创建共享内存成功后会注册on_shmem_exit 回调函数，该共享内存会在调用on_shmem_exit 时被回收掉。
3. 创建的过程中可能会发生`collision-with-existing-segment`,此时会打印错误信息，其它类型的错误是不可恢复的。

```
/*
 * Set up shared memory allocation mechanism
 */
if (!IsUnderPostmaster)
	InitShmemAllocation();

/*
 * Now initialize LWLocks, which do shared memory allocation and are
 * needed for InitShmemIndex.
 */
if (!IsUnderPostmaster)
	CreateLWLocks();
```

CreateSharedMemoryAndSemaphores

创建全局的共享内存块和信号量，并对各种用途的共享内存初始化;

1. 先计算出整个GP所需要的共享内存块的大小
2. 调用`PGSharedMemoryCreate`整体创建共享内存；
3. 针对各种用途的共享内存调用其初始化函数进行初始化，（包括对LWLocks的初始化）

```
计算整体需要的共享内存空间；
...
size = add_size(size, LWLockShmemSize());
...

/*****/
if (!IsUnderPostmaster)
	CreateLWLocks();
```

**InternalIpcMemoryCreate(IpcMemoryKey memKey, Size size)**

用来创建新的共享内存段（shared memory segment）， 如果创建成功，将该新建的共享内存attach到创建它的线程上。

同时在创建共享内存成功后会注册on_shmem_exit 回调函数，该共享内存会在调用on_shmem_exit 时被回收掉。

创建的过程中可能会发生`collision-with-existing-segment`,此时会打印错误信息，其它类型的错误是不可恢复的。

```
/* Register on-exit routine to delete the new segment */
on_shmem_exit(IpcMemoryDelete, Int32GetDatum(shmid));

/* OK, should be able to attach to the segment */
//将创建的共享内存连接到进程的地址空间
memAddress = shmat(shmid, NULL, PG_SHMAT_FLAGS);

if (memAddress == (void *) -1)
	elog(FATAL, "shmat(id=%d) failed: %m", shmid);

/* Register on-exit routine to detach new segment before deleting */
on_shmem_exit(IpcMemoryDetach, PointerGetDatum(memAddress));

/* Record key and ID in lockfile for data directory. */
// Append information about a shared memory segment to the data directory lock file.
RecordSharedMemoryInLockFile((unsigned long) memKey,
							 (unsigned long) shmid); 
```

删除共享内存：

```
/****************************************************************************/
/*	IpcMemoryDelete(status, shmId)		deletes a shared memory segment		*/
/*	(called as an on_shmem_exit callback, hence funny argument list)		*/
/****************************************************************************/
static void
IpcMemoryDelete(int status, Datum shmId)
{
	if (shmctl(DatumGetInt32(shmId), IPC_RMID, NULL) < 0)
		elog(LOG, "shmctl(%d, %d, 0) failed: %m",
			 DatumGetInt32(shmId), IPC_RMID);
}
```

将共享内存从进程中分离：

```
/****************************************************************************/
/*	IpcMemoryDetach(status, shmaddr)	removes a shared memory segment		*/
/*										from process' address spaceq		*/
/*	(called as an on_shmem_exit callback, hence funny argument list)		*/
/****************************************************************************/
static void
IpcMemoryDetach(int status, Datum shmaddr)
{
	if (shmdt(DatumGetPointer(shmaddr)) < 0)
		elog(LOG, "shmdt(%p) failed: %m", DatumGetPointer(shmaddr));
}
```

