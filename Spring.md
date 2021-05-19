## Maven

1.优点
  - 能够自动生成项目信息
  - 能够单独发出输出，也可以与项目源代码管理系统(如git)集成
  - 向后兼容
  - 并行构建：编译速度一高20%-50%
  - 提供更完整的错误报告
  - 配置简单，随时可以访问新功能，依赖管理包括自动更新

2.项目对象模型(POM)

maven的基本工作单元，maven通过读取dom获取所需要的配置信息然后执行，比如项目依赖，插件，执行目标，项目版本等信息

## Spring优点

1.降低代码耦合

2.提供基于切面的声明式事务管理

3.方便集成其他框架

4.降低开发难度

## SpringIOC

1.ioc容器：用于管理对象的实例化，初始化，对象和对象之间的依赖关系配置，销毁，对象的查找等功能，控制对象的生命周期，ioc控制反转实现的前提，通过程序启动时提供的清单创建清单内的对象和其依赖关系，抽象的说spring就是一个大型工厂，容器就是生产线，ioc容器有很多种，最基本的就是BeanFactory，它是一个顶层接口，有各种实现类(list,hierarchical之类的)

2.ioc控制反转：将对象的创建交给spring处理，不需要手动new，是一种面向对象设计原则，降低系统耦合度
  - 大致原理：使用反射根据注解或xml配置文件信息获取Class对象，动态创建对象实例(可以是单例也可以是多个不同对象)

3.DI依赖注入：用于查找清单中对象的依赖，比如a对象创建依赖b和c，那么会提前创建好b和c对象然后注入给a对象(a对象创建前是不知道其他对象是否存在或，在哪里以及他们如何创建，完全依靠dI被动注入)

## springBean

springBean就是spring容器生产的产品

1.用法：
  - 使用<bean id="..." class="..." / >定义
  - id：确定该Bean的唯一标识符，容器对Bean管理、访问、以及该Bean的依赖关系，都通过该属性完成。Bean的id属性在Spring容器中是唯一的
  - class：指定该Bean的具体实现类。通常情况下，Spring会直接使用new关键字创建该Bean的实例，因此，这里必须提供Bean实现类的全限定类名

2.Bean的生命周期：
  - 实例化: instanceWrapper = this.createBeanInstance(beanName, mbd, args);
  - 属性赋值: this.populateBean(beanName, mbd, instanceWrapper);
  - 初始化：exposedObject = this.initializeBean(beanName, exposedObject, mbd);
    - 检查aware接口：aware接口能够让bean感知到自己在spring容器中的各种属性
    - 若bean实现了BeNameAware接口，spring将bean的id传给setBeanName方法，这里就是bean对spring容器进行aware的体现了
    - 若bean实现了BeanFactoryAware接口，spring调用setBeanFactory方法将BeanFactory的实现类实例注入给bean(这里的BeanFactory需要自己传入参数指定)
    - 若bean实现了applicationContextAware接口，则调用setApplicationContext方法，作用和上面一样，区别是spring会把自己作为参数注入，不需要指定容器
    - 若bean实现了BeanPostPrecess接口，则spring将调用PostProcessBeforeInitialization方法执行实例创建成功后的前置增强(后面还有个后置处理)
    - 若bean实现了InitializingBean接口，则spring将调用afterpropertie方法执行初始化(相当于构造方法，就是xml配置中的init-method)，这个初始化是在属性赋值后执行的！！！
    - 后置处理，这次调用的是PostProcessAfterInitialization
  - 初始化后bean将和applicationContext绑定，applicationContext销毁后bean也销毁
  - 销毁：若bean实现了DispostibleBean接口，则会调用destory方法，类似于c++里的析构函数
  
3.BeanDefinition
  - 作用：用于对bean进行描述，BeanDefinition和BeanFactory一样也是个顶层接口，下面有很多具体的实现类
  
  ```java
  public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

    // 单例、原型标识符
    String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
    String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

      // 标识 Bean 的类别，分别对应 用户定义的 Bean、来源于配置文件的 Bean、Spring 内部的 Bean
    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;

      // 设置、返回 Bean 的父类名称
    void setParentName(@Nullable String parentName);
    String getParentName();

      // 设置、返回 Bean 的 className
    void setBeanClassName(@Nullable String beanClassName);
    String getBeanClassName();

      // 设置、返回 Bean 的作用域
    void setScope(@Nullable String scope);
    String getScope();

      // 设置、返回 Bean 是否懒加载
    void setLazyInit(boolean lazyInit);
    boolean isLazyInit();

    // 设置、返回当前 Bean 所依赖的其它 Bean 名称。
    void setDependsOn(@Nullable String... dependsOn);
    String[] getDependsOn();

    // 设置、返回 Bean 是否可以自动注入。只对 @Autowired 注解有效
    void setAutowireCandidate(boolean autowireCandidate);
    boolean isAutowireCandidate();

    // 设置、返回当前 Bean 是否为主要候选 Bean 。
    // 当同一个接口有多个实现类时，通过该属性来配置某个 Bean 为主候选 Bean。
    void setPrimary(boolean primary);
    boolean isPrimary();

      // 设置、返回创建该 Bean 的工厂类。
    void setFactoryBeanName(@Nullable String factoryBeanName);
    String getFactoryBeanName();

    // 设置、返回创建该 Bean 的工厂方法
    void setFactoryMethodName(@Nullable String factoryMethodName);
    String getFactoryMethodName();

    // 返回该 Bean 构造方法参数值、所有属性
    ConstructorArgumentValues getConstructorArgumentValues();
    MutablePropertyValues getPropertyValues();

      // 返回该 Bean 是否是单例、是否是非单例、是否是抽象的
    boolean isSingleton();
    boolean isPrototype();
    boolean isAbstract();

      // 返回 Bean 的类别。类别对应上面的三个属性值。
    int getRole();

      ...
  }
  ```
  
  
