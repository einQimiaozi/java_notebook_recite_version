## fail-fast

所有线程不安全的集合都有该机制，用于检查错误

集合对象和迭代该集合的迭代对象各自维护一个modcount，每次修改集合结构的时候执行modcount++，当两个modcount不一致时说明发生了数据同步问题，会抛出ConcurrentModifactionExcaption错误。

## ArrayList

继承自Abstractlist

构造函数为空的Object数组，protected修饰，transient修饰

transient：使得实现了Serilizable接口的对象中的被修饰属性不可序列化，保证数据在传输过程中的安全性

第一次执行add操作时扩容，默认大小为10,容量上限为Integer.MAX_VALUE-8，也可以指定容量

扩容时使用位操作(int new capacity = oldcapacity + oldcapacity>复杂度>1)，扩容大小为1.5倍，扩容后如果小于指定大小就按指定大小扩容，超过最大容量抛出OOM。

底层为数组，占用连续内存。

插入和删除如果不是尾部追加，或者扩容时，使用System.arraycopy方法移动出插入位置，消耗较大，需要频繁插入删除的场景不适合ArrayList，如果能够根据应用场景预先设置好大小可以避免频繁移动数组

缩容操作不会自动触发，需要手动调用，使用syste.arraycopy移动，缩容后大小为当前size。

实现了RandomAccess接口。

支持插入null元素。

复杂度：

  - get O(1)
  - add(E) O(1)
  - add(index,E) O(n)
  - remove() O(n) 

## LinkedList

底层是双向链表结构，元素节点使用protected和transient修饰。

支持插入null元素。

对其他Collection对象的批量插入采用Collection.toArray方法(遍历，转成Object数组返回)，不断尾插，使用整型size记录链表长度。

尾部追加元素使用尾插法，其他情况下使用头插法(调用node方法，该方法会根据index判断当前插入的位置在链表的前半段还是后半段，遍历返回该位置，因为插入后该位置实际上要后移，所以使用头插法)。

支持按元素查询，会先判断元素是否为null，因为null和普通元素的判断相等的方法不同(==和equals)，判断后根据情况遍历返回第一个满足条件的节点。

不需要扩容，插入删除效率高，查找效率低。

复杂度：

  - get O(n)
  - add(E) O(1)
  - add(index,E) O(n/2)
  - remove O(1)

## HashMap

底层构造函数是空的Node数组，也叫哈希桶，protected和transient修饰，还有一个加载因子，默认为0.75，Node对象结构如下(单向链表)

```java
static class Node<k,v> implements Map.Entry<k,v> {
  final int hash; // 哈希值
  final k key;
  v value;
  Node<k,v> next; // 后继节点
}
```

每一个节点的hash值通过key的hash值和value的hash值异或得到

第一次add执行初始化，threshold=容量×加载因子，超过threshold会触发扩容

默认容量为16,每次两倍扩容(位运算 int newcapacity = oldcapacity>>1),容量必须为2的n次方，可以指定容量，底层会调用tablesizefor方法返回一个大于并最接近指定容量的容量(使用或+位运算)

当桶内链表长度大于7时转换结构为红黑树，小于7时降级为链表，等于7时不做任何操作

```java
static final int tablesizefor(int cap) {
   int n = cap-1;
   n |= n>>1;
   n |= n>>2;
   n |= n>>4;
   n |= n>>8;
   n |= n>>16;
   return (n<0)? |= (n>=MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY = n+1; // 边界检查
}
```

hash桶位置计算方法：元素的hash值模以容量(源码中使用位操作 [e.hash & cap-1]),因此容量必须是2的n次方(这样才能用位操作取模)

扩容过程

  - 1.判断是否要初始化
  - 2.边界判断(hashmap的容量上限为2^31-1)
  - 3.计算扩容后容量(位操作)
  - 4.如果未初始化则执行初始化，如果未初始化但是有threshold，则初始化同时设置当前的容量为threshold
  - 5.计算新阈值并构建新的hash桶
  - 6.如果旧hash桶内有元素，则遍历老桶，每次遍历将引用指向null，方便GC
  - 7.由于每次扩容2倍，所以老桶内元素的下标应该不变or变为[老桶长度+原来的下标]，如果没有hash冲突则在原下标(此时初始化一个链表作为桶内结构)，若发生hash冲突则计算当前hash和老桶的大小，小于则还在原来的位置，大于则在新位置(源码里使用位操作 if((e.hash & oldcap)==0))
  - 8.使用两个指针记录链表的头尾节点，尾插法插入元素，处理完链表之后放入对应下标




