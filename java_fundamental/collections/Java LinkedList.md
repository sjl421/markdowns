# Java LinkedList

## 一、概述

​      LinkedList与ArrayList一样实现List接口，只是ArrayList是List接口的大小可变数组的实现，LinkedList是List接口**双向链表**的实现。基于链表实现的方式使得LinkedList在插入和删除时更优于ArrayList，而随机访问则比ArrayList逊色些。

​      LinkedList实现所有可选的列表操作，并允许所有的元素包括null。

​      除了实现 List 接口外，LinkedList 类还为在列表的开头及结尾 get、remove 和 insert 元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。

​      此类实现 Deque 接口，为 add、poll 提供先进先出队列操作，以及其他堆栈和双端队列操作。

> 实现的List接口和Deque接口可以使得LinkedList实现双向链表和队列的功能；

​      所有操作都是按照双重链接列表的需要执行的。在列表中编索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。

​      同时，与ArrayList一样此实现不是同步的,如果需要保证LinkedList在非同步操作下的安全性，最好使用(最好的还是不要使用LinkedList类，换成其他的线程安全的类)

​     LinkList的迭代器也是fail-fast的；在interator创建后所有不是经由list自身的interator而引起的list `structurally modified `都将抛出`ConcurrentModificationException`

```
 List list = Collections.synchronizedList(new LinkedList(...));
```

​      （以上摘自JDK 6.0 API）。

## 二、源码分析

###       2.1、定义

​      首先我们先看LinkedList的定义：

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

​      从这段代码中我们可以清晰地看出LinkedList继承AbstractSequentialList，实现List、Deque、Cloneable、Serializable。其中AbstractSequentialList提供了 List 接口的骨干实现，从而最大限度地减少了实现受“连续访问”数据存储（如链接列表）支持的此接口所需的工作,从而以减少实现List接口的复杂度。Deque一个线性 collection，支持在两端插入和移除元素，定义了双端队列的操作。

### 2.2、属性

​      在LinkedList中提供了两个基本属性size、header。

```
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
     //指向第一个元素
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
     //指向最后一个元素
    transient Node<E> last;
```

由于LinkList的数据结构是双向链表，所以需要同时保留从头和尾访问链表的起点。

​      first表示链表的表头， last表示的是链表的表尾，Node为节点对象。

```
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

​      上面为Entry对象的源代码，Entry为LinkedList的内部类，它定义了存储的元素。该元素的前一个元素、后一个元素，这是典型的双向链表定义方式。

### 2.3、构造方法

​      LinkedList提供了两个构造方法：LinkedList()和LinkedList(Collection<? extends E> c)。

```
    //默认构造函数，什么都没干
    public LinkedList() {
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     */
    //构造一个LinkList,并将入参中的所有元素都包含进来；
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

​      LinkedList()构造一个空列表。里面没有任何元素，仅仅只是将header节点的前一个元素、后一个元素都指向自身。

​      LinkedList(Collection<? extends E> c)： 构造一个包含指定 collection 中的元素的列表，这些元素按其 collection 的迭代器返回的顺序排列。该构造函数首先会调用LinkedList()，构造一个空列表，然后调用了addAll()方法将Collection中的所有元素添加到列表中。以下是addAll()的源代码：

jdk 1.7 中的实现：

 ```java
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
          	//node函数，来获取指定位置的节点，从这个节点开始添加c中的元素
            succ = node(index);
            pred = succ.prev;
        }
		//将c中的元素添加到list中
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }
		//更新size
        size += numNew;
      	//更新modCount，这里只加1，并不是加入参c的length
        modCount++;
        return true;
    }
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

 ```

在addAll()方法中，涉及到了两个方法，一个是node(int index)，该方法为LinkedList的私有方法，主要是用来查找index位置的节点元素。

```java
	//node()方法在获取通过索引的位置来决定是从头部开始遍历和是从尾部开始遍历来获取指定位置的节点；
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

​      从该方法有两个遍历方向中我们也可以看出LinkedList是双向链表，

> (jdk 1.6) 这也是在构造方法中为什么需要将header的前、后节点均指向自己。

​      如果对数据结构有点了解，对上面所涉及的内容应该问题，我们只需要清楚一点：LinkedList是双向链表，其余都迎刃而解。

​      由于篇幅有限，下面将就LinkedList中几个常用的方法进行源码分析。

### 2.4、增加方法

​      add(E e): 将指定元素添加到此列表的结尾。

```
    public void addFirst(E e) {
        linkFirst(e);
    }
```

​      该方法调用linkFirst方法，然后直接返回true，对于linkFirst()而已，它为LinkedList的私有方法。

```
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

​      在linkFirst方法中无非就是做了这件事：构建一个新节点newEntry，然后修改其前后的引用。

​      LinkedList还提供了其他的增加方法：

​      add(int index, E element)：在此列表中指定的位置插入指定的元素。

​      addAll(Collection<? extends E> c)：添加指定 collection 中的所有元素到此列表的结尾，顺序是指定 collection 的迭代器返回这些元素的顺序。

​      addAll(int index, Collection<? extends E> c)：将指定 collection 中的所有元素从指定位置开始插入此列表。

​      addFirst(E e): 将指定元素插入此列表的开头。

​     	 addLast(E e): 将指定元素添加到此列表的结尾。

### 2.5、移除方法

​      remove(Object o)：从此列表中移除首次出现的指定元素（如果存在）。该方法的源代码如下：

```
public boolean remove(Object o) {
        if (o==null) {
            for (Entry<E> e = header.next; e != header; e = e.next) {
                if (e.element==null) {
                    remove(e);
                    return true;
                }
            }
        } else {
            for (Entry<E> e = header.next; e != header; e = e.next) {
                if (o.equals(e.element)) {
                    remove(e);
                    return true;
                }
            }
        }
        return false;
    }
```

​      该方法首先会判断移除的元素是否为null，然后迭代这个链表找到该元素节点，最后调用remove(Entry<E> e)，remove(Entry<E> e)为私有方法，是LinkedList中所有移除方法的基础方法，如下：

```
private E remove(Entry<E> e) {
        if (e == header)
            throw new NoSuchElementException();

        //保留被移除的元素：要返回
        E result = e.element;
        
        //将该节点的前一节点的next指向该节点后节点
        e.previous.next = e.next;
        //将该节点的后一节点的previous指向该节点的前节点
        //这两步就可以将该节点从链表从除去：在该链表中是无法遍历到该节点的
        e.next.previous = e.previous;
        //将该节点归空
        e.next = e.previous = null;
        e.element = null;
        size--;
        modCount++;
        return result;
    }
```

​      其他的移除方法：

​      clear()： 从此列表中移除所有元素。

​      remove()：获取并移除此列表的头（第一个元素）。

​      remove(int index)：移除此列表中指定位置处的元素。

​      remove(Objec o)：从此列表中移除首次出现的指定元素（如果存在）。

​      removeFirst()：移除并返回此列表的第一个元素。

​      removeFirstOccurrence(Object o)：从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表时）。

​      removeLast()：移除并返回此列表的最后一个元素。

​      removeLastOccurrence(Object o)：从此列表中移除最后一次出现的指定元素（从头部到尾部遍历列表时）。

###和队列操作相关的方法

peek（）： Retrieves, but does not remove，如果list为空，返回null

```
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```

element（）： Retrieves, but does not remove，如果list为空，抛异常`NoSuchElementException`

```
    public E element() {
        return getFirst();
    }
```

poll(): Retrieves and remove, 如果list为空，返回null

```
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```

remove(): Retrieves and removes the head (first element) of this list.如果list为空，抛`NoSuchElementException`异常

```
    public E remove() {
        return removeFirst();
    }
```

offer()：在list的尾部插入制定的元素

```
public boolean offer(E e) {
    return add(e);
}
```

push（）：

```
/**
 * Pushes an element onto the stack represented by this list.  In other
 * words, inserts the element at the front of this list.
 *
 * <p>This method is equivalent to {@link #addFirst}.
 *
 * @param e the element to push
 * @since 1.6
 */
public void push(E e) {
    addFirst(e);
}
```

pop():

```
/**
 * Pops an element from the stack represented by this list.  In other
 * words, removes and returns the first element of this list.
 *
 * <p>This method is equivalent to {@link #removeFirst()}.
 *
 * @return the element at the front of this list (which is the top
 *         of the stack represented by this list)
 * @throws NoSuchElementException if this list is empty
 * @since 1.6
 */
public E pop() {
    return removeFirst();
}
```

### 2.6、查找方法

​      对于查找方法的源码就没有什么好介绍了，无非就是迭代，比对，然后就是返回当前值。

​      get(int index)：返回此列表中指定位置处的元素。

```
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

​      getFirst()：返回此列表的第一个元素。

```java
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
```

​      getLast()：返回此列表的最后一个元素。

```java
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
```

​      indexOf(Object o)：返回此列表中首次出现的指定元素的索引，如果此列表中不包含该元素，则返回 -1。

```
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

​      lastIndexOf(Object o)：返回此列表中最后出现的指定元素的索引，如果此列表中不包含该元素，则返回 -1。