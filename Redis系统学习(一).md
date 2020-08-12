# Redis系统学习(一)

## 数据类型

- **String(字符串)**

   **简述：**可以包含任何数据，如jpg或者序列化的对象，一个键最大能存储512M。

- **Hash(字典)**

   **简述：**键值对集合，即编程语言中的Map类型。

   **特性：**适合存储对象，并且可以像数据库update一个属性一样只修改某一项属性值。

  ​            （Memcached中需要去除整个字符串反序列化成对象修改完再序列化存回去。）

   **应用场景：**存储，读取，修改用户属性。

- **List(列表)**

   **简述：**链表（双向链表）

   **特性：**增删快，提供了操作某一段元素的API

   **应用场景：**最新消息排行等功能（比如朋友圈的时间线）；消息队列

- **Set(集合)**

   **简述：**哈希表实现，元素不重复。

   **特性：**添加，删除，查找的复杂度都是O(1);为集合提供了求交集，并集，差集等操作。

   **应用场景：**1.共同好友 

  ​                    2.利用唯一性，统计访问网站的所有独立ip 

  ​                    3.好友推荐时。

  

- **Sorted Set(有序集合)**

​       **简述:**  将Set中的元素增加一个权重参数score，元素按照score有序排列。

​       **特性**：数据插入集合时，已经进行天然排序。

​       **应用场景：**1.排行榜；2.带权重的消息队列。

### 5中数据结构的底层实现：

- **String**

   底层是一个redis定义的简单动态字符串SDS

- **List**

  底层是一个双向链表。其中的具体属性有：

  - head/tail 节点分别代表是头节点和尾节点。所以获取头节点和尾节点中的值的时间复杂度均为O(1)。

    头节点的prev和尾节点的next均为null。

  - 链表中除了头节点和尾节点的普通节点均带有prev和next，所以获取一个普通节点所需要的时间复杂度为O(N).

  - len--- 链表中存有记录链表长度的属性len。

  - 多态： 链表节点使用void* 指针来保存节点值，并且可以通过list结构的dup，free，match三个属性为节点值设置类型特定函数。

- **hash**

  底层是通过字典实现的，字典，又称为关联数组，符号表，或者映射。

  内部结构：

  ```c++
  typedef struct dictEntry {
      void *key;
      union {
          void *val;
          uint64_t u64;
          int64_t s64;
          double d;
      } v;
      struct dictEntry *next;
  } dictEntry;
  
  typedef struct dictType {
      unsigned int (*hashFunction)(const void *key);
      void *(*keyDup)(void *privdata, const void *key);
      void *(*valDup)(void *privdata, const void *obj);
      int (*keyCompare)(void *privdata, const void *key1, const void *key2);
      void (*keyDestructor)(void *privdata, void *key);
      void (*valDestructor)(void *privdata, void *obj);
  } dictType;
  
  /* This is our hash table structure. Every dictionary has two of this as we
   * implement incremental rehashing, for the old to the new table. */
  typedef struct dictht {
      dictEntry **table;
      unsigned long size;
      unsigned long sizemask;
      unsigned long used;
  } dictht;
  
  typedef struct dict {
      dictType *type;
      void *privdata;
      dictht ht[2];
      long rehashidx; /* rehashing not in progress if rehashidx == -1 */
      int iterators; /* number of iterators currently running */
  } dict;
  ```

  #### 原理讲解：

  #### 字段：

  **ht字段：**

  ht是一个两个项的数组，其中ht[0]用于存放哈希表，ht[1]在哈希重排时使用。

  **rehashidx字段：**

  当rehashidx==-1时，表示当前不是rehash，用于表示在进行rehash操作时，进行到了哪一步。

  #### 如何操作数据

  假设现在需要添加一个数据<k,v>,首先需要计算哈希值：

  ```c
  hash = dict->type->hashFunction(k);
  ```

  然后根据sizemark计算出索引值：

  ```c
  index = hash & dict->ht[x]->sizemark
   //x表示实际存放哈希表的索引，一般为0，当在进行rehash时为1.
  ```

  这样就可以将 <k,v>存储在 dict->ht[x]->table[index]中，如果table[index]中已经有数据，则新添加的数据防砸i链表表头。

  （这是因为table[index]中的链表是一个单链表，没有指向链表尾部的指针，添加到表头更快）

  #### rehash过程

  当哈希表中保存的数据太多或者太少时，**需要对哈希表进行相应的扩容或者收缩。**

  如果进行扩容操作，**那么ht[1]的大小第一个不小于ht[0].used*2的2^n(n为正整数)**，

  如：used=5,ht[0].used*2 = 10<2^4 = 16,所以ht[1]的大小为：16

  **然后就可以将ht[0]的数据哈希到ht[1]中，当ht[0]中得数据全部哈希到 ht[1]后，释放ht[0]，将ht[1]变为ht[0].**

  #### 扩展的触发条件

  1.在没有执行bgsave或bgrewriteaof时，哈希表的负载因子>=1

  2.在执行bgsave或bgrewriteaof时，哈希表的负载因子>=5

​      注意：

​       在进行bgsave或bgrewriteaof时，提高负载因子是为了避免扩容，避免不必要的内存写入。

####      收缩的触发条件

​        负载因子<0.1

####       渐进式哈希

​         在扩容或者收缩时，需要将ht[0]全部哈希到ht[1]中，如果一次性哈希，当数据足够时，会存在效率问题。因此redis通过逐步哈希的方式。具体过程如下：

        1. 为ht[1]分配空间
           2. 设置 rehashidx = 0
           3. 新添加的数据不在加入到ht[0]，而是加入到ht[1]中，保证ht[0]不会增加数据。
           4. 将ht[0]->table[rehashidx]的数据全部哈希到ht[1]中
           5. rehashidx++
           6. 随着不断的执行，ht[0]的数据哈希到了ht[1],这时可以设置 rehashidx = 1，表明rehash结束，释放ht[0],设置ht[1]为ht[0].

- **zset**

  zset中的基本数据结构使用的是跳跃表(skiplist)

  