#HashMap(Jdk1.8)

**在JDK1.6，JDK1.7中，HashMap采用位桶+链表实现，即使用链表处理冲突，**同一hash值的链表都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。**而JDK1.8中，HashMap采用位桶+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树**，这样大大减少了查找时间。

**简单说下HashMap的实现原理：**

**首先有一个每个元素都是链表（可能表述不准确）的数组，当添加一个元素（key-value）时，就首先计算元素key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，但是形成了链表，同一各链表上的Hash值是相同的，所以说数组存放的是链表。而当链表长度太长时(8)，链表就转换为红黑树，这样大大提高了查找的效率。**

​     当链表数组的容量超过初始容量的0.75时，再散列将链表数组扩大2倍，把原链表数组的搬移到新的数组中

即HashMap的原理图是：
![img](http://img.blog.csdn.net/20160605101246837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**一，JDK1.8中的涉及到的数据结构**

1. 位桶数组

```
transient Node<k,v>[] table;//存储（位桶）的数组</k,v>  
```

2. 数组元素Node<K,V>实现了Entry接口

```
//Node是单向链表，它实现了Map.Entry接口  
static class Node<k,v> implements Map.Entry<k,v> {  
    final int hash;  
    final K key;  
    V value;  
    Node<k,v> next;  
    //构造函数Hash值 键 值 下一个节点  
    Node(int hash, K key, V value, Node<k,v> next) {  
        this.hash = hash;  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }  
   
    public final K getKey()        { return key; }  
    public final V getValue()      { return value; }  
    public final String toString() { return key + = + value; }  
   
    public final int hashCode() {  
        return Objects.hashCode(key) ^ Objects.hashCode(value);  
    }  
   
    public final V setValue(V newValue) {  
        V oldValue = value;  
        value = newValue;  
        return oldValue;  
    }  
    //判断两个node是否相等,若key和value都相等，返回true。可以与自身比较为true  
    public final boolean equals(Object o) {  
        if (o == this)  
            return true;  
        if (o instanceof Map.Entry) {  
            Map.Entry<!--?,?--> e = (Map.Entry<!--?,?-->)o;  
            if (Objects.equals(key, e.getKey()) &&  
                Objects.equals(value, e.getValue()))  
                return true;  
        }  
        return false;  
    } 
```

3. 红黑树

```
//红黑树  
static final class TreeNode<k,v> extends LinkedHashMap.Entry<k,v> {  
    TreeNode<k,v> parent;  // 父节点  
    TreeNode<k,v> left; //左子树  
    TreeNode<k,v> right;//右子树  
    TreeNode<k,v> prev;    // needed to unlink next upon deletion  
    boolean red;    //颜色属性  
    TreeNode(int hash, K key, V val, Node<k,v> next) {  
        super(hash, key, val, next);  
    }  
   
    //返回当前节点的根节点  
    final TreeNode<k,v> root() {  
        for (TreeNode<k,v> r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    }  
```

**二，源码中的数据域**

**加载因子（默认0.75）：为什么需要使用加载因子，为什么需要扩容呢**？**因为如果填充比很大，说明利用的空间很多，如果一直不进行扩容的话，链表就会越来越长，这样查找的效率很低，因为链表的长度很大（当然最新版本使用了红黑树后会改进很多），扩容之后，将原来链表数组的每一个链表分成奇偶两个子链表分别挂在新链表数组的散列位置，这样就减少了每个链表的长度，增加查找效率**

HashMap本来是以空间换时间，所以填充比没必要太大。但是填充比太小又会导致空间浪费。如果关注内存，填充比可以稍大，如果主要关注查找性能，填充比可以稍小。

```
public class HashMap<k,v> extends AbstractMap<k,v> implements Map<k,v>, Cloneable, Serializable {  
    private static final long serialVersionUID = 362498820763181265L;  
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  
    static final int MAXIMUM_CAPACITY = 1 << 30;//最大容量  
    static final float DEFAULT_LOAD_FACTOR = 0.75f;//填充比  
    //当add一个元素到某个位桶，其链表长度达到8时将链表转换为红黑树  
    static final int TREEIFY_THRESHOLD = 8;  
    static final int UNTREEIFY_THRESHOLD = 6;  
    static final int MIN_TREEIFY_CAPACITY = 64;  
    transient Node<k,v>[] table;//存储元素的数组  
    transient Set<map.entry<k,v>> entrySet;  
    transient int size;//存放元素的个数  
    transient int modCount;//被修改的次数fast-fail机制  
    int threshold;//临界值 当实际大小(容量*填充比)超过临界值时，会进行扩容   
    final float loadFactor;//填充比（......后面略）  
```

**三，HashMap的构造函数**

HashMap的构造方法有4种，主要涉及到的参数有，指定初始容量，指定填充比和用来初始化的Map

```
//构造函数1  
public HashMap(int initialCapacity, float loadFactor) {  
    //指定的初始容量非负  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException(Illegal initial capacity:  +  
                                           initialCapacity);  
    //如果指定的初始容量大于最大容量,置为最大容量  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    //填充比为正  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException(Illegal load factor:  +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);//新的扩容临界值  
}  
   
//构造函数2  
public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}  
   
//构造函数3  
public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted  
}  
   
//构造函数4用m的元素初始化散列映射  
public HashMap(Map<!--? extends K, ? extends V--> m) {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
    putMapEntries(m, false);  
}  
```

**四，HashMap的存取机制**

**get(Objet key)**

```
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        /*找到插入的第一个Node，方法是hash值和n-1相与，这种方式相当于取模，tab[(n - 1) & hash]*/
        //也就是说在一条链上的hash值相同的
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    //说明元素是以树的形式组织的
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    //说明元素还是以链表的形式组织的
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

get(key)方法时获取key的hash值，计算hash&(n-1)得到在链表数组中的位置first=tab[hash&(n-1)],先判断first的key是否与参数key相等，不等就遍历后面的链表找到相同的key值返回对应的Value值即可.

**put(key，value)**

```
public V put(K key, V value) {  
        return putVal(hash(key), key, value, false, true);  
    }  
     /** 
     * Implements Map.put and related methods 
     * 
     * @param hash hash for key 
     * @param key the key 
     * @param value the value to put 
     * @param onlyIfAbsent if true, don't change existing value 
     * @param evict if false, the table is in creation mode. 
     * @return previous value, or null if none 
     */  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
                   boolean evict) {  
        Node<K,V>[] tab;   
    Node<K,V> p;   
    int n, i;  
        if ((tab = table) == null || (n = tab.length) == 0)  
            n = (tab = resize()).length;  
    /*如果table的在（n-1）&hash的值是空，就新建一个节点插入在该位置*/  
        if ((p = tab[i = (n - 1) & hash]) == null)  
            tab[i] = newNode(hash, key, value, null);  
    /*表示有冲突,开始处理冲突*/  
        else {  
            Node<K,V> e;   
        K k;  
    /*检查第一个Node，p是不是要找的值*/  
            if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))  
                e = p;  
            else if (p instanceof TreeNode)  
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
            else {  
                for (int binCount = 0; ; ++binCount) {  
        /*指针为空就挂在后面*/  
                    if ((e = p.next) == null) {  
                        p.next = newNode(hash, key, value, null);  
               //如果冲突的节点数已经达到8个，看是否需要改变冲突节点的存储结构，　　　　　　　　　　　　　  
　　　　　　　　　　　　//treeifyBin首先判断当前hashMap的容量，如果不足64，只进行  
                        //resize，扩容table，如果达到64，那么将冲突的存储结构为红黑树  
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                            treeifyBin(tab, hash);  
                        break;  
                    }  
        /*如果有相同的key值就结束遍历*/  
                    if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))  
                        break;  
                    p = e;  
                }  
            }  
    /*就是链表上有相同的key值*/  
            if (e != null) { // existing mapping for key，就是key的Value存在  
                V oldValue = e.value;  
                if (!onlyIfAbsent || oldValue == null)  
                    e.value = value;  
                afterNodeAccess(e);  
                return oldValue;//返回存在的Value值  
            }  
        }  
        ++modCount;  
     /*如果当前大小大于门限，门限原本是初始容量*0.75*/  
        if (++size > threshold)  
            resize();//扩容两倍  
        afterNodeInsertion(evict);  
        return null;  
    }  
```

**treeifyBin**

1. 先根据hash值索引到tab中指定的列；
2. 将该list中的所有Node类型的节点元素转换成TreeNode类型的元素，形式还是List；
3. TreeNode组成的List重构城红黑树；

```
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            //如果table的容量 < 64，则只进行resize()
            resize();
        //根据hash值定位到指定的某列
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                //将tab中的该list中的Node类型的元素用TreeNode替换
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
            	//生成红黑树
                hd.treeify(tab);
        }
    }
```

**treeify**

```
        /**
         * Forms tree of the nodes linked from this node.
         * @return root of tree
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }
```

**resize()**

```
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    /**
     *初始化或者将size扩至2倍大小。如果满了，就分配符合初始容量目标下的门阀值
     *否则，因为我们是进行2的幂的扩展操作，每个位桶处的数据要么呆在相同的索引处，要么移动
     *处，要么移动2的幂的位移量。
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;//超过1>>30大小,无法扩容只能改变 阈值
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold     //门限值*2
        }
        else if (oldThr > 0) // initial capacity was placed in threshold 
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults   初始化操作
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//table在这里产生了。
        table = newTab;//下面对原table中已存储的数据进行迁移，分树和链表2种情况处理
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;//将单节点移动到新位置
                    else if (e instanceof TreeNode)         //下面是分开2种情况操作，一种是发生碰撞的节点以树结构进行存储，另一种是以链表结构存储
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);//这里是处理树的情况
                    else { // preserve order 保持顺序
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {//根据hash值与oldCap的运算结果，将链表中集结的元素分开
                                if (loTail == null)     //运算结果为0的元素，用lo记录并连接成新的链表
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {                      //运算结果不为0的数据，
                                if (hiTail == null)         //用li记录
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead; //lo仍然放在“原处”，这个“原处”是根据新的hash值算出来的。
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;//li放在j+oldCap位置
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

