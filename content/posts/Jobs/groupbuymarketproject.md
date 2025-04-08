---
date: '2025-03-23T19:36:21+08:00'
draft: false
title: 'Grouptradingproject'
tags: ["Projects"]
categories: ["Projects"]
description: 'Grouptradingproject'
ShowToc: true
TocOpen: true
---



# 拼团交易平台系统

**项目背景：**为了盘活沉睡用户，需要适当降低商品价格。但为了达到传播的效果，所以需要引入拼团方式，以客带客，靠用户自身传播的方式进行交易拉新。这样的处理方式对比于 KOL，会让利商品价值到用户自身。



### 第1-2节 拼团库表设计

***业务流程：***

![image-20250323194301660](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323194301660.png)

- *运营角度：* 1. 给哪些商品配置拼单；2. 拼团商品提供的规则信息：折扣、时间、人数等。
- *用户角度：* 1. 参与拼团，首次发起or参与现有，拼单完成回调通知。
- *库表设计：* 1. 人群设计，将所有符合某个条件的用户ID，全部写入特定Redis记录中。
- 拼团活动，折扣的多种迭代。

**库表设计：**

*拼团配置表*：

![image-20250323194814202](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323194814202.png)

- 拼团活动表：设定了拼团的成团规则，人群标签的使用可以限定哪些人可见，哪些人可参与。
- 折扣配置表：拆分出拼团优惠到一个新的表进行多条配置。如果折扣还有更多的复杂规则，则可以配置新的折扣规则表进行处理。
- 人群标签表：专门来做人群设计记录的，这3张表就是为了把符合规则的人群ID，也就是用户ID，全部跑任务到一个记录下进行使用。 比如黑玫瑰人群、高净值人群、拼团履约率90%以上的人群等。



***参与拼团表：***

![image-20250323195108167](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323195108167.png)

- 拼团账户表：记录用户的拼团参与数据，一个是为了限制用户的参与拼团次数，另外是为了人群标签任务统计数据。
- 用户拼单表：当有用户发起首次拼单的时候，产生拼单id，并记录所需成团的拼单记录，另外是写上拼团的状态、唯一索引、回调接口等。这样拼团完成就可以回调对接的平台，通知完成了。【微信支付也是这样的设计，回调支付结果，这样的设计可以方便平台化对接】当再有用户参与后，则写入用户拼单明细表。直至达成拼团
- 回调任务表：当拼团完成后，要做回调处理。但可能会有失败，所以加入任务的方式进行补偿。如果仍然失败，则需要对接的平台，自己查询拼团结果。



### 第1-3节 研发系统设计

进行研发系统设计。包括：库表设计、用例图、系统建模、工程模型、功能流程、UML时序图。

- 用例图：用户与系统交互最简表示形式；
- 流程图：功能节点的串联关系；
- 时序图：展示了整个拼团过程所涉及的系统模块和流转关系

**系统架构：**

*MVC架构*

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323200411716.png" alt="image-20250323200411716" style="zoom:50%;" />

*DDD架构*

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323200448795.png" alt="image-20250323200448795" style="zoom:50%;" />





### 第2-2节 试算模型抽象模板设计

***引入设计模式进行解耦和实现，提高工程代码的扩展性***

=> 设计模式抽象模板的通用结构定义，添加一个 tree规则树抽象模型，在引入到工程中进行使用。这样后续工程中就可以不断的定义通用的设计模式被不同的场景统一使用了。



**模型设计**

链式的多分支规则树模型结构，由功能节点自行决定后续流程的执行链路。它的设计比责任链的扩展性更好，自由度也更高

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323212011895.png" alt="image-20250323212011895" style="zoom:50%;" />

- 首先，定义抽象的通用规则树模型结构
  - 涵盖：StrategyMapper, StrategyHandler /ˈstrætədʒi/、AbstractStrategyRouter\<T, D, R>。 通过泛型设计允许使用方可以自定义出入参和动态上下文，让抽象模板模型具有通用性。
  - 之后，由使用方自定义出工厂、功能抽象类和一个个流程流转的节点。



**编码实现**

项目工程的Types模块中，添加通用设计模式模板

*1.1 策略映射器*

````java
public interface StrategyMapper<T, D, R> {

    /**
     * 获取待执行策略
     *
     * @param requestParameter 入参
     * @param dynamicContext   上下文
     * @return 返参
     * @throws Exception 异常
     */
    StrategyHandler<T, D, R> get(T requestParameter, D dynamicContext) throws Exception;

}
````

- 用于获取每个执行的节点，责任链加强版
- T，D，R：入参，上下文，反参

*1.2 策略受理器*

```java
public interface StrategyHandler<T, D, R> {

    StrategyHandler DEFAULT = (T, D) -> null;

    R apply(T requestParameter, D dynamicContext) throws Exception;

}
```

- 执行具体的业务流程，利用上下文传递信息给后面执行的节点。

*1.3 策略路由器*

```java
public abstract class AbstractStrategyRouter<T, D, R> implements StrategyMapper<T, D, R>, StrategyHandler<T, D, R> {

    @Getter
    @Setter
    protected StrategyHandler<T, D, R> defaultStrategyHandler = StrategyHandler.DEFAULT;
	
    // 伪代码
    public R router(T, D) throws Exception {
        StrategyHandler strategyHandler = get(T, D);
        if(null != strategyHandler) return strategyHandler.apply(T, D);
        return defaultStrategyHandler.apply(T, D);
    }

}
```

- 执行具体节点，转发给下一个节点或者传递到默认节点中



### 第2-3节 多线程异步数据加载

**首页试算**

当一个用户进入到购物首页查看商品的优惠信息，我们可以把这个过程定义为试算过程。试算；试试算一下，这个用户进入到首页看这个商品的时候，商品的营销优惠信息。包括：原始价格、折扣价格、拼团目标、拼团时效、是否可见优惠、是否可参与拼团等，这些东西都是试算拿到的结果。

**1. Model定义对象； 2. 服务功能实现，trial试算模块；3. 业务流转的功能节点**

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250323214143312.png" alt="image-20250323214143312" style="zoom:50%;" />



**模型链路**

```java
IIndexGroupBuyMarketService 
=> RootNode 
	=> multiThread[前置处理|检查信息] => doApply[业务+路由] 
=> SwitchNode 
	=> multiThread[前置处理] => doApply[业务+路由] 
=> MarketNode 
	=> multiThread[⭐前置处理⭐拼单试算] => doApply[业务+路由] 
=> EndNode 
	=> multiThread[前置处理] => doApply[业务|构建结果]     
```

**路由策略模板[节点模板]**

```java
public abstract class AbstractMultiThreadStrategyRouter<T, D, R> implements StrategyMapper<T, D, R>, StrategyHandler<T, D, R> {
    
    public R router(T requestParameter, D dynamicContext) throws Exception {}
    @Override
    public R apply(T requestParameter, D dynamicContext) throws Exception {
        multiThread() // 前置处理
        return doApply() // 业务逻辑
    }
    protected abstract void multiThread(T requestParameter, D dynamicContext) throws;
    protected abstract R doApply(T requestParameter, D dynamicContext) throws;
```

```java
public abstract class AbstractGroupBuyMarketSupport<T, D, R> extends AbstractMultiThreadStrategyRouter<T, D, R> {

    protected long timeout = 500;
    @Resource
    protected IActivityRepository repository;

    @Override
    protected void multiThread(T, D) {
        // 空方法，需要实现的话，让子类重写
    }

}
```

**具体节点实例**

工作流：前置处理 => 业务逻辑 => 路由

```java
public class RootNode extends AbstractGroupBuyMarketSupport<MarketProductEntity, DynamicContext, TrialBalanceEntity> {

    @Resource
    private SwitchNode nextNode;

    @Override
    protected TrialBalanceEntity doApply(T, D, R) throws Exception {
        log.info("业务处理")
        return router(requestParameter, dynamicContext); // 下一个节点
    }

    @Override
    public StrategyHandler<T, D, R> get(T, D, R) {
        return switchNode;
    }
}
```

⭐⭐⭐

***异步数据加载***

```java
// 异步查询活动配置
XXXThreadTask taskA = new XXXThreadTask(...);
FutureTask<XXX> xxxFutureTaskA = new FutureTask<>(XXXThreadTask);
threadPoolExecutor.execute(xxxFutureTask);

// 异步查询商品信息 -
XXXThreadTask taskB = new XXXThreadTask(...);
FutureTask<XXX> xxxFutureTaskB = new FutureTask<>(XXXThreadTask);
threadPoolExecutor.execute(xxxFutureTask);

// 写入上下文， 前置查询数据 
dynamicContext.setXXXA(xxxFutureTaskA.get(TIMEOUT, TimeUnit.MINUTES)); // 阻塞指定时间
dynamicContext.setXXXB(xxxFutureTaskB.get(TIMEOUT, TimeUnit.MINUTES));
```



---

### 第2-4节 **策略模式优惠折扣计算**

不同优惠模式 => 策略模式实现

**定义接口**

```java
public interface IDiscountCalculateService {
    BigDecimal calculate(...);
}
```

**抽象模板**

```java
public abstract class AbstractDiscountCalculateService implements IDiscountCalculateService {

    @Override
    public BigDecimal calculate(...) {
        // 1. 人群标签过滤
        if (IsFilter){
            boolean isCrowdRange = filterTagId(userId, groupBuyDiscount.getTagId());
            if (!isCrowdRange) return originalPrice;
        }
        // 2. 折扣优惠计算
        return doCalculate(originalPrice, groupBuyDiscount);
    }

    // 人群过滤 - 限定人群优惠
    private boolean filterTagId(String userId, String tagId) {
        // todo xiaofuge 后续开发这部分
        return true;
    }
	
    // 具体计算函数
    protected abstract BigDecimal doCalculate(...);

}
```

**折扣方法**

```java
@Slf4j
@Service("MJ")  // 满减优惠
public class MJCalculateService extends AbstractDiscountCalculateService {

    @Override
    public BigDecimal doCalculate(BigDecimal originalPrice, GroupBuyActivityDiscountVO.GroupBuyDiscount groupBuyDiscount) {
        log.info("优惠策略折扣计算:{}", groupBuyDiscount.getDiscountType().getCode());

        // 折扣表达式 - 100,10 满100减10元
        String marketExpr = groupBuyDiscount.getMarketExpr();
        String[] split = marketExpr.split(Constants.SPLIT);
        BigDecimal x = new BigDecimal(split[0].trim());
        BigDecimal y = new BigDecimal(split[1].trim());

        // 不满足最低满减约束，则按照原价
        if (originalPrice.compareTo(x) < 0) {
            return originalPrice;
        }

        // 折扣价格
        BigDecimal deductionPrice = originalPrice.subtract(y);

        // 判断折扣后金额，最低支付1分钱
        if (deductionPrice.compareTo(BigDecimal.ZERO) <= 0) {
            return new BigDecimal("0.01");
        }

        return deductionPrice;
    }

}
```

**营销调用**

```java
// MarketNode

public TrialBalanceEntity doApply(T, D) throws Exception {
    log.info("拼团商品查询试算服务");

    dynamicContext.getGroupBuyActivityDiscountVO();
    groupBuyActivityDiscountVO.getGroupBuyDiscount();

    dynamicContext.getSkuVO();

    // 具体的策略，优惠模式
    IDiscountCalculateService discountCalculateService = discountCalculateServiceMap.get(groupBuyDiscount.getMarketPlan());
    if (null == discountCalculateService) {
        log.info("...", JSON.toString(discountCalculateServiceMap.keys()))
    	throw ...
    }

    // 折扣价格
    BigDecimal deductionPrice = discountCalculateService.calculate(requestParameter.getUserId(), skuVO.getOriginalPrice(), groupBuyDiscount);
    dynamicContext.setDeductionPrice(deductionPrice);

    return router(requestParameter, dynamicContext);
}
```

学习点：

```java
@Service("Key")
XXX implement IDiscountCalculateService {}
    
@Resource   // ⭐先按名称，找不到再按类型注入
private Map<String, IDiscountCalculateService> discountCalculateServiceMap;
```





### 第2-5节 人群标签数据采集

精准的对这些用户做定向活动投放，比如；特定的券、特定的通知等。以此达到更加精准的运营效果

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250324161606013.png" alt="image-20250324161606013" style="zoom:50%;" />

人群标签是根据用户业务行为采集的

**数据库表：**

crowd_tags ：统计功能；

crowd_tags_detail：tag_id => user_id;  具体每个人群都有谁；

crowd_tags_job：tag_id => batch_id, tag_relu:消费规则;

![image-20250324170251768](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250324170251768.png)

流程：

1. 根据tag_id, batch_id 查询crowd_tags_job得到对应批次任务
2. 采集用户数据
3. 数据写入记录 // 暂存
4. 数据写入数据库  => crowd_tags_detail
5. 更新人群标签统计 => crowd_tags 



**数据写入数据库**

```java
@Override
public void addCrowdTagsUserId(String tagId, String userId) {
    CrowdTagsDetail crowdTagsDetailReq = new CrowdTagsDetail();
    crowdTagsDetailReq.setTagId(tagId);
    crowdTagsDetailReq.setUserId(userId);
    try {
        crowdTagsDetailDao.addCrowdTagsUserId(crowdTagsDetailReq);
        // 获取BitSet
        RBitSet bitSet = redisService.getBitSet(tagId); // tagId:{userId, ...}:BitSet
        bitSet.set(redisService.getIndexFromUserId(userId), true);  // Long类型啥的转换一下跟bitset长度一致，尽可能均匀
    } catch (DuplicateKeyException ignore) {
        // 忽略唯一索引冲突
    }
}
```

❗❗❗redis中 BitSet 就是bit类型的hashtable， 而 bitmap是字符串；



大致思路：

```java
// 数据写入数据库时，同时把bitset中位置置1，表示这个用户存在于这个集合
// 查询时，根据用户Id，查bitset                    // 这里用户ID=>bitset:key, 使用一个转换函数，使得散列尽可能均匀
```





---

### 第2-6节 库表拆分

将商品SKU与活动表 解耦 => SKU&Activity关联表。使得多个商品可以绑定同一个活动ID(优惠拼团活动)

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250324185432351.png" alt="image-20250324185432351" style="zoom: 67%;" />

```java
# Market节点 判断 商品是否存在优惠活动
=> ErrorNode => 不存在优惠活动
=> EndNode => 返回优惠的试算价格    
```

流程：

```java
// MarketNode => multi-thread 检查是否存在商品<=> 活动关联表
@Override
public GroupBuyActivityDiscountVO call() throws Exception {
    SCSkuActivityVO scSkuActivityVO = activityRepository.querySCSkuActivityBySCGoodsId(goodsId);
    // log.error(JSON.toJSONString(scSkuActivityVO));
    if (scSkuActivityVO == null) return null;
    return activityRepository.queryGroupBuyActivityDiscountVOById(scSkuActivityVO.getActivityId());
}
```

```java
// marketNode 路由
if (dynamicContext.getGroupBuyActivityDiscountVO() == null || dynamicContext.getSkuVO() == null) {
    return errorNode;
}
return tagNode;
```

```java
// ErrorNode 判断上下文，抛出无拼团配置
log.info("拼团商品查询试算服务-NoMarketNode userId:{} requestParameter:{}", requestParameter.getUserId(), JSON.toJSONString(requestParameter));

if (dynamicContext.getGroupBuyActivityDiscountVO() == null || dynamicContext.getSkuVO() == null) {
    log.info("商品无拼团营销配置 {}", requestParameter.getGoodsId());
    throw new AppException(ResponseCode.ILLEGAL_PARAMETER.getCode(), ResponseCode.ILLEGAL_PARAMETER.getInfo());
}

return TrialBalanceEntity.builder().build();
```





---

### 第2-7节 人群标签节点过滤

限制拼团的人群

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250324214800601.png" alt="image-20250324214800601" style="zoom:67%;" />

流程1：

```java
MarketNode =>  TagNode => EndNode
```

```java
// TagNode 检查请求的用户是否 能看见拼团，是否参与拼团  tag_scope = {1,2} 第一位限制是否可见，第二位是否限制参与
// redis:bitset => tag_id:{...} 存放满足条件的用户

// 获取拼团信息
GroupBuyActivityDiscountVO activityDiscountVO = dynamicContext.getGroupBuyActivityDiscountVO();
String tagId = activityDiscountVO.getTagId();
boolean visible = activityDiscountVO.isVisible();
boolean enable = activityDiscountVO.isEnable();

// 无特定人群信息, 人人能操作
if (StringUtils.isBlank(tagId)) {
    dynamicContext.setVisible(true);
    dynamicContext.setEnable(true);
    return router(requestParameter, dynamicContext);
}

// 是否在人群范围内，特定人群能看
boolean isWithin = repository.isTagCrowdRange(tagId, requestParameter.getUserId());
dynamicContext.setVisible(visible || isWithin);
dynamicContext.setEnable(enable || isWithin);

return router(requestParameter, dynamicContext);


// => isTagCrowdRange function
// xiaofuge、liergou在里面
RBitSet bitSet = redisService.getBitSet(tagId);
if (!bitSet.isExists()) return true;
// 判断用户是否存在人群中
return bitSet.get(redisService.getIndexFromUserId(userId));    
```



---

### 第2-8节  动态配置开关操作

目的：如何不停车就给汽车换个轮子？

=> 在程序运行过程中，直接动态变更某些属性配置。这些动态变更的配置包括降级和切量的开关，也包括一些功能程序的白名单用户测试。

**使用Redis作为集中化储存系统的好处，方便一处修改变量，通知所有的实例进行变更！**

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250324220542463.png" alt="image-20250324220542463" style="zoom:50%;" />



**流量降级或切量**

```java
// 自定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
public @interface DCCValue {
    String value() default "";
}

// 动态中心控制 -- 作用于降级
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

// SwitchNode中 进行流量控制
// 根据用户ID切量
String userId = requestParameter.getUserId();

// 判断是否降级
if (activityRepository.downgradeSwitch()) {
    log.info("拼团活动降级拦截 {}", userId);
    throw new AppException(ResponseCode.E0004.getCode(), ResponseCode.E0004.getInfo());
}

// 判断是否切量
if (activityRepository.cutRange(userId)) {
    log.info("拼团活动切量拦截 {}", userId);
    throw new AppException(ResponseCode.E0004.getCode(), ResponseCode.E0004.getInfo());
}
```

**利用Redis的Topic事件发布订阅 进行 修改**

BeanPostProcessor 能够当SpringBean初始化完成后再执行

```java
// DCCValueBeanFactory implements BeanPostProcessor 
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


@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    // 注意：增加 AOP 代理后，获得类的方式要通过 AopProxyUtils.getTargetClass(bean); 不能直接 bean.class 因为代理后类的结构发生了变化，这样不能获得自己的自定义注解了
    Class<?> targetBeanClass = bean.getClass();
    Object targetBeanObject = bean;
    if (AopUtils.isAopProxy(targetBeanObject)) {
        targetBeanClass = AopUtils.getTargetClass(targetBeanObject);
        targetBeanObject = AopProxyUtils.getSingletonTarget(bean);
    }

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

**Controller**

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

⏱️25/3/24-23:34



### 第2-9节 拼团交易营销锁单

25/3/25/ 20:37

完整的业务流程如下：

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250325203753717.png" alt="image-20250325203753717" style="zoom: 80%;" />

- 首先，团购商品下单。下单过程分为：创建流水单、锁定营销优惠（拼团、积分、卷），创建支付订单、唤起收银台支付、用户扫码支付、支付完成核销优惠等。
- 用户以拼团的方式下单，创建流水单完成后，需要与拼团系统交互，锁定营销优惠。更新流水单优惠金额和支付金额。创建订单使用最终的支付金额。
- 拼团表(目标量、完成量、锁单量)，锁单量达到目标量后，不能再参与该拼团。拼团超时，释放空闲位置让其他用户参与.

*Trade Repository  / rɪˈpɒzət(ə)ri /*

```java
// 简单实现步骤
// OutTradeNO 模拟生成 订单ID模拟生成, TeamID模拟生成. 两张表:1.记录拼团中每个用户的信息 2. 记录该拼团信息.

// ⭐仓储业务实现
// 判断是否有团  -- 一行记录存储每个拼团的信息
// 首次则创建拼团订单. 初始化表2
// 否则加入其他人的团 表2 拼团人数+1

// 插入该用户的拼团信息到表1; 每个用户一行

// ⭐控制器业务实现
// 1. 检查用户信息
// 2. 查询用户outTradeNo是否已经使用过,幂等性
// 3. 查询拼团是否人数已满足
// 4. 营销优惠试算
// 5. 判断用户是否能够看见拼团和参与
// 6. 拼团锁单 => 仓储业务
// 7. 返回结果
```



### 第2-10节 责任链抽象模板设计

25/4/8/ 15:53

先前的责任链模板都是写死的。本章内容为：将每个节点的执行逻辑和链接逻辑解耦。

抽象

![image-20250408155522466](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250408155522466.png)

简单的单例链

```java
// 1. IlogicChainArmory:   装配链，提供添加节点方法和获取节点
// 		next()
//		appendNext()
// 2. ILogicLink extends IlogicChainArmory:  并提供一个受理业务逻辑的方法
//		apply()
// 3. AbstractLogicLink extends ILogicLink:  封装添加节点和执行 next 下一个节点的方法
//      next()
//		appendNext()
// 		apply()
```

这里是由统一的**类**维护责任链，也就是一份责任链。如果有2个独立的链要处理，就需要使用到非单例的类进行填充，否则会修改同一份链。也就是 `ILogicLink<T, D, R> next `被几份链反复调整，也就不是一个单独的链了。



多例-链路模式

多例链的设计要解耦链路和执行，把链路当做一个 LinkedList 列表处理，之后执行当做是单独的 for 循环。

```java
// 需要设计的类
// 1. Node & LinkedList extends ILink
// 双向链表节点 & 双向链表

// 2. BusinessLinkedList extends LinkedList
// 从头到尾执行各个节点

// 3. ILogicHandler  -- 定义任务

// 4. LinkArmory -- 创建链表并装配节点
// public LinkArmory(String linkName, ILogicHandler<T, D, R>... logicHandlers)
// private final BusinessLinkedList<T, D, R> logicLink;
```

```java
// 用法
// 1. 定义 RuleLogicNode extends ILogicHandler
// 2. new LinkArmory<>("demo01", ruleLogic201, ruleLogic202, ...);
```

具体伪代码

```java
public class LinkedList<E> implements ILink<E> {

    /**
     * 责任链名称
     * */
    @Getter
    private final String name;
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;

    public LinkedList(String name) {
        this.name = name;
    }

    // 头插节点
    void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
    }

    // 尾插法
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        size++;
    }

    @Override
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public boolean addFirst(E e) {
        linkFirst(e);
        return true;
    }

    @Override
    public boolean addLast(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x=first; x!=null; x=x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x=first; x!=null; x=x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
	
    // 删除节点，注意是双向链表
    E unlink(Node<E> x) {
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        // x为头节点
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        // x为末尾
        if (next == null) {
            last = x.prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        return element;
    }

    @Override
    public E get(int index) {
        return node(index).item;
    }	
	
    // 根据index，判断是前向还是后向
    private Node<E> node(int index) {
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++) {
                x = x.next;
            }
            return x;
        } else {
            Node<E> x = last;
            for (int i = size-1; i > index; i--) {
                x = x.prev;
            }
            return x;
        }
    }

    @Override
    public void printLinkList() {
        if (this.size == 0) {
            System.out.printf("链表为空");
        } else {
            Node<E> temp = first;
            System.out.print("目前的列表，头节点：" + first.item + " 尾节点：" + last.item + " 整体：");
            while (temp != null) {
                System.out.print(temp.item + "，");
                temp = temp.next;
            }
            System.out.println();
        }
    }
}
```



### 第2-11节 交易规则责任链过滤

2025-4-8 17:30

![image-20250408175143439](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250408175143439.png)

该章节补充：过滤拼团活动配置的规则。包括；活动的有效期、状态，以及个人参与拼团的次数。

```java
// Logic chain
// 1. 检查参数
// 2. 检查是否存在交易记录，根据外部OrderId
// 3. 判断拼团活动是否满
// 4. 营销试算 -- 给出折扣价格
// 5. 人群限定 -- 特定人群才能拼团
// 6. 拼团锁单 -- 1. 修改活动表对应的信息 2. 插入一条拼团记录
// 6.1. 交易规则过滤  -- 检查活动时间 + 检查拼团次数  ⭐本节主要这部分
```

具体就是定义一些 责任链节点(拦截器)

```java
@Slf4j
@Service
public class ActivityUsabilityRuleFilter implements ILogicHandler<TradeRuleCommandEntity, TradeRuleFilterFactory.DynamicContext, TradeRuleFilterBackEntity> {

    @Resource
    private ITradeRepository repository;

    @Override
    public TradeRuleFilterBackEntity apply(TradeRuleCommandEntity requestParameter, TradeRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        log.info("交易规则过滤-活动的可用性校验{} activityId:{}", requestParameter.getUserId(), requestParameter.getActivityId());

        // 查询拼团活动
        GroupBuyActivityEntity groupBuyActivityEntity = repository.queryGroupBuyActivityEntityByActivityId(requestParameter.getActivityId());

        // 校验：活动状态
        if (!ActivityStatusEnumVO.EFFECTIVE.equals(groupBuyActivityEntity.getStatus())) {
            log.info("活动的可用性校验，非生效状态 activityId:{}", requestParameter.getActivityId());
            throw new AppException(ResponseCode.E0101);
        }

        // 校验；活动时间
        Date now = new Date();
        if (now.before(groupBuyActivityEntity.getStartTime()) || now.after(groupBuyActivityEntity.getEndTime())) {
            log.info("活动的可用性校验，非可参与时间范围 activityId:{}", requestParameter.getActivityId());
            throw new AppException(ResponseCode.E0102);
        }

        dynamicContext.setGroupBuyActivity(groupBuyActivityEntity);

        // 走到下一个责任链节点
        return next(requestParameter, dynamicContext);
    }
}
```

```java
@Slf4j
@Service
public class UserTakeLimitRuleFilter implements ILogicHandler<TradeRuleCommandEntity, TradeRuleFilterFactory.DynamicContext, TradeRuleFilterBackEntity> {

    @Resource
    private ITradeRepository repository;

    @Override
    public TradeRuleFilterBackEntity apply(TradeRuleCommandEntity requestParameter, TradeRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        log.info("交易规则过滤-用户参与次数校验{} activityId{}", requestParameter.getUserId(), requestParameter.getActivityId());

        GroupBuyActivityEntity groupBuyActivity = dynamicContext.getGroupBuyActivity();

        Integer count = repository.queryOrderCountByActivityId(requestParameter.getActivityId(), requestParameter.getUserId());

        if (groupBuyActivity.getTakeLimitCount() != null && count >= groupBuyActivity.getTakeLimitCount()) {
            log.info("用户参数次数校验，已达可参与上限 activityId:{}", requestParameter.getActivityId());
            throw new AppException(ResponseCode.E0103);
        }

        return TradeRuleFilterBackEntity.builder()
                .userTakeOrderCount(count)
                .build();
    }
}
```

组装责任链  -- BusinessLinkedList 具体业务

```java
@Bean("tradeRuleFilter")
public BusinessLinkedList<TradeRuleCommandEntity, TradeRuleFilterFactory.DynamicContext, TradeRuleFilterBackEntity>
    tradeRuleFilter(ActivityUsabilityRuleFilter activityUsabilityRuleFilter, UserTakeLimitRuleFilter userTakeLimitRuleFilter) {
    // 组装链
    LinkArmory<TradeRuleCommandEntity, DynamicContext, TradeRuleFilterBackEntity> filter = new LinkArmory<>("交易规则过滤链", activityUsabilityRuleFilter, userTakeLimitRuleFilter);
    // 链对象
    return filter.getLogicLink();
}
```

LinkArmory  -- 真正的抽象装配责任链

```java
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
```



用法

```java
@Resource
private BusinessLinkedList<TradeRuleCommandEntity, TradeRuleFilterFactory.DynamicContext, TradeRuleFilterBackEntity> tradeRuleFilter;

tradeRuleFilter.aaply(...)
```



---

### 第2-12节：拼团组队结算统计

25/4/8 21:19

 ![image-20250408224034286](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250408224034286.png)

拼团的过程是用户在商城下单，锁定拼团优惠（也就是拼团系统里锁单的过程）。之后就是用户给这笔商品完成支付交易，交易后不会直接发货，直至拼团组队完成后才会发货。

那么，这里有一个流程，就是支付完成后，需要做拼团数量的统计结算。如，拼团需要3个用户一起下单，那么每完成一笔支付，就要给拼团的组队加上一笔记录。这个就是本节要实现的流程。

⭐⭐⭐ 所有用户完成支付后, 最后一个用户的结算触发回调事件!

```java
@Transactional(timeout = 500)
@Override
public void settlementMarketPayOrder(GroupBuyTeamSettlementAggregate groupBuyTeamSettlementAggregate) {
    UserEntity userEntity = groupBuyTeamSettlementAggregate.getUserEntity();
    GroupBuyTeamEntity groupBuyTeamEntity = groupBuyTeamSettlementAggregate.getGroupBuyTeamEntity();
    TradePaySuccessEntity tradePaySuccessEntity = groupBuyTeamSettlementAggregate.getTradePaySuccessEntity();

    // 1. 更新拼团订单明细状态
    GroupBuyOrderList groupBuyOrderList = new GroupBuyOrderList();
    groupBuyOrderList.setUserId(userEntity.getUserId());
    groupBuyOrderList.setOutTradeNo(tradePaySuccessEntity.getOutTradeNo());
    int updateOrderStatus2COMPLETE = groupBuyOrderListDao.updateOrderStatus2COMPLETE(groupBuyOrderList);
    if (1 != updateOrderStatus2COMPLETE) {
        throw new AppException(ResponseCode.UPDATE_ZERO);
    }
	
    // 2. 更新拼团达成数量
    int updateAddCount = groupBuyOrderDao.updateAddCompleteCount(groupBuyTeamEntity.getTeamId());
    if (1 != updateAddCount) {
        throw new AppException(ResponseCode.UPDATE_ZERO);
    }

    // 3. 更新拼团完成状态
    if (groupBuyTeamEntity.getTargetCount() - groupBuyTeamEntity.getCompleteCount() == 1) {
        int i = groupBuyOrderDao.updateOrderStatus2COMPLETE(groupBuyTeamEntity.getTeamId());
        if (1 != i) {
            throw new AppException(ResponseCode.UPDATE_ZERO);
        }

        // 查询拼团交易外部订单号列表
        List<String> outOrderNoList = groupBuyOrderListDao.queryGroupBuyCompleteOrderOutTradeNoListByTeamId(groupBuyTeamEntity.getTeamId());

        // 拼团完成写入回调任务记录
        NotifyTask notifyTask = NotifyTask.builder()
            .activityId(groupBuyTeamEntity.getActivityId())
            .teamId(groupBuyTeamEntity.getTeamId())
            .notifyUrl("暂无")
            .notifyCount(0)
            .notifyStatus(0)
            .build();
        notifyTask.setParameterJson(JSON.toJSONString(new HashMap<String, Object>() {{
            put("teamId", groupBuyTeamEntity.getTeamId());
            put("outTradeNoList", outOrderNoList);
        }}));

        notifyTaskDao.insert(notifyTask);
    }

}
```

这里算是支付成功后的回调



---

