# HashTable(jdk1.8)

# HashMap和Hashtable的区别

**两者最主要的区别在于Hashtable是线程安全，而HashMap则非线程安全** 

Hashtable的实现方法里面都添加了synchronized关键字来确保线程同步，因此相对而言HashMap性能会高一些，我们平时使用时若无特殊需求建议使用HashMap，在多线程环境下若使用HashMap需要使用Collections.synchronizedMap()方法来获取一个线程安全的集合（Collections.synchronizedMap()实现原理是Collections定义了一个SynchronizedMap的内部类，这个类实现了Map接口，在调用方法时使用synchronized来保证线程同步,当然了实际上操作的还是我们传入的HashMap实例，简单的说就是Collections.synchronizedMap()方法帮我们在操作HashMap时自动添加了synchronized来实现线程同步，类似的其它Collections.synchronizedXX方法也是类似原理）

**HashMap可以使用null作为key，而Hashtable则不允许null作为key** 

虽说HashMap支持null值作为key，不过建议还是尽量避免这样使用，因为一旦不小心使用了，若因此引发一些问题，排查起来很是费事 
HashMap以null作为key时，总是存储在table数组的第一个节点上



**HashMap是对Map接口的实现，HashTable实现了Map接口和Dictionary抽象类**



**HashMap的初始容量为16，Hashtable初始容量为11，两者的填充因子默认都是0.75**



HashMap扩容时是当前容量翻倍即:capacity*2，Hashtable扩容时是容量翻倍+1即:capacity*2+1



**两者计算hash的方法不同**

Hashtable计算hash是直接使用key的hashcode对table数组的长度直接进行取模

```
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

二、默认构造方法开始

```
public Hashtable() {
        this(11, 0.75f);
    }
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }
    通过学习hashmap知道，这表示hashtable初始是11的大小，也是0.75的容量比例
    table = new Entry<?,?>[initialCapacity]; 说明初始时已经构建了数据结构是Entry类型的数组，Entry源码和hashmap基本元素用的node基本是一样的
    threshold= 11*0.75=8.25 即容量是8
    threshold上限是MAX_ARRAY_SIZE + 1     Integer.MAX_VALUE - 8+1=Integer.MAX_VALUE -7   这个上限根据注释说明，是因为一些JVM的不同限制，为了防止OutOfMemoryError而进行-8的
```

 三、get/put方法

```
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
```

```
public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
    
    1.synchronized 同步  还是方法级别的同步，因此效率可想肯定不是很高
    2.value==null 抛错  说明值不能null
    3.int index = (hash & 0x7FFFFFFF) % tab.length;   这个index的算法，看逼格就没hashmap高啊
        3.1具体0x7FFFFFFF是2 31次幂，即01111111111111111111111111111111   安位与 随便一个与之后好像没变化啊，那这句是干嘛的呢，闲的？我们要相信大佬们不会闲的没事写废代码的
            与的作用其实就是第一位0，为了干掉负的hash的，-1的hash值就是-1，再%，显然不能让出现这个情况
        3.2    %tab.length就好说了，让索引落在table内，这种方式相比hashmap的二次hash再异或，效率肯定差一些的
    4.for循环  只看这个循环就知道了，数据结构是数组加链表了
        主要作用是找当前索引的链表上有没有相同key  又就覆盖值结束
    5.addEntry(hash, key, value, index);当前索引没找到相同key，或者压根索引上就是空的，就在当前索引上加节点
```

四、addEntry增加节点方法

```
private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>) tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }
    1. modCount++;结构变更次数，不谈
    2. if (count >= threshold) {  是否扩容的判断，也是一个主要地方了，看看扩容方案
        2.1 rehash();
        protected void rehash() {
            int oldCapacity = table.length;
            Entry<?,?>[] oldMap = table;

            // overflow-conscious code
            int newCapacity = (oldCapacity << 1) + 1;    说明扩容方案是 扩2倍+1
            if (newCapacity - MAX_ARRAY_SIZE > 0) {        如果扩容后大于最大数组大小，就用最大数组大小
                if (oldCapacity == MAX_ARRAY_SIZE)
                    // Keep running with MAX_ARRAY_SIZE buckets
                    return;
                newCapacity = MAX_ARRAY_SIZE;
            }
            Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];   创建扩容的新数组结构

            modCount++;
            threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);    计算新的容量比例
            table = newMap;    主数组切换到新的数组

            for (int i = oldCapacity ; i-- > 0 ;) {    转移原结构中数据到新结构中
                for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
                    Entry<K,V> e = old;
                    old = old.next;

                    int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                    e.next = (Entry<K,V>)newMap[index];
                    newMap[index] = e;
                }
            }
        }
        2.2 完成扩容了，需要对本次要添加元素重新计算索引位置
    3.     Entry<K,V> e = (Entry<K,V>) tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        创建新元素把添加到数组。
        这里有一个地方要注意到，再有值的情况下，hashmap是在链表末尾挂载新元素的，而这是在链表头也就是数组索引这个位置挂载新元素，一前一后要注意这点
```

```
    /**
     * Increases the capacity of and internally reorganizes this
     * hashtable, in order to accommodate and access its entries more
     * efficiently.  This method is called automatically when the
     * number of keys in the hashtable exceeds this hashtable's capacity
     * and load factor.
     */
    @SuppressWarnings("unchecked")
    protected void rehash() {
        int oldCapacity = table.length;
        Entry<?,?>[] oldMap = table;

        // overflow-conscious code
        int newCapacity = (oldCapacity << 1) + 1;
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

        modCount++;
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;

        for (int i = oldCapacity ; i-- > 0 ;) {
            for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
                Entry<K,V> e = old;
                old = old.next;

                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = (Entry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }
```

五、其它一些
1.关于2n+1的扩展，在hashtable选择用取模的方式来进行，那么尽量使用素数、奇数会让结果更加均匀一些，具体证明，可以看看已经证明这点的技术文章
关于hash，hashmap用2的幂，主要是其还有一个hash过程即二次hash，不是直接用key的hashcode，这个过程打散了数据
总体就是一个减少hash冲突，并且找索引效率还要高，实现都是要考量这两因素的