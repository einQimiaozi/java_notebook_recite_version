## 结构

zookeeper是一个树形结构的文件系统，每个节点称为znode，每个znode最大支持存储1mb大小

## 功能

1.统一命名管理：对多台分布式服务器进行统一命名，外部客户端访问时只需要访问相同地址，zookeeper内部采用负载均衡方法调用请求到不同服务器上

2.配置监听：例如redis等服务器的配置可以设置为znode，zookeeper监听znode实时监控redis配置的变化

3.服务器上下线监控：通过监控服务器节点状态在服务器下线时实时通知客户端

## 选举

zookeeper集群由一个leader和多个follower组成

### 第一次启动选举

1.每个节点有一票，节点会投票给自己

2.对比每个节点的myid(在zookeeper的data中可以配置)，myid大的节点可以将myid小的节点的票抢走

3.票数最多的且超过半数投票的节点当选，leader选举出来后再有新节点加入也不再选举

4.如果所有节点票数都不超过节点数量的一半，则节点进入looking状态等待新节点加入

举例：节点1,2,3,4,5的myid分别为1,2,3,4,5

初次启动时节点1,2加入集群，票数情况

```
节点1 | 节点2 |
  1  |   1   |
```

此时节点进入looking状态

根据myid分配之后

```
节点1 | 节点2 |
  0  |   2   |
```

节点3,4加入

```
节点1 | 节点2 | 节点3 | 节点4 |
  0   |  2   |   1  |   1  |
```

再次根据myid分配

```
节点1 | 节点2 | 节点3 | 节点4 |
  0   |  0   |   0  |   4  |
```

此时节点4当选，节点5加入

```
节点1 | 节点2 | 节点3 | 节点4 | 节点5 |
  0   |  0   |   0  |4 leader|   1   |
```

### 非首次选举

再zookeeper的运行过程中如果节点发生变化则可能触发再次选举

zookeeper依靠节点之间的心跳检测嗅探其他节点，如果follower嗅探leader失败则会发起新选举

zookeeper中除了myid还有任期和事务id，任期是一个leader当选后+1,如果此时没有leader，则全部节点任期一致，事务id再每次提交事务时+1，可以根据事务id的大小判断当前节点是否同步到了最新状态，比如leader的事务id=10,follower2的事务id=8,那么follower的数据并未同步到最新状态

情况1：follower发生了分区错误或自己网络阻塞，可通过其他follower同步leader状态，此时不需要重新选举

情况2：leader挂了，进行再次选举，选举规则如下

  - 任期大的优先
  - 任期相同时事务id大的优先
  - 事务id相同时myid大的优先

## 节点类型

zookeeper中的所有数据视为节点，也叫znode，zookeeper保证znode的插入是原子性的

1.无编号持久型：znode设置后客户端和服务器断开连接该节点的数据也不会被删除，不能重复创建

2.有编号持久性：znode设置后客户端和服务器断开连接该节点的数据也不会被删除，可以重复创建，每次创建后会在节点名称后加一个自增编号，改编号不会自减

3.无编号临时型：znode设置后客户端和服务器断开连接该节点的数据会被删除，不能重复创建

4.无编号临时型：znode设置后客户端和服务器断开连接该节点的数据会被删除，可以重复创建，每次创建后会在节点名称后加一个自增编号，改编号不会自减

## 监听器

zookeeper通过监听器监听znode状态，监听器默认设置一次后只能监听一次变化

1.在程序的主线程中创建zookeeper客户端，该客户端会创建两个线程，一个负责建立连接(connect)，另一个负责监听(listenser)

2.connect线程连接成功后会将客户端要监听的事件注册到zookeeper的监听列表中，该事件的数据结构为 客户端:ip:port:事件路径

3.zookeeper监听到事件发生变化会通过process方法回调给listenser，listenser将事件变化返回给客户端完成监听

## zookeeper请求

如果请求发送到leader：leader会将请求转发给follower，follower写入成功会ackleader，超过半数follower应答，则leader认为写入成功，ack给客户端

如果请求发送到follower，follower会将请求转发给leader，leader负责转发给其他follower，超过半数follower应答，则leader将写入成功ack给和客户端通信的follower，由该follower应答客户端

## 分布式锁

分布式锁是zookeeper的典型应用，因为zookeeper对znode的操作原生支持原子性，可临时化节点并自动编号

  - 1.首个连接zookeeper服务器的客户端创建一个rootlock的znode作为锁，在root下尝试创建znode001作为锁
  - 2.其他客户端连接到zookeeper服务器后获取rootlock下的全部children并创建一个znode00x，如果自己创建的znode00x是最小的节点，则成功获取锁
  - 3.否则客户端阻塞(这里可以使用你语言里自带的sync的各种方法，根据你的业务逻辑选择)并监听znode00x-1(它的前一个znode)
  - 4.znode00x-1释放后znodex就会成为最小节点，此时获取rootlock成功
  - 5.处理完业务逻辑后delete掉znode00x完成锁释放

## paoxs算法

paoxs保证zookeeper满足cap理论中的cp

一个Paxos系统中，首先将所有节点划分为Proposer（提议者），Acceptor（接受者），和Learner（学习者）。（注意：每个节点都可以身兼数职）

  - 1.Proposer向多个Acceptor发出Propose请求承诺，每个承诺有一个全局递增唯一id，一个Acceptor如果接收过承诺3,那么它就不能接受承诺1或承诺2,只能接收承诺3或3以上的id
  - 2.Proposer收到多数Acceptor的承诺后，向Acceptor发出提案(具体要执行啥，过半通过决议)，每个提案也有全局递增唯一id，如果Acceptor接受了提案3,那么它就不能接受提案1，提案2和提案3,只能接收提案4及更大的提案id，Acceptor接收提案后，会将提案id和提案value作为一个kv对持久化到自己的磁盘里
  - 3.Acceptor如果收到多个Proposer的请求承诺，并且id递增，那么会覆盖之前的承诺，提案也是如此，可以理解成毁约，如果Proposer的承诺被毁约并且此时承诺数小于paoxs中总节点数的一半，Proposer会重新向未接收自己承诺的Acceptor发起请求承诺
  - 4.Learner执行决议通过的提案
  - 5.如果paoxs内的Proposer，Acceptor数量总和为奇数(比如两个Proposer，三个Acceptor)，那么有可能形成死锁
    - Acceptor1承诺Proposer1，Acceptor2承诺Proposer2，此时Acceptor3会不断接收Proposer1和Proposer2的承诺，然后不断毁约，Proposer1和Proposer2会不断重新发起请求承诺，造成系统死锁

## zab协议

zab协议是一种类似paoxs的算法，也是zookeeper底层实现分布式的算法

ZAB协议针对事务请求的处理过程类似于一个两阶段提交过程

（1）广播事务阶段

（2）广播提交阶段

广播事务阶段：
  - （1）客户端发起一个写操作请求
  - （2）Leader服务器将客户端的请求转化为事务Proposal 提案，同时为每个Proposal 分配一个全局的ID，即zxid
  - （3）Leader服务器为每个Follower服务器分配一个单独的队列，然后将需要广播的 Proposal依次放到队列中去，并且根据FIFO策略进行消息发送。
  - （4）Follower接收到Proposal后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向Leader反馈一个Ack响应消息。
  - （5）Leader接收到超过半数以上Follower的Ack响应消息后，即认为消息发送成功，可以发送commit消息。

广播提交阶段

  - （1）Leader向所有Follower广播commit消息，同时自身也会完成事务提交。Follower 接收到commit消息后，会将上一条事务提交。
  - （2）Zookeeper采用Zab协议的核心，就是只要有一台服务器提交了Proposal，就要确保所有的服务器最终都能正确提交Proposal

leader崩溃恢复

1.重新选举：
  - 新选举出来的Leader不能包含未提交的Proposal
  - 新选举的Leader节点中含有最大的zxid

2.数据恢复
  - 新leader会首先确认事务日志中的所有的Proposal 是否已经被集群中过半的服务器Commit
  - leader会ping所有follower，保证Follower将所有尚未同步的事务Proposal都从Leader服务器上同步过，并且应用到内存数据中以后，Leader才会把该Follower加入到真正可用的Follower列表中

总结：leader崩溃恢复保证所有已经提交的提案必须先被follower执行完毕，所有前任leader提交失败的提案全部被抛弃

## zookeeper常见面试题

6.1 选举机制：半数机制，超过半数的投票通过，即通过。

（1）第一次启动选举规则：投票过半数时，服务器 id 大的胜出

（2）第二次启动选举规则：
  - ①EPOCH 大的直接胜出
  - ②EPOCH 相同，事务 id 大的胜出
  - ③事务 id 相同，服务器 id 大的胜出
  - 
6.2 生产集群安装多少 zk 合适？安装奇数台。

生产经验：
  - ⚫ 10 台服务器：3 台 zk；
  - ⚫ 20 台服务器：5 台 zk；
  - ⚫ 100 台服务器：11 台 zk；
  - ⚫ 200 台服务器：11 台 zk

服务器台数多：好处，提高可靠性；坏处：提高通信延时

6.3 常用命令：ls、get、create、delete



