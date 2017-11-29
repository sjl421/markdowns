# ConcurrentHashMap

ConcurrentHashMap和HashTable类似，key和value均不能为null（HashMap的key或者value可以为null）；

ConcurrentHashMap的实现是 数组+链表+红黑树实现的，内部所有的元素都存放在Node内部类中；

### ConcurrentHashMap的实现

ConcurrentHashMap内部实现了很多内部类。所有的key-value 对都存放在Node中。红黑树的节点由TreeNode组成；TreeBins记录了红黑树的根节点；



链表的锁机制，将每一个链表的首元素作为锁；

如何实现resizing？

计数器LongAdder的实现

遍历





