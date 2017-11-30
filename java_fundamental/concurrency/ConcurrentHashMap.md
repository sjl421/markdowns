# ConcurrentHashMap

ConcurrentHashMap和HashTable类似，key和value均不能为null（HashMap的key或者value可以为null）；

ConcurrentHashMap的实现是 数组+链表+红黑树实现的，内部所有的元素都存放在Node内部类中；

### ConcurrentHashMap的实现

ConcurrentHashMap内部实现了很多内部类。所有的key-value 对都存放在Node中。红黑树的节点由TreeNode组成；TreeBins记录了红黑树的根节点；



链表的锁机制，将每一个链表的首元素作为锁；

如何实现resizing？

计数器LongAdder的实现

遍历



### 关键常亮

```
// 最大容量：2^30=1073741824
private static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认初始值，必须是2的幕数
private static final int DEFAULT_CAPACITY = 16;

//
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

// 
private static final float LOAD_FACTOR = 0.75f;

// 链表转红黑树阀值,> 8 链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;

//树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）
static final int UNTREEIFY_THRESHOLD = 6;

//
static final int MIN_TREEIFY_CAPACITY = 64;

//
private static final int MIN_TRANSFER_STRIDE = 16;

//
private static int RESIZE_STAMP_BITS = 16;

// 2^15-1，help resize的最大线程数
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

// 32-16=16，sizeCtl中记录size大小的偏移量
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

// forwarding nodes的hash值
static final int MOVED     = -1; 

// 树根节点的hash值
static final int TREEBIN   = -2; 

// ReservationNode的hash值
static final int RESERVED  = -3; 

// 可用处理器数量
static final int NCPU = Runtime.getRuntime().availableProcessors();
```



###Fields

```
/** 
 * The array of bins. Lazily initialized upon first insertion.
 * Size is always a power of two. Accessed directly by iterators.
 */
//用来存放Node节点数据的，默认为null，默认大小为16的数组，每次扩容时大小总是2的幂次方；
transient volatile Node<K,V>[] table;

/**
 * The next table to use; non-null only while resizing.
 */
//扩容时新生成的数据，数组为table的两倍； 
private transient volatile Node<K,V>[] nextTable;

/**
 * Base counter value, used mainly when there is no contention,
 * but also as a fallback during table initialization
 * races. Updated via CAS.
 */
private transient volatile long baseCount;

```



参考文献:

1. http://cmsblogs.com/?p=2283

