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

TreeNode：继承Node，红黑树节点，保存了左右子树，父亲节点及其链表下的next节点

TreeBin：TreeNode的容器，也继承自Node，为TreeNode提供了一些条件和锁的控制

实际上hash槽中存放的是TreeBin而不是TreeNode(红黑树下)，有了TreeNode的next节点指针，TreeBin就可以找到其下一个节点
