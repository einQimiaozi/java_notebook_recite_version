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

## BeanDefinition

BeanDefinition是对Bean对象的描述，spring根据BeanDefinition才能创建Bean对象，注解和xml都会被解析为BeanDefinition对象

重要属性
  - beanClass：表示bean类型，即bean的Class对象
  - scope：bean的作用域，比如singleton，prototype等(单例，多例)
  - isLazy：加载bean时是否启用懒汉模式
  - dependsOn：该bean依赖的其他bean，spring根据该参数会自动将这些依赖创建好并注入该bean
  - primary：主bean标识，进行依赖注入时如果一个类型有多个bean，则主bean会被注入
  - initMethodName：bean的初始化方法名

## BeanFactory

spring容器之一，用来生产bean，是一个顶层接口，下面有很多具体的接口，用于做功能增强

BeanDefinition 被读取---> BeadFactory 创建---> Bean对象

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
  
## 单例池

单例池是用于实现单例Bean的，这里的单例Bean指的是一个Bean在创建后每次getBean时都获得的是这个Bean，而不是指一个Bean所属的Class对象只有它一个实例

单例池底层通过concurrentHashmap实现，key为String，value为Bean对象(Object类型)

单例池属于BeanFactory的一部分

## FactoryBean

FactoryBean是一个创建Bean的接口，可以通过继承该接口创建其他Bean，你可以理解为一个小型工厂，这也是它和BeanFactory的区别

常用接口方法
  - getObject() 返回Bean对象
  - isSingleton() 判断Bean是否是单例
  - getObjectType() 返回bean对象的类型

FactoryBean在创建Bean实例时会同时创建两个对象，一个是getObject返回的Bean对象(假设为Object类型)，另一个是FactoryBean对象(所以说FactoryBean也是个Bean，假设为YourFactoryBean类型)，如果直接getBean("object"，YourFactoryBean.class),实际上会导致spring报错，因为你获取的object类型为getObject的返回类型，而你要求的类型是FactoryBean对象的类型，这里将object环卫&object即可获取FactoryBean对象，如果想获取object对象，则将YourFactoryBean.class改为Object.class即可

## springBean

springBean就是spring容器生产的产品

1.用法：
  - 使用<bean id="..." class="..." / >定义
  - id：确定该Bean的唯一标识符，容器对Bean管理、访问、以及该Bean的依赖关系，都通过该属性完成。Bean的id属性在Spring容器中是唯一的
  - class：指定该Bean的具体实现类。通常情况下，Spring会直接使用new关键字创建该Bean的实例，因此，这里必须提供Bean实现类的全限定类名

2.Bean的生命周期：
  - 创建BeanDefinition对象
  - 推断构造方法(细节多，下面说)
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
  
3.推断构造方法
  - 先从缓存中获取，因为prototype这类的bean会多次创建，所以第一次推断后结果会缓存在一个map中，如果没有则往下执行
  - 先拿到所有的构造方法
  - 遍历所有的构造方法
  - 获取构造方法的 @Autowired 的封装对象
  - 如果多个 @Autowired 都是 required=true，就会抛异常
  - 如果构造方法有 @Autowired ，就会存入 List
  - 如果有 @Autowired 注解，并且 required=true，那么就会返回这一个构造方法
  - 如果有 @Autowired 注解，并且都是 required=false，那么就会返回这些构造方法和一个无参构造方法的数组
  - 如果没有 @Autowired 注解，只有一个有参构造方法，就会会这一个构造方法
  - 如果都不是，就会返回 null(即只有无参构造方法或有其他有参构造方法但是都没有自动注入)
  
4.springBean，javaBean和对象的区别
  - 首先springBean和javaBean都是Bean，Bean本身也是一种对象，所以对于相同Class对象的实例对象，Bean和对象本质没有区别，不同Class对象的话，一定要说区别就是Bean有一套自己的对象定义规则，并且根据spring的配置(注解之类的)可以实现自动赋值，而对象没有
  - SpringBean和javaBean的最大区别在于javaBean必须按照Bean的定义规则编写，而SpringBean只要在配置文件中声明，则不一定要按照Bean的定义规则编写，一个不含任何方法和成员的对象也可以是springBean
  
5.Bean的定义方式
  - bean标签，通过加载xml文件，使用classPathXml上下文管理器，声明式
  - @Bean，通过定义Config类，加载该类的class对象，@Bean加在方法上，使用annotationConfig上下文管理器，声明式
  - @Component，通过在xml文件里定义component-scan扫描的对应路径中的所有@Component标志的类，当然也可以使用@ComponentScan("path")这种注解方法定义被扫描路径，声明式
  - 通过BeanDefinition实现类的对象定义Bean，将配置好的Beandefinition对象注册到上下文管理器中，编程式，注意，前三种方式本质上是基于BeanDefinition实现的，该方法也是上下文管理器的registerBean方法的底层实现(该方法允许直接将一个class注册成一个bean)
  
  
  
