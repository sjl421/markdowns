# GPDB allocateReaderGang

### allocateReaderGang

```
Gang *
allocateReaderGang(GangType type, char *portal_name)
```

gang的类型可以是:

* GANGTYPE_ENTRYDB_READER
*  GANGTYPE_SINGLETON_READER 
* GANGTYPE_PRIMARY_READER