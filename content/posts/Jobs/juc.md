---
author: "LongWei"
title: "并发面试题笔记"
date: "2025-03-05"
tags: ["JUC"]
categories: ["Jobs"]
series: ["JobLearning"]
ShowToc: true
TocOpen: true
---

### 1 如何创建线程

1） 实现**Runnable**接口

- 实现run()方法

```java
new Thread(MyRunnable()).start();
```

2）继承Thread类

- 重写run()方法

3）**Callable接口**&&FutureTask

- 实现Callable call()方法，使用FutureTask包装Callable对象，通过Thread启动

```java
FutrueTask<ReturnType> task = new FutrueTask<>(MyCallable());
Thread(task).start
    
ResultType res = task.get(); // 这里阻塞
```

4）使用线程池

- 通过ExecutorService提交Runnable或者Callable任务



不同方法对比

```java
Runnable vs Callable
Callable:可以返回结果，可以抛出异常

线程池的优势：避免重复创建和销毁线程，减少这部分重复带来的开销；
ThreadPool => (FixedThreadPool, CachedThreadPool, ScheduledThreadPool)
   
虚拟线程：虚拟线程创建和切换开销更低
Thread.startVirtualThread()
```



**ThreadPool实践**

XXXThreadPool.submit(task...)

```java
FixedThreadPool: 固定池中线程数量 => 适合1.执行较长任务；2.控制并发度
    
CachedThreadPool：池中线程数量不固定，根据需要动态创建线程，空的被回收，少了多创建；
⭐适用于任务执行时间短且任务数量不确定的场景；
    
ScheduledThreadPool：适用于需要定时执行或周期性执行任务的场景。    
```

```java
// 定时任务线程池
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
// For 延迟任务
scheduledThreadPool.schedule(() -> {
    // 任务逻辑
}, 10, TimeUnit.SECONDS); // 延迟10秒执行

// For周期执行任务
scheduledThreadPool.scheduleAtFixedRate(() -> {
    // 任务逻辑
}, 0, 1, TimeUnit.SECONDS); // 每隔1秒执行一次
```



Thread.sleep(0) <= 主动让出CPU控制权

thread.join() <= 等待该线程完成



### 2 创建线程池

1） Executors**工厂类**

- Executors.newFixedThreadPool(10) //

2）ThreadPoolExecutor直接创建**线程池**

```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, KeepAliveTime,TimeUnit,new LinkedBlockingQueue<>())
    
LinkedBlockingQueue: 当提交的任务多于线程数时，会将多余的暂时挂起
```

**线程池相关参数解释**

- corePoolSize：核心线程数，即线程池中始终保存的线程数量；
- maximumPoolSize：最大线程数，线程池中允许的最大线程数量；
- KeepAliveTime：线程空闲时间，非核心线程空闲超过这个时间会被销毁；
- workQueue：工作队列，存放待执行的任务；
- threadFactory：线程工厂，用于创建新线程；
- rejectedExecutionHandler：任务拒绝处理器，当前任务无法执行时的处理策略。

**工作队列**

- SynchronousQueue：不存储任务，直接提交任务给线程；
- LinkedBlockingQueue：链表结构的阻塞队列，大小无限；
- ArrayBlockingQueue：数据结构的阻塞队列；
- PriorityBlockingQueue：带优先级的无界阻塞队列。/ praɪˈɒrəti /

**线程池的拒绝策略**

队列满且无空闲线程的添加任务的情况。

- AbortPolicy：抛异常；
- CallerRunsPolicy：由提交线程本身执行；
- DiscardOldestPolicy：删除最早提交的任务；
- DiscardPolicy：直接丢弃当前任务。



### 3 线程安全的集合

- 加锁
- 内置锁的 线程安全的容器
- 原子类
- 自己独享的资源：局部变量+ThreadLocal
- 无锁CAS

```java
concurrentHashMap: 同步HashMap<K->V>
数据结构=> 使用数组 + 链表 + 红黑树的结构;
线程安全=> CAS保证操作的原子性
    
eg:线程不安全的操作：
// 非线程安全的操作
int value = map.get("key"); // 这里，如果有其他线程对key->value进行了修改！，这里的value是过期了的
map.put("key", value + 1);
// ⭐保证原子性
map.compute("key", (k, v) -> v + 1); // 一次完成

// 如果是方法中创建的Map,每个线程专享，不会有线程间安全问题
// 如果是类中，且多个线程共享同一个Class实例，则可能出现线程安全问题
Method中创建：安全
Class-成员变量：多个线程可能共享同一个map实例，需要保证线程安全
Class-静态变量：所有线程共享同一个 map 实例 ，需要保证线程安全

BlockingQueue： -数组实现
LinkedBlockingQueue: 线程安全的阻塞队列-链表实现
```

### 4 线程同步

多线程情况下，避免共享资源同时访问，引发数据不一致。=> 加锁，限制访问。

JAVA中，常见的同步方式：

**synchronized** 同步 // 仅限一个线程访问.  由 JVM 负责管理锁的获取和释放. 非公平锁

生产-消费 模拟

```java
Main：
// 共享队列，用于生产者和消费者之间的数据传递
Queue<Integer> queue = new LinkedList<>(); // or BlockingQueue
int maxSize = 5; // 队列的最大容量

// 创建生产者和消费者线程
Thread producer = new Thread(new Producer(queue, maxSize), "Producer");
Thread consumer = new Thread(new Consumer(queue), "Consumer");

// 启动线程
producer.start();
consumer.start();    
```

```java
Producer: 实现Runnable interface
@Override
public void run() {
    int i = 0;
    while (true) {
        synchronized (queue) { // 对队列加锁
            // 如果队列已满，生产者等待
            while (queue.size() == maxSize) {
                try {
                    System.out.println("队列已满，生产者等待...");
                    queue.wait(); // 释放锁，进入等待状态
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            // 生产数据并添加到队列
            System.out.println("生产者生产数据: " + i);
            queue.add(i++);

            // 通知消费者可以消费了
            queue.notifyAll();

            // 模拟生产耗时
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
Consumer：实现Runnable接口
@Override
public void run() {
    while (true) {
        synchronized (queue) { // 对队列加锁
            // 如果队列为空，消费者等待
            while (queue.isEmpty()) {
                try {
                    System.out.println("队列为空，消费者等待...");
                    queue.wait(); // 释放锁，进入等待状态
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            // 消费数据
            int value = queue.poll();
            System.out.println("消费者消费数据: " + value);

            // 通知生产者可以生产了
            queue.notifyAll();

            // 模拟消费耗时
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

⭐⭐⭐

synchronized（临界区对象）  <=  锁住临界区

while(临界区不可用) => 重试 => .wait() 当前线程进入等待状态**让出CPU**，并且**释放锁**

处理完成=> .notifyAll() // 通知等待状态的线程



**`synchronized`**：通过 `Object.wait()` 和 `Object.notify()/notifyAll()` 进行线程间通信



---

**ReentrantLock**  可重入锁  Re en trant lock.  需要手动加锁和释放锁

```java
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock();
}
```

**`ReentrantLock`**：使用 `Condition` 对象更灵活地控制线程等待和唤醒：

```java
Condition condition = lock.newCondition();
lock.lock();
try {
    condition.await();  // 线程等待
    condition.signal(); // 唤醒单个等待的线程
    condition.signalAll(); // 唤醒所有等待的线程
} finally {
    lock.unlock();
}
```

案例

```java
// 共享队列，用于生产者和消费者之间的数据传递
Queue<Integer> queue = new LinkedList<>();
// 创建 ReentrantLock 和 Condition
Lock lock = new ReentrantLock();
Condition notFull = lock.newCondition(); // 队列未满的条件
Condition notEmpty = lock.newCondition(); // 队列非空的条件

Producer:
while(true) {
    lock.lock()
    try{
        // 
        while(queue.size == maxSize) {
            System.out.println("队列已满，生产者等待...");
            notFull.await(); // 等待队列未满的条件
        }
        
        queue.add(?)
        notEmpty.signalAll()
        
        // ... 生产耗时
        
    }finally{
        lock.unlock()
    }
}

Consumer:
while(true) {
    lock.lock()
    try {
        while(queue.size == 0) {
            sout("没东西消费")
            notEmpty.await()    
        }
        
        queue.poll()
        notFull.signalAll()
            
        // ...消费时长
    }finally{
		lock.unlock()
    }
}
```

⭐⭐⭐ 关键用法：

Lock lock = new ReentrantLock();
Condition notFull = lock.newCondition(); // 队列未满的条件
Condition notEmpty = lock.newCondition(); // 队列非空的条件

lock.lock && lock.unlock

notFull.await(),  notFull.signal(),  notFull.signalAll()



---

### 5 线程安全

保证多线程，乱序执行时候，无论怎么执行，都可以得到预期结果

=> **通过 线程同步** (悲观锁)

通过Synchronized 和 ReentrantLock   ˈsɪŋkrənaɪzd &  riːˈentrənt, lɒk 实现线程同步

=> **通过原子操作类**

AtomicInteger  əˈtɒmɪk 原子，  原子整数

⭐⭐⭐***扩展CAS***（乐观锁）：（**Compare And Swap**，比较并交换）是一种无锁并发编程技术，常用于实现**原子操作**。它的基本原理是：**先比较，再交换**，即**只有当变量的当前值等于预期值时，才会更新，否则重试**。

```java
// 当前值, 预期值, 新值
boolean compareAndSwap(V, E, N) {
    if (V == E) { // 比较当前值是否等于预期值
        V = N;    // 如果相等，更新为新值
        return true; // 操作成功
    }
    return false; // 操作失败
}
```

例子：最佳实践案例=> 线程安全的计数器

```java
private AtomicInteger value = new AtomicInteger(0);

// 线程安全的递增方法
public void increment() {
    int oldValue;
    int newValue;
    do {
        oldValue = value.get(); // 获取当前值, 先获取修改值
        newValue = oldValue + 1; // 计算新值 
    } while (!value.compareAndSet(oldValue, newValue)); // CAS 操作
}
// CAS  compare and swap <=> 访问到修改期间，没有其他线程进行修改的话，就可以执行 
value.compareAndSet(oldValue, newValue) => oldvalue == value.get() ? value = newValue ：nothing
```

**检查第一次得到的旧值与修改时的值是否一致，判断是否被动过，没动过再改**

![image-20250303211427792](http://verification.longcoding.top/FivGPltzjm8vcyR15n4BDC2-1Bq9)

=> **线程安全的容器**：concurrentHashMap or copyonWriteArrayList❌ **不懂**

=> **局部变量**，线程专享

=> **ThreadLocal**, 线程本地资源，线程专享



### 6 线程生命周期

初始(资源) => 可运行(CPU队列) => 运行 => 终止

​                                                阻塞&等待    

![image-20250418231047799](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250418231047799.png)



### 7 线程通信

多线程间的协同工作

1）**共享变量：**访问共享内存变量来交换信息；

2）**同步机制：**  阻塞&唤醒

synchronized() => wait => notify&notifyAll  （Object中的方法）

ReentrantLock.lock => condition.await => condition.signal&signalAll

BlockingQueue => queue.put() 满则阻塞 => queue.take() 空则阻塞

```java
synchronized (lock) {
   while (conditionNotMet) {
       lock.wait(); // 释放锁，进入等待状态
   }
   // 执行操作
   lock.notify(); // 唤醒等待的线程
}
```

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

lock.lock();
try {
   while (conditionNotMet) {
       condition.await(); // 等待
   }
   // 执行操作
   condition.signal(); // 唤醒等待的线程
} finally {
   lock.unlock();
}

```

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

// 生产者
new Thread(() -> {
   try {
       queue.put("Data");
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt();
   }
}).start();

// 消费者
new Thread(() -> {
   try {
       String data = queue.take();
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt();
   }
}).start();

```



### 8 线程池

池化技术，预先创建并管理一组线程，避免线程重复创建和销毁带来的开销

关键配置：核心线程数，最大线程数，空间存活时间，工作队列，拒绝策略

=> 提交任务才会创建线程，或者设置preStartAllCoreThreads

=> 核心线程满了不会创建线程，而是把多余的任务放到工作队列中，等待执行

=> 核心线程满载且工作队列放不下了，才会新增线程执行提交的任务(<最大线程数)

=> 工作队列满了+已最大线程数了 => 拒绝策略?新任务

=> 线程空闲时间超过指定时间 且有多余的非核心线程 => 释放非核心线程



工作队列：

LinkedBlockingQueue 无界队列，链式

ArrayBlockingQueue 有界队列，数组

PriorityBlockingQueue 带有优先级的无界阻塞队列



线程池类型：

```java
FixedThreadPool: 固定线程数
CachedThreadPool：变化，动态新建
SingleThreadPool：单线程的池子
ScheduledThreadPool：定时任务的池子
```



shutdown与shutdownNow的区别

shutdown：提醒关闭，会把已提交的任务执行完毕

shutdownNow：强制停止



⭐**核心线程数、最大线程数、空闲线程最大存活时间、阻塞队列、拒绝策略**

Threads: [⭐⭐⭐⏱️⏱️⏱️]

Task:        [🏷️🏷️🏷️🏷️🏷️🏷️]

满了拒绝



### Java线程池的线程数设置

CPU密集型：CPU核心数+1。 这样刚好满载，并且当有线程下线，马上有替补上场

IO密集型：CPU核心*2.  满载情况，流水线形式，刚好CPU核心的线程等待IO，下一批线程补上。



### Java中的线程池拒绝策略

当线程满载 + 队列满载

- 抛出异常
- 交付调用者线程执行提交的任务
- 删除最早提交的任务
- 丢弃当前提交的任务
- 自定义拒绝策略(根据重要性吧，排序)



### 线程池内部线程异常怎么办

评论

- 执行方法是execute时，可以看到堆栈异常，移除异常线程并补充一个新的线程

- 执行方法是submit时，可以调用Future.get, 捕捉异常，但不移除和补充线程

面试鸭

- 线程内部手动try-catch



### 线程安全的集合

Vector：动态数组

HashTable：线程安全的Hash表

ConcurrentHashmap

CopyOnWriteArrayList

BlockingQueue

LinkedBlockingQueue



### 9 并发工具类

- ConcurrentHashMap：线程安全的HashMap，多线程修改临界区时加锁或者其他方法，=>安全  


- AtomicInterger：əˈtɒmɪk   线程安全的整型  compareAndSet CAS方法，原子类型
- Semaphore 信号量：acquire() and release()

- BlockingQueue：阻塞队列-通信容器  queue.put() and queue.take()
- CyclicBarrier：循环屏障 barrier.await 
- CountDownlatch：计时器 latch.countDown() latch.await()



### 497 ReentrantLock实现

=> 可重入锁，允许同一个线程多次获取同一把锁的锁机制，避免线程因为重复获取锁而导致死锁

案例：1. 递归调用中的锁保护 

```java
# 非公平锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) { // 锁未被占用
        if (compareAndSetState(0, acquires)) { // CAS 尝试获取锁
            setExclusiveOwnerThread(current); // 设置当前线程为独占线程
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) { // ⭐⭐⭐锁已被当前线程占用（重入）
        int nextc = c + acquires; // 增加重入次数
        if (nextc < 0) throw new Error("Maximum lock count exceeded");
        setState(nextc); // 更新状态
        return true;
    }
    return false; // 获取锁失败
}
```



**基于AQS实现的一个可重入锁**，支持公平和非公平两种方式

内部依靠一个**State变量**和两个等待**队列**：同步队列和等待队列

利用CAS修改state来争夺锁

- 争抢不到锁就入**AQS 等待队列**进行等待，**AQS 等待队列**是一个双向队列

- **抢到锁但是条件condition不满足**则入**条件队列(每个condition维护一个)**，单向链表

是否公平锁 => 线程获取锁 是加入同步队列尾部还是直接利用CAS争夺锁



![image-20250305195628416](http://verification.longcoding.top/FqPim2avU5jlSZ9MgW892NKCux-W)

`ReentrantLock` 是基于 AQS 实现的可重入独占锁，支持公平锁和非公平锁两种模式。其核心是通过 AQS 的状态管理（`state`）和等待队列来实现线程的阻塞和唤醒。非公平锁的性能通常优于公平锁，但**公平锁可以避免线程饥饿**问题。`ReentrantLock` 提供了比 `synchronized` 更灵活的锁操作，是 Java 并发编程中的重要工具



### 492 Synchronized实现

依赖于JVM的监视器锁+对象头

当synchronized修饰在方法或者代码块上时，会对特定的对象或者类加锁，确保只有一个线程能运行加锁的代码块；

- synchronized修饰方法：**方法的标志**位会增加一个ACC_SYNCHRONIZED标志，检查标志再获取锁，这部分进行同步控制
- synchronized修饰代码块：会在代码块前后插入monitorenter和monitorexit字节码指令，上锁+解锁

synchronized是可重入锁





### 491 Synchronized和ReentrantLock

![image-20250305205348468](http://verification.longcoding.top/FmLzFQnZFDkECyV1ZTIc2ZEK_yyO)

**Synchronized：**基于JVM实现

**ReentrantLock**：基于AQS实现







### Synchronized 锁升级

无锁阶段：最初(**没有对象拿这个锁**)，多个线程可能并不需要同步访问共享资源，因此可能不存在显式的锁。这是性能最高的阶段，因为不存在锁的开销。



偏向锁阶段：当某个线程多次访问共享资源时(**单一线程重复访问**)，JVM会认为这个线程将来还会继续访问该共享资源，因此会使用偏向锁。意味着被锁定的对象会标记上一个线程的ID，这样其他线程就无法获取到这个锁，只有这个锁可以无竞争的访问资源。



轻量级锁阶段：如果某个线程尝试获取一个被偏向锁锁定的对象，那么就会进入轻量级锁阶段(**多个线程交替，不同时竞争**)。此时，JVM会使用**CAS操作**来尝试获取锁。如果CAS成功，那么就得到锁，否则，进入重量级锁阶段。



重量级锁阶段：当轻量级锁阶段的CAS失败(**有线程CAS失败******，同时竞争******)，升级为重量级锁。意味着拿不到锁的线程会进入**阻塞状态**，需要进行上下文切换，导致性能下降。



### 495 什么是Java中的锁自适应自旋

思想：在锁竞争比较少的情况下，线程进入等待状态前，先执行一段自旋操作竞争锁，而不是直接进入等待状态。这样可以减少线程上下文环境切换带来的性能开销。

- 自旋：竞争锁失败后，线程会自旋一段时间争抢锁。反复检查
- 自适应：动态调整自旋的次数。基于之前的自旋结果，上次很快拿到锁，那么自旋久一点，否则自旋少一些。



### 11373 Volatile 和 synchronized

Volatile 修饰字段-变量：每次读写内存中的变量，保证变量的修改线程及时可见性

synchronized加锁，保证原子性，同一时间仅该线程占有该共享资源。



### 496 如何优化Java中锁的使用？

锁粒度(分段) or 无锁(CAS)



1. **减少锁的粒度**：
   1. 减少加锁的范围，减少锁的持续时间
2. 使用**更细粒度的锁**：提高并发度
   - hashTable:
     - 通过方法上添加synchronized实现锁的安全，仅一个线程，性能较差
   - concurrentHashMap：
     - 通过CAS+synchronized实现线程安全，允许多个线程同时读写，性能更高
3. 减少锁的使用
   1. 通过无锁编程、CAS操作和原子类来避免锁的使用，减少锁带来的性能损失
   2. 通过减少共享资源的使用，避免对临界区的竞争。(本地变量+线程本地变量)

扩展：

- 独占锁：写操作多的场景，仅允许一个线程持有锁
- 读写锁：允许多个线程并发读，但写的时候需要上锁，适合读多写少的场景
- 乐观锁和悲观锁：悲观锁每次都加锁；乐观锁假设没有冲突-CAS或版本号实现



### 499 读写锁

**允许多个线程同时读操作**，但是**写操作需要加锁(**单个线程)。

=> 读写+写写操纵是互斥操作；⭐适合读多写少的情况

可以利用ReadWriteLock和ReentrantReadWriteLock实现

```java
# 代码示例
    
ReentrantReadWriteLock lock = new ReentrantReadWriteLock()
Lock readLock = lock.readLock()
lock writeLock = lock.writeLock()

// 1. 判断写锁(读写互斥)2. 判断读锁,第一个和后续
readLock.lock() 
try{
    ...
} finally {
    readLock.unlock()
}

// 1. 有读锁或写锁且写锁不是当前线程持有，则失败
// 2. 根据公平性策略（公平锁或非公平锁）决定是否需要阻塞
// 3. CAS 更新状态
// 4. 设置写锁持有者 <= 设置可重入
writeLock.lock() // 互斥 写写互斥
try{
    ...
} finally {
    writeLock.unlock()
}
```



### 501 Java JMM java内存模型

 java memory model

**用于描述线程何时从主内存中读取数据、何时把数据写回主存中**

**屏蔽各操作系统之间的硬件差异，描述多个线程环境中的变量如何在内存中储存和传递**

![image-20250419001029125](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250419001029125.png)

JMM核心目标

- **可见性**：确保某个线程的修改，其他线程及时可见。 使用**volatile**关键字强制线程每次读写都直接从主内存中获取新值
- **有序性**：指线程执行操作的顺序，JMM允许某些指令通过指令重拍提高性能，且保证线程内的操作顺序不会被破坏，通过`happens-before`关系保证跨线程的有序性。
- **原子性：**指操作不可分割，线程不会在执行过程中被中断。

Why JMM：

操作系统有自己的内存模型，但JAVA是跨平台实现的，因此需要自己设计一套内存模型屏蔽各操作系统之间的差异。JMM描述了多线程环境下，如何在不通过的线程之间共享变量以及变量的操作顺序。



主内存和工作内存：

- 主内存：JAVA堆内存的一部分，所有的实例变量、静态变量和数组元素都存储在主内存中；
- 工作内存：每个线程都有自己的工作内存，工作内存存储了主内存中的变量副本，线程的所有操作都在工作内存中进行。
- 线程之间的变量，必须经过主内存进行传递



### 506 Why ThreadLocal

每个线程自己**独享的独立变量副本**，避免多个线程间的变量共享和竞争，解决线程安全问题。

**每个**线程维护一个`ThreadLocalMap` 用于存储线程独立的变量副本，ThreadLocalMap以**ThreadLocal实例**为键，不同线程通过自己(线程独立的)ThreadLocal身份获取各自的变量副本。

避免同一个ThreadLocalMap的竞争

**应用场景**

- 用户上下文管理
- 数据库连接管理



### 517 Java中wait、notify和notifyALL

这三个方法都是Object对象定义的方法，用于线程之间的通信，且需要在Synchronized修饰内使用

- **wait** => 线程进入等待状态，释放锁
- **notify** => 唤醒一个在等待的线程
- **notifyALL** => 唤醒所有等待的线程



### 518 死锁 及 避免

- 条件互斥：独享资源
- 占有且等待：不放手，等别人放弃
- 不可抢占：文明
- 循环等待： A=>B=>C=>A



避免：

- 按序申请 => 锁获取的顺序相同，这样就可以在前面卡住
- 超时等待时间=>释放手中资源和锁



### 519 volatile关键字

主要的作用还是保证变量的可见性

- 可见性：每次读取变量值都从内存读取最新的值。修改了volatile变量的值，该值会被立刻刷新回主内存中，及时让其他线程可见。



### 6304 如何知晓子线程是否执行完毕？

- ThreadObj.join() 会等待对应子线程执行完毕，套娃，线程A<=>线程B...
- FutureTask+Callable   futrue.get() 拿到线程执行完成的结果
- 回调机制：完成后，调用回调函数通知主线程，异步了



### 481 Semaphore 信号量

ˈseməfɔːr

主要作用就是确保 只有指定数量的线程能够访问资源，**限制同时访问特定资源的线程数量**

*基本概念*

- 许可 permits: 可以访问资源的线程数量。
- Acquire：尝试获取许可；
- release：释放许可。

- 公平：按照请求顺序获取许可，防止线程饥饿
- 非公平：可以提高性能。

常见用法：

```java
Semaphore semaphore = new Semaphore(10);  // 允许最多10个线程访问
semaphore.acquire(); // 失败会阻塞不会往下执行了
try {
    // 访问共享资源
} finally {
    semaphore.release();
}

Semaphore semaphore = new Semaphore(5);  // 允许最多5个线程同时执行任务
for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire(); // 阻塞
            // 执行任务
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            semaphore.release();
        }
    }).start();
}
```

### 482 CyclicBarrier

ˈsaɪklɪk ˈbæriə(r) 循环障碍 // **可以重用**，当所有线程到达屏障后，刷新

允许一组线程在执行某个任务相互等待，直到所有线程都达到了Barrier屏障后才能继续执行 // 直接全卡住

- 屏障：调用barrier.await() 阻塞，等待所有线程都到达屏障；
- 线程数量：预指定的，当所有线程到达屏障，所有线程才被唤醒；
- 重用性：可以被重用。

```java
# 示例
static CyclicBarrier cyclicbarrier;
cyclicbarrier = new CyclicBarrier(parties=10, () -> Sout("全部准备就绪"))

for(i ...)
    new Thread(() -> {
        Thread.sleep(i * 3000 ms)
        sout(i+"玩家准备完成");
        cyclicbarrier.await() // ⭐等待全部完成
        sout(i+"玩家进入游戏")
    })  
```

### 483 CountDownLatch

倒计时门闩锁  - 不可复用

作用：**使某线程等待其他线程执行完一组操作完成**。每当其他线程完成一个操作，计数器--，到达0则等待的ALL线程会被唤醒

主要功能：

- 等待事件完成：await()；
- 递减计数器：latch.countdown()；
- 线程同步：当计数器变为0，唤醒线程。

```java
CountDownLatch latch = new CountDownLatch(3)

for(int i = 1; i <= 3; i++) { // 异步
    new Thread(() -> {
        Thread,sleep(i*3000)
        sout(i+"???")
        latch.countDown(); // 这里模拟递减计数器
    }).start()
}    

sout("wait all thread finish")
latch.await() // 主线程阻塞等待
sout("all thread finish")    
```

// 并行计算结果的汇总



### 487 如何控制多个线程的执行顺序呢？

- synchronized + awit + notify, A => B => C 多组锁
- ReentrantLock + condition 多组
- Thread.join, 逐步等待
- CountDownLatch，等待其他的线程countdown
- semaphore，限制异步为同步顺序

==

- synchronized  **等待&唤醒**

```java
public void methodA() {
    synchronized (lock) {  // 锁的是 lock
        lock.wait();       // wait() 也是在 lock 上调用 ✅, 释放锁，并进入等待状态
    }
}

public void methodB() {
    synchronized (lock) {  // 锁的是 lock
        lock.notify();     // notify() 也是在 lock 上调用 ✅
    }
}
```

- reentrantLock + condition
- 信号量
- Thread中的Join()会进行等待线程执行完毕，套娃
- 单线程池，有序提交任务，**顺序执行**





### 488 Java阻塞队列

- ArrayBlockingQueue
- LinkedBlockingQueue
- PriorBlockingQueue praɪə(r)

- delayQueue: 当队列中元素延迟时间到了，才能取出来



### 489 原子类

- AtomicInterger                           əˈtɒmɪk ˈɪntɪdʒə(r)
- AtomicLong
- AtomicBoolean
- AtomicStampedReference            stæmpt

```java
.get
.compareAndSet
.getAndIncrement    ˈɪŋkrəmənt
```





### 提问

#### CAS

Compare and swap

比较内存中的值 是否与 之前的预期值(之前拿到的旧值) 相等

相等 => 将该变量的值设置为新值

=> 判断从之前的访问到现在的修改，中间变量的值是否变动过，没变过=>可以修改

优势：

**无锁并发** + CAS是原子性的(线程安全)

缺点：

- ABA问题，如果变量值 从 A=>B=>A,CAS无法检测到这种变化
- **自旋(重复)开销**：导致CPU资源浪费，因为一直比较，直到能够修改为止
- 单变量限制：仅支持修改单变量



```java
public class SpinLock {
    private final AtomicBoolean lock = new AtomicBoolean(false);

    public void lock() {
        while (!lock.compareAndSet(false, true)) {
            // 自旋等待
        }
    }

    public void unlock() {
        lock.set(false);
    }
    
 } // 短时间自旋，避免线程上下文切换的开销
```



ABA问题：引入版本号或者时间戳，每次更新变量的同时更新版本号，从而依靠版本号判断变量是否被调整过。

做法：

```java
private AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(0, 0); // 初始值:版本号 == 0:0

public void updateValue(int expected, int newValue) {
    int[] stampHolder = new int[1];
    Integer currentValue = atomicStampedReference.get(stampHolder);
    int currentStamp = stampHolder[0];
	
    // 
    boolean updated = atomicStampedReference.compareAndSet(expected, newValue, currentStamp, currentStamp + 1);
    if (updated) {
        System.out.println("Value updated to " + newValue);
    } else {
        System.out.println("Update failed");
    }
}
```

AtomicStampedReference：<ObjRef => stampedref>

**尝试更新值和版本号**

boolean updated = atomicStampedReference.compareAndSet(expected, newValue, currentStamp, currentStamp + 1);

期望的当前值，更新值，期望的版本号，更新的版本号; **当期望的两个值相同才更新**

辅助理解

```java
expected, newValue <= ready

// 获取当前引用值和版本号
int[] stampHolder = new int[1];

// 因为Java只能返回一个返回值，将多个结果修改数组的形式间接返回
Integer currentValue = atomicStampedRef.get(stampHolder); 
int currentStamp = stampHolder[0]; // ⭐⭐⭐

"当前值: " + currentValue
版本号: " + currentStamp 
    
atomicStampedReference.compareAndSet(expected, newValue, currentStamp, currentStamp + 1);
```

在CAS基础上，多判断一个版本号，检查变量是否修改过，解决ABA问题

// ##

自旋锁 => 获取锁失败，不会阻塞，而是重复尝试获取锁 ❗非公平:饥饿；性能问题:对同一变量高并发进行CAS

```java
public class SpinLock {
    private final AtomicBoolean lock = new AtomicBoolean(false);

    public void lock() {
        while (!lock.compareAndSet(false, true)) {
            // 自旋等待
        }
    }

    public void unlock() {
        lock.set(false);
    }
    
 }

```

针对自旋锁=>CLH 基于队列的自旋锁

![image-20250418235623832](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250418235623832.png)

CAS之前节点的完成状态



AQS对CLH的改造

![image-20250418235721056](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250418235721056.png)

阻塞和通知



#### ❌❌❌AQS

**Abstract Queued Synchronizer** 是**同步**器**的基础框架**， **起到抽象、封装的作用**，将一些排队、入队、加锁、中断方法提供出来，具体加锁时机、入队时机等需要实现类自己控制。

- **(volatile申明)state状态**，可以通过CAS无锁并发方式竞争锁

AQS支持

1. 独占模式：只有一个线程可以执行，互斥锁； 
2. 共享模式：多个线程可以同时执行，例如信号量；

当一个线程获取锁失败时，AQS将其插入等待队列中，并阻塞线程，直到同步状态可用。

使用AQS实现一个独占锁

```java

public class SimpleLock {
    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int releases) {
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

```java
# 基于AQS的独占锁
tryAcquire: => CAS(原子，比较且设置)检查是否锁可以使用 => 可以则设置当前线程为独占模式 true=>else false
    
tryRelease: => getState => 如果不为1则异常 => else 关闭独占模式并释放锁(state设置为0)
    
private final Sync sync = new Sync() // 作为锁对象
lock: sync.acquire(1)
unlock: sync.release(1)
```



AQS和CAS两者可以经常一起使用，例如在ReentrantLock中，CAS用于实现锁的获取和释放操作，而AQS则管理锁的状态和等待队列。



== 

美团技术团队

```java
// **************************Synchronized的使用方式**************************
// 1.用于代码块
synchronized (this) {}
// 2.用于对象
synchronized (object) {}
// 3.用于方法
public synchronized void test () {}
// 4.可重入
for (int i = 0; i < 100; i++) {
	synchronized (this) {}
}

// **************************ReentrantLock的使用方式**************************
public void test () throw Exception {
	// 1.初始化选择公平锁、非公平锁
	ReentrantLock lock = new ReentrantLock(true);
	// 2.可用于代码块
	lock.lock();
	try {
		try {
			// 3.支持多种加锁方式，比较灵活; 具有可重入特性
			if(lock.tryLock(100, TimeUnit.MILLISECONDS)){ }
		} finally {
			// 4.手动释放锁
			lock.unlock()
		}
	} finally {
		lock.unlock();
	}
}
```



非公平锁

```java
// 非公平锁
static final class NonfairSync extends Sync {
	...
	final void lock() {
		if (compareAndSetState(0, 1))
			setExclusiveOwnerThread(Thread.currentThread());
		else
			acquire(1);
		}
  ...
}
```

- 若通过CAS设置变量State（同步状态）成功，也就是获取锁成功，则将当前线程设置为独占线程。
- 若通过CAS设置变量State（同步状态）失败，也就是获取锁失败，则进入Acquire方法进行后续处理。

某个线程获取锁失败的后续流程是什么呢？有以下两种可能：

(1) 将当前线程获锁结果设置为失败，获取锁流程结束。这种设计会极大降低系统的并发度，并不满足我们实际的需求。所以就需要下面这种流程，也就是AQS框架的处理流程。

(2) 存在某种排队等候机制，线程继续等待，仍然保留获取锁的可能，获取锁流程仍在继续。

![image-20250418234428064](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250418234428064.png)





