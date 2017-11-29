# LWLock

### pg_shmem.h

Platform-independent API for shared memory support.

```
typedef struct PGShmemHeader	/* standard header for all Postgres shmem */
{
	int32		magic;			/* magic # to identify Postgres segments */
#define PGShmemMagic  679834894
	pid_t		creatorPID;		/* PID of creating process */
	Size		totalsize;		/* total size of segment */
	Size		freeoffset;		/* offset to first free space */
	Size		indexoffset;	/* offset to ShmemIndex table */
#ifndef WIN32					/* Windows doesn't have useful inode#s */
	dev_t		device;			/* device data directory is on */
	ino_t		inode;			/* inode number of data directory */
#endif
} PGShmemHeader;
```

###sysv_shmem.c

Implement shared memory using SysV facilities

**InternalIpcMemoryCreate(IpcMemoryKey memKey, Size size)**

用来创建新的共享内存段（shared memory segment）， 如果创建成功，将该新建的共享内存attach到创建它的线程上；

同时在创建共享内存成功后会注册on_shmem_exit 回调函数，该共享内存会在调用on_shmem_exit 时被回收掉。

创建的过程中可能会发生`collision-with-existing-segment`,此时会打印错误信息，其它类型的错误是不可恢复的。

```
/* Register on-exit routine to delete the new segment */
on_shmem_exit(IpcMemoryDelete, Int32GetDatum(shmid));

/* OK, should be able to attach to the segment */
将创建的共享内存连接到进程的地址空间
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



###lwlock.c

Lightweight lock manager

全局数据，用来管理所有的LWLock

```
NON_EXEC_STATIC LWLockPadded *LWLockArray = NULL;
```

