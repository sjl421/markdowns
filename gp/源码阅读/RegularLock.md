#RegularLock(Lock)

Lock的创建过程:

PG中LWLock和RegulartLock均依赖共享内存实现, 这里先描述一下共享内存的创建动作;

![Lock](G:\Download\Lock.png)

1. PG里面创建工作内存的动作由PostMasterMain()完成,reset_shared()为 创建共享内存的总入口; 
2. 共享内存的创建动作在函数CreateSharedMemoryAndSemaphores()中完成, 底层依次调用了PGSharedMemoryCreate() 和InternalIpcMemoryCreate;
3. 在共享内存完成之后需要将共享内存attach到自己的进程空间中,该动作由函数PGSharedMemoryAttach完成;
4. 在共享内存创建完成之后,PG会将共享内存划分为不同的用途并做初始化操作,会依次调用 CreateLWLocks()和InitLocks()来初始化LWLock和RegularLock;

RegularLock 就是一般数据库事务管理中所指的锁；

LOCK 结构体：

```
typedef struct LOCK
{
	/* hash key */
	LOCKTAG		tag;			/* unique identifier of lockable object */

	/* data */
	LOCKMASK	grantMask;		/* bitmask for lock types already granted */
							   //用来表示当前锁已经以哪些LOCKMODE的形式被占用了, 用来做LOCKMODE的互斥检测
	LOCKMASK	waitMask;		/* bitmask for lock types awaited */
							   //用来表是当前有哪些LOCKMODE在等待占用该锁
	SHM_QUEUE	procLocks;		/* list of PROCLOCK objects assoc. with lock */
	PROC_QUEUE	waitProcs;		/* list of PGPROC objects waiting on lock */
	int			requested[MAX_LOCKMODES];		/* counts of requested locks */
	//记录该锁的每种请求模式被请求的此时(包括已经被grant的锁模式)
	int			nRequested;		/* total of requested[] array */
	//requested数组的长度;
	int			granted[MAX_LOCKMODES]; /* counts of granted locks */
	//记录锁在每种请求模式上被分配的锁的数量;
	int			nGranted;		/* total of granted[] array */
	//granted数组的长度;
} LOCK;
```

PG 锁支持的锁类型，共八种：

```
/* NoLock is not a lock mode, but a flag value meaning "don't get a lock" */
#define NoLock					0

#define AccessShareLock			1		/* SELECT */
#define RowShareLock			2		/* SELECT FOR UPDATE/FOR SHARE */
#define RowExclusiveLock		3		/* INSERT, UPDATE, DELETE */
#define ShareUpdateExclusiveLock 4		/* VACUUM (non-FULL),ANALYZE, CREATE
										 * INDEX CONCURRENTLY */
#define ShareLock				5		/* CREATE INDEX (WITHOUT CONCURRENTLY) */
#define ShareRowExclusiveLock	6		/* like EXCLUSIVE MODE, but allows ROW
										 * SHARE */
#define ExclusiveLock			7		/* blocks ROW SHARE/SELECT...FOR
										 * UPDATE */
#define AccessExclusiveLock		8		/* ALTER TABLE, DROP TABLE, VACUUM
										 * FULL, and unqualified LOCK TABLE */
```



支持的两种锁模式：

```
/* These identify the known lock methods */
#define DEFAULT_LOCKMETHOD	1
#define USER_LOCKMETHOD		2
#define RESOURCE_LOCKMETHOD	3
```

The LockTagType enum defines the different kinds of objects we can lock.

```
typedef enum LockTagType
{
	LOCKTAG_RELATION,			/* whole relation */
	/* ID info for a relation is DB OID + REL OID; DB OID = 0 if shared */
	LOCKTAG_RELATION_EXTEND,	/* the right to extend a relation */
	/* same ID info as RELATION */
	LOCKTAG_PAGE,				/* one page of a relation */
	/* ID info for a page is RELATION info + BlockNumber */
	LOCKTAG_TUPLE,				/* one physical tuple */
	/* ID info for a tuple is PAGE info + OffsetNumber */
	LOCKTAG_TRANSACTION,		/* transaction (for waiting for xact done) */
	/* ID info for a transaction is its TransactionId */
	LOCKTAG_VIRTUALTRANSACTION, /* virtual transaction (ditto) */
	/* ID info for a virtual transaction is its VirtualTransactionId */
	LOCKTAG_RELATION_RESYNCHRONIZE,			/* whole relation for resynchronize */
	/* ID info for a relation is DB OID + REL OID; DB OID = 0 if shared */
	LOCKTAG_RELATION_APPENDONLY_SEGMENT_FILE,	/* Segment file within an Append-Only relation */
	/* ID info for a relation is DB OID + REL OID + (LOGICAL) SEGMENT FILE # */
	LOCKTAG_OBJECT,				/* non-relation database object */
	/* ID info for an object is DB OID + CLASS OID + OBJECT OID + SUBID */

	/*
	 * Note: object ID has same representation as in pg_depend and
	 * pg_description, but notice that we are constraining SUBID to 16 bits.
	 * Also, we use DB OID = 0 for shared objects such as tablespaces.
	 */
	LOCKTAG_RESOURCE_QUEUE,		/* ID info for resource queue is QUEUE ID */
	LOCKTAG_USERLOCK,			/* reserved for old contrib/userlock code */
	LOCKTAG_ADVISORY			/* advisory user locks */
} LockTagType;
```

**LOCKTAG：**

> A LOCKTAG value uniquely identifies a lockable object;

表征了数据库对象(数据库, 表),  同时也包含事务与某个锁的对应关系;

```
typedef struct LOCKTAG
{
	uint32		locktag_field1; /* a 32-bit ID field */
	uint32		locktag_field2; /* a 32-bit ID field */
	uint32		locktag_field3; /* a 32-bit ID field */
	uint16		locktag_field4; /* a 16-bit ID field */
	uint8		locktag_type;	/* see enum LockTagType */
	uint8		locktag_lockmethodid;	/* lockmethod indicator */
} LOCKTAG;
```

###InitLocks

全局共享内存创建完成之后的初始化动作, 优PostMasterMain集成来调用;

```
LockMethodLockHash = ShmemInitHash("LOCK hash",
									   init_table_size,
									   max_table_size,
									   &info,
									   hash_flags);
LockMethodProcLockHash = ShmemInitHash("PROCLOCK hash",
										   init_table_size,
										   max_table_size,
										   &info,
										   hash_flags);	
LockMethodLocalHash = hash_create("LOCALLOCK hash",
									  128,
									  &info,
									  hash_flags);                                   
```

###LockAcquire

```
LockAcquireResult
LockAcquire(const LOCKTAG *locktag,
			LOCKMODE lockmode,
			bool sessionLock,
			bool dontWait)
//sessionLock: 加锁模式:true表示未会话加锁,false表示为当前事务加锁;
//dontWait: 表示无法在立即获取锁时是否等待,true表示不等待,false表示等待;
```

1. 以HASH_ENTER的方式在LockMethodLocalHash的本地hashtable中搜索该locktag对应的锁，如果本地HashTabel中没有该锁，则在其中创建该并初始化;

   如果当前线程已经获得了锁，则增加本地的锁计数并返回，不对共享内存做操作；

   ```
   locallock = (LOCALLOCK *) hash_search(LockMethodLocalHash,
   									  (void *) &localtag,
   									  HASH_ENTER, &found);
   ```


1. 本地尚未获取该锁，根据生成的LOCALLOCK->hashcode 计算出该锁对应的LWLockId；

   调用LWLockAcquire 以互斥的方式获取对应的LWLockId 的lock；

   为保证安全，在获取到LWLock后会重新记录该hashcode的锁到LockMethodLockHash（如果失败，一般是超过了共享内存的空间）并做初始化操作，如果记录失败，会释放LWLock;

   ```
   hashcode = locallock->hashcode;
   partition = LockHashPartition(hashcode);
   partitionLock = LockHashPartitionLock(hashcode);

   LWLockAcquisre(partitionLock, LW_EXCLUSIVE);
   lock = (LOCK *) hash_search_with_hash_value(LockMethodLockHash,
   											(void *) locktag,
   											hashcode,
   											HASH_ENTER_NULL,
   											&found);
   ```

2. 创建 PROCLOCK

3. 检查所冲突：

   其中一项的检查就是当前要获取的锁是否和自己已经获取的锁冲突；

   如果冲突了并且是dontWait策略，则清除LocalLock，释放LWLock，返回LOCKACQUIRE_NOT_AVAIL； 如果不是dontWait策略，则进入等待；

4. 如果一切正常, 释放PROCLOCK， LOCKACQUIRE_OK; 此时该进程成功获取了LWLock；

###LockRelease

LockRelease() 函数主要用来表是锁的释放过程;

```
bool LockRelease(const LOCKTAG *locktag, LOCKMODE lockmode, bool sessionLock)
```



![锁的释放](G:\Download\锁的释放.png)

1. 根据LOCKTAG,在本地LockMethodLocalHash表中搜索lock; 如果线程已经获取了该锁,则应该在本地的LocalLock table中是有记录的;

   ```
   locallock = (LOCALLOCK *) hash_search(LockMethodLocalHash,
   										  (void *) &localtag,
   										  HASH_FIND, NULL);
   ```

2. 减小本地locallock->nLocks--的计数.

   如果计数不为零,则表示我们之前是多次获取了同一个锁,此时只减小计数,返回true

   ```
   locallock->nLocks--;
   if (locallock->nLocks > 0)
   	return TRUE;
   ```

3. 如果locallock->nLocks-- 后nLocks计数为0 

   ```
   	/*
   	 * Do the releasing.  CleanUpLock will waken any now-wakable waiters.
   	 */
   	wakeupNeeded = UnGrantLock(lock, lockmode, proclock, lockMethodTable);

   	CleanUpLock(lock, proclock,
   				lockMethodTable, locallock->hashcode,
   				wakeupNeeded);
   	...
       RemoveLocalLock(locallock);
       return TRUE
   ```

### 具体的锁

与表相关的锁:

17308:dbsun的oid;  24588:t1的oid; LockTagType=0: LOCKTAG_RELATION, LockMode = 1: AccessShareLock

![获取表的锁](G:\snap\获取表的锁.PNG)

事务与锁之间的关系:

253669:xid(事务id); LockTagType=4代表LOCKTAG_TRANSACTION

![XACT_LOCK](G:\snap\XACT_LOCK.PNG)



---

12.5

PROCLOCK:

用于存储进程和锁之间的关系,通过该结构可以查询到当前进程租户了哪些进程,在死锁检测和消除中需要频繁使用到该结构;

```
typedef struct PROCLOCK
{
	/* tag */
	PROCLOCKTAG tag;			/* unique identifier of proclock object */

	/* data */
	LOCKMASK	holdMask;		/* bitmask for lock types currently held */
	LOCKMASK	releaseMask;	/* bitmask for lock types to be released */
	SHM_QUEUE	lockLink;		/* list link in LOCK's list of proclocks */
	SHM_QUEUE	procLink;		/* list link in PGPROC's list of proclocks */
	int			nLocks;			/* total number of times lock is held by 
								   this process, used by resource scheduler */
	SHM_QUEUE	portalLinks;	/* list of ResPortalIncrements for this 
								   proclock, used by resource scheduler */
} PROCLOCK;
```

LOCALLOCK:

每个线程都本地维护一份自己已经获取的锁的hashtable, hashtable中处的就是LOCKLOCK对象,

```
typedef struct LOCALLOCK
{
	/* tag */
	LOCALLOCKTAG tag;			/* unique identifier of locallock entry */

	/* data */
	LOCK	   *lock;			/* associated LOCK object in shared mem */
							   //指针，指向该LOCALLOCK所关联的LOCK对象
	PROCLOCK   *proclock;		/* associated PROCLOCK object in shmem */
							   //指针，指向该LOCALLOCK所关联的PROCLOCK对象
	uint32		hashcode;		/* copy of LOCKTAG's hash value */
	bool		preparable;		/* MPP: During prepare we populate this to avoid MPP-1094 */
	int64		nLocks;			/* total number of times lock is held */
							   // 表示获取该锁的总次数
	int			numLockOwners;	/* # of relevant ResourceOwners */
	int			maxLockOwners;	/* allocated size of array */
	LOCALLOCKOWNER *lockOwners; /* dynamically resizable array */
} LOCALLOCK;
```

PROCLOCKTAG  用来确定proclock 在proclock hashtable中的位置,PROCLOCKTAG 唯一的确定了某个LOCK对象的持有者和等待者;

>PROCLOCKTAG is the key information needed to look up a PROCLOCK item in the proclock hashtable.

```
typedef struct PROCLOCKTAG
{
	LOCK   *myLock;	            /* link to per-lockable-object information */
	PGPROC *myProc;	            /* link to PGPROC of owning backend */
} PROCLOCKTAG;
```

PROCLOCK:

每一个获取了某个对象LOCK的进程,需要维护将要获取这个LOCK或者在等待获取该LOCK对象的线程的信息,这些信息存储在PROCLOCK结构体中; 

PROCLOCK对象被连接到了与它关联的LOCK对象中

> Each PROCLOCK object is linked into lists for both the associated LOCK object and the owning PGPROC object. 

```
typedef struct PROCLOCK
{
	/* tag */
	PROCLOCKTAG tag;			/* unique identifier of proclock object */

	/* data */
	LOCKMASK	holdMask;		/* bitmask for lock types currently held */
	LOCKMASK	releaseMask;	/* bitmask for lock types to be released */
	SHM_QUEUE	lockLink;		/* list link in LOCK's list of proclocks */
	//每一个LOCK对象内部都有一个PROCLOCK链表,用来记录所有持有该锁的PROCKLOCk对象,这里的lockLink记录的就是该PROKLOCK对象在该链表中的位置;
	SHM_QUEUE	procLink;		/* list link in PGPROC's list of proclocks */
	//每一个PGPROC对象中都有一个PROCLINK链表,记录的是所有当前进程持有锁或者即将持有锁的PROCLOCK对象,这里的procLink就是记录本PROCKLOCK对象在链表中的位置的
	int			nLocks;			/* total number of times lock is held by 
								   this process, used by resource scheduler */
	SHM_QUEUE	portalLinks;	/* list of ResPortalIncrements for this 
								   proclock, used by resource scheduler */
} PROCLOCK;
```

* Table-level Lock Modes

  RegularLock就是Table-Level Lock

* Row-level Locks

  对某一行的update操作使用的Row-level Locks;

* Deadlocks

* Advisory Locks 

   session level: 持有锁直到显式的释放锁或者直到事务结束;

   transaction level:

  RegularLock就是Table-Level Lock



如何表示一个线程获取了某个锁:

在线程本地的LocalLockHashTable中插入该锁对应的



全局HashTable中的锁与本地HashTable中的锁的对应关系:

​	本地HashTable中的LocalLock中只是保留了RegularLock的指针;



Lock如何表征某个锁被占用了:

Lock对象中通过nGranted, granted[lockmode], grantMask,waitMask四个字段来表示当前锁的状态;

* nGranted 记录当前有多少进程占用了该锁,
* granted[lockmode]用来记录当前锁以某种LOCKMODE被占用的次数;
* grantMask: 所有被占用的LOCKMODE的集合;
* waitMask:所有等待占用该锁的LOCKMODE的集合;
* grantMask和waitMask用来做当前锁的LOCKMODE的互斥检测;

表示占用某个锁只是更新以上四个字段;

```
lock->nGranted++;
lock->granted[lockmode]++;
lock->grantMask |= LOCKBIT_ON(lockmode);
lock->waitMask &= LOCKBIT_OFF(lockmode);
```

### 三种锁的几种关系

SpinLock, LWLock, RegularLock, 

SpinLock是最底层的锁,与操作系统相关, 使用互斥信号量实现, 

LWLock(Lightweight,轻量级锁) 主要用来控制共享内存的互斥访问, 底层的实现依赖SpinLock + 共享内存;

RegularLock(简称Lock): 数据库事务管理中使用的锁,  底层的实现依赖LWLock + 共享内存;

```
...
//获取LWLock
LWLockAcquire(partitionLock, LW_EXCLUSIVE);

//操作共享内存
GrantLock(lock, proclock, lockmode);
GrantLockLocal(locallock, owner);

//释放k
LWLockRelease(partitionLock);
...
```

共享内存:

```
    共享内存是一段可以被多个进程同时访问的内存空间。是一种常见的进程间通信的方式。   
    共享内存作为进程间通信机制的缺陷: 共享内存并未提供同步机制，在并发访问共享内存时需要其他的机制来控制对共享内存的互斥访问;
```



### 锁的几种模式

```
/* NoLock is not a lock mode, but a flag value meaning "don't get a lock" */
#define NoLock					0

#define AccessShareLock			1		/* SELECT */
#define RowShareLock			2		/* SELECT FOR UPDATE/FOR SHARE */
#define RowExclusiveLock		3		/* INSERT, UPDATE, DELETE */
#define ShareUpdateExclusiveLock 4		/* VACUUM (non-FULL),ANALYZE, CREATE
										 * INDEX CONCURRENTLY */
#define ShareLock				5		/* CREATE INDEX (WITHOUT CONCURRENTLY) */
#define ShareRowExclusiveLock	6		/* like EXCLUSIVE MODE, but allows ROW
										 * SHARE */
#define ExclusiveLock			7		/* blocks ROW SHARE/SELECT...FOR
										 * UPDATE */
#define AccessExclusiveLock		8		/* ALTER TABLE, DROP TABLE, VACUUM
										 * FULL, and unqualified LOCK TABLE */
```

###LOCK:

```
typedef struct LOCK
{
	/* hash key */
	LOCKTAG		tag;			/* unique identifier of lockable object */

	/* data */
	LOCKMASK	grantMask;		/* bitmask for lock types already granted */
							   //用来表示当前锁已经以哪些LOCKMODE的形式被占用了, 用来做LOCKMODE的互斥检测
	LOCKMASK	waitMask;		/* bitmask for lock types awaited */
							   //用来表是当前有哪些LOCKMODE在等待占用该锁
	SHM_QUEUE	procLocks;		/* list of PROCLOCK objects assoc. with lock */
	PROC_QUEUE	waitProcs;		/* list of PGPROC objects waiting on lock */
	int			requested[MAX_LOCKMODES];		/* counts of requested locks */
	//记录该锁的每种请求模式被请求的此时(包括已经被grant的锁模式)
	int			nRequested;		/* total of requested[] array */
	//requested数组的长度;
	int			granted[MAX_LOCKMODES]; /* counts of granted locks */
	//记录锁在每种请求模式上被分配的锁的数量;
	int			nGranted;		/* total of granted[] array */
	//granted数组的长度;
} LOCK;
```

Lock和数据库对象之间的关系:

```
typedef struct LOCKTAG
{
	uint32		locktag_field1; /* a 32-bit ID field */
	uint32		locktag_field2; /* a 32-bit ID field */
	uint32		locktag_field3; /* a 32-bit ID field */
	uint16		locktag_field4; /* a 16-bit ID field */
	uint8		locktag_type;	/* see enum LockTagType */
	uint8		locktag_lockmethodid;	/* lockmethod indicator */
} LOCKTAG;
```



