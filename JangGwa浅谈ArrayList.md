---
title: JangGwa浅谈ArrayList
date: 2016-08-30 15:36:25
tags: Java
---
### ArrayList介绍
对于 ArrayList ，相信大家都很熟悉，天天都会接触到它。JangGwa 从源码角度再和你熟悉一遍，这边先简单介绍下 ArrayList 。<!-- more-->

1.基于数组实现，是一个**动态数组**，其容量能自动增长。

2.ArrayList**不是线程安全的**，建议在单线程中使用，多线程可以选择 Vector 或 CopyOnWriteArrayList 。

3.实现了 RandomAccess 接口，可以**通过下标序号进行快速访问**。

4.实现了 Cloneable 接口，能被**克隆**。

5.实现了 Serializable 接口，支持**序列化**。

### ArrayList源码解析

ArrayList 继承了 AbstractList 并实现了 List,RandomAccess, Cloneable, java.io.Serializable 接口，上面做了相应的介绍就不再阐述了。关键我们看两个重要的属性 **elementData** 和 **size**。

**elementData:**保存了添加到 ArrayList 中的元素。实际上，elementData 是个动态数组，我们能通过构造函数 `ArrayList(int initialCapacity)` 来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数`ArrayList()`来创建 ArrayList ，则elementData的容量默认是10。

**size:** 动态数组的实际大小。

```java
public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

private static final long serialVersionUID = 8683452581122892189L;

private transient Object[] elementData;

private int size;

// ArrayList带容量大小的构造函数。
public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
}

// ArrayList无参构造函数。默认容量是10。
public ArrayList() {
    this(10);
}
```

elementData 数组的大小会根据 ArrayList 容量的增长而动态的增长，具体的增长方式，下面的`ensureCapacity()`函数会告诉你答案。

```java
// 若ArrayList的容量不足以容纳当前的全部元素，设置新的容量=“(原始容量x3)/2 + 1”
public void ensureCapacity(int minCapacity) {
    modCount++;
    int oldCapacity = elementData.length;
    if (minCapacity > oldCapacity) {
        Object oldData[] = elementData;
        int newCapacity = (oldCapacity * 3)/2 + 1;
        if (newCapacity < minCapacity)
            newCapacity = minCapacity;
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```


**定位**

```java
// 获取index位置的元素值
public E get(int index) {
    RangeCheck(index);

    return (E) elementData[index];
}

private void RangeCheck(int index) {
if (index >= size)
    throw new IndexOutOfBoundsException(
    "Index: "+index+", Size: "+size);
}
```


**添加**


```java
// 添加元素e
public boolean add(E e) {
    // 确定ArrayList的容量大小
    ensureCapacity(size + 1);  // Increments modCount!!
    // 添加e到ArrayList中
    elementData[size++] = e;
    return true;
}
```


```java
// 将e添加到ArrayList的指定位置
public void add(int index, E element) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(
        "Index: "+index+", Size: "+size);

    ensureCapacity(size+1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
         size - index);
    elementData[index] = element;
    size++;
}

// 将集合c追加到ArrayList中
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacity(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

// 从index位置开始，将集合c添加到ArrayList
public boolean addAll(int index, Collection<? extends E> c) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(
        "Index: " + index + ", Size: " + size);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacity(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
             numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```



**删除**

```java
// 删除ArrayList指定位置的元素
public E remove(int index) {
    RangeCheck(index);

    modCount++;
    E oldValue = (E) elementData[index];

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
             numMoved);
    elementData[--size] = null; // Let gc do its work

    return oldValue;
}

// 删除ArrayList的指定元素
public boolean remove(Object o) {
    if (o == null) {
            for (int index = 0; index < size; index++)
        if (elementData[index] == null) {
            fastRemove(index);
            return true;
        }
    } else {
        for (int index = 0; index < size; index++)
        if (o.equals(elementData[index])) {
            fastRemove(index);
            return true;
        }
    }
    return false;
}

// 快速删除第index个元素
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    // 从"index+1"开始，用后面的元素替换前面的元素。
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // Let gc do its work
}

//删除元素
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
        if (elementData[index] == null) {
            fastRemove(index);
        return true;
        }
    } else {
        // 便利ArrayList，找到“元素o”，则删除，并返回true。
        for (int index = 0; index < size; index++)
        if (o.equals(elementData[index])) {
            fastRemove(index);
        return true;
        }
    }
    return false;
}

// 删除fromIndex到toIndex之间的全部元素。
protected void removeRange(int fromIndex, int toIndex) {
	modCount++;
	int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);
    }
```

**清空 ArrayList**

```java
public void clear() {
    modCount++;

    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```


**克隆函数**

```java
public Object clone() {
    try {
        ArrayList<E> v = (ArrayList<E>) super.clone();
        // 将当前ArrayList的全部元素拷贝到v中
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError();
    }
}
```

### 总结
- ArrayList 实际上是通过一个数组去保存数据的。当我们使用无参构造函数构造 ArrayList 时，则 ArrayList 的**默认容量大小是10**。
- 当ArrayList容量不足以容纳全部元素时，ArrayList会重新设置容量：**新的容量=“(原始容量x3)/2 + 1”**;如果设置后的新容量还不够，则直接把**新容量设置为传入的参数**。
- ArrayList **查找效率高，插入删除元素的效率低。**
- ArrayList 的克隆函数，即是将全部元素克隆到一个数组中。
- ArrayList 实现 java.io.Serializable 的方式。当写入到输出流时，先写入“容量”，再依次写入“每一个元素”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。

- **toArray()会抛出“java.lang.ClassCastException”异常**



 **原因**: `toArray()`返回的是 Object[] 数组，**Java不支持向下转型。**(例如，将Object[]转换为的Integer[])

 **解决方案**

```java
public static Integer[] vectorToArray2(ArrayList<Integer> v) {
	Integer[] newText = (Integer[])v.toArray(new Integer[0]);
	return newText;
}
```

