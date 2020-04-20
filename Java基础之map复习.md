# Java基础之map复习

![1587347164422](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\1587347164422.png)

## 比较HashTable和HashMap

- ### 相同点

  都实现了Map接口，都采用key-value机制。

- ### 不同点

  1. 虽然两者都支持键值为空，但是HashMap允许键值为空（只允许一条记录的键值为空，不允许多条记录的键值为空），但是HashTable不允许。

  2. HashMap把HashTable的contains方法去掉了，将其改为了containsKey和containsValue，因为contains方法容易引起歧义。

  3. 继承的类不同。HashTable继承自Dictionary类，而HashMap继承自AbstractMap类。

  4. HashTable是线程安全的，HashMap不是线程安全的。当多个线程访问HashTabl时，不需要开发人员对其进行同步；但是对于HashMap开发人员必须提供额外的同步机制。

  5. 根据第四点线程安全机制的分析可知，HashMap的使用效率可能高于HashTable。

  6. **fail-fast机制：**快速失败是Java集合的一种错误检测机制。当多个线程对集合进行结构上的改变的操作时，就可能会产生fail-fast事件。例如，现在存在两个线程，他们分别是线程1和线程2，当线程1通过使用Iteator（迭代器）遍历集合A的时候，如果线程2修改了集合A的结构（增加或者删除元素），那么这个时候程序就会抛出ConcurrentModificationException异常，从而产生fail-fast事件。

     由于HashTable是线程安全的，因此没有采用快速失败机制，HashMap是非线程安全的，因此，迭代器采用了快速失败机制。

- ### 注意点

   HashTable它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了**分段锁**。

  **（什么是分段锁？）** 