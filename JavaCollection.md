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

复杂度：

  - get O(1)
  - add(E) O(1)
  - add(index E) O(n)
  - remove() O(n) 


