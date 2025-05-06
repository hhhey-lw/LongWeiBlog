---
author: "LongWei"
title: "Java面试题笔记"
date: "2025-02-19"
tags: ["Java"]
categories: ["Jobs"]
series: ["JobLearning"]
ShowToc: true
TocOpen: true
---

### 415 序列化&反序列化

将Object转为字节流，或反之

- 普通：实现serializable接口 ˈsɪərɪəlaɪzəbl
- 使用Jackon，Obj => json格式



### 416 Java中的不可变类

final 修饰，例如String类

🎉优点：

- 线程安全
- 缓存友好

缺点

- 性能问题，因为不能修改，所有每次状态变化，都需要生成新的对象。



### 411 多态特性

- 继承
- 方法重载，函数名相同，但是函数签名需要有差异(参数类型&数量)
- 重写，子类重写父类方法，通过父类调用方法时，调用的是子类重写后的函数。



### 412 Java参数传值是副本还是引用呢？

- 基本类型是传**值副本**，int...
- 引用数据类型是传**引用副本**。 including：obj，array



### 425 Java中 包装类型和基本类型

🏷️基本类型 => int long float double  ...  位于栈上(局部变量的话) ，性能好，但**不支持为null**

（局部变量在栈上，成员变量在堆上，静态字段在方法区）

🏷️包装类型 => 每一个基本类型都对应一个包装类型。**包装类型是类，在堆中，支持null**



**JVM内存模型**  ❗❗❗内存堆和数据结构堆不是同一个东西(不是堆的结构)

```java
+--------------------------------------------+
|               方法区（Method Area）        | ← 线程共享
|  - 类元数据（Class 信息）                   |
|  - 静态变量（static 变量）                  |
|  - 常量池（字符串常量等）                   |
+--------------------------------------------+
        ↑              ↑
        |              |
+----------------+    +-----------------+
|  栈（Stack）   |    |   堆（Heap）     | ← 线程共享
|  - 局部变量    |    |  - Java 对象实例 |
|  - 方法调用栈  |    |  - 数组         |
+----------------+    +-----------------+
```



### 413 interface & abstract class

- interface(自上而下) 知晓某一种行为，基于这些行为约束定义的接口， 一些类需要有这些行为的话，需要实现这些接口
- abstract class(自下而上): 有许多类，它们有共同点，很多代码可以复用，因此将公共逻辑封装为抽象对象。



### 100 hashCode & equals & == 

- hashCode用于散列表(hashMap)用于计算hash值，从而计算存储位置；

- equals比较对象内容是否相等，默认是==。 可能需要重写equals方法逻辑，实现对象成员变量的比较；
- == 引用类型比较两个引用是否指向同一个对象(内存地址)，基本类型则比较值。



### 431 Java注解

注解就是一个标记，提供元数据机制，给予代码添加说明信息。可以在类，方法，成员变量...上标记，标记本身可以设置一些值。

```java
 @Target(Class,...)
@Retention(RetentionPolicy.RUNTIME) // 注解保留时间
public @interface MyInterface {
    String value(); 
}

// 运行时，通过反射机制拿到对应的注解 -- 结合AOP做事情
...Obj.getAnnotation("MyInterface")...
```



### 432 Java反射

运行时获取类的信息，并操作对象

```java
Person person = new Person("Emma", 28);
Class<?> cls = person.getClass();

// 调用无参方法
Method greetMethod = cls.getDeclaredMethod("greet");
greetMethod.invoke(person);

// 调用有参方法
Method setAgeMethod = cls.getDeclaredMethod("setAge", int.class);
setAgeMethod.invoke(person, 35);

System.out.println(person); // Person{name='Emma', age=35}
```



### 434 Java泛型

通过在编译时检查类型安全，使得代码更加通用和灵活，避免运行时发生类型转换错误。 简单理解=> 将运行时的类型转换异常上升到编译时候

- 类型安全
- 代码重用
- 消除显示类型转换，一开始指定好



🏷️泛型类

```java
public class Box<T> {
    private T value;
    public Box(T value) { this.value = value; }
    public T getValue() { return value; }
}
// 这样的话  getValue()就不需要(Type)getValue()了， 直接告诉调用者返回类型  => 避免类型转换,且提高代码复用性
```

🏷️泛型方法

```java
// 泛型方法
public static <T> T getFirst(T[] array) {
    return array[0];
}

// <T> 是申明T是泛型哦， 不然谁知道它是不是一个类的名称
// 🎉更加灵活
String[] words = {"Hello", "World"};
Integer[] numbers = {1, 2, 3};

System.out.println(getFirst(words));  // Hello
System.out.println(getFirst(numbers)); // 1
```

🏷️限制参数类型 防止非法类型使用

```java
class Calculator<T extends Number>
```



### 6306 Java泛型擦除

Java编译器在编译代码的时候，将所有泛型信息删除的过程。 擦除是为了确保与旧版本兼容

```java
Class Box<T> {}      =>      class Box {}  (T => Object)
Class Box<T extends Number> {}      =>      class Box {}  (T => Number)
```



泛型类型上界  ? extends T

泛型类型下届  ? super T

限制类型



---

### 436 Java深拷贝&浅拷贝

- 深拷贝：不仅复制当前对象本身，而且递归的复制对象内的引用。 重写clone方法，或许序列化和反序列化生成新的对象；
- 浅拷贝：只复制对象的本身和基本类型的成员变量，而不复制引用类型的对象。 **Object.clone()**；
- 引用拷贝：仅复制对应引用  **Object o1 = o2; ** 。

<img src="http://verification.longcoding.top/FpC7CiMvtO7t2ecq3_e95NzCHKx1" alt="image-20250310152342480" style="zoom:50%;" />



### 438 Java类加载过程

类加载过程：

- 加载：二进制流读入内存，生成Class对象
- 连接
  - 验证：格式是否规范
  - 准备：为静态变量设置初始值，为它们在方法区开辟。
  - 解析：将常量池的符号引用转化为直接引用。
- 初始化：执行静态代码块，为静态变量赋值(代码逻辑里面的)。



### 440 BigDecimal

高精度的计算类，处理任意精度的数值。  涉及到钱，用这个



### 442 Java中的final、finally

final修饰符，指类和变量不能再修改。

```java
class final MyClass {
    
}
```

不论是否异常，都会执行finally代码块

```java
try{

}catch() {

} finally {

}
```



### 825 Java调用外部可执行程序

使用Runtime.exec()

```java
try {
    // 执行命令
    Process process = Runtime.getRuntime().exec("notepad.exe");
    // 等待外部进程结束
    int exitCode = process.waitFor();
    // 检查执行情况
    if ... else ...
}
```

使用ProcessBuilder

```java
// 配置程序信息
ProcessBuilder builder = new ProcessBuilder("cmd.exe", "/c", "dir");
// builder.xxx() ... 配置信息

// 启动进程
Process process = builder.start();

// 获取进程输出
InputStream is = process.getInputStream();
// ... 

// 等待结束
int code = process.waitFor()
    
// 检查执行情况    
```



### 938 如果一个线程在Java中被两次调用start方法，会发生什么？

报错 => Java中，一个线程只能被启动一次。



### 943 IO流

读取数据和输出数据

- 字节流，用于处理二进制文件, InputStream&OutputStream;
  - InputStream 子类如下：
    - FileInputStream
    - BufferedInputStream // 提供缓冲区，提高性能
- 字符流，用于处理文本。 Reader&Writer；
  - Reader 子类如下：
    - FileReader
    - BufferedReader



### 945 Java网络编程

🏷️用于网络通信，网络编程基本概念

- IP地址
- 端口号
- socket(ip:port)
- 协议，TCP & UDP 等等



TCP实践

```java
// 服务器代码
class ServerThread extends Thread {
    private Socket socket = new ServerSocket("localhost", 8080);
    
    @overwrite
    public void run() {
        try {
            PrintWriter out = new PrintWriter(socket.getOutputStream() ... );
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            
            String message = in.readLine();
            out.println("Hello, client!");
        }
    }
}

// Client
Socket socket = new Socket("localhost", 8080)
    
// ServerSocket & Socket    
```



### 949 Java中的自动装箱和拆箱

**自动装箱/拆箱让 Java 更加易用，减少了手动转换的冗余代码，同时保持了类型安全** 

- 自动拆箱和装箱 ,  解决泛型不支持基本类型

```java
List<Integer> list = new ArrayList<>();  // ✅ 使用包装类 Integer

// 手动装箱和拆箱
list.add(Integer.valueOf(10)); // int => Integer
int a = list.get(0).intValue(); // Integer => int 手动实现

// 自动装箱和拆箱
list.add(10)
int a = list.get(0)    
```

- 基本类型当作对象使用

```java
Map<Integer,String> map = new HashMap<>();
map.put(1, "One") <= 自动装箱
String value = map.get(1) <= 自动装箱    
```

- 简化基本类型和包装类型

```java
public static int add(Integer a, Integer b) {
    return a + b;  // 这里的 a 和 b 需要自动拆箱
}

// 手动拆箱
public static int add(Integer a, Integer b) {
    return a.intValue() + b.intValue();  // 这里的 a 和 b 需要自动拆箱
}

```



### 988 Java中的迭代器 Iterator ɪtəˈreɪtə

作用用于遍历集合

- hasNext(): 返回是否存在下一个元素
- next(): 返回下一个元素

```java
Iterator iterator = list.iterator()
while(iterator.hasNext()) {
    // ...
    iterator.next();
}    
```



### 993 Java特性

- 封装：将对象的状态和行为封装在一个类内部，并通过公开的接口与外部进行交互。只暴露必要功能。
- 继承：子类继承父类，可以继承它的属性和方法，提高代码重用和扩展。

- 多态：重写&重载



### 994 Java中访问修饰符

- public：All
- private：当前类
- protected：当前类，同一包(package声明一致)，子类
- default，当前类，同一包



### 995 静态方法&实例方法

- 静态方法 static声明：属于类而非具体的实例，通过类名调用。
  - 工厂方法，工具类
  - 当类的字节码文件加载到内存，类方法的入口地址就会被分配完成，所以类方法不仅仅被该类的对象调用，也可以直接通过类名称完成调用，类方法的入口地址只有程序退出才消失。

- 实例方法：属于实例，通过实例调用；
  - 当类的字节码加载到内存中的时候，类的实例方法并没有被分配到入口地址，只有当类的对象创建之后，实例方法才分配了入口地址，从而实例方法可以被类创建的所有的对象所调用，还有一点要注意，当我们创建第一个类的对象时，实例方法的入口地址会完成分配，当后续在创建对象时，不会被分配新的入口地址，该类的所有的对象共享实例方法的入口地址，当该类的所有的对象被销毁，入口的地址才会消失。



### 990 java中for和for-each

- for，下标遍历
- for-each，遍历集合，不提供下标，且不能修改集合，否则报错



### 5900 wait()和sleep()

wait() 需要和 notify()搭配使用。 用于在同步代码块同，阻塞和唤醒



sleep() 使线程进行休眠状态，让出CPU使用权，时间到了自动恢复为就绪态等待系统分配CPU



### 5908 Java Object类有什么方法及对应作用

- hashCode() 计算哈希值
- equals() 默认引用是否一致
  - 根据需求重写这个，实现指定对象属性比较

- toString()  默认返回对象的类名+hashCode的16进制表示
  - 重写使其更有描述意义。

- getClass() 返回对象的Class类对象

- wait() 挂起当前线程，使其变为等待状态。
- notify&notifyAll() 唤醒在对象监视器上等待的一个or全部线程

- clone() 浅拷贝对象，其中引用属性不拷贝
  - 重写实现深拷贝，保证完整性



---

### 5909 Java字节码是什么？❌

处于 源代码.java 和 JVM.bin 执行的机器代码.exe(windows)之间的中间表示。

.class 文件 可以被JVM解释器编译为机器码



🎉通过Java反射Api，可以在运行时动态生成或者修改字节码，从而创建代理对象或实现动态方法调用



---

### 166 BIO、NIO、AIO

BIO:人一直盯着水烧开，水烧开之后亲自关火
NIO:人在烧水的时候去干别的事情，时不时看着水烧没烧开，烧开之后亲自关火
AIO:人找了一个帮手，帮手在烧水的时候一直盯着，水烧开之后帮手关火，然后提醒人水烧开了。人全程不管烧水的事情



- BIO：Blocking IO。传统阻塞式IO模式，调用方调用BIO会被阻塞，直到IO服务完成才被唤醒。
  - 场景：同步、阻塞，适合并发连接较少的场景，小型服务。

```java
// 客户端
// 1. 创建服务器 socket，监听端口
serverSocket = 创建 ServerSocket(端口);

// 2. 服务器循环等待客户端连接
while (true) {
    // 3. 阻塞等待客户端连接
    socket = serverSocket.accept(); ⭐这里会卡住

    // 4. 每个连接创建新线程
    启动新线程(() -> {
        inputStream = socket.getInputStream();
        outputStream = socket.getOutputStream();
        
        while (true) {
            // 5. 读取数据（阻塞）
            data = 读取数据(inputStream);
            if (data == null) break;
            
            // 6. 处理数据
            处理数据(data);
            
            // 7. 响应客户端（阻塞）
            outputStream.write(响应数据);
        }
        // 8. 关闭连接
        socket.close();
    });
}


// 客户端
// 1. 创建 Socket 连接服务器
socket = 创建 Socket(服务器IP, 端口);

// 2. 获取输入输出流
inputStream = socket.getInputStream();
outputStream = socket.getOutputStream();

// 3. 发送数据（阻塞）
outputStream.write(请求数据);
outputStream.flush();

// 4. 读取服务器响应（阻塞）
response = 读取数据(inputStream);

// 5. 处理响应
处理数据(response);

// 6. 关闭连接
socket.close();
```

一连接一线程			

- NIO：Non-blocking IO。非阻塞IO模式，调用方发起IO操作即返回，不论任务是否完成。通常结合IO多路复用技术，使得一个线程可以同时管理多个连接。

```java
// 1. 创建 Selector（事件选择器）
selector = 创建 Selector();

// 2. 创建非阻塞 ServerSocketChannel，绑定端口
serverChannel = 创建 ⭐ServerSocketChannel(端口)⭐;
serverChannel.configureBlocking(false);

// 3. 注册到 Selector，监听 "accept" 事件
serverChannel.register(selector, OP_ACCEPT);

// 4. 服务器循环监听事件
while (true) {
    // 5. 轮询监听已准备好的 I/O 事件， 监视多个Channel
    selector.select();

    // 6. 遍历所有发生事件的通道
    for (key : selector.selectedKeys()) {
        if (key.isAcceptable()) {  // 7. 处理新连接
            socketChannel = serverChannel.accept();
            socketChannel.configureBlocking(false);
            socketChannel.register(selector, OP_READ | OP_WRITE);
        } 
        else if (key.isReadable()) {  // 8. 读取数据
            socketChannel = key.channel();
            buffer = 创建 ByteBuffer();
            bytesRead = socketChannel.read(buffer);
            if (bytesRead > 0) {
                处理数据(buffer);
            }
        }
        else if (key.isWritable()) {  // 9. 响应客户端
            socketChannel = key.channel();
            buffer = 创建 ByteBuffer(响应数据);
            socketChannel.write(buffer);
        }
    }
}


// 客户端
// 1. 创建 Selector（事件管理器）
selector = 创建 Selector();

// 2. 创建非阻塞 SocketChannel
socketChannel = 创建 ⭐SocketChannel()⭐;
socketChannel.configureBlocking(false);

// 3. 连接服务器（非阻塞）
socketChannel.connect(服务器IP, 端口);

// 4. 注册到 Selector，监听 "连接完成" 事件
socketChannel.register(selector, OP_CONNECT | OP_READ | OP_WRITE);

// 5. 轮询 Selector 监听事件
while (true) {
    selector.select(); // 轮询等待事件
    for (key : selector.selectedKeys()) {
        if (key.isConnectable()) { // 6. 处理连接完成事件
            if (socketChannel.finishConnect()) {
                socketChannel.register(selector, OP_READ | OP_WRITE);
            }
        } 
        else if (key.isWritable()) { // 7. 发送数据
            buffer = 创建 ByteBuffer(请求数据);
            socketChannel.write(buffer);
        } 
        else if (key.isReadable()) { // 8. 读取服务器响应
            buffer = 创建 ByteBuffer();
            bytesRead = socketChannel.read(buffer);
            if (bytesRead > 0) {
                处理数据(buffer);
            }
        }
    }
}  
```

​	

- AIO：Asynchronous IO 异步IO
  - 利用回调or通知的方式告知调用方

```java
// 1. 创建异步 ServerSocketChannel
serverChannel = 创建 AsynchronousServerSocketChannel(端口);
serverChannel.bind(端口);

// 2. 监听客户端连接（异步回调）
serverChannel.accept(null, new CompletionHandler<>() {
    // 3. 处理新连接
    completed(socketChannel, attachment) {
        // 4. 继续监听新的客户端
        serverChannel.accept(null, this);

        // 5. 读取数据（异步回调）
        buffer = 创建 ByteBuffer();
        socketChannel.read(buffer, buffer, new CompletionHandler<>() {
            // 6. 读取完成，处理数据
            completed(bytesRead, buffer) {
                if (bytesRead > 0) {
                    处理数据(buffer);
                    
                    // 7. 响应客户端（异步回调）
                    socketChannel.write(ByteBuffer.wrap(响应数据), null, new CompletionHandler<>() {
                        completed(bytesWritten, attachment) {
                            // 响应完成
                        }
                    });
                }
            }
        });
    }
});

// 客户端
// 1. 创建异步 SocketChannel
socketChannel = 创建 AsynchronousSocketChannel();

// 2. 连接服务器（异步回调）
socketChannel.connect(服务器IP, 端口, null, new CompletionHandler<>() {
    // 3. 连接成功回调
    completed(attachment) {
        // 4. 发送数据（异步）
        buffer = 创建 ByteBuffer(请求数据);
        socketChannel.write(buffer, buffer, new CompletionHandler<>() {
            completed(bytesWritten, buffer) {
                // 5. 继续读取服务器响应（异步）
                buffer.clear();
                socketChannel.read(buffer, buffer, new CompletionHandler<>() {
                    completed(bytesRead, buffer) {
                        处理数据(buffer);
                    }
                });
            }
        });
    }
});
```



### 439 Java双亲委派模型 ❌❌❌

Java类加载机制的设计模式之一。 核心思想：类加载器在加载某个类时，先委派父类去加载，父类无法加载时，才由当前类加载器加载。 自定义ClassLoader继承系统的&重写findClass即可

GPT: **双亲委派模型（Parent Delegation Model）** 是 Java **类加载机制** 的一个规则。
 **当一个类加载器要加载某个类时，先让父类加载器尝试加载，只有当父类加载器无法加载该类时，才由自身加载**



```java
【Bootstrap ClassLoader】（引导类加载器）
    ↑         加载-1，不行的话到2
【ExtClassLoader】（扩展类加载器）
    ↑         加载-2，不行的话到3
【AppClassLoader】（应用类加载器）
    ↑         加载-3，不行的话到4
【自定义 ClassLoader】（用户自定义加载器）  加载-4  
    
// 倒着的U字形
```



它解决的问题

- 避免类重复加载，如果没有该机制 => 每个类加载器都可以**自行加载同一个类**，可能导致 **ClassCastException**（类转换异常）

```java
ClassLoader cl1 = new MyClassLoader(); // 自定义类加载器
ClassLoader cl2 = new MyClassLoader(); 

Class<?> class1 = cl1.loadClass("com.example.MyClass");
Class<?> class2 = cl2.loadClass("com.example.MyClass");

// class1 和 class2 虽然名字相同，但是两个不同的类
Object obj = class1.newInstance();
MyClass myObj = (MyClass) obj; // 可能抛出 ClassCastException
// 每个类加载器都有自己独立的类加载空间。如果你使用多个类加载器加载同一个类，它们会认为这是两个不同的类，尽管类的名字和内容完全相同。
```

- 保证Java核心类的安全

因为委托父类加载

例如

```java
package java.lang;
public class String {
    public static void hack() {
        System.out.println("Java 被黑了！");
    }
}
```

如果没有双亲委派，那么加载的是自定义的类，可能导致安全漏洞。



