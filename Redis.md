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

![skip1](https://pcsdata.baidu.com/thumbnail/f9ddbe8bevb3c42a5a8b77029ec41dd7?fid=1508469986-16051585-1109070741150446&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-j%2FhERxeb47SE5InD1fitaHfwh1c%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=1961763278&dp-callid=0&time=1617973200&size=c1600_u1600&quality=100&vuk=-&ft=video)

![skip2](https://pcsdata.baidu.com/thumbnail/2aa07a075l57a24ccb752339d9823bc9?fid=1508469986-16051585-466657229385980&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-l3WbK%2FyMUz%2FsLDcPPkiDN87n7jo%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=1961763278&dp-callid=0&time=1617973200&size=c1600_u1600&quality=100&vuk=-&ft=video)

图片转自知乎

为什么不用红黑树/avl：
  - 1.并发环境下调整节点的操作容易产生线程安全问题
  - 2.跳跃表实现更简单

## pub/sub

消息发布和消息订阅，在redis中，你可以对某一个key进行消息发布和订阅，当该key进行发布后，所有订阅它的客户端都会收到对应的消息。

## 持久化(RDB)

RDB是Redis用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。

1.自动触发：通过配置save命令到redis.conf配置文件中，根据save命令后接的参数决定自动触发的时间和频率

2.手动触发
  - 1.save命令:阻塞式持久化，该命令会调用主进程把数据库状态写入RDB命令文件(dump.rdb)中，文件创建完毕之前redis不能处理其他的任何请求和事务
  - 2.bgsave命令：非阻塞持久化，该命令会创建一个子进程(父进程的复制品)专门处理数据库状态的写入，全程只有fork父进程的时候才会有瞬间的阻塞

缺点：需要一定时间内做一次备份，如果redis意外down掉的话，就会丢失最后一次快照后的所有修改(数据有丢失)。对于数据完整性要求很严格的需求

## 持久化(AOF)

通过将数据库的操作记录写入AOF文件来完成持久化

1.append only file：该命令将操作写入appendonly.aof文件内，os在缓冲区满之后会一次性刷入磁盘(也就是说对于redis来说是写入磁盘，但是os实际上不会每次都写入磁盘)，使用主进程fork一个子进程来并发处理，将重写对服务器的影响降低到了最小。
  - no：每30秒同步一次
  - everysex：每秒同步一次
  - always：每接受到一条新命令就同步一次
  - aof缓冲区：子进程创建后如果主进程处理了新的命令，则会将该操作写入aof缓冲区，子进程会将缓冲区的内容也写入aof文件，保证重写的实时性

