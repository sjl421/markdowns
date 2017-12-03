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

```
typedef struct LWLock
{
	//用来控制LWlock的并发访问，在获取LWLock时（更新LWLock信息时，需要先获取LWLock->mutex);
	slock_t		mutex;			/* Protects LWLock and queue of PGPROCs */
	bool		releaseOK;		/* T if ok to release waiters */ //表示锁没有被占用
	char		exclusive;		/* # of exclusive holders (0 or 1) */ // 表示锁被排他占有
	int			shared;			/* # of shared holders (0..MaxBackends) */ //表示锁被同向占有
	int			exclusivePid;	/* PID of the exclusive holder. */
	PGPROC	   *head;			/* head of list of waiting PGPROCs */
	PGPROC	   *tail;			/* tail of list of waiting PGPROCs */
	/* tail is undefined when head is NULL */
} LWLock;
```



**void**
**LWLockAcquire(LWLockId lockid, LWLockMode mode)**

```
	for (;;)
	{
		bool		mustwait;
		int			c;
		/* Acquire mutex.  Time spent holding mutex should be short! */
		SpinLockAcquire(&lock->mutex);
		/* If retrying, allow LWLockRelease to release waiters again */
		if (retry)
			lock->releaseOK = true;

		/* If I can get the lock, do so quickly. */
		if (mode == LW_EXCLUSIVE)
		{
			if (lock->exclusive == 0 && lock->shared == 0)
			{
				lock->exclusive++;
				lock->exclusivePid = MyProcPid;
				mustwait = false;
			}
			else
				mustwait = true;
		}
		else
		{
			if (lock->exclusive == 0)
			{
				lock->shared++;
				mustwait = false;
			}
			else
				mustwait = true;
		}

		if (!mustwait)
		{
			LOG_LWDEBUG("LWLockAcquire", lockid, "acquired!");
			break;				/* got the lock */
		}

		/*
		 * Add myself to wait queue.
		 *
		 * If we don't have a PGPROC structure, there's no way to wait. This
		 * should never occur, since MyProc should only be null during shared
		 * memory initialization.
		 */
		if (proc == NULL)
			elog(PANIC, "cannot wait without a PGPROC structure");

		proc->lwWaiting = true;
		proc->lwExclusive = (mode == LW_EXCLUSIVE);
		lwWaitingLockId = lockid;
		proc->lwWaitLink = NULL;
		if (lock->head == NULL)
			lock->head = proc;
		else
			lock->tail->lwWaitLink = proc;
		lock->tail = proc;
		
		/* Can release the mutex now */
		SpinLockRelease(&lock->mutex);

		/*
		 * Wait until awakened.
		 *
		 * Since we share the process wait semaphore with the regular lock
		 * manager and ProcWaitForSignal, and we may need to acquire an LWLock
		 * while one of those is pending, it is possible that we get awakened
		 * for a reason other than being signaled by LWLockRelease. If so,
		 * loop back and wait again.  Once we've gotten the LWLock,
		 * re-increment the sema by the number of additional signals received,
		 * so that the lock manager or signal manager will see the received
		 * signal when it next waits.
		 */
		LOG_LWDEBUG("LWLockAcquire", lockid, "waiting");

#ifdef LWLOCK_TRACE_MIRROREDLOCK
	if (lockid == MirroredLock)
		elog(LOG, "LWLockAcquire: waiting for MirroredLock (PID %u)", MyProcPid);
#endif

#ifdef LWLOCK_STATS
		block_counts[lockid]++;
#endif

		for (c = 0; c < num_held_lwlocks; c++)
		{
			if (held_lwlocks[c] == lockid)
				elog(PANIC, "Waiting on lock already held!");
		}

		PG_TRACE2(lwlock__startwait, lockid, mode);

		for (;;)
		{
			/* "false" means cannot accept cancel/die interrupt here. */
#ifndef LOCK_DEBUG
			PGSemaphoreLock(&proc->sem, false);
#else
			LWLockTryLockWaiting(proc, lockid, mode);
#endif
			if (!proc->lwWaiting)
				break;
			extraWaits++;
		}

		PG_TRACE2(lwlock__endwait, lockid, mode);

		LOG_LWDEBUG("LWLockAcquire", lockid, "awakened");

#ifdef LWLOCK_TRACE_MIRROREDLOCK
		if (lockid == MirroredLock)
			elog(LOG, "LWLockAcquire: awakened for MirroredLock (PID %u)", MyProcPid);
#endif
		/* Now loop back and try to acquire lock again. */
		retry = true;
```

**void**
**LWLockRelease(LWLockId lockid)**

```
	//获取锁
	volatile LWLock *lock = &(LWLockArray[lockid].lock);
	//获取锁的mutex
	SpinLockAcquire(&lock->mutex);
	/* Release my hold on lock */
	if (lock->exclusive > 0)
	{
		lock->exclusive--;
		lock->exclusivePid = 0;
	}
	else
	{
		Assert(lock->shared > 0);
		lock->shared--;
	}
		head = lock->head;
	if (head != NULL)
	{
		if (lock->exclusive == 0 && lock->shared == 0 && lock->releaseOK)
		{
			/*
			 * Remove the to-be-awakened PGPROCs from the queue.  If the front
			 * waiter wants exclusive lock, awaken him only. Otherwise awaken
			 * as many waiters as want shared access.
			 */
			proc = head;
			if (!proc->lwExclusive)
			{
				while (proc->lwWaitLink != NULL &&
					   !proc->lwWaitLink->lwExclusive)
				{
					proc = proc->lwWaitLink;
					if (proc->pid != 0)
					{
						lock->releaseOK = false;
					}					
				}
			}
			/* proc is now the last PGPROC to be released */
			lock->head = proc->lwWaitLink;
			proc->lwWaitLink = NULL;
			
			/* proc->pid can be 0 if process exited while waiting for lock */
			if (proc->pid != 0)
			{
				/* prevent additional wakeups until retryer gets to run */
				lock->releaseOK = false;
			}
		}
		else
		{
			/* lock is still held, can't awaken anything */
			head = NULL;
		}
	}
	SpinLockRelease(&lock->mutex);
```



### ipc.h

用来注册清理共享内存的会掉函数， 

```
static struct ONEXIT
{
	pg_on_exit_callback function;
	Datum		arg;
}	on_proc_exit_list[MAX_ON_EXITS], on_shmem_exit_list[MAX_ON_EXITS];

static int	on_proc_exit_index,
			on_shmem_exit_index;
```



**shmem_exit(int code)**

该函数由postmaster调用，在有backend意外死掉时，调用所有在创建共享内存块是注册的回调函数重新初始化共享内存，该函数会在调用回调函数前将回调函数在注册的地方清理掉；

> 1. suppose： 如果一个backend的退出会造成全局的共享内存重新初始化，如果有多个backend，有的锁已经被session1获取了， session 2 在执行操作是死掉了引起了全局共享内存的清理，那么session 1获取的或也就被清理掉了？





### proc.h

1. 每一个backend都在共享内存中有一个PROC结构体；

2. PGPROC-> links： 

   > list link for any list the PGPROC is in. When waiting for a lock, the PGPROC is linked into that lock's waitProcs queue.  A recycled PGPROC is linked into ProcGlobal's freeProcs list.

```
struct PGPROC
{
	/* proc->links MUST BE FIRST IN STRUCT (see ProcSleep,ProcWakeup,etc) */
	SHM_QUEUE	links;			/* list link if process is in a list */

	PGSemaphoreData sem;		/* ONE semaphore to sleep on */
	int			waitStatus;		/* STATUS_WAITING, STATUS_OK or STATUS_ERROR */

	Latch		procLatch;		/* generic latch for process */

	LocalTransactionId lxid;	/* local id of top-level transaction currently
								 * being executed by this proc, if running;
								 * else InvalidLocalTransactionId */

	TransactionId xid;			/* id of top-level transaction currently being
								 * executed by this proc, if running and XID
								 * is assigned; else InvalidTransactionId */

	TransactionId xmin;			/* minimal running XID as it was when we were
								 * starting our xact, excluding LAZY VACUUM:
								 * vacuum must not remove tuples deleted by
								 * xid >= xmin ! */

	/*
	 * Distributed transaction information. This is only accessed by the backend
	 * itself, so this doesn't need to be protected by any lock. In fact, it
	 * could be just a global variable in backend-private memory, but it seems
	 * useful to have this information available for debugging purposes.
	 */
	LocalDistribXactData localDistribXactData;

	int			pid;			/* This backend's process id, or 0 */
	BackendId	backendId;		/* This backend's backend ID (if assigned) */
	Oid			databaseId;		/* OID of database this backend is using */
	Oid			roleId;			/* OID of role using this backend */
    int         mppSessionId;   /* serial num of the qDisp process */
    int         mppLocalProcessSerial;  /* this backend's PGPROC serial num */
    bool		mppIsWriter;	/* The writer gang member, holder of locks */

	bool		inCommit;		/* true if within commit critical section */

	uint8		vacuumFlags;	/* vacuum-related flags, see above */

	/* Info about LWLock the process is currently waiting for, if any. */
	bool		lwWaiting;		/* true if waiting for an LW lock */
	bool		lwExclusive;	/* true if waiting for exclusive access */
	struct PGPROC *lwWaitLink;	/* next waiter for same LW lock */

	/* Info about lock the process is currently waiting for, if any. */
	/* waitLock and waitProcLock are NULL if not currently waiting. */
	LOCK	   *waitLock;		/* Lock object we're sleeping on ... */
	PROCLOCK   *waitProcLock;	/* Per-holder info for awaited lock */
	LOCKMODE	waitLockMode;	/* type of lock we're waiting for */
	LOCKMASK	heldLocks;		/* bitmask for lock types already held on this
								 * lock object by this backend */

	/*
	 * Info to allow us to wait for synchronous replication, if needed.
	 * waitLSN is InvalidXLogRecPtr if not waiting; set only by user backend.
	 * syncRepState must not be touched except by owning process or WALSender.
	 * syncRepLinks used only while holding SyncRepLock.
	 */
	XLogRecPtr	waitLSN;		/* waiting for this LSN or higher */
	int			syncRepState;	/* wait state for sync rep */
	SHM_QUEUE	syncRepLinks;	/* list link if process is in syncrep queue */

	/*
	 * All PROCLOCK objects for locks held or awaited by this backend are
	 * linked into one of these lists, according to the partition number of
	 * their lock.
	 */
	SHM_QUEUE	myProcLocks[NUM_LOCK_PARTITIONS];

	struct XidCache subxids;	/* cache for subtransaction XIDs */

	/*
	 * Info for Resource Scheduling, what portal (i.e statement) we might
	 * be waiting on.
	 */
	uint32		waitPortalId;	/* portal id we are waiting on */

	/*
	 * Information for our combocid-map (populated in writer/dispatcher backends only)
	 */
	uint32		combocid_map_count; /* how many entries in the map ? */

	int queryCommandId; /* command_id for the running query */

	bool serializableIsoLevel; /* true if proc has serializable isolation level set */

	bool inDropTransaction; /* true if proc is in vacuum drop transaction */
};
```

