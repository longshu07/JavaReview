# \<center\>对比Vector、ArrayList、LinkedList有何区别？\</center\>

这三者都是实现集合框架中的List，也就是所谓的有序集合，因此功能也比较近似比如都提供按照位置进行定位、添加或者删除的操作，都提供迭代器以遍历其内容。但因为具体设计区别，在行为、性能线程安全等方面，表现又有很大不同。

## Vector

Vector是java早期提供的**线程安全的动态数组**，如果不需要线程安全，并不建议选择。毕竟同步是有额外开销的。Vector内部是使用**数组来保存数据**的。可以根据需要自动的增加容量，当数组已满，会创建新的数组，并拷贝原有数组数据。

## ArrayList

ArrayList是应有更加广泛的**动态数组**实现，它本身不是线程安全，所以性能会好很多。与Vector近似，ArrayList也是可以根据需要调整容量，不过两者的调整逻辑顺序有所区别，**Vector在扩容时会提高一倍，而Arra则是增加50%。**单线程应尽量使用ArrayList，Vector因为同步会有性能损耗；即使在多线程环境下，我们可以利用Collections这个类中为我们提供的synchronizedList(List list)方法返回一个线程安全的同步列表对象。

## LinkedList

LinkedList顾名思义是Java提供的**双向链表**，所以它不需要像上面那样调整容量，它也不是线程安全的。

## 读写机制

  Vector与ArrayList仅在插入元素时**容量扩充机制不一致**。对于Vector，默认创建一个大小为10的Object数组，并将capacityIncrement设置为0；当插入元素数组大小不够时，如果capacityIncrement大于0，则将Object数组的大小扩大为现有size+capacityIncrement；如果capacityIncrement<=0,则将Object数组的大小扩大为现有大小的2倍。  

ArrayList在执行插入元素是超过当前数组预定义的最大值时，数组需要扩容，扩容过程需要调用**底层System.arraycopy()方法<u>进行大量的数组复制操作</u>**；在删除元素时并不会减少数组的容量（如果需要缩小数组容量，可以调用trimToSize()方法）；在查找元素时要遍历数组，对于**非null的元素采取equals的方式寻找**。

LinkedList在插入元素时，须创建一个**新的Entry**对象，并更新相应元素的前后元素的引用；在查找元素时，需遍历链表；在删除元素时，要遍历链表，找到要删除的元素，然后从链表上将此元素删除即可。



## 三者读写效率

LinkedList由于基于链表方式存放数据，增加和删除元素的速度较快，但是检索速度慢。为什么LinkedList的检索会慢呢？因为通过循环，在查找是需要把当前节点指向上一个或者下一个，来找到索引位置的节点。增删快是只需要**改变节点之间的引用关系。**

ArrayList对元素的**增加和删除都会引起数组的内存分配空间动态变化**（即对数组进行大量的复制（把数组向后移动））。因此对其进行插入和删除的速度比较慢，但是检索速度快。

~~~java
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

/**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
//每个节点都会保存前一个和后一个节点。现在来看一下它是如何进行查找的
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

/**
     * Returns the (non-null) Node at the specified element index.
     */
//最后调用了node(int index)方法
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


~~~



~~~java
/**
     * Returns the element at the specified position in this list.
     *
     * @param  index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        rangeCheck(index);
 
        return elementData(index);
    }

@SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

~~~

## 衍生问题

### java集合框架的设计结构（整体印象）

### Java提供的主要容器（集合和Map）类型，了解或掌握对应的数据结构、算法，思考具体技术选择。

### 将问题扩展到性能、并发等领域。

### 集合框架的演进与发展。

我这里以需要掌握典型排序算法为例，你至少需要熟知：

1. 内部排序，至少掌握基础算法：归并排序、交换排序（冒泡、快排）、选择排序、插入排序等
2. 外部排序：掌握利用和外部存储处理超大数据集，至少理解过程和思路。
3. 考察算法不仅仅是如何简单实现，而是刨根问底。
4. 比如那些排序是不稳定的（快排、堆排），或者思考稳定意味着什么？
5. 对不同数据集，各种排序的最好或最差情况
6. 从某个角度如何进一步优化（比如空间占用，假设业务场景需要最小辅助空间，这个角度堆排序就比归并优异等。

## 三大集合类

Collection接口时所有集合的根，然后扩展提供三大类的集合

### List

**有序集合**，它提供了方便的访问、插入、删除等操作。

### Set

Set是**不允许重复元素**的，这是和List最明显的区别，也就是不存在两个对象equals返回true。日常开发中我们有很多需要保证元素唯一性的场合。

### Queue/Deque

是Java提供的**标准队列结构的实现**，除了集合的基本功能，它还支持类型**先入先出（FIFO，First-In-First-Out）**或者**后入先出（LIFO， Last-In-First-Out)**等特定行为，这里不包括BlockingQueue，因为通常是并发编程场合，所以被放置在并发包里。

了解基本特征和典型使用场景，以Set的几个实现为例：

1. **TreeSet**支持**自然顺序访问**，但是添加、删除、包含等操作要相对低效（log（n）时间）。
2. **HashSet**是利用**哈希算法**，理想情况下，如果哈希散列正常，可以提供常数时间的添加、删除、包含等操作，但是**不保证有序**。
3. **LikedHashSet**，内部构建一个**记录插入顺序的双向链表**，因此提供了按照插入顺序遍历的能力，与此同时，也保证了常数时间的添加、删除、包含等操作，这些操作性能略低于HashSet，因为**需要维护列表的开销**。
4. 在**遍历元素**时，**HashSet**性能受自身容量影响，所以初始化时候，除非有必要，不然不要将其背后的HashMap容量设置过大。而对**LinkHashSet**，由于其**内部链表提供的方便**，**遍历元素只和元素多少有关**。

## 理解 Java 提供的默认排序算法，具体是什么排序方式以及设计思路

### Arrays.sort()还是Collections.sort()(底层调用Arrays.sort())；

### 什么数据类型；

- 对与**原始数据类型**，目前使用的是所谓的**双轴快速排序**（Dual-Pivot QuickSort),是一种改进的快速排序算法。
- 对与**对象数据类型**，目前则是使用**TimSort**，是一种归并和二分插入排序结合的优化排序算法。

### 多大的数据集（太小的数据集，复杂排序是没有必要的，java会直接进行二分插入排序）等。



## 一课一练

需要实现一个云计算任务调度系统，希望可以保证VIP客户的任务被优先处理，你可以利用哪些数据结构或者标准的的集合类型呢？类型场景大多是基于什么数据结构呢

在这个题目下，自然就会想到优先级队列了，但还需要额外考虑vip再分级，即同等级vip的平权的问题，所以应该考虑除了直接的和vip等级相关的优先级队列优先级规则问题，还得考虑同等级多个客户互相不被单一客户大量任务阻塞的问题，数据结构确实是基础，即便这个思考题考虑的这个场景，待调度数据估计会放在redis里面吧

既然是Java的主题，那就用PriorityBlockingQueue吧。
如果是真实场景肯定会考虑高可用能持久化的方案。
其实我觉得应该参考银行窗口，同时三个窗口，就是三个队列，银台就是消费者线程，某一个窗口vip优先，没有vip时也为普通客户服务。要实现，要么有个dispatcher，要么保持vip通道不许普通进入，vip柜台闲时从其他队列偷

























