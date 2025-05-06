请作为面试官的角度，回答。场景 + 方法 or 问题 + 措施



![image-20250428184345824](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428184345824.png)

**学习要点**

- 编写的Java文件是怎么运行的？
  - JVM - 类加载
- JVM的内存是如何管理的呢？分配和回收？
  - JVM 内存结构
  - JMM相关
  - JVM内存分配
  - JVM垃圾回收
- 如何结合实际的机器调整JVM参数，最佳实践呢？



### JVM基础 - 类字节码详解

Java代码是运行在JVM上的，而JVM能够屏蔽掉不同具体的设备差异，这提供了很好的**移植性**。

`.Java文件` 需要编译为 `.class文件`字节码文件才能被JVM运行。其中`.class文件`就是以字节为单位，将类数据按照顺序紧凑的排列在class文件中(有格式模板)。JVM按照预设的模板解析。



**Class文件的结构**：

- 字节码(Class)文件信息

- 常量池: 存储字面量、字段的名称和描述符号、方法的名称和描述符等...
  - 字面量类似于java中的常量概念，如文本字符串，final常量等;
- 方法表集合：对类内部的方法描述(类型，权限修饰)。包括成员变量和方法的描述；
  - code内的主要属性：
    - **stack**：最大操作数栈，根据这个分配栈帧
    - **locals**: 局部变量所需的存储空间
    - **args_size**: 方法参数的个数，每个实例方法都会有一个隐藏参数this
    - **attribute_info**: 方法体内容
    - **LineNumberTable**: 该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。
    - **LocalVariableTable**: 该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系

```java
Classfile /E:/JavaCode/TestProj/out/production/TestProj/com/rhythm7/Main.class
  Last modified 2018-4-7; size 362 bytes
  MD5 checksum 4aed8540b098992663b7ba08c65312de
  Compiled from "Main.java"
public class com.rhythm7.Main
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER

Constant pool:
   #1 = Methodref          #4.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#19         // com/rhythm7/Main.m:I
   #3 = Class              #20            // com/rhythm7/Main
   #4 = Class              #21            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/rhythm7/Main;
  #14 = Utf8               inc
  #15 = Utf8               ()I
  #16 = Utf8               SourceFile
  #17 = Utf8               Main.java
  #18 = NameAndType        #7:#8          // "<init>":()V
  #19 = NameAndType        #5:#6          // m:I
  #20 = Utf8               com/rhythm7/Main
  #21 = Utf8               java/lang/Object

{
  private int m;
    descriptor: I
    flags: ACC_PRIVATE

  public com.rhythm7.Main();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/rhythm7/Main;

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/rhythm7/Main;
}
SourceFile: "Main.java"
```



### JVM基础 - 类的加载过程

类加载过程包括：`加载`，`验证`，`准备`， `解析`， `初始化` 这几个阶段；

- 加载：根据类的全限定名将Class文件加载到**方法区**，并转化为运行时的数据结构；并且在**堆中**生成一个Class对象，作为方法区数据的访问入口

```java
Class<?> clazz = Class.forName("com.example.Demo");
Method method = clazz.getMethod("test"); // 通过Class对象访问方法区的方法信息
```

- 连接 
  - 验证 - 保证被加载的类的**正确性**，校验字节码文件信息
  - 准备 - 为类的**静态变量分配内存**，并初始化默认值（分配在方法区中）
  - 解析 - 把类中的符合引用转换为**直接引用**。
- 初始化：为类的静态变量赋予正确的初始值
  - 声明类变量是指定初始值
  - 使用静态代码块为类变量指定初始值




**类加载器**：

- Bootstrap ClassLoader：**启动类加载器**，负责加载存放在**JDK**\jre\lib下，或者被虚拟机识别的**类库**。JDK java.*开头的类

- Extension ClassLoader：**扩展类加载器**，负责加载JDK\jre\lib\ext目录中的所有库，javax.*开头的类
- Application ClassLoader：负责加载用户类路径(ClassPath)所指定的类；
- UserImpl ClassLoader



`双亲委派机制`, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

优势：

- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行



---

### JVM基础 - JVM内存结构

Java虚拟机(JVM)内存结构是指JVM在**执行Java程序**时所管理的**内存区域划分**，这些内存区域各有**不同的用途**和生命周期。

❗注意：和JMM不是一个东西！

![image-20250428202406497](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428202406497.png)

- 线程私有：程序计数器、虚拟机栈、本地方法区
- 线程共享：堆、方法区、堆外内存



#### 一、程序计数器

**作用**：记录 JVM 字节码下一条执行的指令地址。

**扩展**：因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行。JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

#### 二、虚拟机栈

**作用**：主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。**每个线程**都有自己的栈，栈中的数据都是以**栈帧（Stack Frame）的格式存在**。<u>每个方法都各自有对应的一个栈帧。</u> 



**栈帧** - 方法调用和执行的基本单位

- **局部变量**表：方法内定义的局部变量
- **操作数**栈：用于**方法执行时的计算**，存储**临时数据**（如算术运算、**方法参数传递**）
  - 可理解为草稿纸，函数中执行的计算，不可能一步完成，可能需要多步记录，这个栈可以用来暂存信息或函数调用时传参数。
- 动态链接：存储**指向运行时常量池（Runtime Constant Pool）的引用**，用于**方法调用时的符号解析** 【符合引用修改为内存中的入口地址-直接引用】
  - 存储**方法在运行时常量池中的引用**，用于支持**多态方法调用**（如接口方法、重写方法）；运行时才链接实际调用的方法(支持多态)；
- 方法返回地址：存储**方法执行完毕后返回的位置**（调用者的程序计数器 PC 值）
- 附加信息。



#### 三、堆内存

**作用：** 存放对象的实例。被所有线程共享。

**高效回收：** 将堆内存逻辑上分为三块 **<u>（优化GC的性能）</u>**

- 新生代：新对象和没到一定年龄的对象都放在新生代；
- 老年代：被长时间使用的对象；
- 元空间：



**具体：**

- 新生代：
  - 伊甸园区(Eden Memory) + 两个幸存区(Survivor Memory), 默认比例是 8:1:1
  - 大多数对象创建的时候都放置在Eden内存空间中；(首次)
  - 当Eden区被填充满，会执行Minor GC，将幸存的对象移动到一个幸存区中；
  - 两个两个幸存区使用的是复制算法；
  - 经过多轮Minor GC仍存活的对象会复制到 Eden区。默认是15次，因为这个标识使用4个比特；
- 老年代
  - 存放多次Minor GC仍存活的对象。当老年代内存满了，会执行Major GC
  - 大的对象，占据大量连续内存的对象会直接放入到老年代中，避免在两个幸存区中进行大量的内存拷贝；

- 设置JVM堆内存大小
  - -Xms来设置堆的起始内存
  - -Xmx来设置堆的最大内存



**对象在堆中的生命周期**

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
   - 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
   - 此时 JVM 会给对象定义一个**对象年轻计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
   - JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
   - 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代



#### 四、方法区

目的：存放**已被加载的类信息、常量、静态变量、即时编译器编译后的代码** 

1. **类信息的仓库**：存储加载的类的元数据（字段、方法、常量池等）
   1. 常量池：字面量("abc",123, final常量) 和 符号引用(类和接口的全限定名+字段和方法名称和描述符)
   2. 运行时常量池：类加载时，将常量池转变为运行时常量池。主要就是修改符号引用，调整为直接引用(内存实际地址)，运行时动态添加常量(String.intern())
2. **静态变量的家**：存放 `static` 变量和编译期常量。
3. **JIT 的缓存**：存储热点代码的本地机器码。
4. **动态性的基石**：支持反射、动态代理等高级特性



#### 五、本地方法栈

**目的：**用于支持 **Native 方法（本地方法）** 的执行。非Java语言的方法，例如操作系统提供的方法；原理与虚拟机栈类似





### ❌JVM基础 - JMM (Java 内存模型)

现在的电脑：CPU - Cache - 内存三级架构，并且CPU是多核的。这样的话，线程之间可能是同时运行的。

**会导致两个问题**

- 两个或多个线程共享一个对象，而没有正确使用volatile声明或同步，则一个线程对共享对象的更新可能对其他线程不可见。(CPU将结果存储在Cache中，并没有及时的写回内存)
  - volatile 确保线程修改共享变量后对其他线程立即可见
- 两个线程将共享对象的变量计数读入其CPU缓存中，例如初始为1，每个线程自增1，同步的情况下应该为3，但是未合理同步导致为2.
  - 使用Java synchronized块, 保证同一时间只有一个线程可以进入临界资源。



**JMM基础**

定义：定义了线程如何与内存交互，以及线程之间如何通过内存进行通信。

![image-20250506111746427](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250506111746427.png)

核心概念：

- 主内存和工作内存
  - 主内存：共享变量
  - 工作内存：线程私有的内存区域，存放局部变量和共享变量副本
- 内存间交互操作
  - 从主内存读取共享变量的线程工作内存
  - 线程将工作内存的共享变量写回主内存
- **happens-before原则** (重排序)
  - 编译器和处理器常常会对指令做重排序，编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。提高执行效率
  - 这些重排序都可能会导致多线程程序出现内存可见性问题
  - 插入特定类型的内存屏障指令，通过内存屏障指令来禁止特定类型的处理器重排序



JMM带来的问题

- 可见性
  - **volatile**关键字，读取和修改都及时从内存中进行读写，保证修改的结果对其他线程及时可见
- 有序性
  - synchronized、**concurrent包**提供的，原子类和锁机制实现。
- 原子性
  - **synchronized**关键字进行加锁，通过互斥实现线程安全；





### GC 垃圾回收

**目的：**释放掉不使用了的内存空间，主要针对的是堆和方法区



#### 判断一个对象是否可以被回收

- 引用计数
  - 给对象添加一个引用计数器，每被引用一次则+1。当值为0表示没有被其他对象引用，可以回收
  - 缺点：会产生循环引用的情况，就是对象A引用对象B，对象B引用对象A，但是这俩对象都不再使用，但是无法回收，导致内存泄漏。
- 可达性分析
  - 通过 GC root作为起始点进行搜索，能够到达的对象都是存活对象，不可达的对象会被回收。
  - GC root对象一般包括：
    - 虚拟机栈、本地方法栈、方法区静态属性、方法区常量引用中引用的对象

类的卸载条件：(需要满足以下三点，且满足了也不一定回收)

- 所有实例被回收
- 对应的类加载器被回收
- 类的Class对象被回收，那么在任何地方都无法通过Class对象反射访问该类方法



#### 引用类型

- 强引用: 被强关联的对象

```java
Object obj = new Object();
```

- 软引用：只有内存空间不足时才会被回收；使用场景：外部资源，比如图片缓存等... 可以再次获取。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

- 弱引用：被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。使用场景：临时缓存

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

- 虚引用：为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```





#### 垃圾回收算法

- 标记 - 清除
  - 标记存活的对象，清理掉未被标记的对象

![image-20250428213710917](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428213710917.png)

- 标记 - 整理![image-20250428213740094](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428213740094.png)
  - 让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。
- 标记 - 复制
  - 将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

![image-20250428213806913](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428213806913.png)

- 分代回收
  - 一般将堆分为新生代和老年代。
    - 新生代使用: 标记 - 复制算法
    - 老年代使用: 标记 - 整理 算法



#### 垃圾收集器 

设计目标：**吞吐量** or **延迟**

![image-20250428213937801](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428213937801.png)



![image-20250428214942234](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428214942234.png)

- Serial new + old 收集器  ˈsɪəriəl
  - **串行**进行垃圾回收。**单CPU环境下性能高**，因为没有上下文切换的开销。会存在stop the world 死区。
  - 年轻代-**标记复制**；老年代-**标记整理**



![image-20250428220451369](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428220451369.png)

- Parallel Scavenge + Parallel Old收集器  ˈpærəlel ˈskævɪndʒ
  -  Serial的多线程版
  - 年轻代-**标记复制**(多线程)；老年代-**标记整理**(多线程)



ParNew：![image-20250428220851117](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428220851117.png)

CMS： Concurrent Mark Sweep   swiːp![image-20250428220322299](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428220322299.png)

- ParNew - CMS 收集器
  - 为了配合CMS产生的ParNew 
  - 年轻代-多线程复制；老年代-**并发标记-清除**(CMS)
  - CMS收集器： **降低延迟**
    - **初始标记**: 仅仅只是**标记一下 GC Roots 能`直接`关联到的对象**，速度很快（只扫第一层引用），需要停顿。 **STW暂停**
    - **并发标记**: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。(****递归标记**所有存活对象（从初始标记的对象出发，遍历整个对象图）**)  让GC和用户线程**交替工作**，减少停顿时间
    - **重新标记**: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。(**修正并发标记期间的引用变化**) **STW暂停**  (卡表标记变更的区域)
    - **并发清除**: 不需要停顿。
- G1 收集器
  - 新生代：分区复制 + 标记-整理
  - 老年代：分区标记-整理

![image-20250428222244144](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428222244144.png)

通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。**通过记录每个 Region 垃圾回收时间以及回收所获得的空间(这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。**



每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![image-20250428222229462](C:\Users\韦龙\AppData\Roaming\Typora\typora-user-images\image-20250428222229462.png)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

- 初始标记：
- 并发标记
- 最终标记: 为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
- 筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点:

- 空间整合: 整体来看是基于“标记 - 整理”算法实现的收集器，从局部(两个 Region 之间)上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。





---



#### 内存分配和回收策略

- Minor GC、Major GC、 Full GC

  - Minor GC: 收集新生代垃圾

  - Major GC： 收集老年代垃圾

  - Mixed GC：G1收集器，收集新生代垃圾和部分老年代垃圾

  - Full GC：收集整个堆和方法区垃圾

    

- 内存分配策略

  - 对象优先在Eden区分配
    - 当Eden区内存不足，发起minor GC
  - 大对象直接进老年代
    - 避免在Eden区和survivor区之间的大量内存复制
  - 长期存活对象进老年代
    - 默认minor gc 15次之后
  - 动态年龄判断：当Survivor 中相同年龄所有对象大小总和超过一般的Survivor区，将大于等于该年龄的对象移动到老年代中；

- Full GC触发条件
  - 调用System.gc()：手动调用，只是建议并不一定执行。不推荐
  - 老年代空间不足

