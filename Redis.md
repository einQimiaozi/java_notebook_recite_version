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
  - 1.并发环境下调整节点的操作容易产生线程安全问题(所谓的支持无锁操作)
  - 2.跳跃表实现更简单
  - 3.不需要旋转节点，插入极快

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

## redis的事件处理模型

1.6.0之前的redis使用io多路复用处理事件，该模型被称为事件处理器，包含了多个socket管理客户端链接，io多路复用程序，文件事件分派器，事件处理器四个模块

2.执行流程：
  - 文件处理器基于io多路复用模型监听多个socket
  - 当socket的读/写/应答/关闭等操作任意之一就绪时，多路复用程序将socket通过事件分派处理器分派给与socket相关连的事件处理器进行处理

3.非阻塞执行流程(redis6.0后支持非阻塞，并发只存在于网络事件io，其他部分还是单线程，不存在线程安全问题)：
  - 主线程建立链接请求，获取socket放入全局等待读队列
  - 主线程处理完读事件之后通过RR策略将等待队列平均分给io线程们(socket和线程绑定)
  - 主线程阻塞，等待io线程读取socket完毕
  - io线程组并行对socket解析(只解析不执行)，主线程阻塞，解析全部完成后主线程执行所有请求命令，将数据写入缓冲区
  - 主线程阻塞等待socket回写完毕后解除等待队列和线程的绑定，清空等待队列

4.为什么不用多线程
  - redis的性能瓶颈不在cpu而在io和内存
  - 单线程容易编程和维护
  - 死锁和上下文切换会影响性能

5.为什么后来加入多线程
  - 加入的多线程只负责网络io，解决io瓶颈问题

## 事务指令

1.multi：标记事务块的开始，总是返回ok，redis会将后续命令加入到执行队列中再执行exec

2.watch：用于监听一个或多个key，如果事务exec之前该key被修改则触发事务打断，exec执行结果返回null

3.exec：执行所有被放入执行队列中的事务，执行结束后恢复正常链接状态，使用cas同步，执行成功则返回该队列中所有原子化事务的执行结果组成的数组

4.discard：取消事务块内所有命令的执行，总是返回ok

redis的事务不支持rollback，不满足原子性和持久性

## key的过期删除策略

redis使用key+该key的过期时间组成一个过期字典，通过查询该字典判断key是否过期，使用expire设置

1.定期删除：定期检查一部分key是否过期，需要人工设置检查频率，设置的好则能平衡cpu和内存的消耗，设计的不好会导致获取到一个已经过期的key

2.定时删除：创建多个定时器对每个key进行检查，一旦发现过期立刻删除，频繁删除不cpu友好，对内存友好

3.惰性删除：读取key时才检查是否过期，删除频率低对cpu友好，对内存不友好

4.redis默认使用定期删除+惰性删除，平衡cpu和内存，同时惰性删除可以避免获取到一个过期的key

## 内存淘汰策略

1.volatile-lru：利用LRU算法移除最近使用较少的(only expire)

2.allkeys-lru：利用LRU算法移除最近使用较少的

3.volatile-random:瞎jb删(only expire)

4.allkeys-random:瞎jb删

5.volatile-ttl：优先移除快过期的(only expire)

6.noeviction:默认策略，oom后直接抛错误，别的什么也不做





