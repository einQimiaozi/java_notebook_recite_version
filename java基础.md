## 面向对象三大特性

1.封装：把对象的属性和行为以某种关系整合到一个及合理，通常使用类来实现

2.多态：允许以同一个编程风格来编写程序，以处理种类繁多的已经存在的类
  - 重写 overrideing：运行期动态分配，要求子类方法覆盖父类的方法时参数和返回值都相同，抛出的异常不能超过父类，访问级别不能低于父类的级别
  - 重载 overloading：编译期静态分配，一个类中的两个或以上方法拥有相同的方法名，但是其他的都不相同，典型的例子就是构造函数

3.继承：类似与父子关系，子类能够使用父类的大多数方法和属性(private和final除外)

## 常量和静态变量

1.常量：final修饰，编译期初始化，运行时不能改变其引用或数值，如果方法被声明为final，则不能被子类重写，private默认为final修饰

2.静态变量：static修饰，属于类变量(所有实例共享，和实例的生命周期无关，可以直接通过类名访问，内存中只有一份)，如果是方法被修饰则在类加载阶段就存在，不能是抽象方法，并且因此只能访问该类的静态成员

3.静态语句块：类初始化时直接运行一次

4.静态内部类：常用于实现高效同步的工厂设计模式，不依赖于外部类的实例创建

## String

1.存储：jdk8中使用char数组，jdk9之后使用byte数组

2.性质：不可变对象，可以用于缓存hash，线程安全

3.创建：直接初始化会从String pool中取，如果没有的话则会创建一个Stirng对象，并将其缓存入String pool中，String pool使用堆中内存，本质上一个new String可能会创建两份副本

## 基本数据类型

  | 类型 | 大小 | 取值范围 | 包装器 |
  | :----: | :----:  | :----: | :----: |
  | byte | 8bit | [-128,+127] | Byte |
  | short | 16bit | [-2^15,+2^15-1] | Short |
  | int | 32bit | [-2^31,+2^31-1] | Integer |
  | long | 64bit | [-2^63,+2^63-1] | Long |
  | float | 32bit | [IEEE754,IEEE754] | Float |
  | double | 64bit | 同上 | Double |
  | char | 16bit |[Unicode 0,Unicode 2^16-1] | Character |
  | void | --- | --- | --- |
  
## 缓存池

  | 类型 | 缓存范围 |
  | :-----: | :-----: |
  | boolean | | 所有boolean值 |
  | byte | 所有byte值 |
  | short | -128,127 |
  | int | -128,127 |
  | char | \u0000,\u007F | 
  
  - Integer.valueOf(x) 和 new Integer(x) 一个会使用缓存池中的对象(多次调用指向的是同一个对象) 另一个会新建一个对象
  - valueOf(x) 实现的方法就是判断值是否在缓存池中，如果不在就返回，不在就new一个，然后返回

## 拷贝

1.深拷贝和浅拷贝在基本数据类型下对值拷贝

2.引用类型下深拷贝对对象拷贝副本，浅拷贝对引用进行拷贝

3.java中Object类针对拷贝使用自带的clone方法，要重写clone()方法需要类实现Cloneable接口，该接口是标注接口，如果没有实现Cloneable接口就重写clone方法会抛出异常，不想实现Cloneable方法实现拷贝可以使用拷贝构造函数或拷贝工厂

## 接口和抽象类

1.抽象类可以有类成员和方法体，接口不可以(只能通过反射实例化接口然后添加成员和方法体)，只能有抽象方法

2.抽象类不允许多重继承，接口可以

3.标识接口：用于对类做标记，如果某类需要实现某功能，则必须先实现某接口，比如RandomAccess(随机访问能力)，Cloneable(拷贝)，Seriallizable(序列化)

## 异常

1.运行时异常(RuntimeException)：由jvm管理，编译期不检查，可以捕获和一抛出，比如空指针异常

2.编译时异常：编译期会检查，必须进行处理，否则不能通过编译，RuntimeException以外的异常都是编译时异常，比如IOException,sqlException

3.错误(Error):一种严重的错误，和异常的最大区别在于程序无法处理，比如OOM

## 数组

1.数组的类没有class文件(因为是由jvm运行时动态创建的)，操作由jvm指令直接执行，比如newarray：创建一个数组

2.数组的类名以 [ 开头，和普通的类不一样，并且一维数组和二维数组的类并不一样( [I 和 [[I)，在jvm看来一维数组和二维数组不是同一个类

## equals和hashcode

1.Object对象的hashcode方法是本地方法，c/c++实现，判断地址，需要根据具体需求重写

2.equals通过对象的hashcode方法判断是否相等，所以重写equals方法必须重写hashcode方法
  - 自反性
  - 一致性
  - 对称性
  - 传递性
  - 非null对象调用该方法传入null返回必为false

## 动态绑定和静态绑定

1.java中的绑定：对方法或变量调用方式的操作就叫做绑定

2.静态绑定：在编译期确定的绑定就是静态绑定，常见的有重载，使用private或static修饰的方法或变量，静态绑定使用类信息来完成

3.动态绑定：在运行时才确定调用方式的就是动态绑定，常见的有重写和反射，使用对象信息来完成

## javabean

javabean是一种用于传递数据的特殊类，成员由private修饰，通过public的get和set开头+成员变量名(首字母大写)的方法访问和设置成员变量，方便ide对的读取和分析

## 反射

1.反射值在运行时对任意一个类都能够动态的知道这个类的所有方法和属性，对于任意一个对象都能够调用它的任意方案发

2.原理：jvm在类加载时会通过类全限定名查找并加载class字节码创建Class对象，反射就是通过获取Class对象来实现动态调用该类和其实例的

3.常用方法：
  - Class.ForName(String name)：根据类名获取Class对象，有可能找不到所以需要手动抛异常
  - getClass()：通过对象实例获取Class对象，好处是不用处理异常，坏处是必须得先有个实例才行
  - Class.class:通过调用某个类的.class属性来获取，比上面两个方法方便简单且不需要处理异常,而且该方法不会触发类的加载(java中的类是按需加载的，第一次创建静态成员的引用时才会加载)，省内存，而且可以应用于数组，接口和基本数据类型
  
  ```java
  Class clazz = Person.class;
  ```
  
  - Class.newInstace()方法可以通过反射获取到的class对象来创建一个实例，但要求该Class必须有空的构造函数
  - 使用Class对象获取指定的Constructor对象(一个可以构造类的东西)，再调用Constructor对象的newInstance方法就可以选定创建实例的构造方法，不需要必须有空的构造函数了

## 代理

代理主要负责对委托类进行增强，包括过滤消息，转发消息给委托类，事后处理消息等，一般调用委托类的方法而不是自己重新实现一遍，只实现增强部分

### 静态代理

1.编译期确定，需要针对委托类编写代理类，实现委托类的增强

2.程序运行前，代理类的class字节码文件就存在

### jdk动态代理

1.运行时确定，使用反射实现，要求委托类必须是一个接口的实现类，不需要编写重复的代码

2.没有class字节码(因为运行时动态生成，程序结束的话代理类就不存在了)

3.原理：通过委托类的接口确定委托类的结构(这就是为什么委托类必须是接口的实现类)，根据该接口的Class对象反射创建代理类

4.核心方法
  - 动态代理的关键在于两个类:Proxy和InvocationHandler
  - InvocationHandler：代理类的方法调用会经过该接口，每个代理对象实例都会绑定一个Handler，通过该接口的invoke方法调用委托类的方法并执行方法前后的增强逻辑
  - Proxy：通过该类生成一个继承该类并实现委托类接口的代理对象，该代理对象的方法调用会被InvocationHandler拦截并转发给委托类调用

  ```java
  // 委托类接口
  public interface UserInfoImplements {
    public String getName(int id);
    public int getAge(int id);
  }
  
  // 委托类实现
  public class UserInfo implements UserInfoImplements {
    public UserInfo(){}
    @Override
    public String getName(int id) {
      if(id==1) return "John";
      if(id==2) return "Jack";
    }
    @Override
    public int getAge(int id) {
      if(id==1) return 18;
      if(id==2) return 23;
    }
  }
  
  // 定义InvocationHandler
  class MyInvocation implements InvocationHandler {
    private Object obj;
    public MyInvocation(Object obj) {
      this.obj = obj;
    }
    // 重点
    @Override
    public Object invoke(Object proxy,Method method,Object[] args) throws Threadable{
      // 使用method对象调用委托类的方法getName
      if(method.getName().equlas("getName")) {
        // 前增强逻辑 
        System.out.println("before getName!!!");
        // 参数为委托对象和该对象对应方法的参数数组
        Object res = method.invoke(obj,args);
        // 后增强逻辑
        System.out.println("after getName!!!");
      }
      // 调用getAge (这里因为只有两个方法所以我就简写了，实际上最好再做个name的判断)
      else {
        // 前增强逻辑 
        System.out.println("before getAge!!!");
        // 参数为委托对象和该对象对应方法的参数数组
        int res = (int)method.invoke(obj,args);
        System.out.println(res+1);
        // 后增强逻辑
        System.out.println("after getAge!!!");
      }
    }
  }
  ```
  
5.源码分析
  - invoke：定义增强逻辑
  - newProxyInstance(ClassLoader loader,Class<?>[] interface,InvocationHandler h)：该方法是Proxy类的静态方法。通过类加载器保证代理对象和委托类被同一类加载器加载，interface用于定义代理类实现增强的方法，h用于定义代理类的增强逻辑
  - ProxyClassFactory工厂：通过Class的字节码加载代理类(这个字节码不会保存在磁盘里，通过sun包的ProxyGenerator类动态生成)
  - 小结：Proxy通过ProxyGenerator生成代理对象的字节码并加载生成Class对象，通过反射创建代理类的实例

### cglib代理

1.解决了jdk动态代理的委托类必须实现接口的问题，原理是对委托类生成一个子类，并覆盖其方法实现增强，所以不能对final修饰的类进行代理，底层使用ASM字节码生成框架生成代理类，比java反射创建代理类效率更高，需要引入cglib.jar和asm.jar两个包，使用只需要一个参数和一个回调函数，进入毁掉函数后和jdk代理一样通过拦截执行增强

2.核心方法
  - MethodInterceptor接口：用于拦截代理类调用委托类的方法
  - intercept(Object Proxy,Method method,Object[] args,MethodProxy methodproxy)
    - proxy：代理实例
    - method：委托类要被调用的方法
    - args：方法参数数组
    - methodProxy：代理类对方法的代理引用

3.cglib总结
  - 可以传入接口也可以传入类作为委托对象，接口用实现代理，类用继承代理
  - static和final方法描述的方法不能被代理
  - 会默认继承委托对象的所有Object的方法(比如finallize，equals，toString等)
  - 提供了回调函数设计，可以使得拦截器的编码更加灵活
  
