---
author: "LongWei"
title: "Spring笔记"
date: "2025-03-01"
tags: ["spring"]
categories: ["Jobs"]
series: ["JobLearning"]
ShowToc: true
TocOpen: true
---


### 1. 什么是循环依赖

两个及以上的类之间相互依赖。模块A依赖于模块B，模块B依赖于模块A，导致依赖链的循环，无法确定加载/初始化顺序。

=> 多个Bean循环引用导致Spring容器无法正常初始化它们。

延迟某些Bean的初始化时间，使用@Lazy进行懒加载，只有当实际使用了该对象才创建。



### 2. Spring如何解决循环依赖

**提前暴露未完全创建的Bean**

三级缓存解决：

- 一级缓存(Single Objects Map)：用于初始化单例Bean； (成品)
- 二级缓存(Early Singleton Objects Map): 用于存储尚未完全初始化，但实例化的Bean，用于提取暴露对象，避免循环依赖问题；(半成品，成员变量未初始化)
- 三级缓存(Singleton Factories Map): 用于存储对象工厂，可以通过工厂创建早期Bean



**解决步骤：**例如AB两个相互依赖，三级缓存策略

- 创建A，查询一级缓存看看有没有完全体B，没有则看看二级缓存有没有半成品B，都没有则创建A的Bean，调用ceateBean方法(实例化，属性注入，初始化)；
- 之后，A往**三级缓存**加入一个A的getObject方法
- 到了属性注入，因为A依赖B，那么需要创建B。同样的路线，B查询到二级缓存都没发现A，调用createBean创建B实例。到了B的属性注入，发现三级缓存有A工厂，调用getObject创建半成品A，放到二级缓存中，完成B的第二步属性注入。后面initializeBean完成B的创建，并放到一级缓存中。
- 回到A，A调用一级缓存的B完成注入。

未解决的问题：

而如果说 A 是构造器注入，B 是 set 注入。则说明 A 需要 B 的时刻提前了，在实例化 new A(B b)的时候就需要 B。此时 A 没有往三级缓存放getObject，因此到了创建依赖 B的时候，无法获取 A的 getObject 工厂方法，只能继续 new A，造成循环依赖的死循环。



### 4. Spring重要的模块组成

**Core Container 核心容器：**

- Spring Core：提供依赖注入DI和控制反转IOC的实现。
- Spring Beans：负责管理Bean的定义和生命周期，通过IOC完成Bean的创建、依赖注入、初始化、销毁等操作；
- Spring Context：基于Core和Beans的高级容器，提供类似JNDI的上下文功能，包含国际化、事件传播、资源访问等功能；
- Spring Expression Language：用于运行时查询和操作对象的值。

**AOP面向切面编程**

- Spring AOP：提供面向切面编程的功能，可以在方法执行前后或抛出异常时动态插入额外的业务逻辑。

**Data Access 数据访问**

- Spring JDBC：操作数据库
- Spring ORM：支持注意ORM框架，简化持久层开发
- Spring Transaction事务管理：提供声明式事务和编程式的事务管理机制

**Web层**

- 

- Spring Web：提供基础Web开发支持，包括Servlet Api的集成
- Spring MVC：实现了Model-View——Controller(MVC)模式的框架，用于构建HTTP请求的Web应用；

![image-20250415185859345](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250415185859345.png)



### 5. Spring IOC

控制反转，通过依赖注入让容器负责管理对象的创建和管理

- 核心：对象的创建和依赖关系交由IOC容器进行负责。
- 依赖注入：构造器注入，setter注入或接口注入。

控制：控制对象的创建过程

反转：创建对象的主题变为Spring



### 6. Spring Bean

Spring应用中的对象

生命周期

- 实例化
- 依赖注入：将依赖的其他对象注入进来
- 初始化：如果Bean实现了Initializing接口或者@PostConstruct，Spring依赖注入后会调用该初始化方法；
- 销毁

创建方式：

- XML配置
- 基于注解：@Component @Service @Repository @Controller等
- Java配置类：通过结合@Configuration和@Bean实现



### 7. Spring注入方式

- 构造器注入
- setter注入
- 字段注入 @Autowired
- 方法注入 @Autowired
- 接口回调注入：注入实现类



### 8. AOP面向切面

- 核心：将与业务逻辑无关的横切关注点抽取出来，通过声明式动态的应用到业务方法中，而不是嵌入业务逻辑中。

- 关键概念：切面(Aspect)、连接点(Joint Point)、通知(advice)、切入点(PointCut)和织入(Weaving)

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* UserService.createUser(..))")
    public void logBefore() {
        System.out.println("准备创建用户...");
    }
}
```





### 9. Spring拦截链的实现

责任链模式，指一系列拦截器依次生效

- HandlerInterceptor（MVC拦截器）：用于拦截HTTP请求
- Filter（过滤器）：基于Servlet API的过滤器
- AOP拦截链（切面）：实现方法的前后处理。@Before @After @Around

💡在Spring AOP中，@before和@after注解都对应一个Interceptor拦截器，

![image-20250415205311425](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250415205311425.png)

责任链模式



### 10. Spring MVC知识点

基于经典的MVC模式 (Model-View-Controller)来开发Web应用的模块。

基于Servlet API构建，核心是`DispatcherServlet`, 即请求分发器，将HTTP请求映射到控制器方法中，并将数据返回给视图层进行渲染

Model：业务模型和数据模型 分别是Servie、Repository；

Controller：分别是DispatcherServlet和Controller；(分发&处理)

**主要功能：** 请求映射、数据绑定、视图解析、表单处理、异常处理等。

工作流程：

- HTTP请求 => DispatcherServlet 根据URL进行分发
  - HandlerMapping：缓存URL到具体处理器对象的映射关系
  - HandlerAdapter：DispatcherServlet通过该适配器间接调用Handler
- URL => Controller 接收请求并进行业务处理
- ModelAndView：数据封装到模型对象中。
- ViewResolver:负责呈现数据，前后端分离，不需要这个。



### 11. Spring MVC的具体工作原理

![image-20250415210418513](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250415210418513.png)

- DispatcherServlet接收请求
- 请求映射：根据请求URL查询HandlerMapping，返回Handler执行链(拦截器和处理器)
- 依次执行拦截器preHandle()xN=>Controller，若中断则直接转到afterComplete()
- 获取HandlerAdapter：通过HandlerAdapter执行handle方法，解决不同处理器的差异
- 拦截器postHandle()

- AfterCompletion()
- 返回响应数据





### 12. Spring中的设计模式

工厂模式：BeanFactory

模板方法：RestTemplate、JDBCTemplate，RedisStringTemplate

代理模式：AOP

单例模式：IOC下的Bean

责任链模式：SpringMVC拦截器

观察者模式：Spring中的监听器

适配器模式：handlerAdapter



### 13 Spring中的事务传播行为

父事务与子事务之间的关系？一个事务被另一个事务调用

作用：定义和管理事务边界，定义多个事务方法嵌套时，是否开启新事务、复用事务还是挂起事务。。。

- Propagation_Required 必须传播
  - 最常用的事务传播行为，如果存在事务，执行的方法会**加入**到该事务中，若不存在，则会**创建**一个新事务；
  - 例如下单场景，扣减库存、生产订单记录、更新用户积分等多个操作，应该在一个事务中，以保证数据的一致性
- Propagation_SUPPORTS 支持的传播行为
  - 当前存在事务则加入，否则非事务方式执行
  - 例如电商场景下，用户查看商品评论列表，这是正常操作。但是在一个事务中调用该方法(进行商品推广活动统计时，同时查看商品评论列表)，则可以加入该事务。
- Propagation_MANDATORY 强制的传播行为
  - 必须要求存在当前事务，否则抛出异常
  - 场景：促销活动中，进行优惠计算并更新订单金额和扣减库存时，必须在同一个事务中。因为这两是紧密关联的。
- Propagation_REQUIRES_NEW 需要新事务的传播行为
  - 无论是否存在事务，调用的子方法都会创建一个新的事务。
  - 独立小团体
  - 例子：业务操作和日志操作，这俩相对独立的操作，即使业务失败，日志也应该成功
- Propagation_NOT_SUPPORTED  不支持事务的传播行为
  - 子方法以非事务的方式执行，若存在事务，先挂起
  - 场景：xxx
- Propagation_NERVER 不允许事务的传播
  - 要求当前绝对不能存在事务，否则抛出异常
  - 例如: 计算商品的推荐指数时，不涉及事务操作
- Propagation_NESTED 嵌套传播行为
  - 若当前存在事务，方法将在嵌套事务中执行，否则创建新事务
  - 例子：多个步骤，检查库存，生成订单，发送通知等。生成订单时异常，仅回滚这个子事务，不影响前面的检查库存操作。



### 14. Spring IOC容器初始化过程

四阶段

- 启动阶段
  - 配置加载：加载配置文件or配置类，XML配置文件、JAVA配置类或配置注解
  - 创建容器：Spring创建IOC容器(BeanFactory\ApplicationContext), 加载和管理Bean
- Bean定义和注册阶段
  - 解析Bean定义，注册到容器中
- 实例化和依赖注入
  - 实例化：根据BeanDefinition创建Bean实例
  - 依赖注入：根据构造函数、Setter方法、字段注入，将依赖注入到Bean中
- 初始化
  - BeanPostProcessor：处理器会在Bean初始化完成后执行
  - @Post Construct Bean的初始化方法

![image-20250415213456343](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250415213456343.png)



### 15 Qualifier注解

例如：当多个接口有多个实现时，指定哪个具体的Bean，**消除歧义**

```java
@Component
public class Client {

    private final Service service;

    @Autowired
    public Client(@Qualifier("serviceImpl1") Service service) {
        this.service = service;
    }

    public void doSomething() {
        service.serve();
    }
}
```

**@Primary** 多个Bean实现了继承了某个类，依赖注入优先为带有@Primary的类

```java
@Component
@Primary
public class DefaultService implements Service {
    public void serve() {
        System.out.println("Default Service");
    }
}

@Component
@Qualifier("specificService")
public class SpecificService implements Service {
    public void serve() {
        System.out.println("Specific Service");
    }
}

@Component
public class Client {

    private final Service service;

    @Autowired
    public Client(@Qualifier("specificService") Service service) {
        this.service = service;
    }

    public void doSomething() {
        service.serve();
    }
}
```



### 16 事务的失效场景

使用@Transactional声明式事务时，存在以下事务失效情况

- ⭐rollbackFor：没设置对，默认为(RuntimeException or Error回滚才生效)，而自定义异常和IOException等并不会回滚

  ```java
  @Transactional(rollbackFor = Exception.class)
  ```

- ⭐异常被捕获了，异常被catch了，仅打印log，并没有将异常往回抛出。

- ⭐同一个类中方法调用，因为事务是基于动态代理实现的，自己调用自己的方法不会走代理方法

  - 只有通过代理对象调用的方法才会被事务拦截器增强
  - 当类内部方法A调用方法B时，实际上是`this.methodB()`调用，而不是`proxy.methodB()`

  ```java
  ((XXXService)AopContext.currentProxy())
              .applyMethod(xxx);
  ```

- @Transactional应用在**非public**修饰上，Spring事务判断非公开方法不执行事务

- ⭐事务传播设置不当，设置为Requires_new，会创建子事务，两个无关

- 多线程环境，@Transactional基于ThreadLocal，因此多线程不能保持事务同步。



### 17. Spring 启动过程

- **创建<容器>**：
  - 根据配置文件中的信息创建容器 ApplicationContext，容器启动阶段实例化BeanFactory，并加载容器总的BeanDefinitions
- **加载<配置>**：
  - 读取XML配置文件，JAVA Config类，和基于注解的Bean，包括数据库连接配置、AOP配置等等。
- **注册Bean<定义>**
  - 解析配置文件中定义的BeanDefinition和声明的Bean元数据
- **<实例化>**Bean：实例化Bean对象，并放入容器中进行管理
- **<注入依赖>**：
  - 注入Bean对象中的依赖对象，构造器，setter，字段注入
- **<初始化>Bean**
  - InitializingBean，or afterPropertiesSet ? 这是什么呢 & BeanPostProcessors

---

### 18. @Value注解

注入外部资源的**值**

- 配置文件注入

例子：

```properties
app.name=MyApp
app.version=1.0.0
```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppConfig {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    public String getAppName() {
        return appName;
    }

    public String getAppVersion() {
        return appVersion;
    }
}    
```



### 19 常用的注解

- @PostConstruct：Bean初始化完成后调用
  - 提前加载数据
- @PreDestory： Bean销毁前调用
  - 关闭连接，释放资源



- @Scheduled 定时任务

- @Conditional 有条件的装配Bean对象

  ```java
  @Conditional(OnLinuxCondition.class)  // 仅这个类存在时才装配这个Bean到容器中
  @Bean
  public MyService myService() {
      return new MyService();
  }
  ```

- @ControllerAdvice 全局异常处理

  ```java
  @ControllerAdvice
  public class GlobalExceptionHandler {
  
      @ExceptionHandler(Exception.class)
      public ResponseEntity<String> handleAllExceptions(Exception ex) {
          return new ResponseEntity<>("An error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
      }
  }
  
  ```

  

