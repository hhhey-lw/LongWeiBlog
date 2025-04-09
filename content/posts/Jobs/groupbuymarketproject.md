---
date: '2025-03-23T19:36:21+08:00'
draft: false
title: '拼团营销交易拼团'
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

### **第2-13节：交易结算责任链过滤**

⏱️25/4/9 12:31

![image-20250409152543493](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250409152543493.png)

本节诉求：

拼团交易结算的过程，需要一些列的规则过滤。包括；我们上一节提到的校验外部交易单的时间是否在拼团有效时间内，同时还有关于这笔外部交易单是否为有效的拼团锁单订单。另外像是 SC 渠道的有效性也需要在结算时进行校验。

所以，本节我们需要实现一套规则链，来处理这些业务规则。因为规则链已经被抽取为通用的模板了，那么本节使用起来会非常容易。



责任链流程：1. 校验渠道是否为黑名单 2. 外部订单号是否正确 3. 交易时间是否在拼团有效期内 4. 打包信息到上下文

***责任链包装对象***

```java
@Bean("tradeSettlementRuleFilter")
public BusinessLinkedList<TradeSettlementRuleCommandEntity,
TradeSettlementRuleFilterFactory.DynamicContext, TradeSettlementRuleFilterBackEntity> tradeSettlementRuleFilter(
    SCRuleFilter scRuleFilter, OutTradeNoRuleFilter outTradeNoRuleFilter, SettableRuleFilter settableRuleFilter, EndRuleFilter endRuleFilter
) {
    LinkArmory<TradeSettlementRuleCommandEntity, DynamicContext, TradeSettlementRuleFilterBackEntity> tradeSettlementFilter =
        new LinkArmory<>("交易结算规则过滤链", scRuleFilter, outTradeNoRuleFilter, settableRuleFilter, endRuleFilter);

    return tradeSettlementFilter.getLogicLink();

}
```

1 SC过滤器

```java
@Slf4j
@Service
public class SCRuleFilter  implements ILogicHandler<TradeSettlementRuleCommandEntity,
        TradeSettlementRuleFilterFactory.DynamicContext, TradeSettlementRuleFilterBackEntity> {

    @Resource
    private ITradeRepository repository;

    @Override
    public TradeSettlementRuleFilterBackEntity apply(TradeSettlementRuleCommandEntity requestParameter, TradeSettlementRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        // 1. 打印函数流程信息
        log.info("结算规则过滤-渠道黑名单校验{} outTradeNo:{}", requestParameter.getUserId(), requestParameter.getOutTradeNo());

        // 2. 校验黑名单信息
        Boolean intercept = repository.isSCBlackIntercept(requestParameter.getSource(), requestParameter.getChannel());
        if (intercept) {
            log.info("{}-{} 渠道黑名单拦截", requestParameter.getSource(), requestParameter.getChannel());
            throw new AppException(ResponseCode.E0105);
        }

        return next(requestParameter, dynamicContext);
    }
}
```

2 订单号校验过滤器

```java
@Slf4j
@Service
public class OutTradeNoRuleFilter  implements ILogicHandler<TradeSettlementRuleCommandEntity,
        TradeSettlementRuleFilterFactory.DynamicContext, TradeSettlementRuleFilterBackEntity> {

    @Resource
    private ITradeRepository repository;

    @Override
    public TradeSettlementRuleFilterBackEntity apply(TradeSettlementRuleCommandEntity requestParameter, TradeSettlementRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        log.info("结算规则过滤-外部单号校验{} outTradeNo:{}", requestParameter.getUserId(), requestParameter.getOutTradeNo());

        MarketPayOrderEntity marketPayOrderEntity = repository.queryMarketPayOrderEntityByOutTradeNo(requestParameter.getUserId(), requestParameter.getOutTradeNo());

        if (null == marketPayOrderEntity) {
            log.info("不存在的外部交易单号或用户已退单，不需要做支付订单结算：{} outTradeNo:{}", requestParameter.getUserId(), requestParameter.getOutTradeNo());
            throw new AppException(ResponseCode.E0104);
        }

        dynamicContext.setMarketPayOrderEntity(marketPayOrderEntity);

        return next(requestParameter, dynamicContext);
    }
}
```

3 有效期校验器

```java
@Slf4j
@Service
public class SettableRuleFilter  implements ILogicHandler<TradeSettlementRuleCommandEntity,
        TradeSettlementRuleFilterFactory.DynamicContext, TradeSettlementRuleFilterBackEntity> {

    @Resource
    private ITradeRepository repository;

    @Override
    public TradeSettlementRuleFilterBackEntity apply(TradeSettlementRuleCommandEntity requestParameter, TradeSettlementRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        log.info("结算规则过滤-有效时间校验：{}, {}", requestParameter.getUserId(), requestParameter.getOutTradeNo());

        // 1. 上下文获取数据
        MarketPayOrderEntity marketPayOrderEntity = dynamicContext.getMarketPayOrderEntity();

        // 2. 查询拼团对象
        GroupBuyTeamEntity groupBuyTeamEntity = repository.queryGroupBuyTeamByTeamId(marketPayOrderEntity.getTeamId());

        // 3. 外部交易时间 - 支付时间需要在拼团有效时间内完成
        if (requestParameter.getOutTradeTime().after(groupBuyTeamEntity.getValidEndTime())){
            // 4. 判断，外部交易时间是否小于拼团结束时间
            log.info("订单交易时间不在拼团有效时间范围内");
            throw new AppException(ResponseCode.E0106);
        }

        // 5. 设置上下文
        dynamicContext.setGroupBuyTeamEntity(groupBuyTeamEntity);

        return next(requestParameter, dynamicContext);
    }
}
```

4 打包节点

```java
@Slf4j
@Service
public class EndRuleFilter implements ILogicHandler<TradeSettlementRuleCommandEntity,
        TradeSettlementRuleFilterFactory.DynamicContext, TradeSettlementRuleFilterBackEntity> {
    @Override
    public TradeSettlementRuleFilterBackEntity apply(TradeSettlementRuleCommandEntity requestParameter, TradeSettlementRuleFilterFactory.DynamicContext dynamicContext) throws Exception {
        log.info("结算规则过滤-结束节点{} outTradeNo:{}", requestParameter.getUserId(), requestParameter.getOutTradeNo());

        GroupBuyTeamEntity groupBuyTeamEntity = dynamicContext.getGroupBuyTeamEntity();

        return TradeSettlementRuleFilterBackEntity.builder()
                .teamId(groupBuyTeamEntity.getTeamId())
                .activityId(groupBuyTeamEntity.getActivityId())
                .completeCount(groupBuyTeamEntity.getCompleteCount())
                .targetCount(groupBuyTeamEntity.getTargetCount())
                .validStartTime(groupBuyTeamEntity.getValidStartTime())
                .validEndTime(groupBuyTeamEntity.getValidEndTime())
                .lockCount(groupBuyTeamEntity.getLockCount())
                .status(groupBuyTeamEntity.getStatus())
                .build();
    }
}
```

5 SC来源渠道黑名单动态配置

```java
@Slf4j
@Service
public class DCCService 

@DCCValue("scBlacklist:s02c02")  // 默认黑名单渠道
private String scBlacklist;

public Boolean isSCBlackIntercept(String source, String channel) {
    List<String> list = Arrays.asList(scBlacklist.split(Constants.SPLIT));
    return list.contains(source + channel);
}

// Http请求key=scBlacklist&value=s02c02;s03c03

// => Redis (scBlacklist) => scBlacklist,s02c02;s03c03 
```



---

### **第2-14节：拼团回调通知任务**

⏱️25/4/9 15:32

拼团组队交易结算完结后，实现一个回调通知的任务处理。告知另外的微服务系统可以进行后续的流程了。

RPC、MQ，这一类的都是需要有一个公用的注册中心，它的技术架构比较适合于公司内部的统一系统使用。如果是有和外部其他系统的对接，通常我们会使用 HTTP 这样统一标准协议的接口进行使用。

注意：微信支付，支付宝支付，也是在完成支付后，做的这样的回调处理。

![image-20250409153440408](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250409153440408.png)

💡**流程逻辑 1：锁单**

```java
// 1. 检查锁单请求信息 - 请求参数是否为Null or 空串
// 2. 检查outTradeNo是否重复锁单 - 检查检查outTradeNo是否重复锁单是否存在
// 3. 拼团是否能够加入 - 判断拼团当前人数小于目标人数
// 4. 营销优惠试算
// 	  4.1. RootNode => SwitchRoot => MarketNode => TagNode => EndNode
//                                              => ErrorNode
// 		4.1.1 RootNode检查信息
// 		4.1.2 SwitchRoot 降级和限流
// 		4.1.3 MarketNode 1:异步查询商品信息+活动信息 2:进行折扣验算(根据Map拿到对应的折扣对象 - 每个折扣对象实现同一接口)
// 		4.1.4-1 TagNode：根据bitset，检查用户Id是否有资格参与or可见该折扣活动
//		4.1.4-2 ErrorNode：无折扣活动返回
// 		4.1.5 EndNode 组装试算结果
// 5. 根据试算结果判断是否可见和可参与
// 6. 进行锁单
//	  6.1. 交易规则检查 - 校验活动状态+活动时间 && 校验参与次数  -责任链
//      LinkArmory(BusinessLinkedList:ActivityUsabilityRuleFilter+UserTakeLimitRuleFilter)
//		6.1.1 ActivityUsabilityRuleFilter: 检查活动状态和时间
//		6.1.2 UserTakeLimitRuleFilter:     检查用户参与次数
//    6.2. 判断是否有团 - 生成拼团订单 or 加入别人的订单  => 插入or修改记录 GroupBuyOrder
//    6.3. 生成订单明细  => 插入记录到GroupBuyOrderList
```

**💡流程逻辑 2：结算**

```java
// 1. 检查拼团信息 - 结算规则
// 	  1.1 检查 渠道合法性=>外部订单合法性=>拼团时间合法性=>包装检查结果
//    		1.1.1 SCRuleFilter: 检查source和channel是否在黑名单中，@DCCValue("scBlacklist:s02c02"), 可动态配置
//   		1.1.2 OutTradeNoRuleFilter：检查OutTradeNo是否合法，是否GroupBuyOrderList存在初始锁单的拼团订单 
//			(这里，外部进行了支付，但是系统还没修改结算状态呢)
// 			1.1.3 SettableRuleFilter：检查订单交易时间是否在拼团有效时间内
// 			1.1.4 EndRuleFilter：包装结果
// 2. 进行拼团结算
// 	  2.1 更新拼团订单明细 => GroupBuyOrderList对应用户的完成状态
//    2.2 更新拼团达成数量 => GroupBuyOrder对应的用户完成支付数量
//    2.3 更新拼团完成状态 => 最后一个人支付完成触发 => GroupBuyOrder拼团信息状态 => 写入回调任务记录(包装TeamId+所有的外部订单ID)
// 3. 组队回调处理 - 这里显示的执行，使得实时性比较好，只有最后一个人完成才会执行成功，不过有定时任务进行兜底
// 	  3.1 发送Http远程调用，将(TeamId+所有的外部订单ID)发送给第三方服务，通知发货or其他
//    3.2 定时任务：查询所有初始状态和重试状态的 通知任务
```

🏷️**本章节完成 3 触发回调部分**

```java
// 指定触发回调通知任务  - 实时性
@Override 
public Map<String, Integer> execSettlementNotifyJob(String teamId) throws Exception {
    log.info("拼团交易-执行结算通知回调，指定 teamId:{}", teamId);

    List<NotifyTaskEntity> notifyTaskEntityList = repository.queryUnExecutedNotifyTaskList(teamId);
    return execSettlementNotifyJob(notifyTaskEntityList);
}

// 辅助定时扫描，进行通知 - 兜底
@Override
public Map<String, Integer> execSettlementNotifyJob() throws Exception {
    log.info("拼团交易-执行结算通知任务");

    // 查询未执行任务
    List<NotifyTaskEntity> notifyTaskEntities = repository.queryUnExecutedNotifyTaskList();

    return execSettlementNotifyJob(notifyTaskEntities);
}

// 公用通知函数
private Map<String, Integer> execSettlementNotifyJob(List<NotifyTaskEntity> notifyTaskEntities) throws Exception {
    int successCount = 0, errorCount = 0, retryCount = 0;
    for (NotifyTaskEntity notifyTask : notifyTaskEntities) {
        // 回调处理
        String response = port.groupBuyNotify(notifyTask);

        if (NotifyTaskHTTPEnumVO.SUCCESS.getCode().equals(response)) {
            int i = repository.updateNotifyTaskStatusSuccess(notifyTask.getTeamId());
            if (1 == i) {
                successCount += 1;
            }
        } else if (NotifyTaskHTTPEnumVO.ERROR.getCode().equals(response)) {
            int i = repository.updateNotifyTaskStatusError(notifyTask.getTeamId());
            if (1 == i) {
                errorCount += 1;
            }
        } else {
            int i = repository.updateNotifyTaskStatusRetry(notifyTask.getTeamId());
            if (1 == i) {
                retryCount += 1;
            }
        }
    }

    Map<String, Integer> resultMap = new HashMap<>();
    resultMap.put("waitCount", notifyTaskEntities.size());
    resultMap.put("successCount", successCount);
    resultMap.put("errorCount", errorCount);
    resultMap.put("retryCount", retryCount);

    return resultMap;
}

// 远程调用
@Override
public String groupBuyNotify(NotifyTaskEntity notifyTask) throws Exception {
    RLock lock = redisService.getLock(notifyTask.lockKey());
    try {
        // 拼团服务器部署在多台应用服务器上，多任务执行，避免任务被多次执行，锁的粒度 xxx:teamId
        if (lock.tryLock(3, 0, TimeUnit.SECONDS)) {
            try {
                if (StringUtils.isBlank(notifyTask.getNotifyUrl()) || "暂无".equals(notifyTask.getNotifyUrl())) {
                    return NotifyTaskHTTPEnumVO.SUCCESS.getCode();
                }
                return ⭐groupBuyNotifyService.groupBuyNotify(notifyTask.getNotifyUrl(), notifyTask.getParameterJson());⭐ // 这里进行远程调用
            } finally {
                if (lock.isLocked() && lock.isHeldByCurrentThread())
                    lock.unlock();
            }
        }
        return NotifyTaskHTTPEnumVO.NULL.getCode();
    }catch (Exception e) {
        Thread.currentThread().interrupt();
        return NotifyTaskHTTPEnumVO.NULL.getCode();
    }
}

// OKHttp发送请求
public String groupBuyNotify(String apiUrl, String notifyRequestDTOJSON) {
    try {
        // 1. 构建参数
        MediaType mediaType = MediaType.parse("application/json");
        RequestBody body = RequestBody.create(mediaType, notifyRequestDTOJSON);
        Request request = new Request.Builder()
            .url(apiUrl)
            .post(body)
            .addHeader("content-type", "application/json")
            .build();

        // 2. 调用接口
        Response response = okHttpClient.newCall(request).execute();

        // 3. 返回结果
        return response.body().string();
    } catch (IOException e) {
        log.info("拼团回调 HTTP 接口服务异常 {}", apiUrl, e);
        throw new AppException(ResponseCode.HTTP_EXCEPTION);
    }
}
```



---

### 第2-15节：根据UI展示封装接口

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250409202305147.png" alt="image-20250409202305147" style="zoom:60%;" />

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250409202122844.png" alt="image-20250409202122844" style="zoom:50%;" />



- 紫色圈：拼团的统计信息；
- 灰色圈：商品信息；
- 红色圈：参与拼团，随机挑选几个团；
- 绿色圈：参与拼团；单独开始一个新的团，还是参与其他人的团

- 黄色圈：拼团结算。

**ResponseDTO**

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class GoodsMarketResponseDTO {

    private Goods goods;
    private List<Team> teamList;
    private TeamStatistic teamStatistic;

    /**
     * 商品信息
     */
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Goods {
        // 商品ID
        private String goodsId;
        // 原始价格
        private BigDecimal originalPrice;
        // 折扣金额
        private BigDecimal deductionPrice;
        // 支付价格
        private BigDecimal payPrice;
    }

    /**
     * 组队信息
     */
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Team {
        // 用户ID
        private String userId;
        // 拼单组队ID
        private String teamId;
        // 活动ID
        private Long activityId;
        // 目标数量
        private Integer targetCount;
        // 完成数量
        private Integer completeCount;
        // 锁单数量
        private Integer lockCount;
        // 拼团开始时间 - 参与拼团时间
        private Date validStartTime;
        // 拼团结束时间 - 拼团有效时长
        private Date validEndTime;
        // 倒计时(字符串) validEndTime - validStartTime
        private String validTimeCountdown;
        /** 外部交易单号-确保外部调用唯一幂等 */
        private String outTradeNo;

        public static String differenceDateTime2Str(Date validStartTime, Date validEndTime) {
            if (validStartTime == null || validEndTime == null) {
                return "无效的时间";
            }

            long diffInMilliseconds = validEndTime.getTime() - validStartTime.getTime();

            if (diffInMilliseconds < 0) {
                return "已结束";
            }

            long seconds = TimeUnit.MILLISECONDS.toSeconds(diffInMilliseconds) % 60;
            long minutes = TimeUnit.MILLISECONDS.toMinutes(diffInMilliseconds) % 60;
            long hours = TimeUnit.MILLISECONDS.toHours(diffInMilliseconds) % 24;
            long days = TimeUnit.MILLISECONDS.toDays(diffInMilliseconds);

            return String.format("%02d:%02d:%02d", hours, minutes, seconds);
        }

    }


    /**
     * 组队统计
     */
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class TeamStatistic {
        // 开团队伍数量
        private Integer allTeamCount;
        // 成团队伍数量
        private Integer allTeamCompleteCount;
        // 参团人数总量 - 一个商品的总参团人数
        private Integer allTeamUserCount;
    }

}
```

```java
// 增删改查
// 注意：Team的查询
// 1. 先查自己的拼团
// 2. 查其他人的拼团。对应活动，非当前用户，状态为可加入拼团，时间有效，队伍in(对应活动，且锁单人数小于目标人数)

@Override
public List<UserGroupBuyOrderDetailEntity> queryInProgressUserGroupBuyOrderDetailListByRandom(Long activityId, String userId, Integer randomCount) {
    // 1. 根据用户ID、活动ID、查询非当前用户参与的拼团队伍
    GroupBuyOrderList buyOrderListReq = GroupBuyOrderList.builder()
        .activityId(activityId)
        .userId(userId)
        .build();
    buyOrderListReq.setCount(randomCount * 2); // 查询两倍的量，再随机取一半

    // 1. ==ActivityId; 2. !=userId 3; status == 0 or 1; 4. endTime > now(); 5. teamId in (select teamId from group_buy_order where status = 0 and activity = #{activity})
    List<GroupBuyOrderList> groupBuyOrderLists = groupBuyOrderListDao.queryInProgressUserGroupBuyOrderDetailListByRandom(buyOrderListReq);
    if (groupBuyOrderLists == null || groupBuyOrderLists.isEmpty())
        return null;

    // 随机选 randomCount 条
    if (groupBuyOrderLists.size() > randomCount) {
        Collections.shuffle(groupBuyOrderLists);
        groupBuyOrderLists = groupBuyOrderLists.subList(0, randomCount);
    }

    // 2. 过滤队伍获取 TeamId
    Set<String> teamIds = groupBuyOrderLists.stream().map(GroupBuyOrderList::getTeamId)
        .filter(teamId -> teamId != null && !teamId.isEmpty())
        .collect(Collectors.toSet());


    // 3. 查询队伍明细，组装Map结构
    List<GroupBuyOrder> groupBuyOrders = groupBuyOrderDao.queryGroupBuyProgressByTeamIds(teamIds);
    if (groupBuyOrders == null || groupBuyOrders.isEmpty()) return null;

    Map<String, GroupBuyOrder> groupBuyOrderMap = groupBuyOrders.stream()
        .collect(Collectors.toMap(GroupBuyOrder::getTeamId, order -> order));

    // 4. 转换数据
    ArrayList<UserGroupBuyOrderDetailEntity> userGroupBuyOrderDetailEntities = new ArrayList<>();
 	// ... 根据 groupBuyOrders 和 groupBuyOrderLists组装结果

    return userGroupBuyOrderDetailEntities;
}
```

```java
// 锁单 接口
⬇️
// outTradeNo - 调用第三方支付生成
⬇️
// 结算 接口
```

<img src="C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250410001830020.png" alt="image-20250410001830020" style="zoom:50%;" />



---



