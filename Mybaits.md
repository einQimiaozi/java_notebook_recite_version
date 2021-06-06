## JDBC

JDBC本质上是一个java提供的多数据库统一访问接口

使用流程：
  - 1.加载指定数据驱动
  - 2.创建链接对象(DriverManager.Connection)
  - 3.getConnection获取连接并键入用户名和密码
  - 4.createStatement创建stateMent对象用于执行sql语句并获取结果
  - 5.Conncetion.close()关闭链接

## MyBatis

MyBatis是JDBC的一种上层封装，简化了一些JDBC的链接，异常处理等操作，可以使开发聚焦与sql语句本身

1.ORM：对象关系映射，将关系性数据库和对象之间进行映射，这样可以用操作对象的方式进行sql操作，mybatis是一种半自动orm工具(半自动是因为sql语句还要手动编写)

2.mybatis结构

![mybatis](https://upload-images.jianshu.io/upload_images/9033085-ca3974e212f792a0.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

  - sqlSession:负责和数据库交互的顶层API
  - Executor:负责sql语句生成和查询缓存维护
  - StateMentHandler:对JDBC的Statement封装，负责设置参数返回结果等
  - ParameterHandler：负责将用户传递的参数专程JDBC需要的参数
  - ResultSetHandler：负责将JDBC查询的结果转成List
  - TypeHandler：负责java数据类型和JDBC数据类型之间的映射
  - MappedStatement：本质上就是个sql语句的封装
  - Configuration：Mybatis中的配置信息

3.Mybatis缓存
  - 一级缓存：sqlSession缓存，多个不同的sqlSession之间相互隔离，默认开启，和mysql中的缓存一样，底层是一个Hashmap，该缓存的存储介质是内存，sqlSession执行增删改操作时会自动清除缓存，也可以通过clearCache或xml中配置flushCache清除
  - 二级缓存：mapper级别缓存，和mapper中的namespace绑定，这样多个同样namespace下的sqlSession就可以共用同一个二级缓存，优先于一级缓存访问

