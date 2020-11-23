---
title: Java容器类
date: 2020-04-30 23:03:36
tags: 
  - 面试
  - 基础
  - 设计模式
categories: 
  - Java
  - 基础
  - 容器
---
# java集合框架图

java集合框架图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606105558630.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

Java集合的类结构图

![image.png](https://upload-images.jianshu.io/upload_images/4118241-8af2a357fe900055.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

java集合继承关系图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190607095910509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)



在Java容器中一共定义了2种集合, 顶层接口分别是Collection和Map。但是这2个接口都不能直接被实现使用，分别代表两种不同类型的容器。

简单来看，Collection代表的是单个元素对象的序列，（可以有序/无序，可重复/不可重复 等，具体依据具体的子接口Set，List，Queue等）；Map代表的是“键值对”对象的集合（同样可以有序/无序 等依据具体实现）

- List：有序、可重复；索引查询速度快；插入、删除伴随数据移动，速度慢；
- Set：无序，不可重复；
- Map：键值对，键唯一，值多个；

# Collection 集合接口

Collection是最基本的集合接口，存储对象元素集合。一个Collection代表一组Object（元素）。有些容器允许重复元素有的不允许，有些有序有些无需。由Collection接口派生的两个接口是List和Set。这个接口的设计目的是希望能最大程度抽象出元素的操作。

```
public interface Collection<E> extends Iterable<E> {
    ...
}
```

泛型即该Collection中元素对象的类型，继承的Iterable是定义的一个遍历操作接口，采用hasNext next的方式进行遍历。具体实现还是放在具体类中去实现。

主要方法

- boolean add(Object o) 添加对象到集合
- boolean remove(Object o) 删除指定的对象
- int size() 返回当前集合中元素的数量
- boolean contains(Object o) 查找集合中是否有指定的对象
- boolean isEmpty() 判断集合是否为空
- Iterator iterator() 返回一个迭代器
- boolean containsAll(Collection c) 查找集合中是否有集合c中的元素
- boolean addAll(Collection c) 将集合c中所有的元素添加给该集合
- void clear() 删除集合中所有元素
- void removeAll(Collection c) 从集合中删除c集合中也有的元素
- void retainAll(Collection c) 从集合中删除集合c中不包含的元素



## List子接口

List是一个允许重复元素的指定索引、有序集合。

从List接口的方法来看，List接口增加了面向位置的操作，允许在指定位置上操作元素。用户可以使用这个接口精准掌控元素插入，还能够使用索引index（元素在List中的位置，类似于数组下标）来访问List中的元素。List接口有两个重要的实现类：ArrayList和LinkedList。

**Set里面和List最大的区别是Set里面的元素对象不可重复。**

### ArrayList 数组

ArrayList的底层数据结构就是一个数组，数组元素的类型为Object类型，对ArrayList的所有操作底层都是基于数组的。默认列表长度10，也可以自己指定长度。ArrayList中的对象数组的最大数组容量为Integer.MAX_VALUE – 8。

#### 定义

- ArrayList实现了List接口的可变大小的数组。（数组可动态创建，如果元素个数超过数组容量，那么就创建一个更大的新数组）
- 它允许所有元素，包括null
- 它的size, isEmpty, get, set, iterator,add这些方法的时间复杂度是O(1),如果add n个数据则时间复杂度是O(n)
- ArrayList没有同步方法

#### 优点

- ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快。
- ArrayList在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已。
- 根据下标遍历元素，效率高。
- 根据下标访问元素，效率高。
- 可以自动扩容，默认为每次扩容为原来的1.5倍。

#### 缺点

- 插入和删除元素的效率不高。
- 根据元素下标查找元素需要遍历整个元素数组，效率不高。
- 线程不安全。

#### 常用方法

- Boolean add(Object o)将指定元素添加到列表的末尾
- Boolean add(int index,Object element)在列表中指定位置加入指定元素
- Boolean addAll(Collection c)将指定集合添加到列表末尾
- Boolean addAll(int index,Collection c)在列表中指定位置加入指定集合
- Boolean clear()删除列表中所有元素
- Boolean clone()返回该列表实例的一个拷贝
- Boolean contains(Object o)判断列表中是否包含元素
- Boolean ensureCapacity(int m)增加列表的容量,如果必须,该列表能够容纳m个元素
- Object get(int index)返回列表中指定位置的元素
- Int indexOf(Object elem)在列表中查找指定元素的下标
- Int size()返回当前列表的元素个数

#### 常见源码分析

##### add()

```

 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
// 确认能否装得下size+1的对象
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
//计算容量
 private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果是默认长度，就比较默认长度和size+1,取最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

 private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        //如果容量大于数组的长度
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }
private void grow(int minCapacity) {
        //取数组的长度
        int oldCapacity = elementData.length;
        //计算新长度，新长度=旧长度+旧长度/2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //最后按照新容量进行扩容，复制。
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

在add()方法中主要完成了三件事：首先确保能够将希望添加到集合中的元素能够添加到集合中，即确保ArrayList的容量（判断是否需要扩容）；然后将元素添加到elementData数组的指定位置；最后将集合中实际的元素个数加1。

ArrayList的实际默认容量直到调用add()方法才会真正扩容到10，这里通过new ArrayList（）在内存分配的是一个空数组，并没有直接new Object[10],这样设计是很巧妙的，可以节省很多空间。

#### add(int index, E element)方法

```
 public void add(int index, E element) {
    //判断是否越界
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 重新复制数组，把index+1位置往后的对象全部后移
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        //覆盖index位置的对象                 
        elementData[index] = element;
        size++;
    }
```

ArrayList的指定位置添加对象方法，需要把指定位置后面的全部对象后移，所以这样也是ArrayList相对于linkList添加耗时的地方。

#### get(int index)方法

```
   public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

ArrayList的get(int index) 方法比较简单，只有两步，第一，检查是否越界，第二，返回数组索引位置的数据。

#### remove(int index)方法

```
  public E remove(int index) {
        rangeCheck(index);
        
        //父类的属性，用来记录list修改的次数，后续迭代器中会用到
        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
        //把index位置后面的元素左移
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

ArrayList 的remove（int index）方法主要分为 3步:

- 第一步，判断下标是否越界
- 第二步，记录修改次数，并左移index位置后面的元素，
- 第三，把最后位置赋值为null，用于快速垃圾回收。

#### for循环问题

```
   List<Integer> integers = new ArrayList<>(5);
        integers.add(1);
        integers.add(2);
        integers.add(3);
        integers.add(4);
        integers.add(5);

        for (int i = 0; i < integers.size(); i++) {
            integers.remove(i);
        }
        System.out.println(integers.size());
```

这里首先申明一个长度为5的ArrayList的集合，然后添加五个元素，最后通过循环遍历删除，理论结果输出0，但是输出的结果却是2，为什么呢？之前分析remove源码我们知道，ArrayList每删除一次就会把后面的全部元素左移，以这5个元素为例，第一个正常删除没问题，删除后，元素就只剩下[2,3,4,5],这个时候remove(1),还剩[2,4,5],再remove(2),剩下[2,4],后面再remove没有元素了，所以最后size为2。

### LinkedList 链表

**`LinkedList` 保存链表的第一个节点和最后一个节点,每个节点上有三个字段：当前节点的数据字段（data）,指向上一个节点的字段（prev），和指向下一个节点的字段（next）。**

#### 定义

- **`LinkedList` 集合底层实现的数据结构为双向链表**

- **`LinkedList` 集合中元素允许为 null**
- **`LinkedList` 允许存入重复的数据**
- **`LinkedList` 中元素存放顺序为插入顺序。**
- `LinkedList`实现`Deque`接口使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
  - **当 `Deque` 当做队列使用时（FIFO），只需要在头部删除，尾部添加即可**
  - **当`Deque` 作为栈使用的时候，遵循 FILO 准则，入栈和出栈是通过添加和移除头节点实现的**。
- LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。
- 分配内存空间不是连续的。
- 插入、删除操作很快，只要修改前后指针就OK了，时间复杂度为O(1)；
- 访问比较慢，必须得从第一个元素开始遍历，时间复杂度为O(n)；

#### 结点定义（双向链表）

```
private static class Node<E> {
    // 当前节点的元素值
   E item;
   // 下一个节点的索引
   Node<E> next;
   // 上一个节点的索引
   Node<E> prev;

   Node(Node<E> prev, E element, Node<E> next) {
       this.item = element;
       this.next = next;
       this.prev = prev;
   }
}
```

#### 链表定义

每个LinkedList中会持有链表的头指针和尾指针,LinkedList 主要成员变量有下边三个：

```
//LinkedList 中的节点个数
transient int size = 0;
//LinkedList 链表的第一个节点
transient Node<E> first;
//LinkedList 链表的最后一个节点
transient Node<E> last;
```

之所以 LinkedList 要保存链表的第一个节点和最后一个节点是因为，我们都知道，链表数据结构相对于数组结构， 优点在于增删，缺点在于查找。如果我们保存了LinkedList 的头尾两端，当我们需要以索引来查找节点的时候，我们可以根据 `index` 和 `size/2` 的大小,来决定从头查找还是从尾部查找，这也算是一定程度上弥补了单链表数据结构的缺点。

#### LinkedList 的构造函数

```
/**
 * 空参数的构造由于生成一个空链表 first = last = null
 */
 public LinkedList() {
 }

/**
 * 传入一个集合类，来构造一个具有一定元素的 LinkedList 集合
 * @param  c  其内部的元素将按顺序作为 LinkedList 节点
 * @throws NullPointerException 如果 参数 collection 为空将抛出空指针异常
 */
public LinkedList(Collection<? extends E> c) {
   this();
   addAll(c);
}

public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * 在 index 节点前插入包含所有 c 集合元素的节点。
 * 返回值表示是否成功添加了对应的元素.
 */
public boolean addAll(int index, Collection<? extends E> c) {
   // 查看索引是否满足 0 =< index =< size 的要求
   checkPositionIndex(index);
    // 调用对应 Collection 实现类的 toArray 方法将集合转为数组
   Object[] a = c.toArray();
   //检查数组长度，如果为 0 则直接返回 false 表示没有添加任何元素
   int numNew = a.length;
   if (numNew == 0)
       return false;
   // 保存 index 当前的节点为 succ，当前节点的上一个节点为 pred
   Node<E> pred, succ;
   // 如果 index = size 表示在链表尾部插入
   if (index == size) {
       succ = null;
       pred = last;
   } else {
       succ = node(index);
       pred = succ.prev;
   }
    
    // 遍历数组将对应的元素包装成节点添加到链表中
   for (Object o : a) {
       @SuppressWarnings("unchecked") E e = (E) o;
       Node<E> newNode = new Node<>(pred, e, null);
       //如果 pred 为空表示 LinkedList 集合中还没有元素
       //生成的第一个节点将作为头节点 赋值给 first 成员变量
       if (pred == null)
           first = newNode;
       else
           pred.next = newNode;
       pred = newNode;
   }
   // 如果 index 位置的元素为 null 则遍历数组后 pred 所指向的节点即为新链表的末节点，赋值给 last 成员变量
   if (succ == null) {
       last = pred;
   } else {
       // 否则将 pred 的 next 索引指向 succ ，succ 的 prev 索引指向 pred
       pred.next = succ;
       succ.prev = pred;
   }
   // 更新当前链表的长度 size 并返回 true 表示添加成功
   size += numNew;
   modCount++;
   return true;
}
```

LinkedList 批量添加节点的实现。大体分下面几个步骤：

- 检查索引值是否合法，不合法将抛出角标越界异常
- 保存 index 位置的节点，和 index-1 位置的节点。
- 将参数集合转化为数组，循环将数组中的元素封装为节点添加到链表中。
- 更新链表长度并返回添加 true 表示添加成功。

越界检查最后都调用

```
private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```

#### LinkedList 添加节点的方法

LinkedList 作为链表数据结构的实现，不同于数组，它可以方便的在头尾插入一个节点，而 add 方法默认在链表尾部添加节点：

```
/**
 * Inserts the specified element at the beginning of this list.
 *
 * @param e the element to add
 */
 public void addFirst(E e) {
    linkFirst(e);
 }

/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #add}.
 *
 * @param e the element to add
 */
 public void addLast(E e) {
    linkLast(e);
 }
    
/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
 public boolean add(E e) {
    linkLast(e);
    return true;
 }
```

添加方法默认调用`linkLast`添加到最后一个节点。

##### `linkXXX` 方法：`linkFirst、linkLast`

```
 /**
  * 添加一个元素在链表的头节点位置
  */
private void linkFirst(E e) {
   // 添加元素之前的头节点
   final Node<E> f = first;
   //以添加的元素为节点值构建新的头节点 并将 next 指针指向 之前的头节点
   final Node<E> newNode = new Node<>(null, e, f);
   // first 索引指向将新的节点
   first = newNode;
   // 如果添加之前链表空则新的节点也作为未节点
   if (f == null)
       last = newNode;
   else
       f.prev = newNode;//否则之前头节点的 prev 指针指向新节点
   size++;
   modCount++;//操作数++
}

/**
 * 在链表末尾添加一个节点
 */
 void linkLast(E e) {
   final Node<E> l = last;//保存之前的未节点
   //构建新的未节点，并将新节点 prev 指针指向 之前的未节点
   final Node<E> newNode = new Node<>(l, e, null);
   //last 索引指向末节点
   last = newNode;
   if (l == null)//如果之前链表为空则新节点也作为头节点
       first = newNode;
   else//否则将之前的未节点的 next 指针指向新节点
       l.next = newNode;
   size++;
   modCount++;//操作数++
}
```

##### add(int index, E element)、addAll

```
/**
 * 在指定 index 位置插入节点
 */
public void add(int index, E element) {
   // 检查角标是否越界
   checkPositionIndex(index);
   // 如果 index = size 代表是在尾部插入节点
   if (index == size)
       linkLast(element);
   else
       linkBefore(element, node(index));
}
```

###### node(index) 方法的实现

```

/**
 * 返回一个非空节点，这个非空节点位于 index 位置
 */
 Node<E> node(int index) {
   // assert isElementIndex(index);
    // 如果 index < size/2 则从0开始寻找指定角标的节点
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
         // 如果 index >= size/2 则从 size-1 开始寻找指定角标的节点
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
 }
```

大家可能会疑惑为什么这里注释为返回一个非空节点？其实仔细想下就明白了，这里的节点一定不为 null，如果一开始链表为空的时候，index 为 0 的位置肯定为 null，为什么不会产生异常情况呢？其实如果一开始链表中没有元素 size = 0，如果我们向 `index = 0` 的位置添加元素是不会走到 else 中的，而是会调用 `linkLast(element);` 方法去添加元素。 因此**node 方法可以用于根据指定 index 去以 size/2 为界限搜索index 位置的 Node;**

###### linkBefore(E e, Node<E> succ)实现

为什么要叫做 linkBefore 呢，因为在链表的中间位置添加节点，其实就是将 index 原来的节点前添加一个节点，添加节点我们需要知道该节点的前一个节点和当前节点

```
void linkBefore(E e, Node<E> succ) {
   // assert succ != null;
   // 由于 succ 一定不为空，所以可以直接获取 prev 节点
   final Node<E> pred = succ.prev;
   // 新节点 prev 节点为 pred，next 节点为 succ
   final Node<E> newNode = new Node<>(pred, e, succ);
   // 原节点的 prev 指向新节点
   succ.prev = newNode;
   // 如果 pred 为空即头节点出插入了一个节点，则将新的节点赋值给 first 索引
   if (pred == null)
       first = newNode;
   else
       pred.next = newNode;//否则 pred 的下一个节点改为新节点
   size++;
   modCount++;
}
```

- 将构造的新节点前指针 prev 指向 index 的前一个元素，

- 新节点前指针 next 指针指向 index 位置的节点，

- index 位置节点 prev 指针指向新节点

- index 位置前节点（pred）的 next 指针指向新节点

#### LinkedList 删除节点的方法

##### `removeFirst`、`removeLast`

```
/**
 *  删除头节点
 * @return 删除的节点的值 即 节点的 element
 * @throws NoSuchElementException 如果链表为空则抛出异常
 */
 public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
 }

/**
 *  删除尾节点
 *
 * @return  删除的节点的值 即 节点的 element
 * @throws NoSuchElementException  如果链表为空则抛出异常
 */
 public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
 }
```

###### `unlinkFirst`、`unlinkLast`

```
/**
 * 移除头节点
 */
 private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    // 头节点的 element 这里作为返回值使用
    final E element = f.item;
    // 头节点下个节点
    final Node<E> next = f.next;
    // 释放头节点的 next 指针，和 element 下次 gc 的时候回收这个内部类
    f.item = null;
    f.next = null; // help GC
    // 将 first 索引指向新的节点
    first = next;
    // 如果 next 节点为空，即链表只有一个节点的时候，last 指向 null
    if (next == null)
        last = null;
    else
        next.prev = null; //否则 next 的 prev 指针指向 null
    size--;//改变链表长度
    modCount++;//修改操作数
    return element;//返回删除节点的值
 }

/**
 * 移除未节点
 */
 private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    //未节点的前一个节点，
    final Node<E> prev = l.prev;
    //释放未节点的内容
    l.item = null;
    l.prev = null; // help GC
    //将 last 索引指向新的未节点
    last = prev;
    // 链表只有一个节点的时候，first 指向 null
    if (prev == null)
       first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
 }
```

##### remove(int index)

```
public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

刚才已经分析过node(index)方法了，就是找到index位置的节点。

###### unlink(Node<E> x)

```
/**
 * Unlinks non-null node x.
 */
E unlink(Node<E> x) {
   // assert x != null;
   final E element = x.item;
   //保存 index 节点的前后两个节点
   final Node<E> next = x.next;
   final Node<E> prev = x.prev;
    // 如果节点为头节点，则做 unlinkFirst 相同操作
   if (prev == null) {
       first = next;
   } else {//否则将上一个节点的 next 指针指向下个节点
       prev.next = next;
       // 释放 index 位置 prev 指针
       x.prev = null;
   }
    // 如果节点为尾节点，则将 last 索引指向上个节点
   if (next == null) {
       last = prev;
   } else {//否则下个节点 prev 指针指向上个节点
       next.prev = prev;
       x.next = null;
   }

   x.item = null;
   size--;
   modCount++;
   return element;
}
```

看完 `unlink` 操作结合之前说的 `node(index)`，下边两种删除节点的操作，就很好理解了

```
/**
 * 删除指定索引位置的节点
 */
public E remove(int index) {
   checkElementIndex(index);
   return unlink(node(index));
}

/**
 *删除从头节点其第一个与 o 相同的节点
 */
public boolean remove(Object o) {
    // 区别对待 null 元素，比较元素时候使用 == 而不是 equals
   if (o == null) {
       for (Node<E> x = first; x != null; x = x.next) {
           if (x.item == null) {
               unlink(x);
               return true;
           }
       }
   } else {
       for (Node<E> x = first; x != null; x = x.next) {
           if (o.equals(x.item)) {
               unlink(x);
               return true;
           }
       }
   }
   return false;
}
```

看完单个删除节点的方法 LinkedList 实现了 List 接口的 clear 操作，用于删除链表所有的节点：

```
/**
* Removes all of the elements from this list.
* The list will be empty after this call returns.
*/
public void clear() {
   // 依次清除节点，帮助释放内存空间
   for (Node<E> x = first; x != null; ) {
       Node<E> next = x.next;
       x.item = null;
       x.next = null;
       x.prev = null;
       x = next;
   }
   first = last = null;
   size = 0;
   modCount++;
}
```

#### LinkedList 查询节点的方法

LinkedList 查询节点的方法，可分为根据指定的索引查询，获取头节点，获取未节点三种。值得注意的是，根据索引去获取节点内容的效率并不高，所以如果查询操作多余增删操作的时候建议用 `ArrayList` 去替代。

##### `get(int index)、getFirst() 、getLast`

```
/**
* 根据索引查询
*
public E get(int index) {
   checkElementIndex(index);
   return node(index).item;
}

/**
* 返回 first 索引指向的节点的内容
*
* @return the first element in this list
* @throws NoSuchElementException 如果链表为空则抛出异常
*/
public E getFirst() {
   final Node<E> f = first;
   if (f == null)
       throw new NoSuchElementException();
   return f.item;
}

/**
* 返回 last 索引指向的节点的内容
*
* @return the last element in this list
* @throws NoSuchElementException 如果链表为空则抛出异常
*/
public E getLast() {
   final Node<E> l = last;
   if (l == null)
       throw new NoSuchElementException();
   return l.item;
}
```

##### `indexOf(Object o)、lastIndexOf(Object o)`

```
/* 
* 返回参数元素在链表的节点索引，如果有重复元素，那么返回值为从**头节点**起的第一相同的元素节点索引，
* 如果没有值为该元素的节点，则返回 -1；
* 
* @param o element to search for
* @return 
*/
public int indexOf(Object o) {
   int index = 0;
  // 区别对待 null 元素，用 == 判断，非空元素用 equels 方法判断 
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

/**
**返回参数元素在链表的节点索引，如果有重复元素，那么返回值为从**尾节点起**的第一相同的元素节点索引，
* 如果没有值为该元素的节点，则返回 -1；
*
* @param o element to search for
* @return the index of the last occurrence of the specified element in
*         this list, or -1 if this list does not contain the element
*/
public int lastIndexOf(Object o) {
   int index = size;
   if (o == null) {
       for (Node<E> x = last; x != null; x = x.prev) {
           index--;
           if (x.item == null)
               return index;
       }
   } else {
       for (Node<E> x = last; x != null; x = x.prev) {
           index--;
           if (o.equals(x.item))
               return index;
       }
   }
   return -1;
}
```

两个方法分别返回从**头节点起**第一个与参数元素相同的节点索引，和从**尾节点起**第一个与参数元素相同的节点索引。

##### `contains(Object o)`

```
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

调用 indexOf 从头结点开始查询元素位置遍历完成后若 返回值 !=-1 则表示存在，反之不存在

#### LinkedList 的修改节点方法

`LinkedList` 只提供了 `set(int index, E element)` 一个方法。

```
public E set(int index, E element) {
   // 判断角标是否越界
   checkElementIndex(index);
   // 采用 node 方法查找对应索引的节点
   Node<E> x = node(index);
   //保存节点原有的内容值
   E oldVal = x.item;
   // 设置新值
   x.item = element;
   // 返回旧的值
   return oldVal;
}
```

#### LinkedList 作为双向队列的增删改查

`LinkedList`实现`Deque`接口，可实现双向队列。

##### Deque 双端队列

`Queue` 是一个队列，遵循 FIFO 准则，我们也知道 `Stack` 是一个栈结构，遵循 FILO 准则。 而`Deque` 这个双端队列就厉害了,它既可以实现栈的操作，也可以实现队列的操作，换句话说，实现了这个接口的类，既可以作为栈使用也可以作为队列使用。

Deque接口中的方法:

|      |     头部      |     头部      |    尾部     |     尾部     |
| :--- | :-----------: | :-----------: | :---------: | :----------: |
| 插入 |  addFirst(e)  | offerFirst(e) | addLast(e)  | offerLast(e) |
| 移除 | removeFirst() |  pollFirst()  | remveLast() |   pollLast   |
| 获取 |  getFirst()   |  peekFirst()  |  getLast()  |   peekLast   |

由于 `Deque` 接口继承 `Queue` 接口，**当 `Deque` 当做队列使用时（FIFO），只需要在头部删除，尾部添加即可**。我们现在复习下 `Queue` 中的方法及区别：

- `Queue` 的 `offer` 和 `add` 都是在队列中插入一个元素，具体区别在于，对于一些 Queue 的实现的队列是有大小限制的，因此如果想在一个满的队列中加入一个新项，多出的项就会被拒绝。此时调用 `add()`方法会抛出异常，而 `offer()` 只是返回的 false。
- `remove()` 和 `poll()` 方法都是从队列中删除第一个元素。remove()也将抛出异常，而 `poll()` 则会返回 `null`
- `element()` 和 `peek()` 用于在队列的头部查询元素。在队列为空时， `element()` 抛出一个异常，而 `peek()` 返回 `null`。

##### Deque 和 Queue 添加元素的方法

```
// queue 的添加方法实现，
public boolean add(E e) {
   linkLast(e);
   return true;
}
// Deque 的添加方法实现，
public void addLast(E e) {
   linkLast(e);
} 
  
// queue 的添加方法实现，
public boolean offer(E e) {
   return add(e);
}

// Deque 的添加方法实现，
public boolean offerLast(E e) {
        addLast(e);
        return true;
}
```

上面提及到 `Queue`的 `offer` 和 `add` 的区别针对容量有限制的实现，很明显 `LinkedList` 的大小并没有限制，所以在 `LinkedList` 中他们的实现并没有实质性不同。

##### Deque 和 Queue 删除元素的方法

```
// Queue 删除元素的实现 removeFirst 会抛出 NoSuchElement 异常
public E remove() {
   return removeFirst();
}

// Deque 的删除方法实现
public E removeFirst() {
   final Node<E> f = first;
   if (f == null)
       throw new NoSuchElementException();
   return unlinkFirst(f);
}
    
// Queue 删除元素的实现 不会抛出异常 如果链表为空则返回 null 
public E poll() {
   final Node<E> f = first;
   return (f == null) ? null : unlinkFirst(f);
}

// Deque 删除元素的实现 不会抛出异常 如果链表为空则返回 null 
public E pollFirst() {
   final Node<E> f = first;
   return (f == null) ? null : unlinkFirst(f);
}
```

#### Deque 和 Queue 获取队列头部元素的实现

```
 // Queue 获取队列头部的实现 队列为空的时候回抛出异常
 public E element() {
    return getFirst();
 }
// Deque 获取队列头部的实现 队列为空的时候回抛出异常
public E getFirst() {
   final Node<E> f = first;
   if (f == null)
       throw new NoSuchElementException();
   return f.item;
}

// Queue 获取队列头部的实现 队列为空的时候返回 null
public E peek() {
   final Node<E> f = first;
   return (f == null) ? null : f.item;
}

// Deque 获取队列头部的实现 队列为空的时候返回 null
public E peekFirst() {
   final Node<E> f = first;
   return (f == null) ? null : f.item;
}
```

上述我们分析了，双端队列作为队列使用的时候的各个方法的区别，也可是看出 `LinkedList` 对对应方法的实现，遵循了队列设计原则。

#### `LinkedList`作为`Stack`使用

下面我们来看看下双端队列作为栈 `Stack`使用的时候方法对应关系，与 `Queue` 不同，`Stack` 本身就是实现类，他拥有 FILO 的原则， `Stack` 的入栈操作通过 `push` 方法进行，出栈操作通过 `pop` 方法进行，查询操作通过 `peek` 操作进行。 **`Deque` 作为栈使用的时候，也遵循 FILO 准则，入栈和出栈是通过添加和移除头节点实现的。**

| Stack   | Deque         |
| ------- | ------------- |
| push(e) | addFist(e)    |
| pop()   | removeFirst() |
| peek()  | peekFirst()   |

`LinkedList`中的`push`、`pop`、`peek`方法：

```
//在头部添加一个元素
public void push(E e) {
   addFirst(e);
}

//获取并删除第一个元素
public E pop() {
   return removeFirst();
}


//获取不删除第一个元素
public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
}
```

#### LinkedList 的遍历

在 `ArrayList` 分析的时候，我们就知道 `List` 的实现类，有4中遍历方式：for 循环，高级 for 循环，`Iterator` 迭代器方法， `ListIterator` 迭代方法。

`LinkedList` 没有单独 `Iterator` 实现类，它的 `iterator` 和 `listIterator` 方法均返回 `ListItr`的一个对象。 LinkedList 作为双向链表数据结构，获取上个元素和下个元素很方便。

```
private class ListItr implements ListIterator<E> {
   // 上一个遍历的节点
   private Node<E> lastReturned;
   // 下一次遍历返回的节点
   private Node<E> next;
   // cursor 指针下一次遍历返回的节点
   private int nextIndex;
   // 期望的操作数
   private int expectedModCount = modCount;
    
   // 根据参数 index 确定生成的迭代器 cursor 的位置
   ListItr(int index) {
       // assert isPositionIndex(index);
       // 如果 index == size 则 next 为 null 否则寻找 index 位置的节点
       next = (index == size) ? null : node(index);
       nextIndex = index;
   }

   // 判断指针是否还可以移动
   public boolean hasNext() {
       return nextIndex < size;
   }
    
  // 返回下一个带遍历的元素
  public E next() {
       // 检查操作数是否合法
       checkForComodification();
       // 如果 hasNext 返回 false 抛出异常，所以我们在调用 next 前应先调用 hasNext 检查
       if (!hasNext())
           throw new NoSuchElementException();
        // 移动 lastReturned 指针
       lastReturned = next;
        // 移动 next 指针
       next = next.next;
       // 移动 nextIndex cursor
       nextIndex++;
       // 返回移动后 lastReturned
       return lastReturned.item;
   }

  // 当前游标位置是否还有前一个元素
   public boolean hasPrevious() {
       return nextIndex > 0;
   }
  
  // 当前游标位置的前一个元素
   public E previous() {
       checkForComodification();
       if (!hasPrevious())
           throw new NoSuchElementException();
        // 等同于 lastReturned = next；next = (next == null) ? last : next.prev;
        // 发生在 index = size 时
       lastReturned = next = (next == null) ? last : next.prev;
       nextIndex--;
       return lastReturned.item;
   }
    
   public int nextIndex() {
       return nextIndex;
   }

   public int previousIndex() {
       return nextIndex - 1;
   }
    
    // 删除链表当前节点也就是调用 next/previous 返回的这节点，也就 lastReturned
   public void remove() {
       checkForComodification();
       if (lastReturned == null)
           throw new IllegalStateException();

       Node<E> lastNext = lastReturned.next;
       //调用LinkedList 的删除节点的方法
       unlink(lastReturned);
       if (next == lastReturned)
           next = lastNext;
       else
           nextIndex--;
       //上一次所操作的 节点置位空    
       lastReturned = null;
       expectedModCount++;
   }

    // 设置当前遍历的节点的值
   public void set(E e) {
       if (lastReturned == null)
           throw new IllegalStateException();
       checkForComodification();
       lastReturned.item = e;
   }
    // 在 next 节点位置插入及节点
   public void add(E e) {
       checkForComodification();
       lastReturned = null;
       if (next == null)
           linkLast(e);
       else
           linkBefore(e, next);
       nextIndex++;
       expectedModCount++;
   }
    //简单哈操作数是否合法
   final void checkForComodification() {
       if (modCount != expectedModCount)
           throw new ConcurrentModificationException();
   }
}
```

### ArrayList与LinkedList

链表LinkedList和数组ArrayList的最大区别在于它们对元素的存储方式的不同导致它们在对数据进行不同操作时的效率不同。实际使用时根据特定的需求选用合适的类。

- ArrayList基于数组；LinkedList基于双向链表。

- 查找方面。数组的效率更高，可以直接索引出查找；而链表必须从头查找。
- 插入删除方面。特别是在中间进行插入删除，这时候链表体现出了极大的便利性，只需要在插入或者删除的地方断掉链然后插入或者移除元素，然后再将前后链重新组装；但是数组必须重新复制一份将所有数据后移或者前移。
- 在内存申请方面，当数组达到初始的申请长度后，需要重新申请一个更大的数组然后把数据迁移过去才行。而链表只需要动态创建即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190607102647638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

### Vector 向量

Vector非常类似ArrayList。**Vector是同步的**。当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。

#### Stack 栈

Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。

## Set子接口

Set是一种不包含重复的元素的Collection，即任意的两个元素e1和e2都有e1.equals(e2)=false，Set最多有一个null元素。

### HashSet 散列集

HashSet实现了Set接口，基于HashMap进行存储。遍历时不保证顺序，并且不保证下次遍历的顺序和之前一样。HashSet中允许null元素。

在初始化中，创建HashMap

```
public HashSet() {
        map = new HashMap<>();
    }
```

添加方法调用的也是HashMap中的方法，key就是传入的元素，value是Object对象。

```
private static final Object PRESENT = new Object();

public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

意思就是HashSet的集合其实就是HashMap的key的集合，然后HashMap的val默认都是PRESENT。HashMap的定义即是key不重复的集合。使用HashMap实现，这样HashSet就不需要再实现一遍。

所以所有的add，remove等操作其实都是HashMap的add、remove操作。遍历操作其实就是HashMap的keySet的遍历

```
public Iterator<E> iterator() {
    return map.keySet().iterator();
}

public boolean contains(Object o) {
    return map.containsKey(o);
}

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public void clear() {
    map.clear();
}
```



### LinkedHashSet 链式散列集

LinkedHashSet的核心概念相对于HashSet来说就是一个可以保持顺序的Set集合。HashSet是无序的，LinkedHashSet会根据add，remove这些操作的顺序在遍历时返回固定的集合顺序。这个顺序不是元素的大小顺序，而是可以保证2次遍历的顺序是一样的。

类似HashSet基于HashMap的源码实现，LinkedHashSet的数据结构是基于LinkedHashMap。

### TreeSet 树形集

TreeSet即是一组有次序的集合，如果没有指定排序规则Comparator，则会按照自然排序。（自然排序即e1.compareTo(e2) == 0作为比较）

TreeSet源码的算法即基于TreeMap，扩展自AbstractSet，并实现了NavigableSet，AbstractSet扩展自AbstractCollection，树形集是一个有序的Set，其底层是一颗树，这样就能从Set里面提取一个有序序列了。在实例化TreeSet时，我们可以给TreeSet指定一个比较器Comparator来指定树形集中的元素顺序。树形集中提供了很多便捷的方法。



![在这里插入图片描述](https://img-blog.csdnimg.cn/20190607103330622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

## Queue 队列

队列是一种先进先出的数据结构，元素在队列末尾添加，在队列头部删除。Queue接口扩展自Collection，并提供插入、提取、检验等操作。

- `boolean add(E e)`：

- `boolean offer(E e)`：向队列添加一个元素

- `E poll()`：移除队列头部元素（队列为空返回null）

- `E remove()`：移除队列头部元素（队列为空抛出异常）

- `E element();`：获取头部元素

- `E peek();`：获取头部元素

 

### Deque 双端队列

接口Deque，是一个扩展自Queue的双端队列，它支持在两端插入和删除元素，因为LinkedList类实现了Deque接口，所以通常我们可以使用LinkedList来创建一个队列。PriorityQueue类实现了一个优先队列，优先队列中元素被赋予优先级，拥有高优先级的先被删除。

## Iterator迭代器

如何遍历Collection中的每一个元素？不论Collection的实际类型如何，它都支持一个iterator()的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问Collection中每一个元素。典型的用法如下：

```
Iterator it = collection.iterator(); // 获得一个迭代子  
while(it.hasNext()) {  
Object obj = it.next(); // 得到下一个元素  
} 
```



# Map

![这里写图片描述](https://user-gold-cdn.xitu.io/2018/5/21/1638145a2ead2bbb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Map是图接口，存储键值对映射的容器类。Map提供key到value的映射。一个Map中不能包含相同的key，每个key只能映射一个value。

- Map 是**映射接口**，Map中存储的内容是**键值对**(key-value)
- AbstractMap 是继承于Map的抽象类，它实现了Map中的大部分API。其它Map的实现类可以通过继承AbstractMap来减少重复编码。
- SortedMap 是继承于Map的接口。SortedMap中的内容是**排序的键值对**，排序的方法是通过比较器(Comparator)
- NavigableMap 是继承于SortedMap的接口。相比于SortedMap，NavigableMap有一系列的导航方法；如"获取大于/等于某对象的键值对"、“获取小于/等于某对象的键值对”等等。 
- TreeMap 继承于AbstractMap，且实现了NavigableMap接口；因此，TreeMap中的内容是“**有序的键值对”**！
- HashMap 继承于AbstractMap，但没实现NavigableMap接口；因此，HashMap的内容是“**键值对，但不保证次序**”！
- Hashtable 虽然不是继承于AbstractMap，但它继承于Dictionary(Dictionary也是键值对的接口)，而且也实现Map接口；因此，Hashtable的内容也是“**键值对，也不保证次序”**。但和HashMap相比，Hashtable是线程安全的，而且它支持通过Enumeration去遍历。
- WeakHashMap 继承于AbstractMap。它和HashMap的键类型不同，**WeakHashMap的键是“弱键**”。

## 接口定义

```
public interface Map<K,V> {
    ...
    
    interface Entry<K,V> {
        K getKey();
        V getValue();
        ...
    } 
}
```

泛型<K,V>分别代表key和value的类型。这时候注意到还定义了一个内部接口Entry，其实每一个键值对都是一个Entry的实例关系对象，所以Map实际其实就是Entry的一个Collection，然后Entry里面包含key，value。再设定key不重复的规则，自然就演化成了Map。

## 遍历方法

Map集合提供3种遍历访问方法：

- Set keySet() 获得所有key的集合然后通过key访问value会返回所有key的Set集合，因为key不可以重复，所以返回的是Set格式，而不是List格式。（之后会说明Set，List区别。这里先告诉一点Set集合内元素是不可以重复的，而List内是可以重复的） 获取到所有key的Set集合后，由于Set是Collection类型的，所以可以通过Iterator去遍历所有的key，然后再通过get方法获取value。

- Collection values() 获得value的集合。直接获取values的集合，无法再获取到key。所以如果只需要value的场景可以用这个方法。获取到后使用Iterator去遍历所有的value。
- Set< Map.Entry< K, V>> entrySet() 获得key-value键值对的集合。

通过以上3种遍历方式我们可以知道，如果你只想获取key，建议使用keySet。如果只想获取value，建议使用values。如果key value希望遍历，建议使用entrySet。

Map的访问顺序取决于Map的遍历访问方法的遍历顺序。 有的Map，比如TreeMap可以保证访问顺序，但是有的比如HashMap，无法保证访问顺序。

## 哈希

### 哈希表

散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希(Hash）表，函数f(key)为哈希(Hash) 函数。

常见的哈希函数：

- 直接定址法：直接以关键字k或者k加上某个常数（k+c）作为哈希地址。
- 数字分析法：提取关键字中取值比较均匀的数字作为哈希地址。
- 除留余数法：用关键字k除以某个不大于哈希表长度m的数p，将所得余数作为哈希表地址。
- 分段叠加法：按照哈希表地址位数将关键字分成位数相等的几部分，其中最后一部分可以比较短。然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。
- 平方取中法：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。
- 伪随机数法：采用一个伪随机数当作哈希函数。

哈希表是一种通过哈希函数将特定的键映射到特定值的一种数据结构，他维护者键和值之间一一对应关系。

- 键(key)：又称为关键字。唯一的标示要存储的数据，可以是数据本身或者数据的一部分。
- 桶(slot/bucket)：哈希表中用于保存数据的一个单元，也就是数据真正存放的容器。
- 哈希函数(hash function)：将键(key)映射(map)到数据应该存放的槽(slot)所在位置的函数。
- 哈希冲突(hash collision)：哈希函数将两个不同的键映射到同一个索引的情况。

所有散列函数都有如下一个基本特性：**根据同一散列函数计算出的散列值如果不同，那么输入值肯定也不同。但是，根据同一散列函数计算出的散列值如果相同，输入值不一定相同。**

**两个不同的输入值，根据同一散列函数计算出的散列值相同的现象叫做碰撞。**

### 解决哈希冲突的方法

#### 拉链法

哈希冲突后，用链表去延展来解决。将所有关键字为同义词的记录存储在同一线性链表中。如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190725082450961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

#### 开地址法

哈希冲突后，并不会在本身之外开拓新的空间，而是继续顺延下去某个位置来存放。

开放地执法有一个公式:Hi=(H(key)+di) MOD m i=1,2,…,k(k<=m-1)

其中，m为哈希表的表长。di 是产生冲突的时候的增量序列。如果di值可能为1,2,3,…m-1，称线性探测再散列。

如果di取1，则每次冲突之后，向后移动1个位置.如果di取值可能为1,-1,2,-2,4,-4,9,-9,16,-16,…k*k,-k*k(k<=m/2)，称二次探测再散列。

如果di取值可能为伪随机数列。称伪随机探测再散列。

#### 再哈希法

就是同时构造多个不同的哈希函数：
 Hi = RHi(key)   i= 1,2,3 ... k;
 当H1 = RH1(key)  发生冲突时，再用H2 = RH2(key) 进行计算，直到冲突不再产生，这种方法不易产生聚集，但是增加了计算时间。

#### 建立公共溢出区

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

## HashMap

它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。哈希表为解决冲突，采用了链地址法,简单来说，就是数组加链表的结合,当哈希冲突时，数组上的数据采用链表的方式把新数据插到链尾。

当出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当**链表长度太长（默认超过8）时，链表就转换为红黑树**

HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

### 数据结构

从结构实现来讲，HashMap是:**数组+链表+红黑树**（JDK1.8增加了红黑树部分）实现的，如下如所示。

![阿里P8架构师谈：深入探讨HashMap的底层结构、原理、扩容机制](https://youzhixueyuan.com/blog/wp-content/uploads/2019/07/20190731210635_58179.jpg)

```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
		//默认容量
		static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
		//最大容量
		static final int MAXIMUM_CAPACITY = 1 << 30;
		//默认加载因子
		static final float DEFAULT_LOAD_FACTOR = 0.75f;
		//链表转成红黑树的阈值
		static final int TREEIFY_THRESHOLD = 8;
		//红黑树转为链表的阈值
		static final int UNTREEIFY_THRESHOLD = 6;
		//哈希桶数组
		transient Node<K,V>[] table;
    // 存放具体元素的集
    transient Set<Map.Entry<K,V>> entrySet;

		//存储方式由链表转成红黑树的容量的最小阈值
		static final int MIN_TREEIFY_CAPACITY = 64;
		//HashMap中存储的键值对的数量
		transient int size;
		//扩容阈值，当size>=threshold时，就会扩容
		int threshold;
		//HashMap的加载因子
		final float loadFactor;
		
		public HashMap() {
        //默认构造函数，赋值加载因子为默认的0.75f
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

		//指定初始化容量的构造函数
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

		//同时指定初始化容量 以及 加载因子， 用的很少，一般不会修改loadFactor
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);

				//初始容量最大不能超过2的30次方
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //显然加载因子不能为负数
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    //新建一个哈希表，同时将另一个map m 里的所有元素加入表中
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    
		static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
    
    //根据期望容量cap，返回2的n次方形式的 哈希桶的实际容量 length。 返回值一般会>=cap 
    static final int tableSizeFor(int cap) {
    //经过下面的 或 和位移 运算， n最终各位都是1。
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        //判断n是否越界，返回 2的n次方作为 table（哈希桶）的阈值
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
}
```

HashMap类中有一个非常重要的字段，就是 Node[] table，即哈希桶数组，它是一个Node的数组。Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。上图中的每个黑色圆点就是一个Node对象。

一些重要的参数：

- 初始容量（initialCapacity，默认为16）
- 如果initialCapacity不为2的幂值，HashMap会自动选择比initialCapacity大的下一个2的幂值作为初始容量。
- 负载系数（loadFactor，默认为0.75）
- 当HashMap.size()大于`capacity*load_factor`时，容器将自动扩容并重新哈希。

### 确定哈希桶数组索引位置

```
static final int hash(Object key) { //jdk1.8 & jdk1.7
 int h;
 // h = key.hashCode() 为第一步 取hashCode值
 // h ^ (h >>> 16) 为第二步 高位参与运算
 return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这里的Hash算法本质上就是三步：**取key的hashCode值、高位运算**。

对于任意给定的对象，只要它的hashCode()返回值相同，那么所计算得到的Hash码值总是相同的。

在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

![阿里P8架构师谈：深入探讨HashMap的底层结构、原理、扩容机制](https://youzhixueyuan.com/blog/wp-content/uploads/2019/07/20190731210658_66827.jpg)

### 存储数据 put

```
public V put(K key, V value) {
        //先根据key，取得hash值。再插入节点
        return putVal(hash(key), key, value, false, true);
	}
	
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        //tab存放 当前的哈希桶， p用作临时链表节点  
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果当前哈希表是空的，代表是初始化，执行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
        		//扩容哈希表，并且将扩容后的哈希桶长度赋值给n
            n = (tab = resize()).length;
        //如果当前index的节点是空的，表示没有发生哈希碰撞。 直接构建一个新节点Node，挂载在index处即可。
        //index 是利用 哈希值 & 哈希桶的长度-1，替代模运算
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {//发生了哈希冲突。
            Node<K,V> e; K k;
            //如果哈希值相等，key也相等，则是覆盖value操作
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //将当前节点引用赋值给e
                e = p;
            else if (p instanceof TreeNode)//此处代表红黑树
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {//此处代表链表
	           		 //遍历链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
		                    //遍历到尾部，追加新节点到尾部
                        p.next = newNode(hash, key, value, null);
                        //如果追加节点后，链表数量>=8，则转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //treeifyBin首先判断当前hashMap的长度，如果不足64，只进行resize，扩容table，
                        //如果达到64，那么将冲突的存储结构为红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果找到了要覆盖的节点,结束遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
             //如果e不是null，链表上有相同的key值，
            if (e != null) { // existing mapping for key
		            //则覆盖节点值，并返回原oldValue
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //这是一个空实现的函数，用作LinkedHashMap重写使用。
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //如果执行到了这里，说明插入了一个新的节点，所以会修改modCount，以及返回null。
        //修改modCount
        ++modCount;
    	  //更新size，并判断是否需要扩容。
        if (++size > threshold)
            resize();
        //这是一个空实现的函数，用作LinkedHashMap重写使用。
        afterNodeInsertion(evict);
        return null;
    }
```



![阿里P8架构师谈：深入探讨HashMap的底层结构、原理、扩容机制](https://youzhixueyuan.com/blog/wp-content/uploads/2019/07/20190731210713_35028.jpg)

1. 判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；
2. 根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向6，如果table[i]不为空，转向3；
3. 判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向4，这里的相同指的是hashCode以及equals；
4. 判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向5；
5. 遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
6. 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

### 获取数据 get

```
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 定位键值对所在桶的位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //直接命中
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 桶中不止一个节点
            if ((e = first.next) != null) {
            //如果 first 是 TreeNode 类型，则调用黑红树查找方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
	                	//对链表进行查找
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

get方法做了4件事：

- 计算key的hash值；
- 找到key所在的桶及其第一个元素；
- 如果第一个元素的key等于待查找的key，直接返回；
- 如果第一个元素是树节点就按树的方式来查找，否则按链表方式查找；

### 扩容机制

扩容(resize)就是重新计算容量，Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。

```
final Node<K,V>[] resize() {
        //oldTab 为当前表的哈希桶
        Node<K,V>[] oldTab = table;
        //当前哈希桶的容量 length
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //当前的阈值
        int oldThr = threshold;
        //初始化新的容量和阈值为0
        int newCap, newThr = 0;
        //如果当前容量大于0
        if (oldCap > 0) {
            //如果当前容量已经到达上限
            if (oldCap >= MAXIMUM_CAPACITY) {
                //则设置阈值是2的31次方-1
                threshold = Integer.MAX_VALUE;
                //同时返回当前的哈希桶，不再扩容
                return oldTab;
            }//否则新的容量为旧的容量的两倍。 
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)//如果旧的容量大于等于默认初始容量16
                //那么新的阈值也等于旧的阈值的两倍
                newThr = oldThr << 1; // double threshold
        }//如果当前表是空的，但是有阈值。代表是初始化时指定了容量、阈值的情况
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;//那么新表的容量就等于旧的阈值
        else {}//如果当前表是空的，而且也没有阈值。代表是初始化时没有任何容量/阈值参数的情况               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;//此时新表的容量为默认的容量 16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//新的阈值为默认容量16 * 默认加载因子0.75f = 12
        }
        //如果新的阈值是0，对应的是  当前表是空的，但是有阈值的情况
        if (newThr == 0) {
		        //根据新表容量 和 加载因子 求出新的阈值
            float ft = (float)newCap * loadFactor;
            //进行越界修复
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //更新阈值 
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        		//根据新的容量 构建新的哈希桶
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //更新哈希桶引用
        table = newTab;
        //如果以前的哈希桶中有元素
        //下面开始将当前哈希桶中的所有节点转移到新的哈希桶中
        if (oldTab != null) {
            //遍历老的哈希桶
            for (int j = 0; j < oldCap; ++j) {
                //取出当前的节点 e
                Node<K,V> e;
                //如果当前桶中有元素,则将链表赋值给e
                if ((e = oldTab[j]) != null) {
                    //将原哈希桶置空以便GC
                    oldTab[j] = null;
                    //如果当前链表中就一个元素，（没有发生哈希碰撞）
                    if (e.next == null)
                        //直接将这个元素放置在新的哈希桶里。
                        //注意这里取下标 是用 哈希值 与 桶的长度-1 。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果发生过哈希碰撞 ,而且是节点数超过8个，转化成了红黑树
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
                    else { // preserve order
                        //新计算在新表的位置，并进行搬运
                        Node<K,V> loHead = null, loTail = null;                   
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;//临时节点 存放e的下一个节点
                        do {
                            next = e.next;
                            //这里又是一个利用位运算 代替常规运算的高效点： 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位
                            if ((e.hash & oldCap) == 0) {
                                //给头尾节点指针赋值
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }//高位也是相同的逻辑
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }//循环直到链表结束
                        } while ((e = next) != null);
                        //将低位链表存放在原index处，
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //将高位链表存放在新index处
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

HashMap 按当前桶数组长度的2倍进行扩容，阈值也变为原来的2倍（如果计算过程中，阈值溢出归零，则按阈值公式重新计算）。扩容之后，根据hash值重新计算下角标（newTab[e.hash & (newCap - 1)]），并把它们放置到合适的位置上去。

源码做了三件事：

1. 计算新桶数组的容量 newCap 和新阈值 newThr
2. 根据计算出的 newCap 创建新的桶数组，桶数组 table 也是在这里进行初始化的
3. 将键值对节点重新映射到新的桶数组里。如果节点是 TreeNode 类型，则需要拆分红黑树。如果是普通节点，则节点按原顺序进行分组。

### 线程安全性 

在多线程使用场景中，应该尽量避免使用线程不安全的HashMap，而使用线程安全的ConcurrentHashMap。

**在并发的多线程使用场景中使用HashMap可能造成死循环。**

### 重点及面试题

- HashMap是一种散列表，采用（数组 + 链表 + 红黑树）的存储结构；
- HashMap的默认初始容量为16（1<<4），默认装载因子为0.75f，容量总是2的n次方；
- HashMap扩容时每次容量变为原来的两倍；
- 当桶的数量小于64时不会进行树化，只会扩容；
- 当桶的数量大于64且单个桶中元素的数量大于8时，进行树化；
- 当单个桶中元素数量小于6时，进行反树化；
- HashMap查找添加元素的时间复杂度都为O(1)；
- HashMap是非线程安全的容器
- 允许使用null值和null键(HashMap最多只允许一条记录的键为null，允许多条记录的值为null)。
- HashMap中不允许出现重复的键（Key）

## HashTable

Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。

添加数据使用put(key, value)，取出数据使用get(key)，这两个基本操作的时间开销为常数。HashTable是**同步**方法，线程安全但是效率低。

Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

### 数据结构

HashTable使用数组+单向列表。

```
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    		
        // Hashtable保存key-value的数组。
		    // Hashtable是采用拉链法实现的，每一个Entry本质上是一个单向链表
        private transient HashtableEntry<?,?>[] table;
		    // Hashtable中元素的实际数量
				private transient int count;
				// 阈值，用于判断是否需要调整Hashtable的容量（threshold = 容量*加载因子）
				private int threshold;
				// 加载因子
				private float loadFactor;
				
		    // 指定“容量大小”和“加载因子”的构造函数
				public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        //创建对象时创建数组并非像HashMap那样懒加载
        table = new HashtableEntry<?,?>[initialCapacity];
        // Android-changed: Ignore loadFactor when calculating threshold from initialCapacity
        // threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        threshold = (int)Math.min(initialCapacity, MAX_ARRAY_SIZE + 1);
    }
    
    
     // 指定“容量大小”的构造函数
     public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }
     // 默认构造函数。
    public Hashtable() {
        // 默认构造函数，指定的容量大小是11；加载因子是0.75
        this(11, 0.75f);
    }
    
     public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }
    
    private static class HashtableEntry<K,V> implements Map.Entry<K,V> {
    		final int hash;
        final K key;
        V value;
        HashtableEntry<K,V> next;
    		
    		 protected HashtableEntry(int hash, K key, V value, HashtableEntry<K,V> next) {
    		 		/**hash值*/
            this.hash = hash;
             /**key表示键*/
            this.key =  key;
            /**value表示值*/
            this.value = value;
             /**节点下一个元素*/
            this.next = next;
        }
    	}
    
    }
```

### 常用方法

#### put

put() 的作用是**对外提供接口，让Hashtable对象可以通过put()将“key-value”添加到Hashtable中。**

```
public synchronized V put(K key, V value) {
        // Make sure the value is not null
        // Hashtable中不能插入value为null的元素！！！
        if (value == null) {
            throw new NullPointerException();
        }
				
        // Makes sure the key is not already in the hashtable.
         // 若“Hashtable中已存在键为key的键值对”，
		    // 则用“新的value”替换“旧的value”
        HashtableEntry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        HashtableEntry<K,V> entry = (HashtableEntry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }
				//“Hashtable中不存在键为key的键值对”，
        addEntry(hash, key, value, index);
        return null;
    }
    
    private void addEntry(int hash, K key, V value, int index) {
    		//将“修改统计数”+1
        modCount++;
        HashtableEntry<?,?> tab[] = table;
        //若“Hashtable实际容量” > “阈值”(阈值=总的容量 * 加载因子)
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            //则调整Hashtable的大小
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        //将“Hashtable中index”位置的Entry(链表)保存到e中
        @SuppressWarnings("unchecked")
        HashtableEntry<K,V> e = (HashtableEntry<K,V>) tab[index];
        //创建“新的Entry节点”，并将“新的Entry”插入“Hashtable的index位置”，并设置e为“新的Entry”的下一个元素(即“新Entry”为链表表头)。       
        tab[index] = new HashtableEntry<>(hash, key, value, e);
        //将“Hashtable的实际容量”+1
        count++;
    }
```

![img](http://www.justdojava.com/assets/images/2019/java/image-jay/ebdd7991f4634d4393568b0632d769ab.jpg)

put方法执行流程：

1. 校验null值，value不允许null值。
2. 计算出key的哈希值，用哈希值和数组的长度得到index下角标。
3. 根据index找到节点。
4. 如果节点中有元素则遍历链表如果找到则替换旧值并返回旧值。
5. 如果没找到旧值则执行`addEntry`方法，创建节点并加入哈希桶中。`addEntry`方法：
   1. 首先判断是否需要扩容
   2. 创建节点
   3. 添加哈希桶中

#### get

```
public synchronized V get(Object key) {
        HashtableEntry<?,?> tab[] = table;
         // 计算索引值，
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        // 找到“key对应的Entry(链表)”，然后在链表中找出“哈希值”和“键值”与key都相等的元素
        for (HashtableEntry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
```

根据index直接找到节点，并进行比较，找到了就返回value，没找到返回null。

![img](http://www.justdojava.com/assets/images/2019/java/image-jay/82a5ff28b5c14da29f3081db4cc2cd23.jpg)

#### remove()

remove() 的作用就是**删除Hashtable中键为key的元素**

```
public synchronized V remove(Object key) {
    Entry tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    // 找到“key对应的Entry(链表)”
    // 然后在链表中找出要删除的节点，并删除该节点。
    for (Entry<K,V> e = tab[index], prev = null ; e != null ; prev = e, e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            modCount++;
            //重新排列链表
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }
            count--;
            V oldValue = e.value;
            e.value = null;
            return oldValue;
        }
    }
    return null;
}
```

![img](http://www.justdojava.com/assets/images/2019/java/image-jay/7486b49ac2f4410099b5764082a6d352.jpg)

#### rehash 扩容

```
protected void rehash() {
				//旧数组的长度
        int oldCapacity = table.length;
        //旧数组
        HashtableEntry<?,?>[] oldMap = table;

        // overflow-conscious code
        //新数组的长度是=将长度变成原来的(2倍+1)
        int newCapacity = (oldCapacity << 1) + 1;
        //是否超过可存储数量的最大值
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
        		//已经到了最大值直接返回，不允许扩容
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            //没有达到最大值，直接把新数组赋值到最大值
            newCapacity = MAX_ARRAY_SIZE;
        }
        //创建新的数组
        HashtableEntry<?,?>[] newMap = new HashtableEntry<?,?>[newCapacity];

        modCount++;
        //计算阀值 newCapacity * loadFactor 如果到达最大存储容量，以后都不会触法
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;
				//循环获取获取所有节点
        for (int i = oldCapacity ; i-- > 0 ;) {
        		//循环获取节点下的所有元素
            for (HashtableEntry<K,V> old = (HashtableEntry<K,V>)oldMap[i] ; old != null ; ) {
                HashtableEntry<K,V> e = old;
                old = old.next;
								//获取元素新的index，并存入对应的哈希桶中
                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = (HashtableEntry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }
```

可以看到所有操作数据的方法都是`synchronized`线程安全的，尽管，Hashtable 虽然是线程安全的，但是我们一般不推荐使用它，因为有比它更高效、更好的选择 ConcurrentHashMap，在后面我们也会讲到它。



## HashMap和Hashtable的区别

- HashMap线程不安全；Hashtable线程安全。
- 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。
- HashMap使用时候初始化哈希桶；Hashtable创建对象时创建哈希桶。
- HashMap扩容是原来的两倍；Hashtable扩容是原来的两倍+1。
- HashMap的初始容量为16；Hashtable初始容量为11，两者的填充因子默认都是0.75
- HashMap允许有null值；Hashtable不允许有null值。
-  HashMap数组+链表+红黑树；Hashtable数组+链表。

## HashMap和HashSet的区别

- HashMap实现Map接口；HashSet实现Set接口。
- HashMap储存键值对；HashSet仅存储对象（`value`是空`Object`对象）。
- HashSet基于HashMap，内部持有HashMap引用，核心方法调用都是HashMap。

## LinkedHashMap

LinkedHashMap继承自HashMap实现了Map接口。基本实现同HashMap一样，不同之处在于HashMap是无序的而LinkedHashMap保证了迭代的有序性。其内部维护了一个双向链表，解决了 HashMap不能随时保持遍历顺序和插入顺序一致的问题。

**默认情况下，LinkedHashMap的迭代顺序是按照插入节点的顺序。也可以通过改变accessOrder参数的值，使得其遍历顺序按照访问顺序输出。**

LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构。该结构由数组和链表或红黑树组成。本质上，HashMap和双向链表合二为一即是LinkedHashMap。所谓LinkedHashMap，其落脚点在HashMap，因此更准确地说，它是一个将所有Entry节点链入一个双向链表双向链表的HashMap。在LinkedHashMapMap中，所有put进来的Entry都保存在如下面第一个图所示的哈希表中，但由于它又额外定义了一个以head为头结点的双向链表(如下面第二个图所示)，因此对于每次put进来Entry，除了将其保存到哈希表中对应的位置上之外，还会将其插入到双向链表的尾部。

![img](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166327271293.jpg)

LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。其结构可能如下图：

![img](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166338955699.jpg)

上图中，淡蓝色的箭头表示前驱引用，红色箭头表示后继引用。每当有新键值对节点插入，新节点最终会接在 tail 引用指向的节点后面。而 tail 引用则会移动到新的节点上，这样一个双向链表就建立起来了。

### 数据结构

```
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{

		static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
     transient LinkedHashMapEntry<K,V> head;
     
     transient LinkedHashMapEntry<K,V> tail;
     
     final boolean accessOrder;
     
     //默认初始容量 (16)和默认负载因子(0.75)的空 LinkedHashMap
     public LinkedHashMap() {
     		// 调用HashMap对应的构造函数
        super();
        // 迭代顺序的默认值
        accessOrder = false;
    }
    
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    
     public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }
    
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }

}
```

LinkedHashMap增加了两个属性用于保证迭代顺序，分别是 **双向链表头结点header、tail** 和 **标志位accessOrder** (值为true时，表示按照访问顺序迭代；值为false时，表示按照插入顺序迭代)。

LinkedHashMap采用的hash算法和HashMap相同，但是它重新定义了Entry。LinkedHashMap中的Entry增加了两个指针 before 和 after，它们分别用于维护双向链接列表。特别需要注意的是，next用于维护HashMap各个桶中Entry的连接顺序，before、after用于维护Entry插入的先后顺序的。



### 插入数据

链表的建立过程是在插入键值对节点时开始的，初始情况下，让 LinkedHashMap 的 head 和 tail 引用同时指向新节点，链表就算建立起来了。随后不断有新节点插入，通过将新节点接在 tail 引用指向节点的后面，即可实现链表的更新。

Map 类型的集合类是通过 put(K,V) 方法插入键值对，LinkedHashMap 本身并没有覆写父类的 put 方法，而是直接使用了父类的实现。但在 HashMap 中，put 方法插入的是 HashMap 内部类 Node 类型的节点，该类型的节点并不具备与 LinkedHashMap 内部类 Entry 及其子类型节点组成链表的能力。那么，LinkedHashMap 是怎样建立链表的呢？在展开说明之前，我们先看一下 LinkedHashMap 插入操作相关的代码：

```
// HashMap 中实现
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// HashMap 中实现
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) {...}
    // 通过节点 hash 定位节点所在的桶位置，并检测桶中是否包含节点引用
    if ((p = tab[i = (n - 1) & hash]) == null) {...}
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) {...}
        else {
            // 遍历链表，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 未在单链表中找到要插入的节点，将新节点接在单链表的后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) {...}
                    break;
                }
                // 插入的节点已经存在于单链表中
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null) {...}
            afterNodeAccess(e);    // 回调方法，后续说明
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold) {...}
    afterNodeInsertion(evict);    // 回调方法，后续说明
    return null;
}

// HashMap 中实现
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}

// LinkedHashMap 中覆写
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 将 Entry 接在双向链表的尾部
    linkNodeLast(p);
    return p;
}

// LinkedHashMap 中实现
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    // last 为 null，表明链表还未建立
    if (last == null)
        head = p;
    else {
        // 将新节点 p 接在链表尾部
        p.before = last;
        last.after = p;
    }
}
```

上面就是 LinkedHashMap 插入相关的源码，这里省略了部分非关键的代码。我根据上面的代码，可以知道 LinkedHashMap 插入操作的调用过程。如下：

![img](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166881843975.jpg)



newNode()这一步比较关键。LinkedHashMap 覆写了该方法。在这个方法中，LinkedHashMap 创建了 Entry，并通过 linkNodeLast 方法将 Entry 接在双向链表的尾部，实现了双向链表的建立。

HashMap中有三个回调方法：

```
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

根据这三个方法的注释可以看出，这些方法的用途是在增删查等操作后，通过回调的方式，让 LinkedHashMap 有机会做一些后置操作。上述三个方法的具体实现在 LinkedHashMap 中。



### 删除数据

与插入操作一样，LinkedHashMap 删除操作相关的代码也是直接用父类的实现。在删除节点时，父类的删除逻辑并不会修复 LinkedHashMap 所维护的双向链表，这不是它的职责。那么删除及节点后，被删除的节点该如何从双链表中移除呢？当然，办法还算是有的。上一节最后提到 HashMap 中三个回调方法运行 LinkedHashMap 对一些操作做出响应。所以，在删除及节点后，回调方法 `afterNodeRemoval` 会被调用。LinkedHashMap 覆写该方法，并在该方法中完成了移除被删除节点的操作。相关源码如下：

```
// HashMap 中实现
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

// HashMap 中实现
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode) {...}
            else {
                // 遍历单链表，寻找要删除的节点，并赋值给 node 变量
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode) {...}
            // 将要删除的节点从单链表中移除
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);    // 调用删除回调方法进行后续操作
            return node;
        }
    }
    return null;
}

// LinkedHashMap 中覆写
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // 将 p 节点的前驱后后继引用置空
    p.before = p.after = null;
    // b 为 null，表明 p 是头节点
    if (b == null)
        head = a;
    else
        b.after = a;
    // a 为 null，表明 p 是尾节点
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

删除的过程并不复杂，上面这么多代码其实就做了三件事：

1. 根据 hash 定位到桶位置
2. 遍历链表或调用红黑树相关的删除方法
3. 从 LinkedHashMap 维护的双链表中移除要删除的节点

### 访问顺序的维护过程

默认情况下，LinkedHashMap 是按插入顺序维护链表。不过我们可以在初始化 LinkedHashMap，指定 accessOrder 参数为 true，即可让它按访问顺序维护链表。访问顺序的原理上并不复杂，当我们调用`get/getOrDefault/replace`等方法时，只需要将这些方法访问的节点移动到链表的尾部即可。相应的源码如下：

```
// LinkedHashMap 中覆写
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    // 如果 accessOrder 为 true，则调用 afterNodeAccess 将被访问节点移动到链表最后
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

// LinkedHashMap 中覆写
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        // 如果 b 为 null，表明 p 为头节点
        if (b == null)
            head = a;
        else
            b.after = a;
            
        if (a != null)
            a.before = b;
        /*
         * 这里存疑，父条件分支已经确保节点 e 不会是尾节点，
         * 那么 e.after 必然不会为 null，不知道 else 分支有什么作用
         */
        else
            last = b;
    
        if (last == null)
            head = p;
        else {
            // 将 p 接在链表的最后
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### 基于 LinkedHashMap 实现缓存

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 根据条件判断是否移除最近最少被访问的节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

上面的源码的核心逻辑在一般情况下都不会被执行，所以之前并没有进行分析。上面的代码做的事情比较简单，就是通过一些条件，判断是否移除最近最少被访问的节点。看到这里，大家应该知道上面两个方法的用途了。当我们基于 LinkedHashMap 实现缓存时，通过覆写`removeEldestEntry`方法可以实现自定义策略的 LRU 缓存。比如我们可以根据节点数量判断是否移除最近最少被访问的节点，或者根据节点的存活时间判断是否移除该节点等。本节所实现的缓存是基于判断节点数量是否超限的策略。在构造缓存对象时，传入最大节点数。当插入的节点数超过最大节点数时，移除最近最少被访问的节点。实现代码如下：

```
public class SimpleCache<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_NODE_NUM = 100;

    private int limit;

    public SimpleCache() {
        this(MAX_NODE_NUM);
    }

    public SimpleCache(int limit) {
        super(limit, 0.75f, true);
        this.limit = limit;
    }

    public V save(K key, V val) {
        return put(key, val);
    }

    public V getOne(K key) {
        return get(key);
    }

    public boolean exists(K key) {
        return containsKey(key);
    }
    
    /**
     * 判断节点数是否超限
     * @param eldest
     * @return 超限返回 true，否则返回 false
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > limit;
    }
}
```



## WeakHashMap

WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

举例：声明了两个Map对象，一个是HashMap，一个是WeakHashMap，同时向两个map中放入a、b两个对象，当HashMap remove掉a 并且将a、b都指向null时，WeakHashMap中的a将自动被回收掉。出现这个状况的原因是，对于a对象而言，当HashMap remove掉并且将a指向null后，除了WeakHashMap中还保存a外已经没有指向a的指针了，所以WeakHashMap会自动舍弃掉a，而对于b对象虽然指向了null，但HashMap中还有指向b的指针，所以WeakHashMap将会保留。

## TreeMap

TreeMap实现了SotredMap接口，它是有序的集合。而且是一个红黑树结构，每个key-value都作为一个红黑树的节点。如果在调用TreeMap的构造函数时没有指定比较器，则根据key执行自然排序。

## 特点

- 内部红黑树实现
- key-value不为空
- TreeMap有序

### 数据结构

```
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
    
    //比较器，是自然排序，还是定制排序 ，使用final修饰，表明一旦赋值便不允许改变
		private final Comparator<? super K> comparator;  
		private transient Entry<K,V> root = null;  //红黑树的根节点
		private transient int size = 0;     //TreeMap中存放的键值对的数量
		private transient int modCount = 0;   //修改的次数
    
    private static final boolean RED   = false;
    private static final boolean BLACK = true;
    
    static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;    //键
    V value;    //值
    Entry<K,V> left = null;     //左孩子节点
    Entry<K,V> right = null;    //右孩子节点
    Entry<K,V> parent;          //父节点
    boolean color = BLACK;      //节点的颜色，在红黑树种，只有两种颜色，红色和黑色

    //构造方法，用指定的key,value ,parent初始化，color默认为黑色
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }
  }  
  
  //构造方法，comparator用键的顺序做比较
		public TreeMap() {
		    comparator = null;
		}

		//构造方法，提供比较器，用指定比较器排序
		public TreeMap(Comparator<? super K> comparator) {
		    his.comparator = comparator;
		}

		//将m中的元素转化daoTreeMap中，按照键的顺序做比较排序
		public TreeMap(Map<? extends K, ? extends V> m) {
		    comparator = null;
		    putAll(m);
		}

		//构造方法，指定的参数为SortedMap
		//采用m的比较器排序
		public TreeMap(SortedMap<K, ? extends V> m) {
		    comparator = m.comparator();
		    try {
		        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
		    } catch (java.io.IOException cannotHappen) {
		    } catch (ClassNotFoundException cannotHappen) {
		    }
		}
  
}
```

TreeMap提供了四个构造方法，实现了方法的重载。无参构造方法中比较器的值为null,采用自然排序的方法，如果指定了比较器则称之为定制排序.

- 自然排序：TreeMap的所有key必须实现Comparable接口，所有的key都是同一个类的对象
- 定制排序：创建TreeMap对象传入了一个Comparator对象，该对象负责对TreeMap中所有的key进行排序，采用定制排序不要求Map的key实现Comparable接口。等下面分析到比较方法的时候在分析这两种比较有何不同。

### put

```
public V put(K key, V value) {
				////红黑树的根节点
        TreeMapEntry<K,V> t = root;
        //红黑树是否为空
        if (t == null) {
            compare(key, key); // type (and possibly null) check
            //构造根节点，因为根节点没有父节点，传入null值。
            root = new TreeMapEntry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
         //定义节点
        TreeMapEntry<K,V> parent;
        // split comparator and comparable paths
        //获取比较器
        Comparator<? super K> cpr = comparator;
        //如果定义了比较器，采用自定义比较器进行比较
        if (cpr != null) {
        //do while 做循环排序 找到父节点
            do {
             //将红黑树根节点赋值给parent
                parent = t;
                 //比较key, 与根节点的大小
                cmp = cpr.compare(key, t.key);
                //如果key < t.key , 指向左子树
                if (cmp < 0)
                //t = t.left  , t == 它的左孩子节点
                    t = t.left;
                else if (cmp > 0)//如果key > t.key , 指向它的右孩子节点
                    t = t.right;
                else //如果它们相等，替换key的值
                    return t.setValue(value);
            } while (t != null);
        }
        else {//自然排序方式，没有指定比较器
        		//key == null 抛出异常
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
            //一样 循环排序 找到父节点
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        //创建新节点，并制定父节点
        TreeMapEntry<K,V> e = new TreeMapEntry<>(key, value, parent);
       //根据比较结果，决定新节点为父节点的左孩子或者右孩子
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        //新插入节点后重新调整红黑树
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```



#### compare方法 比较器

比较方法，如果comparator==null ,采用comparable.compartTo进行比较，否则采用指定比较器比较大小

```
final int compare(Object k1, Object k2) {
        return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
            : comparator.compare((K)k1, (K)k2);
    }
```

#### fixAfterInsertion 重新调整红黑树

```
private void fixAfterInsertion(TreeMapEntry<K,V> x) {
		    //插入的节点默认的颜色为红色
        x.color = RED;
				 //情形1：新节点x 是树的根节点，没有父节点不需要任何操作
		    //情形2：新节点x 的父节点颜色是黑色的，也不需要任何操作
        while (x != null && x != root && x.parent.color == RED) {
            //情形3：新节点x的父节点颜色是红色的
				    //判断x的节点的父节点位置，是否属于左孩子
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
              //获取x节点的父节点的兄弟节点，上面语句已经判断出x节点的父节点为左孩子，所以直接取右孩子
                TreeMapEntry<K,V> y = rightOf(parentOf(parentOf(x)));
               //判断是否x节点的父节点的兄弟节点为红色。
                if (colorOf(y) == RED) {
		                // x节点的父节点设置为黑色
                    setColor(parentOf(x), BLACK);
                    // y节点的颜色设置为黑色
                    setColor(y, BLACK);
                    // x.parent.parent设置为红色
                    setColor(parentOf(parentOf(x)), RED);
                    // x == x.parent.parent ,进行遍历。
                    x = parentOf(parentOf(x));
                } else {//x的父节点的兄弟节点是黑色或者缺少的
	                  //判断x节点是否为父节点的右孩子
                    if (x == rightOf(parentOf(x))) {
                   		 //x == 父节点
                        x = parentOf(x);
                        //左旋转操作
                        rotateLeft(x);
                    }
                    //x节点是其父的左孩子
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    //进行右旋转
                    rotateRight(parentOf(parentOf(x)));
                }
            } else {
		             //y 是x 节点的祖父节点的左孩子
                TreeMapEntry<K,V> y = leftOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        //通过节点位置的调整，最终将红色的节点条调换到了根节点的位置，根节点重新设置为黑色
        root.color = BLACK;
    }
```

红黑树是一个更高效的检索二叉树，有如下特点：

- 每个节点只能是红色或者黑色
- 根节点永远是黑色的
- 所有的叶子的子节点都是空节点，并且都是黑色的
- 每个红色节点的两个子节点都是黑色的（不会有两个连续的红色节点）
- 从任一个节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点（叶子节点到根节点的黑色节点数量每条路径都相同）

### get

TreeMap底层是红黑树结构，而红黑树本质是一颗二叉查找树，所以在获取节点方面，使用二分查找算法性能最高；

```
//通过key获取对应的value：
public V get(Object key) {
    //获取TreeMap中对应的节点：
    java.util.TreeMap.Entry<K,V> p = getEntry(key);
    //获取节点的值：
    return (p==null ? null : p.value);
}

//通过key获取Entry对象：
final java.util.TreeMap.Entry<K,V> getEntry(Object key) {
    //TreeMap自定义比较器不为空，使用自定义比较器对象来获取节点：
    if (comparator != null)
        //获取节点：
        return getEntryUsingComparator(key);

    //如果key为null，则抛出异常，TreeMap中不允许存在为null的key：
    if (key == null)
        throw new NullPointerException();

    //将传入的key转换成Comparable类型，传入的key必须实现Comparable接口
    Comparable<? super K> k = (Comparable<? super K>) key;
    //获取根节点：
    java.util.TreeMap.Entry<K,V> p = root;

    // 使用二分查找方式，首先判断传入的key与根节点的key哪个大：
    while (p != null) {
        //传入的key与p节点的key进行大小比较：
        int cmp = k.compareTo(p.key);
        //传入的key小于p节点的key,则从根的左子树中搜索：
        if (cmp < 0)
            //左边
            p = p.left;
        else if (cmp > 0)
            //传入的key大于p节点的key,则从根的右边子树中搜索：
            //右边
            p = p.right;
        else
            //传入的key等于p节点的key,则直接返回当前节点：
            return p;
    }
    //以上循环没有找对对应的节点，则返回null：
    return null;
}

//使用自定义比较器进行元素比较，获取对节点：
final java.util.TreeMap.Entry<K,V> getEntryUsingComparator(Object key) {
    K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        java.util.TreeMap.Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```



### remove

```
/**
     * Removes the mapping for this key from this TreeMap if present.
     *
     */
    public V remove(Object key) {
        Entry p = getEntry(key);
        if (p == null)
            return null;

        V oldValue = p.value;
        deleteEntry(p);
        return oldValue;
    }
```

  通过这段代码可以看出，TreeMap的remove()方法中执行删除的真正方式是deleteEntry()方法。deleteEntry()代码如下：

```
/**
     * Delete node p, and then rebalance the tree.
     */
    private void deleteEntry(Entry p) {
        modCount++;//修改次数 +1；
        size--;//元素个数 -1

        // If strictly internal, copy successor's element to p and then make p
        // point to successor.
		/*
         * 被删除节点的左子树和右子树都不为空，那么就用 p节点的中序后继节点代替 p 节点
         * successor(P)方法为寻找P的替代节点。规则是右分支最左边，或者 左分支最右边的节点
         * ---------------------（1）
         */
        if (p.left != null && p.right != null) {
            Entry s = successor (p);
            p.key = s.key;
            p.value = s.value;
            p = s;
        } // p has 2 children

        // Start fixup at replacement node, if it exists.
		//replacement为替代节点，如果P的左子树存在那么就用左子树替代，否则用右子树替代
        Entry replacement = (p.left != null ? p.left : p.right);
		/*
         * 删除节点，分为上面提到的三种情况
         * -----------------------（2）
         */
        //如果替代节点不为空
        if (replacement != null) {
            // Link replacement to parent
			//replacement来替代P节点
            replacement.parent = p.parent;
            if (p.parent == null)
                root = replacement;
            else if (p == p.parent.left)
                p.parent.left  = replacement;
            else
                p.parent.right = replacement;

            // Null out links so they are OK to use by fixAfterDeletion.
            p.left = p.right = p.parent = null;

            // Fix replacement
            if (p.color == BLACK)
                fixAfterDeletion(replacement);
        } else if (p.parent == null) { // return if we are the only node.
            root = null;
        } else { //  No children. Use self as phantom replacement and unlink.
            if (p.color == BLACK)
                fixAfterDeletion(p);

            if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                p.parent = null;
            }
        }
  }
```

# 总结

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019060710430212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

- 如果涉及到堆栈，队列等操作，应该考虑用List，对于需要快速插入，删除元素，应该使用LinkedList，如果需要快速随机访问元素，应该使用ArrayList。
- 如果程序在单线程环境中，或者访问仅仅在一个线程中进行，考虑非同步的类，其效率较高，如果多个线程可能同时操作一个类，应该使用同步的类。
- 要特别注意对哈希表的操作，作为key的对象要正确复写equals和hashCode方法。
- 尽量返回接口而非实际的类型，如返回List而非ArrayList，这样如果以后需要将ArrayList换成LinkedList时，客户端代码不用改变。这就是针对抽象编程。

