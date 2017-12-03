#RegularLock(Lock)

RegularLock 就是一般数据库事务管理中所指的锁；

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

​	A LOCKTAG value uniquely identifies a lockable object

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

LOCK 结构体：

```
typedef struct LOCK
{
	/* hash key */
	LOCKTAG		tag;			/* unique identifier of lockable object */

	/* data */
	LOCKMASK	grantMask;		/* bitmask for lock types already granted */
	LOCKMASK	waitMask;		/* bitmask for lock types awaited */
	SHM_QUEUE	procLocks;		/* list of PROCLOCK objects assoc. with lock */
	PROC_QUEUE	waitProcs;		/* list of PGPROC objects waiting on lock */
	int			requested[MAX_LOCKMODES];		/* counts of requested locks */
	int			nRequested;		/* total of requested[] array */
	int			granted[MAX_LOCKMODES]; /* counts of granted locks */
	int			nGranted;		/* total of granted[] array */
} LOCK;
```



 * We may have several different backends holding or awaiting locks
 * on the same lockable object.  We need to store some per-holder/waiter
 * information for each such holder (or would-be holder).  This is kept in
 * a PROCLOCK struct.



###数据库对象和Lock之间的对应关系

LOCKTAG 类决定了数据库对象和锁之间的关系；

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

###LockAcquire

```
LockAcquireResult
LockAcquire(const LOCKTAG *locktag,
			LOCKMODE lockmode,
			bool sessionLock,
			bool dontWait)
```

1. 以HASH_ENTER的方式在LockMethodLocalHash 的hashtable中搜索该localtag的锁，如果hashtable中没有该锁，则在hashtable中创建；

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