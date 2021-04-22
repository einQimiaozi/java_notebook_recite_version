## 重要参数

1.最大容量:Integer.MAX_VALUE-8

2.默认容量16

3.加载因子0.75

4.链表和红黑树转换阈值8

5.帮助扩容的最大线程数2^15-1

6.forwarding的node的hash值:int MOVED = -1

7.private transient volatile int sizectl
  - 该值为0时：未初始化
  - 该值为-1时：正在初始化
  - 该值为-n时：n-1个线程正在扩容
  - 该值为n时：下次大小为n时扩容

## 内部构造

Node：链表节点，concurrenthashmap的构造基本单元，可读不可写

```java
// 数组，volatile控制同步
transient volatile Node<K,V>[] table;
```

TreeNode：继承Node，红黑树节点，保存了左右子树，父亲节点及其链表下的next节点

TreeBin：TreeNode的容器，也继承自Node，为TreeNode提供了一些条件和锁的控制

实际上hash槽(table)中存放的是TreeBin而不是TreeNode(红黑树下)，有了TreeNode的next节点指针，TreeBin就可以找到其下一个节点

## put

1.put操作本身是建立在对当前table的死循环之上的

2.如果没有初始化则先进行初始化(通过sizectl判断)

3.像hashmap一样计算索引位置，若没有hash冲突则插入(这里还是乐观锁)

4.若发现当前table在进行扩容(通过hash值==MOVED来判断，即当前的node是不是forwardingnode)，就挂起当前插入操作，当前线程帮助扩容

5.若存在hash冲突，则使用synchronized块进行同步插入(这里变为真正的锁)

6.如果插入后发现当前table长度小于64且链表节点数超过8,则扩容，若长度大于64且链表节点超过8个，则链表转红黑树(调用treeifybin方法)

7.插入成功后判断是否需要扩容

## 初始化

1.若sizectl<0,则当前线程挂起，等待table初始化完毕

2.否则使用cas对sizectl-1,声明当前线程要开始初始化了，并初始化table并使用sizectl记录下次扩容的大小

## helperTransfer(帮助扩容)

该方法会调用多个当前正在对table进行操作的线程一起帮助扩容，内部调用了transfer方法

transfer方法会让每个线程在处理到一个节点时，将该节点置为forwardingNode(将其hash值置为MOVED)，这样其他线程访问到该节点时会直接跳过

## get

1.计算当前key的hash值，根据该值找到索引位置

2.若get时遇到扩容且该节点正好是ForwardingNode的话，那么会调用ForwardingNode的find方法查找该节点并返回

3.若不在扩容，则根据索引按照链表or红黑树的方法遍历，找到节点并返回

## 总结

1.jdk1.7中为分段加锁，即多个node为一段，一起加一个锁，粒度粗，使用lreentrantlock，1.8后基于node加粒度更细的锁，使用synchonized+volatile控制同步

2.不允许kv为null，因为如果允许key为null，那么多线程下你不知道是key=null还是查找失败，在单线程下是可以他哦难过map.containsKey(key)来判断的，但多线程下调用该方法前后可能会有其他线程对table进行了修改导致结果线程不安全

3.扩容总结：
  - 先new一个nextTable，2倍于旧table，单线程完成
  - 多线程遍历处理所有节点
  - 其他部分和hashmap基本相同
