---
author: "LongWei"
title: "集合面试题笔记"
date: "2025-03-13"
tags: ["Collection"]
categories: ["Jobs"]
series: ["JobLearning"]
ShowToc: true
TocOpen: true
---

### 6319 HashMap 原理

数据结构：数组 + 链表  （JAVA8之后：数组+链表+红黑树）

- 数据结构 
  - jdk1.7：数组+链表，数组每个元素是一个链表的表头，当发生冲突，将新元素添加在头部(头插法)
    - hash冲突：头插法，❗扩容时可能造成环形链表
  - jdk1.8：
    - 引入红黑树，当链表节点超过8个，那么这个链表会转换为红黑树。查询时间由 O(n) 优化为 O(logn)。当节点小于6个，再转换为链表。
    - hash冲突：尾插法，避免环形链表❗
  - 扩容机制
    - jdk1.7
      - 扩容，元素会重新计算hash值，并分配到新的扩容数组中。⭐比较耗时
      - 扩容时，头插法，在多线程情况下，可能造成环形链表
    - jdk1.8
      - 扩容时，利用了元素哈希值和旧数组容量关系，减少了重新计算的哈希次数
      - 扩容时，尾插法，避免环形链表





- 使用键的`hashcode()`计算hash值，然后(n-1) & hash确定数组中的位置。
  - n-1 => 10000 - 1 = 01111
- HashMap的初始默认容量为16，负载因子为0.75。当存储的元素达到75%时，进行扩容，扩容为原来的2倍空间



**扩展知识：**

- hashmap的红黑树优化：
  - JAVA8开始，为了优化hash冲突时的查找性能。但链表的长度**超过8**时，链表会转变为红黑树。红黑树是一种自平衡的二叉搜索树 查找时间 O(n) => O(logn)。 当元素少于6个，切换为链表

- hashCode() 和 equals()的重要性：
  - hashCode计算hash值，决定键的存储位置。 而hashCode相同=>冲突。 equals()比较的值



```java
// HashMap Node class
static class Entry<K,V> {
    final K key;    // 键
    V value;        // 值
    final int hash; // 计算后的哈希值
    Entry<K,V> next; // 指向下一个元素的指针（形成链表）
}
```

**为什么头插法&多线程&扩容会造成环形链表?**

```java
void transfer(Entry[] newTable) {
  Entry[] src = table; 
  int newCapacity = newTable.length;
  for (int j = 0; j < src.length; j++) {  // 遍历旧表
      Entry<K,V> e = src[j];              // 取出当前桶的头结点
      if (e != null) {                     // 旧桶不为空
          src[j] = null;                   // 释放旧桶的引用（防止 GC 误回收）
          do { 
              Entry<K,V> next = e.next;      // 记录下一个节点
              int i = indexFor(e.hash, newCapacity);  // 计算新索引（线程1暂停）
              e.next = newTable[i];          // 头插法：将 e 连接到 newTable[i] 上
              newTable[i] = e;               // 将 e 作为 newTable[i] 的新头
              e = next;                      // 继续处理下一个节点
          } while (e != null);               // 直到旧链表遍历完
      }
  }
}

old: [ ][ ][ ]     old: [ ][ ][ ]
     [1]                [2]
     [2]        =>

new: [ ][ ][ ]     new: [ ][ ][ ]   // 头插法，将节点进行转移
                        [1]

// 环的情况，HashMap多个线程共享 
Thread1:   e ->  [1][ ][ ]             Thread2: [3][ ][ ]
          next-> [2]                   扩容成功  [2]
                 [3]                            [1]
              
执行
{
 e.next = newTable[i];
 newTable[i] = e;
 e = next;
}              

newTabel[0] => [1]->[3]->[2]->[1]
                ⬆️  ⬅️      ⬅️
```

<img src="http://verification.longcoding.top/FmtTH6S92haO65uNpE51bzyvFmy0" alt="image-20250308200751105" style="zoom:70%;" />



### 4948 HashMap，有哪些提升性能的技巧？

- 合理设置初始容量，减少resize，减少扩容操作。默认16
- 调整负载因子，查找多的设置小一点，减少冲突情况，冲突少了(查询效率就高了)，但会提高内存占用情况，反之亦然。默认0.75

- 确保hashCode(), 均匀的是均匀分布的，以减少hash冲突。



扩展：

- 当元素数量 > 装填因子 * 数组大小 => 进行扩容，新数组为当前的2倍，并把当前hash中的数据重新分配到新的hashmap中；
- LinkedHashMap，拉链法，能够保存插入顺序
- 保存有序 => TreeMap.
- 需要线程安全 => ConcurrentHashMap



### 4947 Hash碰撞 & 解决

不同key使用hashcode()&n-1得到相同的坑，然后冲突

- 链地址法(拉链法)
- 开放地址法，顺序往下or左右横跳  
  - 线性探测
  - 平方探测
- 再hash法(双重hash)
  - 出现碰撞后，使用第二个hash函数计算新的索引位置，减少碰撞的概率。



### 4946 CopyOnWriteArrayList & Collections.synchronizeddList 区别

**CopyOnWriteArrayList** 是线程安全的List实现，特性是写时复制

每次对List的修改操作(add, remove, set)都会创建一个新的底层数组。 读操作不需要加锁，而写操作需要加锁

*优点*：

- 读操作无锁，写操作会创建数组副本 => 读写不冲突
- ⭐读操作在当前数组上执行； 写操作(add,set,remove)会创建一个新数组，新数组上修改再替换旧的(加锁)。

*缺点*：

- 写操作开销大：每次写操作都会创建并复制新数组，并且将数据复制到新的数组中，**写频繁的场景下性能会比较低**
- 内存消耗大

🎉**适合读多写少的场景**



**Collections.synchronizeddList** 包装方法，将任何数组转换为线程安全的版本，会对每个访问方法(get, set, add, remove)进行同步(加锁)，线程安全。

优点：

- 方便

缺点：

- 并发效率低

🎉**适用于简单的将List转为线程安全的版本使用。**



---

### 444 Java中有哪些集合类？

- List接口
  - ArrayList
  - LinkedList
  - Vector
    - 基于动态数组实现，且线程安全，所有方法都添加了synchronized

- Set接口

  - HashSet // 冲突的链表元素无序
  - LinkedHashSet // 继承HashSet，底层由双向链表实现，保证**插入顺序**
  - TreeSet // 基于红黑树, **自动排序**，提供排序功能

- Queue接口

  - PriorityQueue， 优先队列，内部元素按级别排序。只能比较器
  - LinkedList，可以作为队列使用，FIFO

- Map接口

  - HashMap
  - LinkedHashMap
  - TreeMap
  - HashTable<span style="color:red;">不推荐使用</span>，线程安全的哈希表。**早期线程安全的HashMap，锁的粒度为整个表，这样并发能力不高。**底层使用synchronized关键字实现
  - ConcurrentHashMap，线程安全的hashmap。
    - 高性能的线程安全HashMap
    - 提供更细粒度的锁 **分段锁和CAS操作**
    - 读操作不加锁
    - 写操作 => CAS + synchronized细粒度锁

  

### 9179 ArrayList的扩容机制是什么？

默认容量10，当元素数量超过其当前容量，会触发扩容机制。  扩容大小为原数组的1.5倍



### 449 HashSet & HashMap

HashSet存储不重复元素 => 不能冲突的HashMap

HashMap，K=>V, 键唯一，但不同键的值可以相同



### 451 HashMap的扩容机制

负载因子默认0.75, 但比例超过负载因子，进行扩容，扩容为原来的2倍。

当前hashmap中的元素在新hashmap的位置怎么计算？ 就是hashCode() & (n-1) => hashCode() & (2n-1)

优化过程

```java
// example:
 n:16= 1,0000    n-1: 0,1111      2^4
2n:32=10,0000   2n-1:01,1111      2^5
    
// 通过hash值 & 2n-1:
// 只要判断最高位是否即可
    
if 第5位== 0：then
    位置不变；
else
    位置偏移16位
```



### 452 为什么HashMap扩容采用2的n次方？

- 计算散列位置决定=> hashcode & (n-1)   其中 n-1=01111.. 可以将位置映射到数组的每个元素上，分布均匀。

**位运算比取余效率高**





### 457 Java中的LinkedhashMap

基于HashMap & 双向链表 & 额外的顺序指针

🏷️作用：保持插入顺序or访问顺序

💠定义如下：

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Ectry(){...} // constructor
}
```

🏷️图示：

<img src="http://verification.longcoding.top/FgiN-yE6NKTgTyTo8lwgP6bAlHwY" alt="image-20250308215346671" style="zoom:50%;" />

构想=> 由于可以根据插入时间排序，因此可以模拟LRU(least recently used)算法，进行淘汰最不常用的节点。



---

### 461 ConcurrentHashMap

🏷️背景：在多线程并发条件下，普通的共享HashMap会存在线程安全问题，因为没有加锁，导致数据被覆盖...；

🏷️作用：通过同步机制加锁，使得HashMap在背景下是线程安全的，且还需要保证效率；

🏷️定义：

- JDK1.7之前，采用**分段锁**，即每个**Segment**是独立的，将锁的粒度下方，提高线程并发度
  - 利用数组+链表实现

<img src="http://verification.longcoding.top/FpkreN_HAJJZ5eBpsILirxXMj-Lx" alt="image-20250308220417373" style="zoom: 33%;" />

=> 锁的粒度是segment；

缺点：segment不能扩容，而HashMap会扩容

- JDK1.8后，移除Segment，锁的粒度更加细化，锁只在链表or红黑树**节点级别**上竞争锁。通过CAS进行插入操作，只有在更新链表or红黑树时才使用synchronized关键字加锁，并且只锁住链表的头节点。
  - 同步HashMap，数组+链表+红黑树实现；

<img src="http://verification.longcoding.top/FlB1SjHxwwX53SerODlPwBfEXLjP" alt="image-20250308220632518" style="zoom:33%;" />

🏷️添加节点过程

- 计算key的hash后的下标，如果没有元素 => 利用CAS添加元素；
- 冲突 => 给这个节点上锁使用synchronized。这样其他线程就无法访问该节点以及之后的节点了

🏷️额外扩展（为什么分为CAS&Synchronized呢？）

- 无锁机制CAS，能进一步提高并发性能； 因此首次添加节点时，利用CAS很OK，判断&插入
- 而后续对链表的修改，很难原子化操作orCAS比较交换。因此使用Synchronized，实现互斥，且该锁的粒度很小。

```java
// 自旋：如果 CAS 失败就重试
while (true) {
    currentNode = bucket.get();

    if (currentNode == null) {
        // 如果桶位置为空，尝试通过 CAS 插入新节点
        if (bucket.compareAndSet(null, newNode)) {
            return null; // 插入成功
        }
    } else {
        // 如果当前位置有节点，进行链表遍历，寻找合适位置插入
        synchronized (currentNode) {  // 在访问链表节点时加锁，防止并发修改
            Node<K, V> prev = null;
            while (currentNode != null) {
                if (currentNode.key.equals(key)) {
                    // 如果找到相同的 key，更新 value
                    currentNode.value = value;
                    return currentNode.value;
                }
                prev = currentNode;
                currentNode = currentNode.next;
            }
            // 如果没有找到相同的 key，在链表末尾添加新的节点
            prev.next = newNode;
            return null; // 插入成功
        }
    }
}
```





---

### 464 CopyOnwriteArrayList

**线程安全的动态数组**，通过**写时复制机制**，保证数据最终一致性和隔离性，并不保证强一致性。



- 写操作和读操作(Volatile修饰)分离，读完全ok(可能读到旧版本)；写需要加锁且复制数组，再修改旧数组的引用。

  

***线程安全问题：***

- 竞争条件
  - 多个线程对共享变量进行累加，可以导致统计不对
- 死锁
  - 多个线程操作数据时，没有正确的请求资源和释放顺序，相互等待
- **可见性**问题
  - 某个线程对共享变量的修改，其他线程不一定及时可见
    - => volatile 强制从内存中修改&读取
- **原子性**问题
  - 逻辑由多个操作步骤组成，其中CPU时间片到期了，中途其他线程修改了共享变量，导致不一致。
- 线程饥饿
  - 多线程争抢CPU，非公平，有些永远抢不到
- **不一致**视图
  - 多个线程同时修改一个共享对象，导致一部分数据不一致
    - 强制视图一致，就是我第一次拿到的数据，和最终的数据要一致，其他线程别给我改了一部分啊 => 加synchronized