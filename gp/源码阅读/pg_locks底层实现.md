#pg_locks底层实现

```
Datum
pg_lock_status(PG_FUNCTION_ARGS)
```

```
//调用函数获取当前segment（也就是master）的所有Lock		
mystatus->lockData = GetLockStatusData();
```

```
//去所有的segment上获取上面的锁；			
			appendStringInfo(&buffer,
					"SELECT * FROM  pg_lock_status() L "
					 " (locktype text, database oid, relation oid, page int4, tuple int2,"
					 " virtualxid text, transactionid xid, classid oid, objid oid, objsubid int2,"
					 " virtualtransaction text, pid int4, mode text, granted boolean, "
					 " mppSessionId int4, mppIsWriter boolean, gp_segment_id int4) ");
			CdbDispatchCommand(buffer.data, DF_WITH_SNAPSHOT, &cdb_pgresults);
```

```
/*
 * GetLockStatusData - Return a summary of the lock manager's internal
 * status, for use in a user-level reporting function.
 *
 * The return data consists of an array of PROCLOCK objects, with the
 * associated PGPROC and LOCK objects for each.  Note that multiple
 * copies of the same PGPROC and/or LOCK objects are likely to appear.
 * It is the caller's responsibility to match up duplicates if wanted.
 *
 * The design goal is to hold the LWLocks for as short a time as possible;
 * thus, this function simply makes a copy of the necessary data and releases
 * the locks, allowing the caller to contemplate and format the data for as
 * long as it pleases.
 */
LockData *
GetLockStatusData(void)
...
	for (i = 0; i < NUM_LOCK_PARTITIONS; i++)
		LWLockAcquire(FirstLockMgrLock + i, LW_SHARED);

	/* Now we can safely count the number of proclocks */
	els = hash_get_num_entries(LockMethodProcLockHash);
...
    while ((proclock = (PROCLOCK *) hash_seq_search(&seqstat)))
        {	
            PGPROC	   *proc = proclock->tag.myProc;
            LOCK	   *lock = proclock->tag.myLock;

            memcpy(&(data->proclocks[el]), proclock, sizeof(PROCLOCK));
            memcpy(&(data->procs[el]), proc, sizeof(PGPROC));
            memcpy(&(data->locks[el]), lock, sizeof(LOCK));

            el++;
        }
...        
    for (i = NUM_LOCK_PARTITIONS; --i >= 0;)
    	LWLockRelease(FirstLockMgrLock + i);
...    	
```

