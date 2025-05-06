---
date: '2025-03-11T16:45:11+08:00'
draft: false
title: '项目重难点'
tags: []
categories: []
description: '简历项目重难点'
ShowToc: true
TocOpen: true
---



---



### 黑马点评：

#### 登录相关

- 使用 JWT 实现无状态认证，登录成功后生成一个包含用户信息的 Token 返回给前端，由前端每次请求时携带在请求头中。后端通过拦截器统一拦截请求，解析并校验 Token 的合法性。 

<用户信息只包括：userId，username，role等必要信息 => 安全>

- 校验通过后，我们会将解析出来的用户信息保存到 `ThreadLocal` 中，构建一个线程级的用户上下文，在整个请求处理流程中都可以方便地获取当前登录用户，避免层层传参，提升代码整洁度和可维护性。

- ThreadLocal原理：每个线程拥有独立的ThreadLocalMap对象。所有线程使用同一个 `ThreadLocal` 对象作为键，但值互不影响。通过线程级别的变量隔离，避免多线程竞争，实现线程安全。



#### 超卖问题：

需要先判断是否有库存，有则扣减库存并返回成功；没有则返回失败。这包括**两步操作**



#### [1.如何解决卖超问题](https://github.com/qiurunze123/miaosha/blob/master/docs)

```
--在sql加上判断防止数据变为负数 =乐观锁
--redis预减库存减少数据库访问　内存标记(过滤掉无库存的访问)减少redis访问　请求先入队列缓冲，异步下单，增强用户体验

异步下单：
-- 请求先入队缓冲，异步下单，增强用户体验
-- 请求出队，生成订单，减少库存
-- "客户端"定时轮询检查是否秒杀成功 
```



1. ***数据库层面：***
   - ***悲观锁（FOR UPDATE-行锁）*** ❌ 别提，浪费时间
   - ***乐观锁（版本号法 - SQL加上判断防止数据变成负数）***  ⭐⭐⭐
2. ***应用层面：***
   - ***分布式锁（互斥）*** ❌ 别提，浪费时间
   - ***缓存+消息队列***   ⭐⭐⭐
   - ***限流+熔断***

❌❌❌***1.1 行锁*** 

```sql
BEGIN;
SELECT stock FROM products WHERE id = 1 FOR UPDATE;
-- 业务逻辑：检查库存是否足够，执行扣减操作
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;
```

10000条(10线程循环1000次)请求，吞吐量：791.2/sec       互斥锁，串行访问

```java
Integer stock = baseMapper.selectSecKillStockForUpdate(voucherId); // Fordate加行锁，事务结束会释放
if (stock <= 0){
    throw new BusinessException(500, "fail");
}

boolean update = lambdaUpdate().set(SeckillVoucher::getStock, stock - 1)
    .eq(SeckillVoucher::getVoucherId, voucherId)
    .update();
```

多事务的并发逻辑

```sql
-- 事务1（加X锁）
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- 持有X锁

-- 事务2（普通SELECT）
BEGIN;
SELECT * FROM accounts WHERE id = 1; -- ✅ 正常读取（MVCC快照）
                                  -- ❌ 如果事务2也执行 FOR UPDATE 则被阻塞

-- 事务3（尝试修改）
UPDATE accounts SET balance = 100 WHERE id = 1; -- ❌ 被阻塞（X锁互斥）
```



⭐⭐⭐***1.2 CAS方法***

SQL加上防止为负数

```sql
-- （X锁互斥）
UPDATE products 
SET stock = stock - 1
WHERE id = 1 AND stock > 0;  
```

10000条(10线程循环1000次)请求，吞吐量：944.7/sec

```java
// 注意把Stock不满足条件的过滤！

boolean update = lambdaUpdate()
                .setSql("stock = stock - 1") // 这个很关键 不要使用 set(::getStock, stock-1) 错误原因：基于应用层计算的值，而不是数据库的当前值。
                .eq(SeckillVoucher::getVoucherId, voucherId)
                .gt(SeckillVoucher::getStock, 0) // eq(::getStock, stock) 
                .update();
```

⭐⭐⭐ MVCC(版本链+ReadView)  - 读操作而言

- **事务 B 的更新操作**：
  - 更新操作会直接操作数据库的当前值，而不是基于事务 B 的 ReadView。
  - 事务 B 的更新操作会检查数据库的当前值，发现 `stock = 0`，因此更新失败。



🎉🎉🎉讲库存加载到Redis中，在Redis中进行扣减库存

```java
// seckill_key = "seckill:product:1"
// =Lua脚本(判断+扣减)=> 再处理MySQL   // 原子化查询+修改 
```

然后，也不需要扣减MySQL中的库存，这样减少锁竞争，修改为直接插入订单(根据订单和Redis数据，进行核对)。

为了进一步加速响应，引入MQ，解耦下单的库存校验和写入MySQL操作。



***2.3 🏷️缓存+消息队列 （服务解耦）:***

```java
// 库存初始化：将MySQL商品库存存入Redis中，提升查询和扣减效率;

// 库存预扣减(Redis)：用户请求时，先在 Redis 中扣减库存;  == Lua脚本
Long ok = redisTemplate.execute(SECKILL_SCRIPT, Collections.emptyList(), "1");
rabbitTemplate.convertAndSend(exchangerName, routingKey, message);

// 订单异步处理(MQ):扣减成功 => MQ => 订单处理接口 => 插入新订单。
```

10000条(10线程循环1000次)请求，吞吐量：1485.7/sec



🏷️***限流+熔断***: （仅理论）

```java
// 1. Nginx限流(最基础，拦截恶意流量)  - 只针对IP
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate 100r/s;
    server {
        location /seckill {
            limit_req zone=mylimit burst=50 nodelay;
            proxy_pass http://backend;
        }
    }
}

// 2. Redis令牌桶(应用层用户级限流，精准控制)
// def try_acquire_token(user_id, rate=5, capacity=10):
bucket_key = f"token_bucket:{user_id}"
last_time_key = f"last_time:{user_id}"

// 下面需要为lua脚本  ====
now = time.time()
last_time = float(r.get(last_time_key) or now)
elapsed = now - last_time    
    
// 计算补充的令牌
tokens_to_add = int(elapsed * rate)
current_tokens = min(capacity, int(r.get(bucket_key) or capacity) + tokens_to_add)
    
if current_tokens > 0:
    r.set(bucket_key, current_tokens - 1)  # 消耗 1 个令牌
    r.set(last_time_key, now)  # 更新时间
    return True
return False    
    
//  ====
        
        
// 3. 前端排队(减少无效请求，避免一窝蜂涌入)
```





#### 一人一单问题： 

***（首先不能重复下单，还要扣减库存，并且新增订单）***

- ***单体架构* **
  - ***synchronized加锁***   +  插入时的唯一索引
- ***集群架构***
  - *** Redis-Lua脚本原子化查询库存+重复下单***
  



***1.1 🏷️Redis预扣减+检查：*** 使用set结构存储购买过的用户，避免重复下单

```lua
-- 检查库存
-- 检查重复下单
if (redis.call('exists', orderKey) == 0) then
    redis.call('sadd', orderKey, '')  -- ！！！初始化需要从MySQL中加载已经买过的用户
    return 2
end
if (redis.call('sismember', orderKey, userId) == 1) then
    return 2
end

redis.call('sadd', orderKey, userId)
-- 在Redis中，使用Set集合存储买过的用户
```



***1.2 🏷️JVM锁+MySQL唯一索引实现：***

唯一索引减少查询是否重复下单的时间，直接融入到插入操作成功or失败中

```sql
ALTER TABLE orders ADD UNIQUE INDEX idx_uid_item (user_id, item_id);
```

实际场景中，我们需要的是 **"同一用户对同一商品只能购买一次"**，而非单纯限制用户或商品ID的唯一性。 唯一索引为商品ID+用户ID

```java
@Transactional
public Result createOrder(Long userId, Long itemId) {
    boolean stockReduced = false; // 标记是否已扣减库存
    try {
        // 1. 检查库存（加锁）
        Item item = itemMapper.selectForUpdate(itemId);
        if (item.getStock() <= 0) {
            return Result.fail("库存不足");
        }

        // 2. 扣减库存（捕获可能异常）
        int affectedRows = itemMapper.reduceStock(itemId);
        if (affectedRows == 0) { // 扣减失败（如库存不足）
            throw new BusinessException("库存不足");
        }
        stockReduced = true; // 标记已扣减

        // 3. 创建订单
        orderMapper.insert(new Order(userId, itemId));
        return Result.success();

    } catch (DuplicateKeyException e) {
        // 仅当库存已扣减时才回滚
        if (stockReduced) {
            itemMapper.recoverStock(itemId);
        }
        throw new BusinessException("请勿重复购买");

    } catch (Exception e) {
        // 仅当库存已扣减时才回滚
        if (stockReduced) {
            itemMapper.recoverStock(itemId);
        }
        throw new BusinessException("下单失败，请重试");
    }
}
```



***2.1 🏷️分布式锁现实***

解决单机JVM锁，在集群下失效的问题。

```lua
-- 上锁，分布式的
不存在:创建并返回True
```

业务 => 扣库存 + 创建订单

```lua
-- 解锁，分布式，只能解锁自己上的锁
// 如果可以释放别人的锁，会出现以下情况
// 事务A的锁过期释放了(这里没有续时间or暂时卡了) => 事务B加锁 => 事务A活了过来*解锁* => 事务C加锁 ... 
事务B和C拿了同一把锁，寄了  => 只允许删自己加的锁
事务A和B拿了同一把锁，寄了  => 看门狗续时间
```



***lua脚本***

```java
Boolean lock = redisTemplate.opsForValue().setIfAbsent(lock_key, lock_value, 3, TimeUnit.SECONDS);

try {
    if (lock) {
        ...
    }
} finally {
	redisTemplate.execute(SECKILL_UNLOCK_SCRIPT, Collections.singletonList(lock_key), lock_value); // 删除锁和判断锁是否当前线程拥有 原子化
}    
```

***Redission***

```java
RLock lock = redissonClient.getLock(lock_key);
try {
    boolean ok = lock.tryLock(1, 3, TimeUnit.SECONDS);  // 不阻塞
    if(ok) {...}
    finally {
        if (lock.isHeldByCurrentThread())  lock.unlock();
    }
}

try {
    boolean ok = lock.lock();  // 阻塞,且未指定过期时间 => 触发看门狗
    if(ok) {...}
    finally {
        if (lock.isHeldByCurrentThread())  lock.unlock();
    }
}
```



超卖问题:

- 一开始我们是使用synchronzied在扣减库存的代码块中加锁，进行互斥，讲查询MySQL库存和扣减库存原子化。但是这样并发的性能不高。然后我们调整为CAS乐观锁的形式，我们不加锁，修改sql语句，给where条件中添加上库存大于0的条件，当库存不足时，update操作(**会加行锁**)就执行失败，表示扣减失败

- 之后，为了提高并发效率，我们使用redis，预扣减库存提高秒杀商品的并发，我们先讲秒杀商品的库存缓存到redis中，在redis中进行 库存的检查和扣减(原子化)，因为是基于内存的，效率会比mysql快的多。

一人一单问题：

- 在redis预扣减缓存中进行了修改，我们使用redis中的set集合，存储购买过的用户Id，判断用户是否重复购买。 因为redis是单线程的，因此我们使用lua脚本，讲这一个操作，打包成一个脚本文件，给redis执行，保证操作的原子性。
- 然后我们发现当库存不足的时候，用户会一直点击下单，给redis带来了不必要的查询，因此我们在服务中引入一个布尔变量，标识是否有库存，当没有时，直接返回，降低redis的请求量。
- 另一种不使用redis的解决方法： 





后面我们在**单体系统**下遇到了**一人多单超卖**问题，我们通过**乐观锁(update sql添加库存不能为负)**解决；我们对业务进行了变更，将一人多单变成了**一人一单**，结果在**高并发**场景下同一用户发送相同请求仍然出现了超卖问题，我们通过**悲观锁**解决了；由于用户量的激增，我们将单体系统升级成了**集群**，结果由于**锁只能在一个JVM中可见**导致又出现了，在高并发场景下同一用户发送下单请求出现超卖问题，我们通过实现**分布式锁**成功解决集群下的**超卖**问题；释放锁时，**判断锁是否是当前线程 和 删除锁两个操作**不是原子性的，可能导致超卖问题，我们通过将两个操作封装到一个**Lua脚本**成功解决了；

为了解决锁的不可重入性，我们通过将锁以hash结构的形式存储，锁的值为线程ID拼接锁重入次数，每次释放锁都value-1，获取锁value+1，从而实现锁的可重入性，并且将释放锁和获取锁的操作封装到Lua脚本中以确保原子性。

最最后，我们发现可以直接使用现有比较成熟的方案Redisson来解决上诉出现的所有问题🤣，什么不可重试、不可重入、超市释放、原子性等问题Redisson都提供相对应的解决方法（。＾▽＾）





#### 优惠卷秒杀

异步下单：

```java
// 主线程
1. Lua脚本
2. 成功=>放入队列；失败=>返回异常    
    
1.1. Lua脚本 => 库存 =扣库存=> 是否下单 =记录用户下单 => 返回0
               返回1            返回2

// 子线程
队列中取出订单信息 => 插入订单 => 是否付款 => 更新成功
                            => End
```

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250316232011370.png" alt="image-20250316232011370" style="zoom:50%;" />





#### 缓存三兄弟：

💡***缓存穿透***： 恶意请求查询不存在的数据，绕过缓存直接访问数据库

![image-20241121195359815](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20241121195359815.png)

- ***布隆过滤器***

1. 商品ID，通过多个哈希函数，将元素映射到bitMap中的多个位置，只有每个位置都为1才表明这个对象存在。
2. guava库有实现好的，直接用。

```python
# 伪代码，仅供学习
class BloomFilter:
    def __init__(self, size, hash_count):
        """
        初始化布隆过滤器
        :param size: 位数组大小（m）
        :param hash_count: 哈希函数数量（k）
        """
        self.size = size
        self.hash_count = hash_count
        self.bit_array = [0] * size  # 初始化位数组（全0）

    def add(self, item):
        """
        添加元素到布隆过滤器
        """
        for seed in range(self.hash_count):
            # 通过不同的种子模拟多个哈希函数
            index = self._hash(item, seed) % self.size
            self.bit_array[index] = 1  # 设置对应位为1

    def contains(self, item):
        """
        检查元素是否可能存在
        :return: 
            - True: 可能存在（可能有误判）
            - False: 一定不存在
        """
        for seed in range(self.hash_count):
            index = self._hash(item, seed) % self.size
            if self.bit_array[index] == 0:
                return False  # 只要有一位为0，则一定不存在
        return True  # 所有位均为1，可能存在

    def _hash(self, item, seed):
        """
        哈希函数（伪代码，实际可用MurmurHash等）
        :param item: 输入元素
        :param seed: 哈希种子（模拟不同哈希函数）
        :return: 哈希值
        """
        hash_value = 0
        for char in str(item):
            hash_value = (hash_value * seed + ord(char)) & 0xFFFFFFFF
        return hash_value
```



- ***缓存空对象***

*🏷️ 缓存空对象实现方案*

![image-20241121195847292](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20241121195847292.png)

```java
// 缓存命中
if (StrUtil.isNotBlank(s)) { // 不为null，不为""
    SeckillVoucher seckillVoucher = JSONUtil.toBean(s, SeckillVoucher.class);
    return Result.ok(seckillVoucher);
}

// 未命中，是否是之前缓存的空对象
if (Objects.nonNull(s)) { // "" 不为null
    throw new BusinessException(500, "查询缓存空对象！");
}

SeckillVoucher seckillVoucher = lambdaQuery()
    .eq(SeckillVoucher::getVoucherId, id)
    .one();
// 缓存空对象 一个线程重建缓存就好
synchronized (CacheTestService.class) {
    String t = redisTemplate.opsForValue().get(key);
    if (Objects.nonNull(t))
        throw new BusinessException(500, "查询缓存空对象！");

    if (Objects.isNull(seckillVoucher)) {
        redisTemplate.opsForValue().set(key, "", 3, TimeUnit.SECONDS);
        throw new BusinessException(500, "缓存空对象！");
    }
}
// 缓存重建
redisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(seckillVoucher), 3, TimeUnit.SECONDS);

return Result.ok(seckillVoucher);
```



💡***缓存雪崩***：量缓存同时失效，导致请求全部压到数据库

![image-20241121201618172](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20241121201618172.png)

- 过期时间随机化 ⭐⭐⭐ baseline + random_offset
- Hot数据**定时**刷新缓存  @Scheduled
- 多级缓存(本地-Redis-数据库)



💡***缓存击穿***：热点 Key 突然失效，大量并发请求直接打到数据库

- 互斥重建：第一个重建(MySQL=>Redis)，后面的等一下再访问缓存
- 逻辑过期：返回过期数据，但是互斥(第一个访问的)重建缓存  ❗数据一致性不高

![image-20241121210053886](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20241121210053886.png)

```java
// 缓存击穿(热点Key，瞬时重建问题)
public Shop queryWithMutex(Long id) {
    // 1. 先Redis
    String key = "cache:shop:" + id;
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    // Cache命中  shopJson 不为null,"", "\t"(不全是空白字符)
    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return shop;
    }
    // 空对象
    if (shopJson != null) {   // 为""
        return null;
    }

    // 1. 实现缓存重建
    String lockKey = "lock:shop:" + id;
    Shop shop = null;
    try {
        // 1.1 获取互斥锁
        boolean isLock = tryLock(lockKey);
        // 1.2 判断是否获取成功
        if (!isLock) {
            // 1.3 失败则休眠且轮询
            Thread.sleep(50);
            return queryWithMutex(id); ⭐⭐⭐
        }
        // 未命中 => MySQL 缓存重建
        shop = lambdaQuery().eq(Shop::getId, id).one();
        Thread.sleep(500); // 模拟复杂的重建任务
        if (shop == null) {
            // 将空对象缓存
            stringRedisTemplate.opsForValue().set(key, "", RedisConstants.CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
        // log.debug(shop.toString());
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        unlock(lockKey);
    }
    return shop;
}

public Shop queryWithLogicExpire(Long id) {
    // 1. 先Redis
    String key = "cache:shop:" + id;
    // log.debug("Key:" + key);
    String shopJson = stringRedisTemplate.opsForValue().get(key);

    if (StrUtil.isBlank(shopJson)) {
        // 缓存不存在，查询数据库并回填
        return saveShop2Redis(id, 20L);
    }
    // RedisData:  private LocalDateTime expireTime;
    //             private Object data;
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
    Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);

    if (redisData.getExpireTime().isAfter(LocalDateTime.now())) {
        // 未过期
        return shop;
    }
    String lockKey = "lock:shop:" + id;
    // 过期进行更新
    boolean isLock = tryLock(lockKey);
    if (isLock) {
        // 独立线程进行重建缓存
        CACHE_REBUILD_EXECUTOR.submit(() -> {
            try {
                saveShop2Redis(id, 20L);
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                unlock(lockKey);
            }
        });
    }
    return shop;  // 可能会返回过期数据。 因为没有进行等待
}
```



#### Redis&MySQL数据一致性

- 先更新Redis，再更新MySQL ❌不推荐

![image-20250308135639431](http://verification.longcoding.top/FmGqn7lzt08zg5oJu4hJH-WsIxYk)

- 先更新MySQL，再更新Redis ❌不推荐

![image-20250308140251136](http://verification.longcoding.top/Fp0jO35q6BMqKyzIsrb0NXRI8o6P)

- 先删除Redis，再更新MySQL，最后写回Redis ❌不推荐

![image-20250308140614242](http://verification.longcoding.top/FmEB4Dj9_pgrOuyHLfHhGqK04wHb)

- 先更新MySQL，再删除Redis，等请求重新缓存(惰性) ✔️推荐
- 缓存双删除策略。更新MySQL之前，删除一次Redis；更新完MySQL后，再进行一次延迟删除 ✔️推荐

![image-20250308140849778](http://verification.longcoding.top/FsUaIXIOz9NnJkJ0Lf1fUebBQpHI)

数据库没问题，但是缓存有问题，等待一段实践

- 使用Binlog异步更新缓存，监听数据库的binlog变化，通过异步方式更新Redis缓存 ✔️推荐



#### **WebSocket**

```java
@Component
@ServerEndpoint("/websocket")
public class MyWebSocketEndpoint {

    // 会话对象  server:client == 1:n
    private static Map<String, Session> sessionMap = new HashMap<>();


    @OnOpen
    public void onOpen(Session session) {
        System.out.println("onOpen: " + session.getId());
        sessionMap.put(session.getId(), session);
        sendMsg("Hi, can I help you?", session.getId());
    }

    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("receive Message: " + message);
        sendMsg("Yse! I am thinking ... this question!", session.getId());
    }

    @OnClose
    public void OnClose(Session session) {
        sendMsg("Bye!", session.getId());
        System.out.println("OnClose: " + session.getId());
        sessionMap.remove(session.getId());
    }

    public void sendMsg(String msg, String sid) {
        try {
            sessionMap.get(sid).getBasicRemote().sendText(msg);  // 向session对象sendTest
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

前端：

```js
var ws = new WebSocket("ws://localhost:8080/chat");
// 初始化连接
ws.onopen = function (e) {
   
}
//接受消息
ws.onmessage = function (ev) {
    
}
// 关闭连接
ws.onclose = function (ev) {
    
}
```





### 亮点模块：

#### RabbitMQ

- ##### 异步处理

- ***削峰填谷（流量削峰）***

  => **削峰**（缓冲流量）：流量高峰时，MQ 充当“缓冲区”，将请求先存储到消息队列中，避免系统直接崩溃。

  ​     **填谷**（平滑流量）：消费者按**稳定速率**消费消息，保证系统负载均衡。

  - 消息可靠投递
    - 发布确认机制 - 是否重发
    - RabbitMQ持久化
  - 消息可靠消费
    - 手动ACK
    - 限流机制 - 一次只取一个消息
    - 死信队列（❗兜底）
  - 消息唯一消费
    - Redis存储消费过的ID (多个消费者接收到相同的消息时候)  - **发布/订阅（Fanout 交换机-广播）**，消息会被**复制到多个队列。**
    - ⭐RabbitMQ 中，**默认情况下，消息只会被一个消费者消费**，**队列是点对点模型**，一条消息只能被消费一次。**多个消费者订阅同一个队列时，RabbitMQ 采用** **“轮询分发”**（Round Robin），每条消息只会被其中一个消费者处理。

```java
				 RabbitMQ
publisher => [exchanger => queue] => consumer
error     1.     4.      2.   5.  3.    
```

1.2. 投递失败 => 重新投递

4.5. MQ宕机 => 持久化

3.. ACK确认 => 重新消费

```yaml
rebbitmq:
	# ...
    publisher-confirm-type: correlated  # 开启发布确认机制
    publisher-returns: true   # 投递消息抵达队列
    listener:
        simple:
        acknowledge-mode: manual  # 消费者手动确认ACK
            prefetch: 1   # 限制消费者一次处理一条数据
```

***生产者：***

```java
// 确保交换机收到消息
rabbitTemplate.setConfirmCallback(((correlationData, ack, cause) -> {
    if (ack) {
        log.debug("[√] Product: Msg send success!");
    }else {
        log.error("[×] Product: Msg send fail!, 原因: {}", cause.toUpperCase());
        msgRetryCount.putIfAbsent(correlationData.getId(), 3);
        retrySendMsg(correlationData);
    }
}));

// 确保队列收到消息
rabbitTemplate.setReturnsCallback(returnedMessage -> {
    log.error("[×] 消息路由失败!, 原因: {}, Msg: {}", returnedMessage.getReplyText(), returnedMessage.getMessage());
    // 这里Roting Key错误，配置错误，投递死信队列比较好
});
}

private void retrySendMsg(CorrelationData correlationData) {
    Integer count = msgRetryCount.get(correlationData.getId());
    if (count > 0) {
        String message = msgCacheMap.get(correlationData.getId());
        rabbitTemplate.convertAndSend(ReliabilityMQConfig.EXCHANGE_NAME, ReliabilityMQConfig.ROUTING_KEY, message, correlationData);
        msgRetryCount.computeIfPresent(correlationData.getId(), (s, integer) -> integer-1);
    }else {
        log.error("发送到死信交换机！");
    }
}

// ⭐⭐⭐模拟错误情况 1. 2.
public void sendMessage(String message) {
    log.debug("[>] Send Msg: " + message);
    for (int i = 1; i <= 3; i++) {
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        msgCacheMap.put(correlationData.getId(), message + ":" + i); // 缓存消息
        if (i == 2) {
            rabbitTemplate.convertAndSend(ReliabilityMQConfig.EXCHANGE_NAME + "error", ReliabilityMQConfig.ROUTING_KEY, message + ":" + i, correlationData);
        }else if (i == 3) {
            rabbitTemplate.convertAndSend(ReliabilityMQConfig.EXCHANGE_NAME, ReliabilityMQConfig.ROUTING_KEY + "error", message + ":" + i, correlationData);
        }else {
            rabbitTemplate.convertAndSend(ReliabilityMQConfig.EXCHANGE_NAME, ReliabilityMQConfig.ROUTING_KEY, message, correlationData);
        }
    }
}
```

***消费者：***

```java
@RabbitListener(queues = ReliabilityMQConfig.QUEUE_NAME)
public void receiveMessage(String message, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag, Channel channel) throws InterruptedException {
    try {
        log.debug("[×] 开始处理消息: " + message);
        Thread.sleep(2000);
        double p = RandomUtil.randomDouble(0, 1);
        if (p >= 0.5) {
            throw new IOException("手动造成异常");
        }
        // 1. process success
        channel.basicAck(deliveryTag, false);  // deliveryTag:msgId; multiple=false:ackCurrentMsg else ackPreAllMsg
        log.debug("[√] process Msg success: {}", message);
    } catch (IOException e) {
        // 2. process fail
        log.debug("[×] process Msg fail: " + message);
        try {
            // 退回消息
            channel.basicNack(deliveryTag, false, true);  // msgId, 是否批量拒绝:false:仅拒绝当前Msg, true:拒绝所有deliveryTag小于等于当前的deliveryTag消息  ⭐⭐⭐ 消费失败时重新入队，确保消息不丢失。
        } catch (IOException ioException) {
            log.debug("[<] Msg notAck fail");
        }
    }
}
```

- ***微服务解耦***
- ***订单超时取消***



### 拼团交易

#### 抽象责任链模板

定义抽象任务接口

```java
public interface StrategyHandler<T, D, R> {

    StrategyHandler DEFAULT = (T, D) -> null;

    R apply(T requestParameter, D dynamicContext) throws Exception;
}

public interface StrategyMapper<T, D, R> {

    StrategyHandler<T, D, R> get(T requestParameter, D dynamicContext) throws Exception;

}
```

定义抽象节点接口

```java
public abstract class AbstractStrategyRouter<T, D, R> implements StrategyMapper<T, D, R>, StrategyHandler<T, D, R> {

    protected StrategyHandler<T, D, R> defaultHandler = StrategyHandler.DEFAULT;

    public R router(T requestParameter, D dynamicContext) throws Exception {
        StrategyHandler<T, D, R> handler = get(requestParameter, dynamicContext);
        if (handler != null) return handler.apply(requestParameter, dynamicContext);
        return defaultHandler.apply(requestParameter, dynamicContext);
    }

}
```

大致就是 [apply + next]，责任链模式。

```java
// 基本使用
class XXX {
    ObjectXXX nextNode;
    
    apply{... => next()}
    
    next{nextNode.apply()}
}
```



---

**多实例链路模式**

双向链表的实现思想

定义链表：

```Java
public class Node<E> {
    E item;

    Node<E> next;
    Node<E> prev;

    public Node(Node<E> prev, E item, Node<E> next) {
        this.item = item;
        this.next = next;
        this.prev = prev;
    }
}
```

定义节点处理对象

```Java
public interface ILogicHandler<T, D, R> {

    // 子类可以不实现
    default R next(T requestParameter, D dynamicContext) {
        return null;
    }

    R apply(T requestParameter, D dynamicContext) throws Exception;

}
```

实现具体的业务节点

```java
public class RuleLogic201 implements ILogicHandler<String, Rule02TradeRuleFactory.DynamicContext, String> {

    public String apply(String requestParameter, Rule02TradeRuleFactory.DynamicContext dynamicContext) throws Exception{

        // 具体业务逻辑;

        return next(requestParameter, dynamicContext);
    }

}
```

装配业务责任链

```java
public class LinkArmory<T, D, R> {
    private final BusinessLinkedList<T, D, R> logicLink;


    @SafeVarargs
    public LinkArmory(String linkName, ILogicHandler<T, D, R>... logicHandlers) {
        logicLink = new BusinessLinkedList<>(linkName);
        for (ILogicHandler<T, D, R> logicHandler: logicHandlers){
            logicLink.add(logicHandler);
        }
    }
    public BusinessLinkedList<T, D, R> getLogicLink() {
        return logicLink;
    }
}
```



#### 责任链实现业务管道

商品折扣价格试算

```java
// 从工厂拿到对应的责任链出发节点
=> RootNode 
	=> multiThread[前置处理|检查信息] => doApply[业务+路由] 
=> SwitchNode 
	=> multiThread[前置处理] => doApply[业务+路由] 
=> TagNode
    => doApply[业务+路由]
=> MarketNode 
	=> multiThread[⭐前置处理⭐拼单试算] => doApply[业务+路由] 
=> EndNode 
	=> multiThread[前置处理] => doApply[业务|构建结果]    
    
// SwitchNode：根据条件判断业务是否降级or限流
// TagNode：根据用户信息判断拼团活动能否参与与可见
// MarketNode：根据拼团活动，计算折扣价格
// 	      策略模式：根据优惠Id,计算不同的折扣
```

拼团锁单校验

```java
// 实现具体的业务节点

// 1. ActivityUsabilityRuleFilter：校验活动状态和活动时间的有效性

// 2. UserTakeLimitRuleFilter：判断用户参与次数与活动限制次数关系

// 装配业务责任链
```

拼团结算校验

```java
// 1. SCRuleFilter: 校验拼团的渠道来源是否合法，是否在黑名单中
// 2. SettableRuleFilter：拼团活动的用户支付时间是否在有效时间内
// 3. OutTradeNoRuleFilter：外部交易单号是否正确
// 4. EndRuleFilter：打包结果
```



#### AOP+消息订阅实现热更新

通过AOP编程和Redis发布/订阅功能实现业务规则（限流阈值、降级策略）的热更新

自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
public @interface DCCValue {
    String value() default "";
}
```

定义热更新服务

```java
@Service
public class DCCService {

    /**
     * 降级开关 0关闭 1开启
     */
    @DCCValue("downgradeSwitch:0")
    private String downgradeSwitch;

    @DCCValue("cutRange:100")
    private String cutRange;

    public boolean isCutRange(String userId) {
        // 计算哈希码的绝对值
        int hashCode = Math.abs(userId.hashCode());

        // 获取最后两位
        int lastTwoDigits = hashCode % 100;

        // 判断是否在切量范围内
        if (lastTwoDigits <= Integer.parseInt(cutRange)) {
            log.error(cutRange);
            return true;
        }

        return false;
    }

    public boolean isDowngradeSwitch() {
        return this.downgradeSwitch.equals("1");
    }
}
```

⭐注意：在SwitchNode中，调用这些方法进行判断是否放行

基于Redis的发布订阅进行参数调整

```java
/ DCCValueBeanFactory implements BeanPostProcessor 
private static final String BASE_CONFIG_PATH = "group_buy_market_dcc_";

private final RedissonClient redissonClient;

private final Map<String, Object> dccObjGroup = new HashMap<>();

public DCCValueBeanFactory(RedissonClient redissonClient) {
    this.redissonClient = redissonClient;
}

@Bean("dccTopic")
public RTopic dccRedisTopicListener(RedissonClient redissonClient) {
    log.info("初始化Redis Topic监听事件 测试连通{}", redissonClient.getBucket("test").get());
    // 发布订阅模式的Topic实现
    RTopic topic = redissonClient.getTopic("group_buy_market_dcc");
    // 消息类型，回调接口：Topic名称，接收的消息
    topic.addListener(String.class, ((charSequence, s) -> {
        log.info("监听到Redis Topic： {}", charSequence.toString());
        String[] split = s.split(Constants.SPLIT);

        // 获取值
        String attribute = split[0];
        String key = BASE_CONFIG_PATH + attribute;
        String value = split[1];

        // 设置值 => 可以存储任意类型的值（如 String、Integer、自定义对象等），Redisson 会自动进行序列化和反序列化。
        RBucket<String> bucket = redissonClient.getBucket(key);
        boolean exists = bucket.isExists();
        if (!exists) return;
        bucket.set(value);

        Object objBean = dccObjGroup.get(key); // 获取内存中的对象
        if (objBean == null) return;

        Class<?> objBeanClass = objBean.getClass();
        // 检查objBean是否为代理对象
        if (AopUtils.isAopProxy(objBean)) {
            objBeanClass = AopUtils.getTargetClass(objBean);
        }

        try {
            Field field = objBeanClass.getDeclaredField(attribute);
            field.setAccessible(true);
            field.set(objBean, value);
            field.setAccessible(false);

            log.info("DCC 节点监听，动态设置值 {} {}", key,value);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }));
    return topic;
}


// implements BeanPostProcessor 重写方法  -- 并在每个 bean 初始化完成后自动调用该方法。
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    // 注意：增加 AOP 代理后，获得类的方式要通过 AopProxyUtils.getTargetClass(bean); 不能直接 bean.class 因为代理后类的结构发生了变化，这样不能获得自己的自定义注解了
    Class<?> targetBeanClass = bean.getClass();
    Object targetBeanObject = bean;
    if (AopUtils.isAopProxy(targetBeanObject)) {
        targetBeanClass = AopUtils.getTargetClass(targetBeanObject);
        targetBeanObject = AopProxyUtils.getSingletonTarget(bean);
    }
	
    // 这里判断该类是否有对应的自定义注解
    Field[] fields = targetBeanClass.getDeclaredFields();
    for (Field field : fields) {
        if (!field.isAnnotationPresent(DCCValue.class)) {
            continue;
        }

        DCCValue dccValue = field.getAnnotation(DCCValue.class);
        String value = dccValue.value();
        if (StringUtils.isBlank(value)) {
            throw new RuntimeException(field.getName() + "@DCCValue is not config value config case「isSwitch/isSwitch:1」");
        }

        String[] split = value.split(":");
        String key = BASE_CONFIG_PATH.concat(split[0]);
        String defaultValue = split[1];

        String setValue = defaultValue;

        try {
            if (StringUtils.isBlank(defaultValue)) {
                throw new RuntimeException("dcc config error " + key + " is not null - 请配置默认值！");
            }

            RBucket<String> bucket = redissonClient.getBucket(key);
            boolean exists = bucket.isExists();
            if (!exists) {
                bucket.set(defaultValue);
            } else {
                setValue = bucket.get();
            }

            field.setAccessible(true);
            field.set(targetBeanObject, setValue);
            field.setAccessible(false);
            log.info("初始化值 key:{} value:{}", key, setValue);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        }

        dccObjGroup.put(key, targetBeanObject);
    }
    return bean;
}
```

实现Api接口更新Redis中的值

```java
@Override
@GetMapping("update_config")
public Response<Boolean> updateConfig(@RequestParam String key, @RequestParam String value) {
    try {
        log.info("DCC 动态配置值变更 key:{} value:{}", key, value);
        dccTopic.publish(key + "," + value);
        return Response.<Boolean>builder()
            .code(ResponseCode.SUCCESS.getCode())
            .info(ResponseCode.SUCCESS.getInfo())
            .build();
    } catch (Exception e) {
        log.error("DCC 动态配置值变更失败！ key:{} value:{} {}", key, value, e.getMessage());
        return Response.<Boolean>builder()
            .code(ResponseCode.UN_ERROR.getCode())
            .info(ResponseCode.UN_ERROR.getInfo())
            .build();
    }
}
```

更新请求 => Redis => 订阅服务 => 修改参数



---

