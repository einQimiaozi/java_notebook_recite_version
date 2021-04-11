## jvm内存布局

![memoryarch](https://pcsdata.baidu.com/thumbnail/325601426s6725d837583145b4f470c0?fid=1508469986-16051585-923052660936310&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-%2B4xBMQiNdVzbPE8UViU5KrhIkVQ%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=3790468430&dp-callid=0&time=1618077600&size=c1600_u1600&quality=100&vuk=-&ft=video)

方法区：线程共享，非堆(但物理结构上其实是个堆)，存储被jvm加载的常量，类信息，静态变量和编译后的代码数据等，方法区满会抛出OOM，内部包含了运行时常量池，用于存放编译器生成的符号引用和字面量。

堆：线程共享，从存放对象实例，几乎所有对象实例都在堆里，由GC清理内存，内存满后会抛出OOM

java栈：线程私有，跟随线程一起创建，存放栈帧(可以看作是java方法执行的模型)，栈帧存放了操作栈数，局部变量表，引用对象类型，返回地址类型等数据，一个栈帧入栈到出栈的过程可以看作是一个方法的调用到return

本地方法栈：线程私有，用于执行本地方法的数据栈

程序计数器：记录当前执行的代码的行号，线程私有

## JMM

JMM是java的一种逻辑内存模型，物理上不存在，jmm和java内存布局的划分不是一个概念，jmm仅仅是一种控制数据访问模式的方式，非要比较的话，jmm的主内存包括了堆和方法区，工作线程则包括了栈，方法栈和程序计数器

![jmm](https://pcsdata.baidu.com/thumbnail/79ee977e7m00915b3e1e750cc751be72?fid=1508469986-16051585-542448575531019&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-DC9zVeppXtgxxV%2FT5kg5hRGbhfQ%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=3790468430&dp-callid=0&time=1618077600&size=c1600_u1600&quality=100&vuk=-&ft=video)

工作线程维护自己的副本变量(具体内容参看jvm布局的内容)，线程间屏蔽，主内存共享(可能会触发线程安全问题)，不允许直接对主内存修改，工作线程需要拷贝一份要修改的变量副本到自己的内存里，修改后回写给主内存

关于主内存和工作线程中实例对象的存储方法：
  - 成员方法中：不管是引用类型还是基本数据类型都存入栈帧，但是引用指向的对象实例存在主内存中
  - 成员变量中：全部存入主内存(堆区)
  - static和类信息：存入主内存
  - 多个线程调用同一实例的同一方法时：遵循jmm的原则，工作内存需要先拷贝一份副本，操作结束后再回写到主内存

### 一对一线程模型

指jmm的工作线程和硬件处理器的内核线程一一对应，每个被Executor创建的工作线程会从线程池中被取出，然后分配给一个内核线程(一一对应)，再使用线程调度器将内核线程分配给不同的cpu执行其任务

### 硬件内存架构

![cpu](https://pcsdata.baidu.com/thumbnail/fff88abe3j36125f1480712ab4087d3c?fid=1508469986-16051585-214595303277711&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-R05g1QyM1O2nkz%2FzJFLqOULPoqM%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=3666568820&dp-callid=0&time=1618077600&size=c1600_u1600&quality=100&vuk=-&ft=video)

用cpu缓存作为缓冲解决寄存器和内存io速度差距过大的问题，当然也不是每次都会缓存命中(比如volatile变量就不允许cpu进行缓存)，缓存的命中也会影响io效率

实际上上面的一对一模型就是通过硬件内存架构实现的，JMM和硬件内存架构并不一一对应，举例来说就是工作内存中的数据可以存在于寄存器或者cpu缓存或者硬件主内存中，对于jmm主内存也是如此

### JMM存在的必要性

避免多个线程对内存中同一对象同时操作是产生的线程安全问题

### JMM的承诺

1.原子性：
  - 通过synchronized或reentrantlock等锁保证
  - 通过jvm对基本数据类型的原子性保证

2.可见性：
  - 使用synchronized和volatile强制让修改后的值立刻对其他线程可见

3.有序性：
  - voltaile实现禁止指令重排序，指令重排序是指cpu执行命令时由于调用的硬件不同，所以命令1的执行未必在命令2之前，但是能保证代码执行的结果正确
  - happens-before原则：
    - 同一线程内书写在前的代码执行顺序优先于书写在后的
    - 对volatile变量的读操作优先于书写在后的写操作
    - 对同一个锁的lock操作优先于书写在后的unlock操作
    - 操作a若在b之前，b在c之前，则a在c之前(传递性)
    - Thread对象的start()优先于此线程的每一个动作
    - Thread对象的interrput()调用优先于该线程对中断事件检测的代码的执行
    - 线程中的所有操作都优先与线程终结检测之前
    - 实例对象的初始化优先与finalize()方法的执行

## 类加载过程

![process](https://pcsdata.baidu.com/thumbnail/7f8a24eebpa99342234310e3f6f15256?fid=1508469986-16051585-994672685981118&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-xD3CeRYfmrz%2BFjAX73VK6Zfd3go%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=537109366&dp-callid=0&time=1618131600&size=c1600_u1600&quality=100&vuk=-&ft=video)

1.加载：通过一个类的类全限定名查找到该类的class字节码文件并加载，根据字节码文件创建该类的Class对象，jvm对class的加载是按需加载，即第一次使用时才会加载

2.验证：确保class文件包含的信息符合jvm要求
  - 文件格式验证
  - 符号引用验证
  - 字节码验证
  - 元数据验证

3.准备：对类变量(static)分配默认值，不包括final修饰的变量(编译的时候就已经分配了)，可以理解为分配内存但不初始化

4.解析：将符号引用解析为直接引用，符号引用指对目标的一组描述，可以是指针，偏移量，句柄

5.初始化：对类的静态成员初始化其代码中赋予的值，若该类具有父类，则顺手初始化父类

## 类加载器：

1.启动类加载器：
  - 加载{JAVA_HOME}/lib下或使用-X bootclasspath指定路径下的class
  - 非jvm自带，使用c++实现
  - 出于安全考虑，启动类加载器只加载java.javax.sun开头的类
  - 无父类加载器

2.扩展类加载器：
  - jvm自带，java实现。launcher的静态内部类
  - 加载{JAVA_HOMR}/lin/ext下或系统变量Djava.ext.dir指定路径下的包
  - 父类加载器为null

3.系统类加载器(应用类加载器)
  - jvm自带，java实现
  - 加载系统类路径java -classpath或D java.class指定路径下的类
  - 应用中默认的类加载器，可以由开发者直接调用并设置
  - 父类加载器为扩展类加载器

4.自定义类加载器：
  - 用户编写的类加载器，默认父类加载器为系统类加载器
  - 实现的时候需要继承ClasLoader或UrlClassLoader类，具体用哪个看下面

注意，这里的父类指的不是java的继承关系，而是加载器的优先级

## 双亲委派模型

![model](https://pcsdata.baidu.com/thumbnail/5e9cad2a3s9591e47a4c7170105244e2?fid=1508469986-16051585-881829052094408&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-UCjP0PLw7DyVmoZu%2FRy5ZtnhaWo%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=537109366&dp-callid=0&time=1618131600&size=c1600_u1600&quality=100&vuk=-&ft=video)

在jdk1.2之后引入的一种类加载规则

原理：一个类加载器收到类加载请求后会向上传送，如果上位加载器处理不了才会向下送给下位加载器加载

优势和必要性
  - 1.防止同一个类被不同的类加载器加载多次，防止核心类被系统类加载器加载后随意修改
  - 2.java中对类是否相等的判定由类加载器相等+类相等实现

### 双亲委派的破坏和线程上下文类加载器：

java中很多服务器提供者接口允许第三方为他们提供实现(JBDC,JNDI等)，这些接口输入java类核心库，一般存在于lib下，由启动类加载器加载，但是他们的代码实现经常会依赖需要系统类加载器加载的jar包，根据jvm的类加载规则，启动类加载器不能加载这些依赖类，同时由于双亲委派模型的存在，也不能够向下递送这些加载需求

所以需要使用一种叫做线程上文下类加载的classloader来加载这些依赖类，造成双亲委派模型的破坏，线程上下文类加载器属于一种系统类加载器(contextClassLoader)，需要手动获取和设置(getContextClassLoader setContextClassLoader)，如果没有手动设置则会继承父线程的contextClassLoader

为什么不直接通过getSystemClassLoader获取线程上下文类加载器：
  - 在javaweb环境下使用的线程上下文类加载器并不一定是系统类加载器！！！
  - 使用getContextClassLoader可以保证获取到的类加载器和当前线程相同，不需要考虑当前服务下的线程上下文类加载器到底是什么

## 类加载器调用

![classlaoder](https://pcsdata.baidu.com/thumbnail/570b977e5j140cd29522b5c0a1366bd1?fid=1508469986-16051585-902562109188250&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-Wb72JpwayR55fjyLXejkNB9ox9k%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=537109366&dp-callid=0&time=1618131600&size=c1600_u1600&quality=100&vuk=-&ft=video)

所有的非启动类加载器都继承自ClassLoader抽象类

1.LoadClass(String):jdk1.2之前用于传入类名加载类的默认方法，因为jdk1.2之后引入了双亲委派模型，所以现在不建议覆盖该方法进行类加载了

2.findClass(String):jdk1.2之后不再建议覆盖LoadClass方法，而是使用findClass方法，该方法需要自己覆盖编写类加载逻辑，如果没有覆写则会抛出异常

3.defineClass(byte[],int,int):将byte字节解析成jvm能够识别的Class对象，即使用字节流生成Class对象(只生成，不解析)，一般作为findClass的返回值

```java
public class<?> findClass(String name) throws ClassNotFoundException {
  // 获取字节流
  byte[] classData = getClassData(name);
  if(classData==null) throw new ClassNotFoundException;
  else {
    // 需要传入起始位置和偏移量
    return defineClass(classData,0,classData.length);
  }
}
```

4.resolveClass:将创建好的Class对象解析

5.urlClassLoader：该方法会使用一个UrlClassPath类对象，通过该对象找到要加载的字节流，通过defindClass方法生成class对象，整个过程自动加载，只需要传入字节码位置，不需要覆写findClass

6.UrlClassPath：根据传入的字节流文件地址返回该字节码的位置，包含了FileLoader和jarLoader两个class，用于自动判断并返回文件字节码和jar包字节码

以下三种情况不能使用UrlClassLoader，需要手动编写加载逻辑
  - 1.class文件不在系统类加载器能加载的路径范围内
  - 2.class文件通过网络传输并且需要加密
  - 3.需要实现热部署，class文件需要通过不同的类加载器产生不同的对象

### 类加载缓存问题

不同类加载器加载的同一个类不相等是因为类名称空间不同

在loadClass方法的源码中，类全限定名的查找是通过缓存查找，也就是说第一次加载的结果决定了之后所有加载的结果，如果加载出现错误，需要绕过缓存查询才能正确的重新加载class对象，也可以直接通过findClass方法查找

### 显示加载和隐式加载

显式加载：在代码中他哦难过调用ClassLoader加载class对象
  - this.getClass().getClasLoader.loadClass()
  - Class.Forname(String name):通过名称加载(找不到抛错误)
 
隐式加载：通过虚拟机自动加载，该机制在加载某个class时自动加载该class的其他依赖class

开发时通常两种方法混用

## GC

### 如何判断对象已死

1.引用计数法：对象维护自己被引用的次数，一旦引用次数为0,则认为该对象已死
  - 优点：简单易用，无需STW
  - 缺点：无法处理回路，需要额外空间维护计数器

2.可达性分析法：通过选取GC Roots，从GC Roots出发进行图遍历(节点为对象，路径为对象间的引用)，未遍历到的对象就默认死亡
  - 选取GC Roots的标准
    - java栈局部变量表中引用的对象
    - 方法区中的静态属性引用的对象
    - 方法区中常量引用的对象
    - 本地方法栈中JNI方法(本地方法)引用的对象
  - 引用类型
    - 强引用：永远不会被GC
    - 软引用：内存空间不足时才会被GC
    - 弱引用：被垃圾回收器扫描到就会被GC
    - 幻影引用：同弱引用，并且不会被操作，只在GC时发出一个系统通知
 
3.类死亡判断标准：
  - 判定死亡(自救阶段)：若对象未通过可达性分析，则会被记录，对象会被放入一个F-Quene并可以在GC的同时执行自己的finalize方法(如果有的话，没有直接回收)，该方法若能够在GC到自己之前完成自救，则该对象不会被判定为死亡
  - 废弃常量：常量池中的对象未被引用时则出发回收判定
  - 无用的类：
    - 类所有实例被回收
    - 该类的ClassLoader被回收
    - 无法使用反射获取该类的Class对象并且Class对象没有任何引用

4.三色标记：可达性分析的实际执行基础
  - 白色：未被引用的对象，可达性分析开始前所有对象都是白色，结束时如果还是白色则进入最后的自救阶段
  - 灰色：被引用但其引用的对象未检查完
  - 黑色：被引用且其引用的对象也已经检查完毕，即确定生存对象

### StopTheWorld

安全点：就是一个程序会长时间执行的地方(循环，方法调用，异常跳转)，保证线程引用关系不发生变化
  - 抢占式中断：jvm会中断所有线程，如果发现不在安全点上的线程被中断，则恢复执行，直到该线程到达安全，总结：jvm负责管理线程的中断
  - 主动式中断：jvm通过设置中断标志位通知线程中断，线程通过轮询中断标志位是否为true来响应jvm的中断，轮询标志位的位置一般在安全点or创建对象分配的位置，总结：jvm只负责告诉线程哪里可以中断，线程需要自己管理自己的中断，这也是目前常用的方法

安全区域：如果程序处在不执行状态(如阻塞)，那么它就无法响应jvm的中断请求，GC也不可能等待线程唤醒，但是阻塞这个状态本身其实是不会改变引用关系的，所以这类状态就叫做安全区域，本质上就是扩大范围的安全点，进入该类状态的线程会告诉jvm自己处在安全区域，这样GC的时候会直接无视该线程的状态，线程唤醒时检查GC是否结束，如果没有结束则等待GC结束再唤醒(所以安全区域本质上不是什么新东西，而是对一种无法响应就jvm中断请求但是又满足安全点要求的状态的描述)

GC中需要执行GC同时中断用户线程的行为就叫StopTheWorld，它本质上就是利用安全区域和安全点来实现的
  
### GC算法

1.标记清除：
  - 1.标记需要回收的对象
  - 2.统一清除被标记的对象

  会产生内存碎片，使得内存不连续
  
2.标记复制：
  - 1.将内存一分为二
  - 2.一块正常使用，每次GC后将存活的对象拷贝到另一块中
  - 3.对使用的内存进行清理
  - 4：比例划分：采用该算法的GC回收器不一定严格按照1：1比例划分，比如G1和CMS会采用8：1：1的比例划分，8=eden区，1=survive区，每次使用eden和一个survive1
  - 5.eden和survive的回收流程：
    - GC后将存活的对象复制到另一个survive2，清空eden和survive1
    - 重点：复制结束后互换survive1和survive2！！！
    - 理由1：为什么要使用两个survive？因为survive1保存了存活对象，下次GC的时候其实还要扫survive1,所以不能使用survive1进行复制！！！不然会产生内存碎片，总结：防止内存碎片化
    - 理由2：为什么是8：1：1？因为根据实验，90%的新生代活不过一轮，所以每次回收占比平均高达90%
  - 6.空间担保：当survive空间不够的时候，除了OOM外，还可以选择设置空间担保策略

    ![space](https://pcsdata.baidu.com/thumbnail/b89f5e006ufa230247503b4866398abc?fid=1508469986-16051585-91808281140211&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-yJSiuczqALHJVYFYH1k6ILbYSWg%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=537109366&dp-callid=0&time=1618131600&size=c1600_u1600&quality=100&vuk=-&ft=video)
    
    - survive的空间不够时检查是否设置了空间担保
    - 如果设置了空间担保，检查每次老年代剩余空间是否大于每次平均从年轻代晋升到老年代的空间大小
    - 如果满足小于条件，则尝试Young GC，否则直接进行Old GC

    不会产生内存碎片，需要频繁大量复制，效率低，内存利用率也会减少

3.标记整理：和标记复制差不多，区别在于每次将存活对象移动到内存的一端，使用指针进行边界维护，清理指针之外的内存区域

### 分代收集算法

根据对象的生命周期，对象被划分为老年代和年轻代(两块内存，eden和survive都属于年轻代

年轻代对象存活率低，一般使用标记复制算法

老年代对象存活率高，一般使用标记整理或标记清除算法

进入老年代的条件：
  - 每次GC少量存活的年轻代熬过n轮后会进入老年代，如果survive区年龄为x的对象大小的总和>survive空间的一半，则n = x
  - 如果设置了-XX:MaxTenurigThreshold=y，则n=y
  - 大对象直接进入老年代，通过-XX：PretenureSizeThreshold设置

Young GC/Minor GC：对年轻代的GC，eden满了就触发，速度快

Old GC/Major GC：对老年代的GC，老年代空间已满且，调用System.gc()手动触发或 未设置空间担保/空间担保失效 会触发，速度慢但触发频率一般也低

Full GC：全内存的GC，而由于Old GC之前必然会触发Young GC，所以Full GC本质上和Old GC一样

### 垃圾回收器

| 回收器 | 新生代 | 老年代 | 并发 | 标记清除 | 标记复制 | 标记整理 | STW |
| :----: | :----:  | :----: | :----: | :----: | :----:  | :----: | :----: |
| Serial | O | X | X | X | O | X | O |
| Serial Old | X | O | X | X | X | O | O |
| ParNew | O | X | O | X | O | X | O |
| Parallel Scavenge | O | X | O | X | O | X | O |
| Parallel Old | X | O | O | X | X | O | O |
| CMS | X | O | O | O | X | X | O |
| G1 | O | O | O | X | O | X | O |

Paralled收集器的目标并不是单词快速GC，而是保证最大吞吐量(总时长较少即可)

CMS：
  - 执行流程
    - 初始标记：STW，查找GC Roots和其直接子节点
    - 并发标记：No STW，标记所有节点
    - 重新标记：STW，修正并发标记阶段的引用关系变化，使用通过写屏障实现的增量更新算法
    - 并发清除：No STW



  







  
 



