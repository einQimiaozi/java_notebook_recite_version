## 常见数据类型

1.String：二进制安全(就是可以表示任何数据)，最大上限512m，也叫sds，本质是个可以动态扩容的byte数组，每次扩容大小为2n+1

2.Hash：kv集合，String的kv映射，适合存储对象

3.Set：无序集合，底层实现是value为null的HashMap，通过计算Hash值快速重排和快速判断成员是否在集合内，适用于需要取交集的场景，比如共同好友之类的

4.sorted Set:有序集合，底层使用hashmap实现，每个元素对应一个score，依靠score排序，排序后的元素放在跳跃表里，即通过hashmap决定存储结构，通过跳跃表保持有序性，适用于用户评分榜(id:socre)

5.List：字符串双向链表，缓冲区的实现基础

6.zipList：当数据较少时可以用zipList代替List和Hash，zipList本质是个数组，结构如下

|zlbyte|zltail|zllen|数据本身|zlend|

- zlbyte:数据大小 4字节
- zltail：末尾指针(偏移量) 4字节
- zllen:数组长度 2字节
- zlend：结束符 1字节

## 跳跃表

多层链表，查询的时候可以一次性跳跃多个节点，实现O(logn)的查询速度，本质是用空间换时间

插入节点时使用一个随机数作为插入层数，例如插入7时随机层数为4,则在1到4层都可以找到7这个节点，但是在其他层就看不到了

![skip1](https://gitee.com/wang_fu_gui222/image-hosting-on-github/blob/master/skip1.jpg)

![skip2](https://gitee.com/wang_fu_gui222/image-hosting-on-github/blob/master/skip2.jpg)

图片转自知乎

为什么不用红黑树/avl：
  - 1.并发环境下调整节点的操作容易产生线程安全问题
  - 2.跳跃表实现更简单

## pub/sub

消息发布和消息订阅，在redis中，你可以对某一个key进行消息发布和订阅，当该key进行发布后，所有订阅它的客户端都会收到对应的消息。
