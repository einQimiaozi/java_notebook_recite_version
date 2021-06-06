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

2.mybatis架构

![mybatis](https://upload-images.jianshu.io/upload_images/9033085-1b3892beac79af63.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

