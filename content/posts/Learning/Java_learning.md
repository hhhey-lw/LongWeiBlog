---
author: "LongWei"
title: "Java学习笔记"
date: "2024-04-19"
tags: ["Java"]
categories: ["Learning"]
ShowToc: true
TocOpen: true
---





### Java 反射

它提供了动态性和灵活性，使得程序可以在运行时动态地加载类、调用方法、访问字段等，而不需要在编译时确定这些操作。

**1. 动态加载类**

在运行时根据类名动态加载类，而不是在编译时硬编码类名。 -- 插件化架构：根据配置文件或用户输入动态加载类。

**2. 创建对象**

通过反射可以在运行时动态创建对象，即使类的构造函数是私有的。

- 工厂模式：根据配置动态创建对象。
- 依赖注入框架：Spring 通过反射创建 Bean 实例。

**3. 调用方法**

通过反射可以在运行时动态调用对象的方法，即使方法是私有的。

- 测试框架：JUnit 通过反射调用测试方法。

**4. 访问字段**

通过反射可以在运行时动态访问对象的字段，即使字段是私有的

- 序列化和反序列化：通过反射访问对象的字段。
- 对象关系映射（ORM）：Hibernate 通过反射访问实体类的字段。

**7. 注解处理**

通过反射可以获取类、方法、字段上的注解，并根据注解执行相应的逻辑。



*24/4/14*

 1️⃣ **反射的作用**

- 获取类中的所有信息(e.g.持久化、IDE的代码提示)

```java
Field[] fields = cls.getDeclaredFields();
for (int i = 0; i < fields.length; i++) {
    fields[i].setAccessible(true);
    // String name = fields[i].getName();  // 属性名 第一次先生成列名
    Object value = fields[i].get(obj);
    bw.append(value.toString());
    if (i < fields.length - 1) {
        bw.append(", ");
    }
}
bw.newLine();
```



- 结合配置文件动态创建对象

```java
Properties prop = new Properties();

String filename = "prop.properties";
FileInputStream is = new FileInputStream(filename);

prop.load(is);

String classname = prop.getProperty("classname");
String method = prop.getProperty("method");
String name = prop.getProperty("name");

Class cls = Class.forName(classname);
Constructor con = cls.getDeclaredConstructor();
Object o = con.newInstance();

Method setName = cls.getMethod("setName", String.class);
setName.invoke(o, name);

Method met = cls.getDeclaredMethod(method);
met.invoke(o);
```



 2️⃣ **获取Class的三种方式**

- Class.forname("全类名")
- 类名.class
- 对象.getClass()

3️⃣  **常用信息**

- *Constructor(构造器)  Parameter(方法参数)  Field(成员变量)  Modifiers(权限修饰符)  Method(成员方法)  Declared(用于获取私有信息)*





---

### Java 动态代理

***无侵入的给代码添加额外的功能***

流程：

- 准备：具体行为对象 + 抽象行为接口 + 代理类
- 执行：数据 => 代理类(抽象行为接口) => InvocationHandler.invoke

钩子之类的，插入代码

```java
public interface Star                 (抽象接口，定义行为)
public class BigStar implements Star  (具体类名，实现行为)


/*
* ClassLoader: 用于加载代理类的类加载器 (具体)
* Interface  : 代理类需要实现的接口列表 (抽象)
* InvocationHandler:  execute proxy task
*/
Star star = (Star) Proxy.newProxyInstance(
    bigStar.getClass().getClassLoader(),
    new Class[]{Star.class},
    new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if (method.getName().equals("sing")) {
                System.out.println("准备话筒，收钱");
            }else if (method.getName().equals("dance")) {
                System.out.println("准备舞台，收钱");
            }

            return method.invoke(bigStar, args);
        }
    }
);

// 使用代理
BigStar kun = new BigStar("KunKun");
Star proxy = ProxyUtil.createProxy(kun);
String result = proxy.sing("只因你太美");
System.out.println(result);
```





---

### Java 泛型

*类型参数化*

1️⃣ **编译时检查**

不符合的编译通过不了

2️⃣ **代码复用**

```java
public static <T extends Number> Double add(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}

// 函数签名 <T extends Number> 用于限定参数类型
// int float double 都OK
```

- **泛型类**

```java
// Class指定的T会传递给成员变量和成员方法
public class GenericClass <T>{
    private T good;

    public T getGood(T var) {
        System.out.println(var);
        return this.good;
    }

    public void setGood(T good) {
        this.good = good;
    }
}

// 指定多个泛型类型
public class MultiGenericClass<K, V> {
    private K key;
    private V value;

    public K getKey() {
        return key;
    }

    public void setKey(K key) {
        this.key = key;
    }

    public V getValue(K key) {
        return value;
    }

    public void setValue(V value) {
        this.value = value;
    }
}
```

- **泛型接口**

```java
// 就是实例化对象时 再指定参数类型
public interface GenericInterface <K, V>{
    public abstract V getValue(K key);
}

class myMap<K, V> implements GenericInterface<K, V> {

    private HashMap<K, V> map = new HashMap<>();

    @Override
    public V getValue(K key) {
        return map.get(key);
    }

    public void put(K key, V value) {
        map.put(key, value);
    }
}
```

- **泛型方法**

函数签名<T>表示此方法是泛型方法，并且这个<T>与Class的<T>无关，**限定在方法部分**    作用范围！！！

```java
// 泛型结合反射，动态创建对象，可以进行初始化
public <T> T getObject(Class<T> c) throws NoSuchMethodException {
    Constructor<T> constructor = c.getConstructor();
    T t = constructor.newInstance();
    return t;
}

public static <T extends Number> Double add(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

- **泛型边界**

```java
// ？ 是 父类  - 下界
public static void funcAAA(List<? super Student> sList){
    System.out.println(sList);
}

// ？ 是 子类  - 上界
public static void funcAA(List<? extends Person> pList){
    System.out.println(pList);
}
```



**泛型擦除：**

​	为了兼容java没有泛型特性的老版本。在**编译阶段**会进行所谓的“**类型擦除**”。将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型）、

- 消除类型参数声明，即删除`<>`及其包围的部分。
- 据类型参数的上下界推断并替换所有的类型参数为原生态类型(往父类转，更包容)
- 自动产生“**桥接方法**”以保证擦除类型后的代码仍然具有泛型的“多态性”

​	

无限制类型擦除  - `<T>`和`<?>`的类型参数都被替换为Object

有限制类型擦除 - `<T extends Number>`和`<? extends Number>`的类型参数被替换为`Number`，

​				  			`<? super Number>`被替换为Object



***桥接*   - *编译阶段类型擦除时执行***

保证多态特性  - 因为可以多个子类继承父类，父类的类型又是泛型，不能被某个子类的泛型限制住！

```java
// 父类定义
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  
        this.value = value;  
    }  
} 

// 子类定义
class DateInter extends Pair<Date> {  

    @Override  
    public void setValue(Date value) {  
        super.setValue(value);  
    }  

    @Override  
    public Date getValue() {  
        return super.getValue();  
    }  
}

// 编译时 类型擦除后
class Pair {  
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}
```

![image-20240415172031269](http://verification.longcoding.top/Fmw2-h00HMxs2uUZTveY1PTAqy-n)

生成中间的桥接方法，进行连接，维护多态性。

![image-20241122125640349](http://verification.longcoding.top/FvPTfqXKkhldWbQH1j_oVNE67cz8)



---

### Java 注解

```java
@Target({ElementType.METHOD, ElementType.PARAMETER})	// 放置在哪
@Retention(RetentionPolicy.RUNTIME)						// 存活时间
public @interface Log {
    // 模块
    String title() default "";

    // 功能
    BusinessType businessType() default BusinessType.OTHER;

    // 操作人员
    OperatorType operatorType() default OperatorType.MANGE;

    // 是否保存请求参数
    boolean isSaveRequestData() default true;
}
```

**与AOP结合，实现切入**

```java
@Aspect				// 给Spring托管，并且此注解标识这个类实现切入功能
@Component
public class LogAspect {

    @Pointcut("@annotation(log_aop.Log)")	// 检测带有@Log注解的部分
    public void logPointCut(){}


    @Before("logPointCut()")				// 在切入点前执行操作
    public void logPointCutBefore(JoinPoint joinPoint) {
        System.out.println("在切入点前先执行");
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        System.out.println(method.getName());
        Log annotation = method.getAnnotation(Log.class);
        if (annotation != null) {
            System.out.println(annotation.title());
            System.out.println(annotation.isSaveRequestData());
            System.out.println(annotation.businessType());
            System.out.println(annotation.operatorType());
        } else {
            System.err.println("没有获取到注解信息");
        }
    }
}
```

**Spring Boot 简单实现**

```java
@RestController
public class Controller {

    @Log(title = "模拟查询信息", businessType = BusinessType.SELECT, operatorType = OperatorType.PERSON)
    @GetMapping("/test")
    public Map<String, Object> test() {
        System.out.println("执行处理请求操作");
        HashMap<String, Object> map = new HashMap<>();
        map.put("code", 200);
        map.put("msg", "success");

        return map;
    }

}
```





---

### Java 多线程

#### *创建线程的三种方式* 

 *- 单继承多实现*

```java
public class ThreadFirstWay extends Thread{

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println(this.getName() + " print : " + i);
        }
    }
}

new ThreadFirstWay().start()

// 更灵活
-------------------------------------------------------
public class ThreadSecondWay implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            Thread thread = Thread.currentThread();
            System.out.println(thread.getName() + " print : " + i);
        }
    }
}

new Thread(new ThreadSecondWay()).start()
    
// 返回结果
-------------------------------------------------------
class MyCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 20; i++) {
            sum += i;
        }
        return sum;
    }
}

MyCallable callable = new MyCallable();
FutureTask<Integer> task = new FutureTask<>(callable);

Thread thread1 = new Thread(task);
thread1.start();

Integer result = task.get();
System.out.println(result);
```



![image-20250109195552849](http://verification.longcoding.top/FmUwuBD8Gl3L03PtKFZK7pUFjpPO)



#### ***线程池***

```java
// newCachedThreadPool: 会复用之前的线程(空闲, 超出时间会关闭)
// newFixedThreadPool: 固定线程数，任务在阻塞队列中排队

ExecutorService threadPool = Executors.newFixedThreadPool(2);
threadPool.submit(new MyRunnable("@Run 1"));

// 更详细
-------------------------------------------------------
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
    corePoolSize=2,	// 核心线程
    maximumPoolSize=3,  // 临时线程
    KeepAliveTime=60, // 临时线程存活时间
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(3),  // 阻塞队列 
    new ThreadPoolExecutor.AbortPolicy() // 默认抛出异常
);

threadPool.execute(new TestRunnable("@Runnable 1"));
```



#### **单生产单消费者**

使用 **synchronized(唯一对象)** 实现临界区

```java
// 单生产单消费的同步
class Desk {
    // 简化版 1有，0无
    public static int foodFlag = 0;

    // 总个数
    public final static int total = 10;

    // 剩余数
    public static int count = 10;

    // 锁
    public static Object lock = new Object();
}

// 消费者
class Consumer extends Thread {

    @SneakyThrows
    @Override
    public void run() {
        while (true) {
            synchronized (Desk.lock) {
                // 美食家是否吃饱
                if (Desk.count == 0) {
                    break;
                } else {
                    // 是否做好食物
                    if(Desk.foodFlag == 0) {
                        // 等待生产者制作食物
                        Desk.lock.wait();
                    }else {
                        // 消耗掉一份食物
                        Desk.count--;

                        // 吃食物
                        System.out.println("消费者正在吃第" + (Desk.total-Desk.count) + "碗食物");

                        // 通知生产者制作食物
                        Desk.lock.notifyAll();

                        // 是否存在食物
                        Desk.foodFlag = 0;
                    }
                }
            }
        }
    }

}


class Producer extends Thread {
    @SneakyThrows
    @Override
    public void run() {
        while (true) {
            // 占用桌子
            synchronized (Desk.lock) {
                // 美食家是否吃饱
                if(Desk.count == 0) {
                    break;
                }else {
                    // 上份食物是否存在
                    if(Desk.foodFlag == 1) {
                        // 等食物被吃
                        Desk.lock.wait();
                    } else { // 制作食物
                        System.out.println("生产者制作第" + (Desk.total - Desk.count + 1) + "份食物");

                        Desk.foodFlag = 1;

                        Desk.lock.notifyAll();
                    }
                }
            }
        }
    }
}
```

#### **简单抽奖**

```java
// 简单考虑 100%中奖
class LotteryBox extends Thread {
    public final static int[] Pond = {2, 5, 10, 20, 50, 100, 200, 300, 500, 1000};

    public static int[] count = {10, 9, 8, 7, 6, 5, 4, 3, 2, 1};

    public static int totalNumber = 100;
    public static Set<Integer>[] winningNumber = new Set[count.length];
    public static List<Integer> number = new ArrayList<>();

    public static int numberIdx = 0;

    public static ReentrantLock lock = new ReentrantLock();

    public static void initData() {
        // 初始化数组中的每个 Set
        for (int i = 0; i < winningNumber.length; i++) {
            winningNumber[i] = new HashSet<>();
        }

        for (int i = 1; i <= totalNumber; i++) {
            number.add(i);
        }

        Collections.shuffle(number);

        int idx = 0;
        // 指定中奖号码
        for (int i = 0; i < count.length; i++) {
            for (int j = 0; j < count[i]; j++) {
                winningNumber[i].add(number.get(idx));
                idx += 1;
            }
        }

        Collections.shuffle(number);
    }


    @Override
    public void run() {
        while (true) {
            LotteryBox.lock.lock();
            if (LotteryBox.numberIdx < LotteryBox.totalNumber) {

                int n = number.get(LotteryBox.numberIdx);
                LotteryBox.numberIdx += 1;
                LotteryBox.lock.unlock();

                int rank = -1;
                for (int j = 0; j < winningNumber.length; j++) {
                    if (winningNumber[j].contains(n)) {
                        rank = j;
                    }
                }

                if (rank == -1) {
                    System.out.println(getName() + "抽到的" + n + "号码没有中奖！");
                } else {
                    System.out.println(getName() + "抽到的" + n + "号码中了" + Pond[rank] + "元");
                }
            } else {
                LotteryBox.lock.unlock();
                System.out.println(getName() + ": 奖池已经抽取完毕！");
                break;
            }
        }
    }
}
```

#### ***邮件服务***

***ArrayBlockingQueue 资源不够，会自动阻塞和唤醒线程***

```java
package thread;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.SneakyThrows;

import java.io.Serializable;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Date: 2024/4/19
 */
public class EmailServer {
    public static void main(String[] args) throws InterruptedException {
        MailService service = new MailService();

        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        ConsumeEmailQueue runnable1 = new ConsumeEmailQueue("@服务1", service);
        ConsumeEmailQueue runnable2 = new ConsumeEmailQueue("$服务2", service);
        threadPool.submit(runnable1);
        threadPool.submit(runnable2);

        for (int i = 0; i < 5; i++) {
            Email email = new Email("title"+i, "content"+i);
            MailQueue.getMailQueue().produce(email);
            Thread.sleep(3000);
        }
    }
}


class MailService {
    public void sendEmail(Email email) {
        System.out.println("-------------------------");
        System.out.println("来新邮件了！");
        System.out.println(email);
        System.out.println("-------------------------");
    }
}

class ConsumeEmailQueue implements Runnable {
    private MailService mailService;
    private String name;

    public ConsumeEmailQueue(String name, MailService mailService) {
        this.name = name;
        this.mailService = mailService;
    }

    @SneakyThrows
    @Override
    public void run() {
        while (true) {
            Email email = MailQueue.getMailQueue().consume();
            if (email != null) {
                System.out.println(name + "取走了一封邮件");
                System.out.println("剩余邮件: " + MailQueue.getMailQueue().size());
                mailService.sendEmail(email);
            }
            System.out.println("消费者检查了一轮\n");
        }
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Email implements Serializable {
    private String title;
    private String content;
}

class MailQueue {
    // 队列大小
    public static int QUEUE_MAX_SIZE = 1000;

    // 阻塞队列
    public static ArrayBlockingQueue<Email> blockingQueue = new ArrayBlockingQueue<>(100);

    // 私有化构造器
    private MailQueue() {
    }
    // 实现单例模式
    private static class SingletonHolder {
        private static MailQueue mailQueue = new MailQueue();
    }

    public static MailQueue getMailQueue() {
        return SingletonHolder.mailQueue;
    }

    // 生产
    public void produce(Email email) throws InterruptedException {
        blockingQueue.put(email);
    }

    // 消费
    public Email consume() throws InterruptedException {
        return blockingQueue.take();
    }

    public int size() {
        return blockingQueue.size();
    }
}
```





---

### Java Websocket

#### **简易共享聊天室**

##### 服务端

```java
// 自定义终端配置类，目的就是在建立连接时，保存一些数据以供访问  - httpSession 保存了请求的信息
public class GetHttpSessionConfigurator extends ServerEndpointConfig.Configurator {
    @Override
    public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
        HttpSession httpSession = (HttpSession) request.getHttpSession();
        sec.getUserProperties().put(HttpSession.class.getName(), httpSession);
    }
}
```

```java
// 一个Endpoint == 一个客户端
// 服务终端对象， configurator关联自定义的配置器类，目的是让 WebSocket 会话能够访问存储在 HTTP 会话中的数据
@ServerEndpoint(value = "/chat", configurator = GetHttpSessionConfigurator.class)
@Component
public class ChatEndpoint {
    // 维护一份共享的连接终端在线信息。  多个线程频繁地访问和修改共享的键值对集合时，ConcurrentHashMap 是一个合适的选择
    private static Map<String, ChatEndpoint> onlineUsers= new ConcurrentHashMap<>(); // user => endpoint

    // 用来发送信息给对应终端
    private Session session;

    // 用来获取请求的信息，标识发送对象
    private HttpSession httpSession;

    // 首次连接， 在HTTP握手连接时已经把<username, HttpSession>放入config中
    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        // 初始化数据
        this.session = session;
        HttpSession httpSession = (HttpSession) config.getUserProperties().get(HttpSession.class.getName());
        this.httpSession = httpSession;

        String user = (String) httpSession.getAttribute("user");
        onlineUsers.put(user, this);

        // 广播通知 更新在线用户，让别人知道你上线了
        String message = MessageUtils.getMessage(true, null, getAllOnlineUsers());
        broadcastAllUsers(message);
    }

    // 广播信息给所有在线用户
    private void broadcastAllUsers(String message) {
        try {
            Set<String> names = onlineUsers.keySet();
            for (String name : names) {
                ChatEndpoint chatEndpoint = onlineUsers.get(name);
                chatEndpoint.session.getBasicRemote().sendText(message); // @@@ 核心发送信息的代码
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 获取所有在线用户
    private Set<String> getAllOnlineUsers() {
        return onlineUsers.keySet();
    }

    // @@@ 核心 => 转发信息
    @OnMessage
    public void onMessage(String message, Session session) {
        try {
            ObjectMapper mapper = new ObjectMapper();
            Message msg = mapper.readValue(message, Message.class);
            String toName = msg.getToName();
            String data = msg.getMessage();
            String user = (String) httpSession.getAttribute("user");
            String resultMsg = MessageUtils.getMessage(false, user, data);

            onlineUsers.get(toName).session.getBasicRemote().sendText(resultMsg);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @OnClose
    public void onClose(Session session) {
        String user = (String) httpSession.getAttribute("user");
        onlineUsers.remove(user);

        String message = MessageUtils.getMessage(true, null, getAllOnlineUsers());
        broadcastAllUsers(message);
    }
}
```

##### ***客户端***

```js
var ws = new WebSocket("ws://localhost:8080/chat");
        ws.onopen = function (e) {
            $("#username").html("用户："+ username +"<span>在线</span>");
        }
        //接受消息
        ws.onmessage = function (ev) {
            var datastr = ev.data;
            var res = JSON.parse(datastr);
            //判断是否是系统消息
            if(res.system){
                //好友列表
                //系统广播
                var names = res.message;
                var userlistStr = "";
                var broadcastListStr = "";
                for (var name of names){
                    if (name != username){
                        userlistStr += "<a onclick='showChat(\""+name+"\")'>"+ name +"</a></br>";
                        broadcastListStr += "<p>"+ name +"上线了</p>";
                    }
                };
                $("#hylist").html(userlistStr);
                $("#xtlist").html(broadcastListStr);

            }else {
                //不是系统消息
                var str = "<span id='mes_left'>"+ res.message +"</span></br>";
                if (toName == res.fromName)
                    $("#content").append(str);

                var chatdata = sessionStorage.getItem(res.fromName);
                if (chatdata != null){
                    str = chatdata + str;
                }
                sessionStorage.setItem(res.fromName, str);

            };
        }

        ws.onclose = function (ev) {
            $("#username").html("用户："+ username +"<span>离线</span>");
        }

        showChat = function(name){
            // alert("dsaad");
            toName = name;
            //清空聊天区
            $("#content").html("");
            $("#new").html("当前正与"+toName+"聊天");
            var chatdata = sessionStorage.getItem(toName);
            if (chatdata != null){
                $("#content").html(chatdata);
            }
        };
        //发送消息
        $("#submit").click(function () {
            //获取输入的内容
            var data = $("#input_text").val();
            $("#input_text").val("");
            var json = {"toName": toName ,"message": data};
            //将数据展示在聊天区
            var str = "<span id='mes_right'>"+ data +"</span></br>";
            $("#content").append(str);

            var chatdata = sessionStorage.getItem(toName);
            if (chatdata != null){
                str = chatdata + str;
            }
            sessionStorage.setItem(toName,str);
            //发送数据
            ws.send(JSON.stringify(json));
        })
```

##### 伪代码框架

***服务器端***

```java
public class GetHttpSessionConfigurator extends ServerEndpointConfig.Configurator {
    @Override
    public void modifyHandshake() {
		// 保存请求的信息，后续访问
    }
}

// 一个Endpoint == 一个客户端， value==url
// 服务终端 configurator关联自定义的配置器类，目的是让 WebSocket 会话能够访问存储在 HTTP 会话中的数据
@ServerEndpoint(value = "/chat", configurator = GetHttpSessionConfigurator.class)
@Component
public class ChatEndpoint {

    // 首次连接， 在HTTP握手连接时已经把<username, HttpSession>放入config中
    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        // 会话建立逻辑
    }

    // @@@ 核心 => 转发信息  chatEndpoint.session.getBasicRemote().sendText(message); @@@ 核心发送信息的代码
    @OnMessage
    public void onMessage(String message, Session session) {
        // 接受终端消息逻辑
    }

    @OnClose
    public void onClose(Session session) {
		// 会话关闭逻辑
    }
}
```

***客户端***

```js
var ws = new WebSocket("ws://localhost:8080/chat");

ws.onopen = function (e) {
    // 会话建立逻辑
}

ws.onmessage = function (ev) {
    // 消息接收逻辑
}

ws.onclose = function (ev) {
    // 会话关闭逻辑
}

// 主动发送数据
// ws.send(JSON.stringify(data));
```





---

### Nginx

```python
# 正向代理
Client            Server  
X1 \\
X2 == =>  T(VPN) => Y
X3 //

# 反向代理
                      //  Y1
X == =>  T(Nginx) =>  ==  Y2
                      \\  Y3
```

***nginx启停***

注意linux权限，若无权限，则加sudo

```xml
nginx -s ${signal}
quit: 停止
reload: 重新加载配置文件
```

***nginx 配置***

```xml
# 全局配置信息
worker_processes auto; # main进程 { 多个worker进程 }

events {
	worker_connections 1024;   # 网络连接数
}

http {
	...
	server{ ... }
	server{ ... }
}
```

```xml
server {
        listen 80 default_server;	# 监听ipv4-端口(80)  
        listen [::]:80 default_server;	# 监听ipv6-端口(80)

		root /var/www/html;		# 为请求提供文件的根目录
		index index.html index.htm index.nginx-debian.html;  # 定义了当请求为目录时，默认尝试返回的文件列表

		server_name _;
		location / {	# 对于根URL（即/）的请求的处理规则
                # 尝试按顺序找到对应的文件或目录，如果都找不到，则返回404错误（未找到）
                try_files $uri $uri/ =404;
        }   
```

***反向代理***

```xml
# nginx.config 中

http {
	...
	upstream backend {	# default weight=1, 默认进行轮询
		ip_hash; // 终端ip和服务器ip绑定到一起
		server ?.?.?.?:? weight=?;
		server ?.?.?.?:?;
		server ?.?.?.?:?;
	}
	
	server {
		...
		location /app {
			proxy_pass http://backend/;
		}
	}
	...
}
```

解释：⭐upstream ? { ... } 定义了上游服务器信息，ip_hash的作用就是让终端和某个服务器绑定(解决session问题)，weight参数是分发数量的权重，默认1，越大往这个服务器分发的任务越多。 ⭐server中添加映射 location /app 将访问nginx.ip:80/app的请求转发给 http ://backend/ 的服务，进行均衡负载。



***练习***

1️⃣ 0.0.0.0 表示本机127.0.0.1和ip-v4

2️⃣ 创建多个Flask服务，模拟多个后端服务器，实验均衡负载

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, 800?'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=800?)
    
# 终端运行 python .\main8000.py --port=8000
```



### MybatisPlus

*springboot application.yaml 配置*

```yaml
mybatis-plus:
  type-aliases-package: com/yoo/entity
  mapper-locations: classpath:/mapper/**.xml
  configuration:
    map-underscore-to-camel-case: true
```

***Mapper模板增强：SQL语句：查询字段、筛选条件  - SQL片段分离***

```java
@TableName("user")  // 类-绑定数据库-表
Class User{...}

// BaseMapper 内置大量基本的SQL模板
@Mapper
public interface UserMapper extends BaseMapper<User> {}

// example
QueryWrapper wrapper = new QueryWrapper();	// 条件包装器
wrapper.select("name, age, email");			// 显示指定查询字段
wrapper.ge("age", 23);						// 条件 greater than
List list = userMapper.selectList(wrapper);
```

***混合自定义SQL.xml 配置和模板***

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    Type methodID(...params);
    int UPDATEID(@Param(Constants.WRAPPER) LambdaQueryWrapper<User> wrapper, ...params);
}

// Associated XML file
<? id="methodID" resultType="Type">
    SQL Segment
</?>
    
// use Wrapper to build condition
<update id="UPDATEID">
    UPDATE `table_name`
    SET ...
	${ew.customSqlSegment}
</update>
```

***Service模板增强***

```java
// interface extends template abstract method
@Service
public interface IUserService extends IService<User> {}

// class extends template implement method
@Service
public class UserPOServiceImpl extends ServiceImpl<UserPOMapper, UserPO> implements IUserPOService {
    @Override
    public List<UserPO> queryUserByBalance(int minBalance, int maxBalance) {
        // lambdaQuery is method in ServiceImpl super class IService
        return lambdaQuery()
                .between(UserPO::getBalance, (double)minBalance, (double)maxBalance)
                .list();
    }
}
```

***knife4j 接口说明文档***

```java
@Api()
@RestController
@RequestMapping()
class Controller {
    
    @ApiOperation()
    @RequestMapping()
    public void method() {}
    
}

@ApiModel()
class Bean{
    @ApiModelProperty()
    private Type Field;
}
```

***批量操作***

```java
MySQL connection URL: add parameter rewriteBatchedStatements=true
    
userSerivice.saveBatch()
    
// MySQL开启批量操作，MybatisPlus再执行batch级的SQL
```

***⭐ 静态的SQL，避免Service中相互依赖***

```java
Db.lambdaQuery(AddressPO.class)
                .eq(AddressPO::getUserId, user.getId())
                .list();

// => 最终调用AddressPOMapper执行基本SQL操作

// AddressPO @TableName("address") => 关联数据库表，下划线转驼峰

// iService 和 ServiceImpl 封装了基本的SQL操作
```

***枚举***

```java
// application.yaml
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

```java
// 定义状态
@Getter
public enum UserStatus {
    NORMAL(1, "正常"),
    FREEZE(2, "冻结");

    @EnumValue	// 与数据库的转换
    private final int value;
    @JsonValue  // 响应结果的转换
    private final String desc;

    UserStatus(int value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}

// e.g.  Freeze user
lambdaUpdate()
    .eq(UserPO::getId, id)
    .set(UserPO::getStatus, UserStatus.FREEZE)
    .update();
```

⭐ ***分页***

```java
// 注入分页配置，添加分页拦截器
@Configuration
public class MyBatisConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();

        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setDbType(DbType.MYSQL);
        paginationInnerInterceptor.setMaxLimit(1000L);

        mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);

        return mybatisPlusInterceptor;
    }

}
```

```java
// 准备Page信息条件
Page<AddressPO> page = Page.of(pageNum, pageSize);
page.addOrder(OrderItem.desc("id"));  // 根据xxx降序/升序
// 执行查询
Page<AddressPO> addressPOPage = lambdaQuery().page(page);

return addressPOPage.getRecords();
```



---

***封装PageQuery和PageDTO***

```java
@Data
class PageQuery {
    // properity:
    pageSize, pageNum, sortBy, isAsc ...
    // method:
    public <PO> Page<PO> toMpPage(){
        // 1. 分页条件
        Page<PO> page = Page.of(pageNum, pageSize);
        // 2. 排序条件
        if (StrUtil.isNotBlank(sortBy)) {
            page.addOrder(new OrderItem().setColumn(sortBy).setAsc(isAsc));
        }
        return page;
    }
    
    ... // 多态性质
    public <PO> Page<PO> toMpPage(String sortBy, Boolean isAsc) {
        return toMpPage(new OrderItem().setColumn(sortBy).setAsc(isAsc));
    }
    ... // 默认排序...
}

@Data
class UserQuery extends PageQuery {
    // properity: 查询条件
    name, id, xxx
}

// use example
构建UserQuery, 初始化好查询参数 => 调用UserService.method(userQuery)
method(userQuery)  => userQuery.toMpPage() => Page<PO> p => lambdaQuery.condition().page(p) => getRecords()
分步构建：页面配置(复用) + 查询条件(继承额外加)        
```

```java
@Data
public class PageDTO<T> {

    private Long total;

    private Long pages;

    private List<T> list;
    
    // 泛型不指定参数类型， PO=>VO转换器传入 
	public static <PO, VO> PageDTO<VO> of (Page<PO> p, Class<VO> clazz, Function<PO, VO> converter) {
        PageDTO<VO> voPageDTO = new PageDTO<>();
        // 基本信息
        voPageDTO.setTotal(p.getTotal());
        voPageDTO.setPages(p.getPages());

        List<PO> poList = p.getRecords();
        if (CollectionUtils.isEmpty(poList)) {
            voPageDTO.setList(Collections.emptyList());
            return voPageDTO;
        }

        // 拷贝数据
        List<VO> list = poList.stream().map(converter).collect(Collectors.toList());
        voPageDTO.setList(list);

        return voPageDTO;
    }
```



---

### Spring Cloud

<u>*父 pom*</u>

```java
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <org.projectlombok.version>1.18.20</org.projectlombok.version>
    <spring-cloud.version>2021.0.3</spring-cloud.version>
    <spring-cloud-alibaba.version>2021.0.4.0</spring-cloud-alibaba.version>
    <mybatis-plus.version>3.4.3</mybatis-plus.version>
    <hutool.version>5.8.25</hutool.version>
    <mysql.version>8.0.23</mysql.version>
    <knife4j.version>4.3.0</knife4j.version>
</properties>
    
<!-- 对依赖包进行管理 -->
<dependencyManagement>
    <dependencies>
    	<dependency>
    	...
        </dependency>
    </dependencies>
</dependencyManagement>
```

<u>*子项目 pom*</u>

```java
<parent>
    <artifactId>super artifactId</artifactId>
    <groupId>com.yoo</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

***统一dependency version***



**垂直划分项目**

#### User-Service

*nacos托管微服务IP地址*

```java
java:    
	com:
        yoo:
            controller: SpringMVC
            service: IService + ServiceImpl   
            mapper: BaseMapper
            client: @Openfeign, remote call
            domain:
                   po: properity object reflex database table
                   vo: value object, only show required properity

----------------------------------                       
                       
resource:
	application.yaml:
		server:
  			port: 

        spring:
          application:
            name: user-service # service-name
          profiles:
            active: dev
          datasource:
            url: 
            driver-class-name: 
            username: 
            password: 
		  # ⭐ cloud service ip address
          cloud:
            nacos:
              server-addr: 127.0.0.1

    mybatis-plus:
      configuration:
		# 枚举字段类型映射到数据库表指定类型， 下划线转驼峰
        default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
      global-config:
        db-config:
          update-strategy: not_null
          id-type: auto

    knife4j:
      enable: true
      openapi:
        title: 用户服务接口文档
        description: "信息"
        email: 1410124534@qq.com
        concat: longwei
        group:
          default:
            group-name: default
            api-rule: package
            api-rule-resources:
              - com.yoo.controller
```

##### ***OpenFeign 远程调用微服务***

*Step 1.*  *<u>pom添加对应依赖</u>*

```java
<!-- openfeign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
    
<!-- loadbalancer -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

*Step 2.* *<u>开启Openfeign功能</u>*

```java
@EnableFeignClients
...
public class xxxApplicaiotn {
    ...
}
```

*Step 3.  <u>编写Openfeign调用接口</u>*

```java
@FeignClient(value = "order-service")
public interface OrderClient {

    @GetMapping("/order/{id}")
    List<OrderVO> getOrderByUserId(@PathVariable("id") Long userId);  

}

// @Interface : @XXXMapping("/{param}")     <===
//                                             ||
// Method     : (@PathVariable("param") Type name)
```



---

#### Gateway setup

***前端统一请求 =>8080网关 => 分发调度给微服务***   *类似nginx*

*dependencies*

```xml
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos discovery-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--负载均衡-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



⭐⭐⭐*application.yaml*  配置转发接口映射

```yaml
server:
  port: 8080

spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      server-addr: 127.0.0.1
    gateway:
      routes:  ### 路由
        - id: item
          uri: lb://user-service
          predicates:
            - Path=/user/**

        - id: order
          uri: lb://order-service
          predicates:
            - Path=/order/**
```

*StarterClass*

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```



---

#### 网关登录统一拦截

*好比于守卫，统一校验登录Token，进行放行或者拦截*

*pom 依赖*    *支持yaml读取配置*

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

*application.yaml*

```yaml
hm:
  auth:
    excludePaths: # 无需登录校验的路径
      - /user/login
      - /item/search
      - /test/**
```

*属性类*

```java
@Data
@Component
@ConfigurationProperties(prefix = "hm.auth")
public class AuthProperties {
    private List<String> includePaths;
    private List<String> excludePaths;
}
```

*GlobalFilter* *全局拦截*  

先根据配置文件，决定是否拦截，拦=>检查Token=>存入信息在Header => 微服务(从Header中拿取信息)

```java
@Component
@RequiredArgsConstructor
public class AuthGlobalFilter implements GlobalFilter, Ordered {
    // 已加入Spring容器
    private final AuthProperties authProperties;
    // 未加入，手动new
    private final AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取request对象
        ServerHttpRequest request = exchange.getRequest();
        // 判断路径是否需要拦截
        if(isExclude(request.getPath().toString())) {
            // 放行
            return chain.filter(exchange);
        }
        // 获取Header中的Token
        String token = null;
        List<String> headers = request.getHeaders().get("authorization");
        if(!CollUtil.isEmpty(headers)) {
            token = headers.get(0);
        }
        Long userId = null;
        // 简单模拟JwtToken校验
        if(token != null && token.equals("hello")) {
            // 校验成功
            userId = 1L;
        } else {
            // 校验失败 => 直接返回UNAUTHORIZED状态码
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }

        String userInfo = userId.toString();

        // 将用户信息存储在转发请求头中
        ServerWebExchange ex = exchange.mutate()
                .request(builder -> {
                    builder.header("user-info", userInfo);
                })
                .build();

        return chain.filter(ex);
    }

    private boolean isExclude(String path) {
        for (String pathPattern : authProperties.getExcludePaths()) {
            if (antPathMatcher.match(pathPattern, path)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```



---

#### Service Interceptor

ThreadLocal，线程自己的独享空间，用来存储请求中携带的信息

```java
public class UserContext {

    private static final  ThreadLocal<Long> tl = new ThreadLocal<>();

    public static void setUserId(Long userId) {
        tl.set(userId);
    }

    public static Long getUserId() {
        return tl.get();
    }

    public static void removeUserId(){
        tl.remove();
    }

}
```

执行服务前，将request header中携带的信息取出来

```java
public class UserInfoInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String userId = request.getHeader("user-info");
        if (StrUtil.isNotBlank(userId)) {
            UserContext.setUserId(Long.parseLong(userId));
        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        UserContext.removeUserId();
    }
}
```



---

#### 抽取公共部分

#### common-service

```java
com.yoo.common.xxx
```

其他module只要在pom内引用就行



公共部分如何区别不同的服务 - 有些子模块需要但另一些子模块不需要的Class（这个类会被Spring托管，但不想注入Spring容器中）

```java
@ConditionalOnClass(DispatherServlet.class)
```



在其他微服务加上，显示指定配置文件位置

**spring.factories**

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.yoo.common.config.MvcConfig
```



---

### RabbitMQ

```java
publisher => exchanger => queue => consumer
```

exchanger: 

- fanout exchanger:  广播
- direct exchanger：根据key发送给对应queue
- topic exchanger: 根据key进行匹配一组符合的queue



消息订阅

```java
@RabbitListener(queues = "???")
public T method(...) {
    // 
}
```

消息发布

```java
# 直接发送给Queue
rabbitTemplate.convertAndSend(queueName, message)
    
# 发送给交换机
rabbitTemplate.convertAndSend(ExchangeName, "RoutingKey", MessageObject)    
```



消息格式转换

Object => Binary  将发送的对象序列化为JSON格式字符串？ 

```java
@Bean
...
return new Jackson2JSONMessageConverter()
```



队列交换机和绑定

```java
@Bean
public Queue orderQueue() {
    return new Queue("orderQueue", true);
}

// Spring 容器会确保同一个 @Bean 方法只会被调用一次，返回的对象会被缓存并重复使用
@Bean
public DirectExchange orderExchange() {
    return new DirectExchange("orderExchange", true, false);
}

@Bean
public Binding binding() {
    return BindingBuilder.bind(orderQueue()).to(orderExchange()).with("orderRoutingKey");
}
```





---

可靠传输

```yaml
# 发送确认
publisher-confirm-type: none|correlated|simple

none: 无确认机制，性能最好，不可靠
correlated: 异步关联确认,不阻塞线程,高可靠性,性能中、异步回调场景
simple: 同步阻塞确认,阻塞线程,性能底，可靠。

# 接收确认
ackknowledge-mode: none|manual|auto
none: 无确认机制
manual: 手工ack
auto: 自动ack，根据抛出的异常类型，决定重发还是reject


# 重连配置
retry:
	enabled: true  # 启用重试机制
	initial-interval: 1000ms # 第一次失败等待的时长
	multiplier: 1 # 每次失败等待时间成x倍增长
	max-attempts: 3 # 最大重试次数
	stateless: false # 
	
# 如果接口幂等(调用多次都不会重复扣款) => stateless:true 每次重试都是独立的，不保留状态 ⭐性能高
# 非幂等操作（多次执行可能有副作用）
stateless:false # 带状态
```



**死信交换机**

![image-20250113195557165](http://verification.longcoding.top/Fpey0H8HmDHLDSYiTZer2GyQEx8P)

定义交换机和队列

```java
@Bean
public DirectExchange createDLExchanger() {
    return ExchangeBuilder.directExchange("dl.direct").build();
}

@Bean
public Queue createDLQueue() {
    // 队列信息是否持久化到磁盘? durable(持久化?)
    return QueueBuilder.nonDurable("dl.queue").build();
}

@Bean
public Binding toBindingDL() {
    return BindingBuilder.bind(createDLQueue()).to(createDLExchanger()).with("dlRoutingKey");
}

// -----------------------

@Bean
public FanoutExchange createTTLExchanger() {
    return ExchangeBuilder.fanoutExchange("ttl.fanout").build();
}

@Bean
public Queue createTTLQueue() {
    // 队列信息是否持久化到磁盘? durable(持久化?)
    return QueueBuilder.nonDurable("ttl.queue").deadLetterExchange("dl.direct").deadLetterRoutingKey("dlRoutingKey").build();
}

@Bean
public Binding toBinding() {
    return BindingBuilder.bind(createTTLQueue()).to(createTTLExchanger());
}
```

步骤1：发送信息并设置过期时间

```java
// 1:未付款; 2.已付款 ...
// 模拟用户下单
redisTemplate.opsForValue().set("order:userId:status", "1");
// ... 扣货物库存  -1

MessageProperties properties = new MessageProperties();
properties.setExpiration("20000"); // 20秒过期，过期后进入死信队列

Map<String, String> map = new HashMap<>();
map.put("code", "200");
map.put("msg", "Hi, RabbitMQ. This is a dead letter.");

ObjectMapper mapper = new ObjectMapper();

String msgContent = mapper.writeValueAsString(map);
Message message = new Message(msgContent.getBytes(), properties);
rabbitTemplate.convertAndSend("ttl.fanout", "", message);
```

步骤2：用户调用接口进行付款

```java
// 数据库
// ....

// 模拟付款成功
redisTemplate.opsForValue().set("order:userId:status", "2");
```

步骤3：死信队列 处理逻辑

```java
@RabbitListener(queues = "dl.queue")
public void deadLetterQueue(Message message) throws IOException {
    ObjectMapper mapper = new ObjectMapper();
    Map<String, String> msg = mapper.readValue(message.getBody(), Map.class);

    System.out.println("receive Msg ... @@@");
    System.out.println(msg.get("code"));
    System.out.println(msg.get("msg"));

    String status = redisTemplate.opsForValue().get("order:userId:status");
    if (status.equals("1")) {
        // ... 还原货物库存  +1
        return;
    }else if (status.equals("2")){
        // 订单支付完成，实现收尾工作
        System.out.println("订单已支付，生成后续...");
    }
}
```

解耦各个部分。



---

### 苍穹外卖

#### AOP使用-自动补全参数共同信息

```java
// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoFill {
    OperationType value();
}
```

```java
@Aspect  // !!! 交给Spring托管
@Component
public class AutoFillAspect {

    // 任何 returnType package.?.class.method(params)  && with annotation
    @Pointcut("execution(* com.hmwm.service.*.*(..)) && @annotation(com.hmwm.common.AutoFill)")
    public void autoFillPointCut(){}

    @Before("autoFillPointCut()")
    public void autoFill(JoinPoint joinPoint) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        System.err.println("Start fill common field.");

        // get method annotation value
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        AutoFill annotation = signature.getMethod().getAnnotation(AutoFill.class);
        OperationType operationType = annotation.value();

        // 拦截的method的参数列表
        Object[] args = joinPoint.getArgs();
        if (args==null && args.length==0) {
            System.out.println("current method not params");
            return;
        }
        // 第一个⭐参数⭐ 需要填充的对象. 提前交流指定:这里为User Obj
        Object arg = args[0];

        LocalDateTime now = LocalDateTime.now();
        // Long currentId = BaseContext.getCurrentId();

        // 给这个arg填充值
        if (operationType == OperationType.INSERT) {
            // getDeclaredMethod(method_name, params_type)
            Method setCreateTime = arg.getClass().getDeclaredMethod(AutoFillConstant.SET_CREATE_TIME.getMethodName(), Date.class);
            Method setUpdateTime = arg.getClass().getDeclaredMethod(AutoFillConstant.SET_UPDATE_TIME.getMethodName(), Date.class);

            // method-obj: invoke(object, params) 方法(对象, 参数)
            setCreateTime.invoke(arg, Date.from(now.atZone(ZoneId.systemDefault()).toInstant()));
            setUpdateTime.invoke(arg, Date.from(now.atZone(ZoneId.systemDefault()).toInstant()));
        }

    }
```

```java
@AutoFill(OperationType.INSERT)
public String addUser(User user) {
    return user.toString();
}
```

**枚举类**

```java
// 枚举类在 Java 中是通过 enum 关键字定义的，但它本质上是一个类。每个枚举常量都是该枚举类的一个实例。
public enum AutoFillConstant {
    SET_CREATE_TIME("setCreateTime"),
    SET_UPDATE_TIME("setUpdateTime");

    private String methodName; // 存储方法名的字段

    // 构造函数
    AutoFillConstant(String methodName) {
        this.methodName = methodName;
        System.out.println("AutoFillConstant: " + methodName);
    }

    // 获取方法名
    public String getMethodName() {
        return methodName;
    }
}
```

---

#### 定时任务

利用@Scheduled注解

```java
@Component
public class ScheduledTask {
    // 秒 分 时 天 周 月 年                     年/月/日  时/分/秒
    @Scheduled(cron = "0 * * * * ? ") // 这里 xxxx/xx/xx xx:xx:x0 执行
    public void processTimeOutOrder() {
        System.err.println("process Timeout task ...");
    }

}
```

#### ThreadLocal-线程专享资源

可以请求来的时候存储一些 环境变量  ThreadLocal\<Object>

```java
public class BaseContext {
    public static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public static void setCurrentId(Long id) {
        threadLocal.set(id);
    }

    public static Long getCurrentId() {
        return threadLocal.get();
    }

    public static void removeCurrentId() {
        threadLocal.remove();
    }
}
```

#### Websocket

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

#### 文件下载功能

很简单，利用Response进行。 需要设置response内容格式和response-header

```java
ClassPathResource resource = new ClassPathResource("static/file_name.zip");
if (!resource.exists()) {
    System.err.println("file not exists!");
    return ResponseEntity.notFound().build();
}

HttpHeaders headers = new HttpHeaders();
headers.add(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"file_name.zip\""); // attachment 附件

return ResponseEntity.ok()
    .headers(headers)  // <= header
    .contentType(MediaType.APPLICATION_OCTET_STREAM) // <= contentType stream流式
    .body(resource); // <= file
```

#### Excel处理

```java
org.apache.poi // 利用这个Apache POI package
```

```java
XSSFWorkbook excel = new XSSFWorkbook(); // 类似excel文件
XSSFSheet sheet = excel.createSheet("sheet1"); // sheet

XSSFRow row1 = sheet.createRow(0);
row1.createCell(1).setCellValue("姓名");
row1.createCell(2).setCellValue("年纪");
row1.createCell(3).setCellValue("城市");

XSSFRow row2 = sheet.createRow(1);
row2.createCell(1).setCellValue("韦龙");
row2.createCell(2).setCellValue(23);
row2.createCell(3).setCellValue("桂林");

XSSFRow row3 = sheet.createRow(2);
row3.createCell(1).setCellValue("张三");
row3.createCell(2).setCellValue(23);
row3.createCell(3).setCellValue("杭州");

File file = new File("static/staff_info.xlsx");
FileOutputStream stream = new FileOutputStream(file);
excel.write(stream);

stream.close();
excel.close();
```







