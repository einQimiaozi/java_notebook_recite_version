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

3.DI依赖注入：用于查找清单中对象的依赖，比如a对象创建依赖b和c，那么会提前创建好b和c对象然后注入给a对象(a对象创建前是不知道其他对象是否存在或，在哪里以及他们如何创建，完全依靠dI被动注入),DI是ioc的一种实现方法

4.实现原理：
  - 根据Bean配置信息在容器内部创建Bean定义注册表
  - 根据注册表加载、实例化bean、建立Bean与Bean之间的依赖关系(这一步依靠反射实现)
  - 将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用

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

spring容器之一，用来生产bean，是一个顶层接口，下面有很多具体的接口，用于做功能增强，BeanFactory也是其他spring容器的爹

BeanDefinition被读取 ---> BeadFactory创建 ---> BeanFatory注册 ---> 容器扫描被注册的BeanFatory ---> 实例化Bean对象并进行属性赋值

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

spring容器之一

单例池是用于实现单例Bean的，这里的单例Bean指的是一个Bean在创建后每次getBean时都获得的是这个Bean，而不是指一个Bean所属的Class对象只有它一个实例

单例池底层通过concurrentHashmap实现，key为String类型，是该实例的BeanName，value为Bean对象(Object类型)

单例池属于BeanFactory的一部分

## FactoryBean

FactoryBean是一个创建Bean的接口，可以通过继承该接口创建其他Bean，你可以理解为一个小型工厂，这也是它和BeanFactory的区别

常用接口方法
  - getObject() 返回Bean对象
  - isSingleton() 判断Bean是否是单例
  - getObjectType() 返回bean对象的类型

FactoryBean在创建Bean实例时会同时创建两个对象，一个是getObject返回的Bean对象(假设为Object类型)，另一个是FactoryBean对象(所以说FactoryBean也是个Bean，假设为YourFactoryBean类型)，如果直接getBean("object"，YourFactoryBean.class),实际上会导致spring报错，因为你获取的object类型为getObject的返回类型，而你要求的类型是FactoryBean对象的类型，这里将object环卫&object即可获取FactoryBean对象，如果想获取object对象，则将YourFactoryBean.class改为Object.class即可

## springBean

springBean就是spring容器生产的产品，只有被spring管理的对象，才可以叫做springBean

有状态和无状态：有状态是指保存数据的对象，多线程环境下涉及线程安全问题，无状态是指不保存数据的bean，不涉及线程安全问题，一般MVC中的Dao层和Service都是无状态的，Service虽然存Dao实例但是Dao是无状态，Controller是有状态的

1.用法：
  - 使用<bean id="..." class="..." / >定义
  - id：确定该Bean的唯一标识符，容器对Bean管理、访问、以及该Bean的依赖关系，都通过该属性完成。Bean的id属性在Spring容器中是唯一的
  - class：指定该Bean的具体实现类。通常情况下，Spring会直接使用new关键字创建该Bean的实例，因此，这里必须提供Bean实现类的全限定类名

2.Bean的生命周期：
  - 如果是singleton，先调用doGetBean方法尝试从缓存获取，如果失败则继续向下
  - 创建BeanDefinition对象
  - 推断构造方法(细节多，下面说)
  - 实例化前，一段钩子函数，可以做实例化前的准备处理，返回值会被实例化接收，需要返回实例化时和bean对象相同的类型，否则会报错
  - 实例化: instanceWrapper = this.createBeanInstance(beanName, mbd, args);
  - 实例化后，一段钩子函数，可以做实例化后的后续处理
  - 属性赋值: this.populateBean(beanName, mbd, instanceWrapper);
  - 初始化：exposedObject = this.initializeBean(beanName, exposedObject, mbd);(这部分背不下来就算了，反正我背不下来)
    - 检查aware接口：aware接口能够让bean感知到自己在spring容器中的各种属性
    - 若bean实现了BeNameAware接口，spring将bean的id传给setBeanName方法，这里就是bean对spring容器进行aware的体现了
    - 若bean实现了BeanFactoryAware接口，spring调用setBeanFactory方法将BeanFactory的实现类实例注入给bean(这里的BeanFactory需要自己传入参数指定)
    - 若bean实现了applicationContextAware接口，则调用setApplicationContext方法，作用和上面一样，区别是spring会把自己作为参数注入，不需要指定容器
    - 若bean实现了BeanPostPrecess接口，则spring将调用PostProcessBeforeInitialization方法执行实例创建成功后的前置增强(后面还有个后置处理)
    - 若bean实现了InitializingBean接口，则spring将调用afterpropertie方法执行初始化(相当于构造方法，就是xml配置中的init-method)，这个初始化是在属性赋值后执行的！！！
    - 后置处理，这次调用的是PostProcessAfterInitialization
  - 初始化后bean将和applicationContext绑定，applicationContext销毁后bean也销毁，如果使用的是其他BeanFactory，就绑定其他BeanFactory
  - 如果这里使用了AOP，则返回的是代理对象，否则返回的就是初始化后的Bean
  - 如果是单例，从单例池中取，如果是原型，则直接将Bean交给调用者
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
  - 首先springBean和javaBean都是Bean，Bean本身也是一种对象，区别在于Bean有一套自己的对象定义规则，并且根据spring的配置(注解之类的)可以实现自动赋值，而对象没有，所以Bean肯定是对象，对象不一定是Bean，但他们的本质相同
  - SpringBean和javaBean的最大区别在于javaBean必须按照Bean的定义规则编写，而SpringBean只要在配置文件中声明，则不一定要按照Bean的定义规则编写，一个不含任何方法和成员的对象也可以是springBean
  
5.Bean的定义方式
  - bean标签，通过加载xml文件，使用classPathXml上下文管理器，声明式
  - @Bean，通过定义Config类，加载该类的class对象，@Bean加在方法上，使用annotationConfig上下文管理器，声明式
  - @Component，通过在xml文件里定义component-scan扫描的对应路径中的所有@Component标志的类，当然也可以使用@ComponentScan("path")这种注解方法定义被扫描路径，声明式
  - 通过BeanDefinition实现类的对象定义Bean，将配置好的Beandefinition对象注册到上下文管理器中，编程式，注意，前三种方式本质上是基于BeanDefinition实现的，该方法也是上下文管理器的registerBean方法的底层实现(该方法允许直接将一个class注册成一个bean)
  
## ApplicationContext
  
ApplicationContext就是前面说的下上文管理器，也是一种spring容器
  
ApplicationContext继承了BeanFactory的所有加强接口(注意不是直接继承BeanFactory)，所以它可以使用很多增强功能(国家化，父类BeanFactory之类的)，如果不是在极端情况下需要节约性能消耗，一般优先选择ApplicationContext安全
  
ApplicationContext本身也是一个接口，有很多实现类，可以根据不同的场景使用，比如加载相对路径的xml和绝对路径的xml或者使用Config配置方式
  
ApplicationContext支持热部署，一部分实现类可以使用refresh方法对容器进行刷新，即重新创建Bean容器，这样做的结果就是某些单例Bean也会被刷新，导致刷新前后的单例Bean并不在是同一个对象，如果你在刷新前后修改了xml配置文件中某个bean的id，那么有可能出现刷新后获取不到之前的id而报错的情况
  
案例，部署一个自定义的简单MVC结构，包含Dao，Service，Controller
  
applicationContext配置文件
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.2.xsd">
    <!--开启注解扫描-->
    <context:component-scan base-package="com.Qimiaozi"></context:component-scan>
    <bean id="userDao" class="com.Qimiaozi.UserDao" scope="singleton">
        <!--通过constructor这个节点来指定构造函数的参数类型、名称、第几个-->
        <constructor-arg index="0" name="version" type="java.lang.String" value="1.0.0"></constructor-arg>
    </bean>
    <bean id="userService" class="com.Qimiaozi.UserService" scope="singleton">
        <property name="userDao">
            <ref bean="userDao"/>
        </property>
    </bean>
    <bean id="userAction" class="com.Qimiaozi.UserAction" scope="prototype"></bean>
</beans>
```

Dao类
  java
```java
package com.Qimiaozi;

import org.springframework.stereotype.Repository;

@Repository
public class UserDao {

    private String version;

    public UserDao(String version) {
        System.out.println("当前数据库版本:"+version);
        this.version = version;
    }
    public void save(String userInfo) {
        System.out.println("信息成功保存 : "+userInfo);
    }
}
```
  
Service类安全
  
```java
package com.Qimiaozi;

import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

@Service
public class UserService {

    // @Resource(name = "userDao") 注解配置方法，也可以在xml里直接使用属性标签+set方法配置
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        System.out.println("使用标签配置，未使用注解");
        this.userDao = userDao;
    }

    public void save(String userInfo) {
        System.out.println("服务启动，信息保存中.....");
        userDao.save(userInfo);
    }
}
```
  
Controller类

```java
package com.Qimiaozi;

import org.springframework.stereotype.Controller;

import javax.annotation.Resource;

@Controller
public class UserAction {

    @Resource(name = "userService")
    private UserService userService;

    public void execute(String userInfo) {
        userService.save(userInfo);
    }
}
```
  
测试
  
```java
package com.QimiaoziTest;

import com.Qimiaozi.UserAction;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestDemo1 {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserAction action = (UserAction) ac.getBean("userAction");
        action.execute("name:jack age:18");
    }
}
```
  
运行结果
  
```
当前数据库版本:1.0.0
使用标签配置，未使用注解
服务启动，信息保存中.....
信息成功保存 : name:jack age:18

Process finished with exit code 0
```
  
ApplicationContext和BeanFactory的区别
  - ApplicationContext会自动注册后置处理器方法，BeanFactory需要手动ddBeanPostProcessor()注册
  - ApplicationContext的singleton是预先实例化的，BeanFactory在第一次访问时才实例化
  
## Bean装配
  
1.三种方法(常用注解+xml)安全
  - 注解
  - javaConfig
  - xml
  
2.依赖注入方式
  - 属性注入(使用无参构造函数实例化对象后调用setXXX方法注入)
  - 构造函数参数注入
  - 工厂模式注入(用的少)

3.bean对象之间的关系类型
  - ref：引用关系
  - parent：继承关系
  - depends-on：依赖关系
  
4.bean对象的scope：bean对象的成员的scope如果希望和bean对象不一样，直接在配置文件里设置成员的scope就可以
  - singleton：单例，这个单例指的是实例的单例，不是class的单例，单例模式下的bean如果是无状态的，则不涉及线程安全，否则线程不安全
  - prototype：多例，生成多个对象，没有线程安全问题
  - request：用作单个http请求作用域范围内
  - session：用作单个http session会话作用域范围内
  
5.装配方式
  - no：非注解默认，不进行自动装配，通过显式设置ref属性来进行装配
  - byName：通过参数名 自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byname，之后容器试图匹配、装配和该bean的属性具有相同名字的bean
  - byType:注解默认方式，通过参数类型自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byType，之后容器试图匹配、装配和该bean的属性具有相同类型的bean。如果有多个bean符合条件，则抛出错误
  - constructor：这个方式类似于byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常
  - autodetect：首先尝试使用constructor来自动装配，如果无法工作，则使用byType方式
  
## 常见注解
  
1.@Autowired：将依赖的Bean注入

2.@Qualifier("String")：根据指定BeanName注入依赖(解决相同接口不同实现类时选择的问题)
  
3.@Resource(name = "String")：同上，java自带注解
  
4.@Component：用于将当前class声明为一个Bean对象，@Controller, @Service, @Repository都属于@Component，只不过针对Dao，Service，Controller进行了语义的细分
  
5.@Primary：设置该Bean为其接口的不同实现类中的首选Bean(解决相同接口不同实现类时选择的问题)
  
6.@Configuration：该注解会把被注解的类转换成一个用于配置spring的类，相当与使用java编程式做spring配置而不需要xml配置了，一般要搭配@Bean和@ComponentScan使用
  
7.@Bean：加在方法上的注解，方法返回的对象会被spring认为是一个bean对象，该方法在spring中只会被调用一次，之后生成的bean对象会被存入ioc容器中管理，类似与xml中的bean标签
  
8.@ComponentScan(url)：指定扫描某个路径下的全部package中符合条件的Bean，相当与<context:component-scan/>标签
  
9@import(xxx.class)：该注解会把指定的@Configuration的class导入当前类，使得当前类可以使用xxx.class中全部定义的@Bean，并且加载时机跟随当前类，如果xxx。class不是@Configuration，则会在当前类被处理时，将xxx.class自动注册为一个Bean
  
## AOP

AOP：面向切面编程，将需要重复使用的代码在程序运行是从一个切点动态的插入到程序中，这类代码也叫切面类代码，用于将可重复使用的代码和业务代码分离，降低系统耦合读，增加代码重复利用
  
连接点(Join point)：能够被拦截的地方：Spring AOP是基于动态代理的，所以是方法拦截的。每个成员方法都可以称之为连接点

切点(Poincut)：具体定位的连接点：上面也说了，每个方法都可以称之为连接点，我们具体定位到某一个方法就成为切点。

增强/通知(Advice)：
  - 表示添加到切点的一段逻辑代码，就是你的aop要增强的代码，直白地说就是你要干啥，所以这个词其实翻译成增强比通知更准确
  - Spring AOP提供了5种Advice类型给我们：前置、后置、返回、异常、环绕给我们使用

织入(Weaving)：将增强/通知添加到目标类的具体连接点上的过程。

引入/引介(Introduction)：引入/引介允许我们向现有的类添加新方法或属性。是一种特殊的增强！

切面(Aspect)：切面由切点和增强/通知组成，它既包括了横切逻辑的定义、也包括了连接点的定义
  
AOP的实现原理：底层使用动态代理实现，根据被代理对象是否实现接口又分为jdk动态代理和cglib动态代理两种，如果被代理对象为singleton，推荐使用cglib，如果是prototype，推荐使用cglib，因为cglib创建慢但运行快，jdk动态代理则相反
  
srping对AOP的两种使用方法：
  - 注解(常用这个)
  - xml配置
  
## 依赖循环问题
  
1.问题描述：BeanA创建依赖BeanB，BeanB创建依赖BeanA，此时创建这两个Bean时会出现依赖循环导致程序崩溃
  
2.依赖循环的嗅探：spring维护一个singletonsCurrentlyInCreation的Set集合，当Bean被创建时会去遍历该Set，如果当前要创建的Bean在Set中，则说明发生了依赖循环(当前Bean不是首次被创建)，如果不在，则将当前要创建的Bean加入该Set
  
3.三级缓存：
  - 1.Map<String, Object> singletonObjects	用来存放已经完全创建好的单例bean
  - 2.Map<String, Object> earlySingletonObjects	用来存放早期的半成品bean
  - 3.Map<String, ObjectFactory<?>> singletonFactories	用来存放单例bean的ObjectFactory，相当与一个代理对象

4.三级缓存解决依赖循环过程：
  - 假设a依赖b，b依赖a
  - 首先创建a的半成品，将其放入3级缓存
  - 属性注入a时发现依赖b，则创建b的半成品，将其放入3级缓存
  - 属性注入b时发现依赖a，则去2级缓存中取，此时3级缓存中的a会被移动到2级缓存被b获取并注入
  - b创建完毕，将b从3级缓存移入1级缓存
  - 继续创建a，将1级缓存中的成品b注入给a，a创建完毕

5.三级缓存移动时机
  - 3级：创建好对象时就被放入，不需要属性注入完成，即半成品期间
  - 2级：当其他对象尝试将当前对象注入时，从3级缓存中移入
  - 1级：对象属性注入和初始化完成时移入，即成品时
  
6.三级缓存无法解决的情况：通过上面的过程可以观察到，三级缓存要求对象创建时不需要立刻注入属性，所以三级缓存无法解决构造器下依赖循环的情况
  
7.为什么要使用三级缓存
  - 不一定非要使用三级缓存，一级二级也可以，但是某些情况下必须使用三级缓存
  - 三级缓存主要是解决aop问题，因为二级缓存中保存的是半成品，而最终注入的对象是成品，如果对象进行了aop，那么实际上最终的成品是代理对象，而二级缓存中保存的是半成品，不是同一个对象
  - 所以要使用三级缓存提前创建代理对象，保证最终1级缓存对外暴露的对象和2级缓存中被提前注入的半成品对象是同一个！！！
  - 当然，没有aop或者代理的情况下，不需要三级缓存


  





  
  
  
