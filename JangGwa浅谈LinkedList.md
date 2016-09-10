---
title: JangGwa浅谈LinkedList
date: 2016-08-31 15:22:52
tags: Java
---
### LinkedList介绍
JangGwa 从源码角度带你再熟悉一次 LinkedList ，首先简单介绍下 LinkedList 。<!-- more -->

1.基于**双向链表**实现，链表无容量限制。

2.LinkedList 是**非线程安全**的。

3.实现了List接口，实现了`**get(int location)`、`remove(int location)`**等根据索引值来获取、删除节点的函数。

4.实现了 Deque 接口，可以将 LinkedList 当做**双端队列使用**。

5.实现了 Cloneable 接口，能被**克隆**。

6.实现了 Serializable 接口，支持**序列化**。

### LinkedList源码解析

LinkedList 继承了 AbstractSequentialList 并实现了 List , Deque ,  Cloneable  , java.io.Serializable 接口，上面做了相应的介绍就不再阐述了。关键我们看两个重要的属性 **header **和 **size** 。

**header:**双向链表的表头，它是双向链表节点所对应的类 Entry 的实例。Entry 中包含成员变量： previous, next, element 。其中，previous 是该节点的上一个节点，next 是该节点的下一个节点，element 是该节点所包含的值。

**size:**双向链表中节点的个数


```java
public class LinkedList<E>
extends AbstractSequentialList<E>
implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
// 链表的表头，表头不包含任何数据。Entry是个链表类数据结构。
private transient Entry<E> header = new Entry<E>(null, null, null);

// LinkedList中元素个数
private transient int size = 0;

// 默认构造函数：创建一个空的链表
public LinkedList() {
    header.next = header.previous = header;
}

// 包含“集合”的构造函数:创建一个包含“集合”的LinkedList
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

**双向链表的节点所对应的数据结构**。
​    
```java
private static class Entry<E> {
    // 当前节点所包含的值
    E element;
    // 下一个节点
    Entry<E> next;
    // 上一个节点
    Entry<E> previous;

    Entry(E element, Entry<E> next, Entry<E> previous) {
        this.element = element;
        this.next = next;
        this.previous = previous;
    }
}
```

**获取链表指定位置节点**

将 index 与长度 size 的一半比较，如果 index < size/2，就只从位置0往后遍历到位置 index 处，而如果 index >size/2，就只从位置 size 往前遍历到位置 index 处。
​    
```java
private Entry<E> entry(int index) {
    if (index < 0 || index >= size)
        throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
    Entry<E> e = header;
    // 获取index处的节点。
    // 若index < 双向链表长度的1/2,则从前先后查找;
    // 否则，从后向前查找。
    if (index < (size >> 1)) {
        for (int i = 0; i <= index; i++)
            e = e.next;
    } else {
        for (int i = size; i > index; i--)
            e = e.previous;
    }
    return e;
}
```

**get()和set()**

```java
 // 返回LinkedList指定位置的元素
public E get(int index) {
    return entry(index).element;
}

// 设置index位置对应的节点的值为element
public E set(int index, E element) {
    Entry<E> e = entry(index);
    E oldVal = e.element;
    e.element = element;
    return oldVal;
}
```


**添加**

```java
// 将e添加到当前节点的前面
    public void add(E e) {
        checkForComodification();
        lastReturned = header;
        addBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

// 将元素添加到LinkedList的起始位置
public void addFirst(E e) {
    addBefore(e, header.next);
}

// 将元素添加到LinkedList的结束位置
public void addLast(E e) {
    addBefore(e, header);
}

// 将元素(E)添加到LinkedList中
public boolean add(E e) {
    // 将节点(节点数据是e)添加到表头(header)之前。
    // 即，将节点添加到双向链表的末端。
    addBefore(e, header);
    return true;
}
```


**删除**

```java
// 从LinkedList中删除元素(o)
public boolean remove(Object o) {
    if (o==null) {
        // 若o为null的删除情况
        for (Entry<E> e = header.next; e != header; e = e.next) {
            if (e.element==null) {
                remove(e);
                return true;
            }
        }
    } else {
        // 若o不为null的删除情况
        for (Entry<E> e = header.next; e != header; e = e.next) {
            if (o.equals(e.element)) {
                remove(e);
                return true;
            }
        }
    }
    return false;
}

// 将节点从链表中删除
private E remove(Entry<E> e) {
    if (e == header)
        throw new NoSuchElementException();

    E result = e.element;
    e.previous.next = e.next;
    e.next.previous = e.previous;
    e.next = e.previous = null;
    e.element = null;
    size--;
    modCount++;
    return result;
}
```


​    
**清空LinkedList**
​    
```java
// 清空双向链表
public void clear() {
    Entry<E> e = header.next;
    // 从表头开始，逐个向后遍历；对遍历到的节点执行一下操作：
    // (01) 设置前一个节点为null 
    // (02) 设置当前节点的内容为null 
    // (03) 设置后一个节点为“新的当前节点”
    while (e != header) {
        Entry<E> next = e.next;
        e.next = e.previous = null;
        e.element = null;
        e = next;
    }
    header.next = header.previous = header;
    // 设置大小为0
    size = 0;
    modCount++;
}
```

**克隆**

```java
public Object clone() {
    LinkedList<E> clone = null;
    // 克隆一个LinkedList克隆对象
    try {
        clone = (LinkedList<E>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError();
    }

    // 新建LinkedList表头节点
    clone.header = new Entry<E>(null, null, null);
    clone.header.next = clone.header.previous = clone.header;
    clone.size = 0;
    clone.modCount = 0;

    // 将链表中所有节点的数据都添加到克隆对象中
    for (Entry<E> e = header.next; e != header; e = e.next)
        clone.add(e.element);

    return clone;
}
```


### 总结

- LinkedList 的实现是基于**双向循环链表**的。有一个重要的内部类: **Entry** ，是双向链表节点所对应的数据结构。
- 不存在容量不足的问题。
- LinkedList 是基于链表实现的，因此**插入删除效率高，查找效率低**。
- LinkedList 的克隆函数，即是将全部元素克隆到一个新的 LinkedList 对象中。
- LinkedList 实现 java.io.Serializable 。当写入到输出流时，先写入“容量”，再依次写入“每一个节点保护的值”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。

