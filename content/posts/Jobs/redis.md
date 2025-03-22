---
author: "LongWei"
title: "Redis面试题笔记"
date: "2025-03-01"
tags: ["Redis"]
categories: ["Jobs"]
series: ["JobLearning"]
ShowToc: true
TocOpen: true
---

### 637 常见的数据类型

- String
- Set
- Hash
- List
- Zset （Sorted Set）

BitMap => 位图，考勤，或者xxx分配情况

HyperLogLog => 用户访问的独立用户数量

GEO => 地理



应用场景：

- 缓存
- 实时系统
- 消息队列
- 分布式锁
- 计数器：页面访问量、点赞数、评论数



### 651 Redis 主从复制的实现原理

➡️主节点-从节点 的数据同步  

Why 需要主和从

- 数据冗余&故障恢复，某个节点宕机了，但是其他节点还活着；
- 提供负载均衡，配合读写分离策略，主节点写操作，从节点提供读操作；
- 高可用：主从复制是Redis的高可用的基础，也是哨兵和集群实施的基础。



复制流程： 全量 & 增量

- 主节点发送SYNC命令与从节点进行连接，开始同步， 主从之间建立联系；

- 全量复制(第一次连接)： 主节点把全部数据复制到从节点，主节点将当前数据生成RDB文件，发送给从节点；
- 发送的期间，主节点缓存(Replication backlog buffer-复制积压缓冲区)所有写命令。
- 发送缓存的写命令给从节点。
- 持续同步，持续把写命令同步给从节点。



3. 保持连接与断线重连

   Redis中，主节点会和从节点保持长连接，以确保数据的持续同步；

   当连接断开后，重连，请求增量复制，避免全量复制带来的大量开销。

4. 数据一致性和复制延迟

   由于网络延迟，主从之间会存在短暂的数据不一致。 ⭐对于数据一致性严格的任务，要求访问主节点。



---

### 652 Redis集群实现的原理

➡️通过多个Redis实例实现，每个实例存储部分数据，且这些数据是不重复的。**类似于将数据库分库，按业务功能**

➡️具体为采用Hash Slot哈希槽，将键的空间划分为16384个槽。每个Redis实例只负责一部分槽。 

客户端 => 任意Redis实例 => 数据是否在本机上，在返回，否则返回目标节点信息，客户端再路由到其他Redis中。

<= 将单个Redis的压力，分摊到多台Redis实例上，提高并发性能。



特性

- 实现数据分布式存储，对数据进行分片，将不同数据储存在不同节点中
- 去中心化思想，无中心节点，访问任意一个即可。访问正确则响应数据，否则响应对应的节点信息，客户端再次访问。
- 内置高可用性：分为N组，每组提供不同的数据缓存服务，每组中又有一个主节点和K个从节点(主提供读写，从提高读，并进行数据同步功能)

![image-20250307175456538](http://sthda9dn6.hd-bkt.clouddn.com/Fo9jCIPyRQ866mnZdasVsUwqao1j)



布局：

- 集群有多个master，每个master保存不同的数据(海量数据)
- 每个master有多个slave (支持高并发读)
- master之间通过ping检查彼此的健康度
- 客户端请求可以访问集群任意节点，最终都会被转发到正确的节点(路由规则)

存储与读取

- 分片集群引入hash slot，一共有16384个槽
- 不同实例处理不同的槽
- 读写数据：根据有效部分计算hash值，对16384取余，得到插槽，寻找插槽所在实例。



传统哈希

```java
hash(key)%N  N:服务器数量， N一旦变了，大部分数据都需要重新映射到新的服务器上
```

一致性哈希❌❌❌



### 635 Redis为什么快

存储方式、线程模型、IO模型、数据结构

- **基于内存**
- **单线程**事件驱动(避免线程切换开销，避免锁竞争,数据一致性) 结合 **IO多路复用**(单线程同时处理多个*客户端连接*)
- 高效数据结构 => （String，List， Set）

-  多线程引入 => 网络处理并发请求，减少网络IO等待影响 (网络IO可能成为性能瓶颈) 
  - Redis主线程很快，但是网络IO处理不一定更得上它的速度，可能成为累赘。
  - 使用多个IO进程加快网络IO速度(数据序列号&反序列化，客户端请求的解析...)

![image-20250307181554873](http://sthda9dn6.hd-bkt.clouddn.com/Fp7r8RqwKZyeOXh1NE2-FJK7m5jX)

### 638 跳表

多层链表实现，底层链表保存所有元素，每层链表都是下一层的子集 (**有序链表的基础上添加多级索引**)

特性

- 快查找某个元素 & 范围查询 复杂度**O(logn)**

```java
Level 3:       1 ---------------> 9 ----------------> 20
Level 2:       1 ------> 5 -----> 9 ----> 13 -----> 20
Level 1:       1 -> 3 -> 5 -> 7 -> 9 -> 11 -> 13 -> 15 -> 20
Level 0:       1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10 -> 11 -> 12 -> 13 -> 14 -> 15 -> 16 -> 20
```

从高层开始往下找



插入时候呢
**插入的随机层级：概率函数决定从下(level-0)到上(level-n) 概率为 0.25(n)**

跳表结构

```java
Class SkipNode{
    int value;
    SkipNode[] forward;// 指向不同层级的后继， 下一个level节点
    
    public SkipNode(int value, int level) {
        this.value = value;
        this.forward = new SkipNode[level + 1]; // 0 到 level 层 每一个节点，动态的数据长度
    }
}
```

```java
class SkipList {
    private static final int MAX_LEVEL = 16; // 最大层数
    private int level; // 当前层数
    private SkipListNode header; // 头节点
    private Random random; // 随机数生成器，用于决定节点层数
	
    // 初始化头节点，其中有[0,...,MAX_LEVEL]的节点
    public SkipList() {
        this.level = 0;
        this.header = new SkipListNode(Integer.MIN_VALUE, MAX_LEVEL);
        this.random = new Random();
    }
    
    // 插入节点
    public void insert(int value) {
        SkipListNode[] update = new SkipListNode[MAX_LEVEL + 1]; // 记录每层需要更新的节点位置
        SkipListNode current = header;

        // 从最高层开始查找插入位置
        for (int i = level; i >= 0; i--) {
            // 找到插入节点的位置
            while (current.forward[i] != null && current.forward[i].value < value) {
                current = current.forward[i];
            }
            update[i] = current; // 记录插入节点的位置
        }

        // 随机生成新节点的层数
        int newLevel = randomLevel();

        // 如果新节点的层数大于当前层数，更新 update 数组和当前层数
        if (newLevel > level) {
            for (int i = level + 1; i <= newLevel; i++) {
                update[i] = header;
            }
            level = newLevel;
        }

        // 创建新节点
        SkipListNode newNode = new SkipListNode(value, newLevel);

        // 更新每层的指针
        for (int i = 0; i <= newLevel; i++) {
            newNode.forward[i] = update[i].forward[i];
            update[i].forward[i] = newNode;
        }

        System.out.println("Inserted: " + value);
    }

    
}
```

```python
3 []=>^
2 []  =>  [] =>^
1 []=>[]=>[] =>^
0 []=>[]=>[] =>^
  -1   3   6
    
update 记录每一层的插入位置
SkipListNode newNode = new SkipListNode(value, newLevel); 每个节点数组长度为newLevel+1
```



### 639 Redis Hash

键值对集合

ObjKey => {k1:v1, k2:v2}



### 642 Redis数据过期删除策略

- 定时删除 => 定时任务，多少时间检查，删除过期的键 
- 惰性删除 => 访问时，判断，过期了才删



内存回收机制

- 当内存使用达到限制了，主动删除不常用的数据。 LRU算法

### 643 Redis内存淘汰策略

设置了过期时间的数据 淘汰策略

- random
- ttl
- lru  最近最少使用 least recent used
- lfu  最不常用 least frequency used



### 644 Redis  Lua脚本

核心 

- 原子性，避免并发修改带来的安全问题
- 减少网络往返次数
- 复杂操作

Lua本身不具备原子性，但是Redis线程是单线程，因此Lua任务会一同执行，其他任务会被阻塞



### 646 Redis BigKey问题，以及解决

Big Memory Key，key对应的value超级大

- 内存分配不均，在`集群`模式下，如何某个实例的大key过多，负担不均衡
- 客户端超时
- 阻塞其他任务



解决

开发方面

- 开发时，数据**压缩**
- 大化小，**拆成**几份
- 选择合适的数据结构存储

业务方面

- 仅**存储必要信息**

数据分布方面

- 大Key拆分成小key，分散到不同redis实例中



### 647 如何解决redis中的热点key问题

问题：**某个key被频繁访问**，导致redis压力过大

- 热点key拆分：将多个热点数据分散到多个key中，例如通过前缀，使不同请求分散到多个key中，且分布在多实例中，避免集中访问单一key。

  - 全量拷贝，key:datainfo => key:datainfo_1, key:datainfo_2, key:datainfo_3 
  - Key拆分，每个key只存一部分

  例如：热点库存key => stock:product_123,这个key被大量访问

  => 对其进行拆分 4份

  ```java
  stock:product_123:1
  stock:product_123:2
  stock:product_123:3
  stock:product_123:4
      
  # 业务层
  int shardIndex = (productId.hashCode() & Integer.MAX_VALUE) % shardCount;
  String shardKey = "stock:product_123:" + shardIndex;
  ```

  将请求拆分成多个key，分摊压力

  ⭐数据一致性：需要同步所有相关的key

  

- 多级缓存：在Redis前，增加其他缓存层，如(CDN,本地缓存)，分担redis的压力； CDN(**Content Delivery Network** 内容分发网络)；

- 读写分离：通过Redis主从复制，将读请求分发到多个从节点，分摊压力；
- 限流和降级：应用限流策略，减少对redis的请求，必要时回传空值或降级数据。



### 648 Redis的持久化机制

- RDB(redis database)快照
  - 通过生成某一时刻的数据快照来实现持久化，间隔一定时间
  - 二进制文件，数据紧凑，恢复数据快
- AOP(Append only file)日志
  - AOP通过将每个写操作追加到日志中实现持久化，以便根据操作日志进行恢复，重放
  - 恢复精确，但文件体积大
  - 操作记录什么时候写回磁盘
    - 立即，每个操作 =>
    - 间隔, 间隔一定时间=> 写AOF
    - 缓冲区满，再写AOF
- 混合RDB+AOP
  - 利用RDB文件快和AOF的精确
  - 备份的时候，先生成RDB，再将新增的写操作加到AOF文件后面




### 649 Redis在生成RDB快照时，如何处理请求

**写时复制技术**

- RDB快照并不会复制数据，而是复制页表(相当于指针)
- 当Redis处理写请求时，会复制对应的页数据，这样RDB快照和当前数据的页表部分指向将变化了，进行错开. 
- **RDB快照在利用fork子进程存储快照**



### 650 Redis哨兵机制 sentinel ˈsentɪn(ə)l

=> **系统能够在长时间运行中保持较高的`可用性`和`稳定性`**，即使在**出现故障时仍然能快速恢复，确保业务不中断**。

一种高可用性(稳健)解决方案，用于监控Redis主从集群，自动完成主从切换，以及故障自动恢复和通知

- 监控：哨兵会不断的Ping redis主节点和从节点，定时Ping Redis实例，检查存活状态
- 自动故障转移：主节点宕机了，会选择一个从节点晋升为新的主节点，并通知客户端
- 通知，向管理员或其他服务发送通知，以便快速处理redis实例的变化



Redis主节点选举

- 优先级， 通过配置文件， slave_priority

- 优先级相同，则看同步状态offset(数据同步，复制偏移)，数据越一致，越可能成为主节点
- 同步状态一致，则比较ID号

选好master redis后，sentinel会把其他redis实例变为master redis的slave，将新主节点的IP和端口通知给客户端



**主观下线和客观下线**

1） sentinel每个1s Ping所有节点，当超过一段时间没收到对于节点的Pong，主观认为下线。

2）客观下线(主节点而已)：如果主节点宕机了，它向其他哨兵发起投票，只有当下线的投票数大于一半的时候，才认为主节点宕机了。



---

### 653. Redis集群脑裂问题

在网络分区的情况下，可能导致同一个集群出现多个主节点。

分布式系统中，由于**网络分区问题**导致系统多个节点误认为自己是主节点，导致多个主节点提供写入服务，导致数据不一致。

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FoJtPGv_EEQhK52jNYmr48X8KsQ6" alt="image-20250319220151017" style="zoom:40%;" />

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fkn14hzuMQ9at05iEg8_XSrQXZLh" alt="image-20250319220208573" style="zoom:40%;" />

🏷️避免脑裂问题：

- 【`min-slaves-to-write`】设置主节点至少有指定的从节点的情况下才执行写操作。
- 【`min-slaves-max-lag`】设置从节点的最大延迟，如果从节点的延迟超过这个值，则不计入`min-slaves-to-write`。

这样当脑裂后，某个主节点的从节点数量不够或者延迟较大，就无法写入，避免多个主节点写入造成的数据不一致。 【并不能完全解决】



---

### 655 实现分布式锁

set ex nx (set expire_time not-exists) 命令 + lua脚本实现

- 加锁：SET lock_key uniqueValue EX expire_time NX
- 解锁如下

```lua
if redis.call("GET",KEYS[1]) == ARGV[1]   # 避免别人删了这把锁
then
    return redis.call("DEL",KEYS[1])
else
    return 0
end
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FikBBRYAuFGNcOcNoQmgE4um1X2l" alt="image-20250307223835242" style="zoom: 50%;" />



### 656 分布式锁在未完成业务时，过期了怎么办

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FtTngD_Cgbl4xrWxHwGDcclIA0Gf" alt="image-20250307224018962" style="zoom:50%;" />

=> 逻辑途中，给它续期

**扩展知识** - 看门狗机制

抢到锁之后，启动后台定时任务，定时向redis进行锁的续期。比如每过1/3锁的过期时间，给锁续期

业务完成，再把这个后台定时任务给结束



- 并且分布式锁需要满足谁上锁谁解锁
- 释放锁时,需要1.检查锁是不是自己上的,再释放锁. 两步=> Lua脚本



### 654 Redis 订阅&发布

```java
                     => Subscriber 1
publisher => channel => Subscriber 2
                     => Subscriber 3
```

消息的订阅和推送 - 阻塞式消息拉取

```redis
SUBSCRIBE channel
PUBLISH channel message
UNSUBSCRIBE channel
```



### 658 Redis 实现分布式锁可能遇到的问题有哪些?

分布式锁需要满足的要求

- 锁的互斥: Redis setnx+redis单线程
- 可重入性:一个线程可以重复拿到同一把锁
- 锁的性能: 需要基于内存

会出现问题:

- 锁误解 => 需要确保锁自己自己解锁, 或者过期(线程ID判断下)
- 锁的有效时间 => Watchdog机制,给锁定期续时间
- 单点故障问题 => RedLock =>
  - 需要保证一半以上的节点加锁成功才算拿到这把锁

- 可重入锁

```lua
-- tryLock.lua
-- KEYS[1] 锁的Key    
-- ARGV[1] 当前线程标识
-- ARGV[2] 锁的过期时间
    
local lockValue = redis.call('get', KEYS[1]) 
if lockValue == false then
    // 锁不存在
    redis.call('setex', KEYS[1], ARGV[2], ARGV[2])
    return true
else
    // 锁存在
    local parts = {}
	local index = 0
    for match in (localValue .. ":"):gmatch("(.-):") do
    	parts[index] = match
        index = index + 1
    end
    if parts[0] == ARGV[1] then
    	-- 锁由当前线程所有,重入次数+1
        local count = tonumber(parts[1]) + 1
        redis.call('setex', KEYS[1], ARGV[2], ARGV[1] .. ":" .. count)          
        return true
     end
end
return false
```





### 659 Redis中的缓存击穿、缓存穿透和缓存雪崩

**概念**

- *缓存击穿：* 某个**热点**Key数据在缓存中**失效**，导致大量的请求直接访问数据库。 **瞬间的高并发**，可能导致数据库崩溃；  -- 热点过期\删除
- *缓存穿透：* 指查询一个不存在的数据，缓存中没有存储，直接二次映射到数据库中查询，造成数据库负担；
- *缓存雪崩：* 指**多个缓存数据在同一时间过期**，导致大量请求同时访问数据库，从而造成数据库瞬间负载激增。 -- 批量过期 

**解决**

- 缓存击穿：

  - 1 加互斥锁，保证同一请求只有一个请求来重新构建缓存，其他线程等待。

  - 2 热点数据永不过期

  - 3 双重检查

    ```java
    String data = redisTemplate.opsForValue().get("hot_key")
    if(data == null) { // 数据没缓存
        synchronized(this) { // 加速，仅一个线程取构建缓存
            String data = redisTemplate.opsForValue().get("hot_key") // 这里再次判断，这样其他进入同步块的就不用重新访问DB构建缓存了
            if(data == null){
                data = queryFromDB();
                redisTemplate.opsForValue().set("hot_key", data, 30, TimeUnit.MINUTES);
            }
        }
    }
    ```

    

- 缓存穿透(避免二次路由到数据库，拦截它)：

  - 使用布隆过滤器，过滤掉不存在的请求，避免直接访问数据库; hash(where condition)

    实现// 快速判断一个元素是否在BitMap中，将字符串用多个Hash函数映射到不同的二进制位置，将对齐位置设置为1；当查询的时候，需要所有位置都置1，那么该数据才可能存在。因为不会重置为0；

  - 对查询结果进行缓存，即使不存在的数据，也可以换成个标识(空数据，较短的过期时间)，减少对数据库的请求

- 缓存雪崩(多个过期)
  - 数据预热，提前将热门的数据加载到缓存中，避免高并发出现大量的数据库访问。
  - 采用随机分布的方式设置缓存失效时间，避免多个缓存数据同时过期(批量过期)；***添加随机值，进行偏移***
  - 双缓存策略，将数据同时储存在两层缓存中。
    - 在Redis之前，添加一层本地缓存，减少对Redis的依赖。

​					⭐Caffeine高性能缓存库 LRU淘汰策略

```java
@Service
public class CacheService {
    @Resource
    private Cache<String, String> localCache;

    public void put(String key, String value) {
        localCache.put(key, value);
    }

    public String get(String key) {
        return localCache.getIfPresent(key);
    }
}

@Configuration
@EnableCaching
public class CaffeineCacheConfig {

    @Bean
    public com.github.benmanes.caffeine.cache.Cache<String, String> localCache() {
        return Caffeine.newBuilder()
                .maximumSize(1000)  // 最大缓存条目
                .expireAfterWrite(10, TimeUnit.MINUTES)  // 10 分钟自动过期
                .build();
    }
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .maximumSize(1000));
        return cacheManager;
    }
}

// # ----------- 用法
@Cacheable(value = "users", key = "#id")
public String getUserById(String id) {
    System.out.println("查询数据库：" + id);
    return "User_" + id;
}   
```

本地缓存+Redis （多级缓存）

```java
public String getData(String key) {
    // 1. 先查本地缓存
    String value = localCache.getIfPresent(key);
    if (value != null) return value;

    // 2. 查 Redis
    value = redisTemplate.opsForValue().get(key);
    if (value != null) {
        // 回写本地缓存
        localCache.put(key, value);
        return value;
    }

    // 3. 查询数据库（模拟）
    value = queryFromDB(key);

    // 4. 回写 Redis 和本地缓存
    redisTemplate.opsForValue().set(key, value, 10, TimeUnit.MINUTES);
    localCache.put(key, value);
    return value;
}
```

---



### 660 Redis与MySQL的一致性策略

主要就是网络问题，和请求并发问题造成的不一致性

⭐ 让redis和mysql数据一样⭐

- 先更新Redis，再更新MySQL ❌不推荐

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FmGqn7lzt08zg5oJu4hJH-WsIxYk" alt="image-20250308135639431" style="zoom:66%;" />

- 先更新MySQL，再更新Redis ❌不推荐

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fp0jO35q6BMqKyzIsrb0NXRI8o6P" alt="image-20250308140251136" style="zoom:66%;" />

- 先删除Redis，再更新MySQL，最后写回Redis ❌不推荐

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FmEB4Dj9_pgrOuyHLfHhGqK04wHb" alt="image-20250308140614242" style="zoom:66%;" />

- 先更新MySQL，再删除Redis，等请求重新缓存(惰性) ✔️推荐
- 缓存双删除策略。更新MySQL之前，删除一次Redis；更新完MySQL后，再进行一次延迟删除 ✔️推荐

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FsUaIXIOz9NnJkJ0Lf1fUebBQpHI" alt="image-20250308140849778" style="zoom:67%;" />

数据库没问题，但是缓存有问题，等待一段实践

- 使用Binlog异步更新缓存，监听数据库的binlog变化，通过异步方式更新Redis缓存 ✔️推荐

**\# ==========================**

*GPT版本：*

- 📌方案一： 延迟双删(最终一致性)，核心思路如下：
  - 删除Redis缓存
  - 更新MySQL数据
  - 等待一小会时间
  - 再次删除Redis缓存，防止并发请求写回脏数据

```java
// 1. 先删除 Redis
redisTemplate.delete(key);

// 2. 更新 MySQL
databaseService.updateDataInMySQL(key, newValue);

// 3. 延时再删除一次 Redis（解决并发问题）
new Thread(() -> {
    try {
        TimeUnit.MILLISECONDS.sleep(500);
        redisTemplate.delete(key);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();
```

**✅ 适用场景**

✔ 适用于读多写少的场景（如商品详情页）
 ✔ 不影响数据库写入性能
 ✔ 代码简单，容易实现

**🔴 缺点**

❌ 存在 500ms 的数据不一致窗口
 ❌ 不能保证强一致性



📌方案2：异步消息队列（最终一致性）。核心思路如下：

- 更新MySQL数据
- 发送消息到RabbitMQ
- 消息消费者监听更新，删除Redis缓存

```java
public void updateData(String key, String newValue) {
    // 1. 更新 MySQL
    databaseService.updateDataInMySQL(key, newValue);

    // 2. 发送消息到 MQ，通知消费者删除 Redis
    rabbitTemplate.convertAndSend("cache_update_queue", key);
}

@RabbitListener(queues = "cache_update_queue")
public void processCacheUpdate(String key) {
    redisTemplate.delete(key);
}
```

**✅ 适用场景**

✔ 适用于高并发场景
 ✔ 避免了 Redis 和 MySQL 直接耦合
 ✔ 保证最终一致性

**🔴 缺点**

❌ 需要额外的 MQ 服务（Kafka、RabbitMQ）
 ❌ 增加系统复杂度



📌方案3：监听MySQL Binlog（强一致性）。

✅核心思路如下：

- 监听MySQL Binlog日志
- 实时解析SQL变更
- 删除Redis缓存
- 保证Redis和MySQL强一致性

✅实现方式

- Canal 阿里巴巴开源

```java
public void onMessage(String binlogData) {
    String changedKey = parseBinlog(binlogData);
    redisTemplate.delete(changedKey);
}
```

**✅ 适用场景**

✔ 适用于高一致性要求的业务（如金融、电商支付）
 ✔ 高吞吐，低延迟，实时清理缓存
 ✔ 适用于分布式系统

**🔴 缺点**

❌ 依赖 Canal，运维成本较高
 ❌ 需要解析 Binlog，代码实现复杂



评论：

![image-20250308143904146](http://sthda9dn6.hd-bkt.clouddn.com/FoIDpjjifc5pHMUPM-N05X4v1Skz)



------------------

### 661 Redis String 类型底层实现 ❌

Redis String类型底层实现主要基于 SDS(Simple dynamic string)结构，并结合编码方式进行优化存储

```c
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

```

flags: SDS类型，SDS有5个类型 => sdshdr5, sdshdr8 sdshdr16 sdshdr32 sdshdr64

// simple dynamic string header => sdshdr



---

### 662 如何使用Redis快速实现排行榜

[(userInfo, liked_star), ..., ]

- 利用Sorted Set(跳表实现) 存储分数和成员



### 663 如何利用Redis实现布隆过滤器

利用BitMap: (setBit, getBit)



```java
// 添加元素到布隆过滤器
public void add(String value)
    for(SimpleHash hashFunc : hashFuncs) {
        xxx.setbit(BLOOM_FILTER_KEY, hashFUnc(Value))
    }

// 检查某个元素是否存在于布隆过滤器中
public boolean mightContain(String value) {
    for(SimpleHash hashFunc : hashFuncs) {
        if(xxx.getBit(BLOOM_FILTER_KEY, hashFunc(value)) !=1) {
            return false;
        }
}    
```



布隆过滤器的适用场景

- 爬虫：URL去重

- 黑名单：判断
- 分布式系统：判断数据是否在某个节点上
- 推荐系统：判断用户是否看过某个内容



### 664 如何利用Redis统计大量用户唯一访问量

=> HyperLogLog 

- PFADD key element => ADD HyperLogLog中
- PFCOUNT key =>  return unique number



### 665 Redis中的Geo数据结构

- GEOADD key longitude latitude member
- GEODIST key member1 member2
- GEORADIUS key longitude latitude radius unit;



### 668 Redis 性能瓶颈

扩容内存、读写分离、集群模式、多级缓存(本地、redis、mysql)



### 885 Redis Cluster模式 和Sentinel模式

集群与哨兵模式

- 集群 => 数据切片,每个集群负责一部分数据, 集群内又分为主从模式;

- 哨兵：高可用性，当master宕机，根据规则选择最优的slave并晋升为新的master

1.Cluster**集群**模式:集群模式用于对**数据进行分片**，主要用于解决大数据、高吞吐量的场景。将**数据自动分不到多个Redis实例**上，支持自动故障转移(如果某个实例失效，集群会自动重写配置和平衡，不需要手动进行调整，因为内置了哨兵逻辑)
2.Sentinel**哨兵**模式: 哨兵模式用于保证主从节点的高可用，**读写分离**场景。如果主节点宕机，哨兵会将从节点升为主节点.



### 6305 Redisson分布式锁的原理

Redisson是基于Redis+lua脚本实现的分布式锁, 使用redis的原子操作来确保多线程...， 只能有一个线程能够获取到锁，实现互斥。

主要包括：

- 获取锁：(setNX + 过期时间)
- 自动续期：(Watchdog 机制)
- 可重入性：(线程ID计数器，当值为0表示才真正的删除锁)
- 释放锁：(Lua脚本保证原子性)

```java
// ### get lock
RLock lock = redisson.getLock("lock:key")  
// => SET mylock thread-id NX PX 30s;  thread-id保证可重入    
lock.lock()
    
// ### 自动续期 watchdog
// 如果线程未主动释放锁，redisson则每隔10s续期30s
PEXPIRE lock:key 30s
    
// ### 可重入性
// 第一次加锁
lock.lock();

// 同一线程再次加锁
lock.lock();

// 释放锁
lock.unlock();
lock.unlock(); // 只有 count=0 时才真正释放
```

分布式锁最佳实践

```java
RLock lock = redisson.getLock("myLock");

try {
    // 尝试获取锁，最多等待 5 秒，10 秒后自动释放（防止死锁）
    if (lock.tryLock(5, 10, TimeUnit.SECONDS)) {
        try {
            // 执行业务逻辑
            // ...
        } finally {
            lock.unlock(); // 释放锁
        }
    } else {
        System.out.println("获取锁失败");
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

