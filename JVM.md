## jvm内存布局

![memoryarch](https://pcsdata.baidu.com/thumbnail/325601426s6725d837583145b4f470c0?fid=1508469986-16051585-923052660936310&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-bmVveRAyY1CRbgdXXz6LiemCM%2BU%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=2795343168&dp-callid=0&time=1618070400&size=c1600_u1600&quality=100&vuk=-&ft=video)

方法区：线程共享，非堆(但物理结构上其实是个堆)，存储被jvm加载的常量，类信息，静态变量和编译后的代码数据等，方法区满会抛出OOM，内部包含了运行时常量池，用于存放编译器生成的符号引用和字面量。

堆：线程共享，从存放对象实例，几乎所有对象实例都在堆里，由GC清理内存，内存满后会抛出OOM

java栈：线程私有，跟随线程一起创建，存放栈帧(可以看作是java方法执行的模型)，栈帧存放了操作栈数，局部变量表，引用对象类型，返回地址类型等数据，一个栈帧入栈到出栈的过程可以看作是一个方法的调用到return

本地方法栈：线程私有，用于执行本地方法的数据栈

程序计数器：记录当前执行的代码的行号，线程私有

## JMM

JMM是java的一种逻辑内存模型，物理上不存在，jmm和java内存布局的划分不是一个概念，jmm仅仅是一种控制数据访问模式的方式，非要比较的话，jmm的主内存包括了堆和方法区，工作线程则包括了栈，方法栈和程序计数器

![jmm](https://pcsdata.baidu.com/thumbnail/325601426s6725d837583145b4f470c0?fid=1508469986-16051585-923052660936310&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-%2B4xBMQiNdVzbPE8UViU5KrhIkVQ%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=3790468430&dp-callid=0&time=1618077600&size=c1600_u1600&quality=100&vuk=-&ft=video)

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





  
 



