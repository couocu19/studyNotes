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

- ### **String**

   底层是一个redis定义的简单动态字符串SDS

- ### **List**

  底层是一个双向链表。其中的具体属性有：

  - head/tail 节点分别代表是头节点和尾节点。所以获取头节点和尾节点中的值的时间复杂度均为O(1)。

    头节点的prev和尾节点的next均为null。

  - 链表中除了头节点和尾节点的普通节点均带有prev和next，所以获取一个普通节点所需要的时间复杂度为O(N).

  - len--- 链表中存有记录链表长度的属性len。

  - 多态： 链表节点使用void* 指针来保存节点值，并且可以通过list结构的dup，free，match三个属性为节点值设置类型特定函数。

  ![img](https://upload-images.jianshu.io/upload_images/9613667-0fd0a58ec798191f.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

- ### **hash**

  底层是通过字典实现的，字典，又称为关联数组，符号表，或者映射。

  redis的哈希对象的底层可以使用ziplist（压缩列表）或者HashTable（哈希表）。

  **当hash对象可以同时满足以下两个条件时，哈希对象使用压缩列表编码。**
  
    1.哈希对象保存的所有键值对的键和值的长度都小于64字节。
  
    2.哈希对象保存的键值对的个数小于512个。
  
  
  
  内部结构：
  
    redis的hash架构就是标准的hashtable的结构，通过**挂链**（链地址法）解决冲突问题。
  
  ```c++
  //hash表一个节点包含key，value数据对
  
  //哈希表节点
  typedef struct dictEntry {
      void *key; //键
      //值
      union {
          void *val;
          uint64_t u64;
          int64_t s64;
          double d;
      } v;
      struct dictEntry *next;  //指向下一个节点，链接表的方式解决hash冲突
  } dictEntry;
  
  //存储不同数据类型会对应不同操作的回调函数
  typedef struct dictType {
      unsigned int (*hashFunction)(const void *key);
      void *(*keyDup)(void *privdata, const void *key);
      void *(*valDup)(void *privdata, const void *obj);
      int (*keyCompare)(void *privdata, const void *key1, const void *key2);
      void (*keyDestructor)(void *privdata, void *key);
      void (*valDestructor)(void *privdata, void *obj);
  } dictType;
  
  //哈希表：每个字典都使用两个哈希表，以实现渐进式rehash
  typedef struct dictht {
    dictEntry **table; //哈希表数组，数组的每个项都是entry链表的头结点(链地址法解决哈希冲突)
      unsigned long size; //hash表的总大小
  unsigned long sizemask; //计算在table中索引的掩码，值是size-1
      unsigned long used; //hash表已使用的大小
  } dictht;
  
  //字典
  typedef struct dict {
      dictType *type; //典型特定函数，不同数据类型对应不同的函数
      void *privdata; //私有数据
      dictht ht[2];  //两个hash表，rehash时使用
      long rehashidx; /* rehash的索引，-1表示没有进行rehash */
      int iterators; 
  } dict;
  
  ```



  #### 原理讲解：

  #### 字段：

  **ht字段：**

  ht是一个两个项的数组，其中ht[0]用于存放**哈希表**，ht[1]在**哈希重排**时使用。

 **维护两个数组的作用？**

答：相当于维护了一对滚动数组，分为一张新表和一张旧表，当hashtable的大小需要改变时，就将旧表中的数据向新表中迁移，当下一次变动数组大小时，新表代替旧表，这样就达到了资源的重复利用和效率的提升。

  **rehashidx字段：**

因为是渐进式的哈希过程，所以数据的迁移不是一次性完成的，rehashidx字段用来表示rehash的进度，表示rehash进行到了哪一步。

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

  这样就可以将 <k,v>存储在 dict->ht[x]->table[index]中，**如果table[index]中已经有数据，则新添加的数据放在链表表头。**

  （**这是因为table[index]中的链表是一个单链表，没有指向链表尾部的指针，添加到表头更快**）



  #### rehash的步骤：

​    当哈希表中的数据太少或者太多时要进行rehash，即对哈希表的大小进行收缩或者扩张。其具体过程如下：

1. 为ht[1]申请空间，其中所申请的大小根据要进行的操作而定：

   - 如果进行的是扩张操作，则ht[1]的大小要依据ht[0]中包含的键值对的数量而定，即根据HashTable的used的属性而定，ht[1]的大小应设置为第一个大于等于ht[0].used*2的2^n次方幂。
   - 如果进行的是收缩操作，同理，ht[1]的大小应设置为第一个大于等于ht[0].used的[2^n].

2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面，rehash指的是重新计算键的哈希值和索引值，然后将键值对放到ht[1]哈希表的指定位置上。

   具体：在rehash操作期间，每次对HashTable执行增删改查操作时，也会顺带将ht[0]对应的table中rehashidx上的键值对全部copy到ht[1]对应的位置上。，当一次rehash操作完成之后，rehashidx++；

3. 当ht[0]中的所有键值对都copy到ht[1]之后，将rehashidx属性设置为-1，释放ht[0],将ht[1]设置为ht[0],并为ht[1]重新创建一个空的哈希表，并为下一次rehash做准备。



### 渐进式哈希:

代码：

```java
int dictRehash(dict *d, int n) {
    int empty_visits = n*10; /* 最大访问空桶数，作用：进一步减小可能引起阻塞的时间 */
    if (!dictIsRehashing(d)) return 0;
 
    //分n步进行的渐进式哈希主体部分
    while(n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;
 
        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long)d->rehashidx);
        while(d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            if (--empty_visits == 0) return 1;
        }
        de = d->ht[0].table[d->rehashidx];
        /* 将ht[0]中的所有元素都迁移到ht[1]中 */
        while(de) {
            uint64_t h;
 
            nextde = de->next;
            /* 辅助了哈希表的大小的掩码来计算出来的，可以保证不会越界 */
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            de = nextde;
        }
        //每一步rehash结束，都要增加索引值，并把旧表中已经迁移的bucket置为空指针。
        d->ht[0].table[d->rehashidx] = NULL;
        d->rehashidx++;
    }
 
    /* 检查一下旧表是否全部迁移完毕，若是，则回收空间，重置旧表，重置渐进式哈希的索引，否则用返回值告诉调用方，dict内仍有数据未迁移*/
    if (d->ht[0].used == 0) {
        zfree(d->ht[0].table);
        d->ht[0] = d->ht[1];
        _dictReset(&d->ht[1]);
        d->rehashidx = -1;
        return 0;
    }
 
    /* More to rehash... */
    return 1;
}
```



​      dict的ht中包含两个哈希表，对于ht[0]会默认申请一个大小为4的哈希数组，当ht[0]中的数据满之后，会为ht[1]申请一个ht[0].size*2的空间，但是不会直接将ht[0]中的数据全部哈希到ht[1]中，而是采用渐进式哈希的方法，在之后的(find,set,get)方法逐渐copy进去，因此在ht[1]被全部占满的时候，能够保证ht[0]中的全部元素都copy进ht[1]中。

​         在扩容或者收缩时，需要将ht[0]全部哈希到ht[1]中，如果一次性哈希，当数据足够时，会存在效率问题。因此redis通过逐步哈希的方式。具体过程如下：

        1. 为ht[1]分配空间，空间大小为ht[0]的size大小的二倍。
           2. 设置 rehashidx = 0
           3. 新添加的数据不在加入到ht[0]，而是加入到ht[1]中，保证ht[0]不会增加数据。
           4. 将ht[0]->table[rehashidx]的数据全部哈希到ht[1]中
           5. rehashidx++，并将copy到ht[1]中数据对应的ht[0]中的数据置为空指针
           6. 随着不断的执行，ht[0]的数据哈希到了ht[1],这时可以设置 rehashidx = 1，表明rehash结束，释放ht[0],设置ht[1]为ht[0].
           
           

### Q/A:

  #### 扩展的触发条件

  1.在没有执行bgsave或bgrewriteaof时，哈希表的负载因子>=1

  2.在执行bgsave或bgrewriteaof时，哈希表的负载因子>=5

​      注意：

​       在进行bgsave或bgrewriteaof时，提高负载因子是为了避免扩容，避免不必要的内存写入。

####      收缩的触发条件

​        负载因子<0.1

#### ps：什么是加载因子/负载因子/装载因子

答：用于表示哈希表中元素元素填满的程度。冲突的机会越大，查找的成本越高。反之，查找的成本越低，从而查找的时间越少。

#### 怎样计算一个哈希表的装载因子呢？

例如：今有表长为10 的hash表，里面包含7个元素，所以他的装载因子为： 7/10 = 0.7;

#### Redis中bgsave和save：

这两个命令都是用于创建当前数据库的备份。

因为redis持久化选择RDB快照模式，所以redis并不是实时的进行数据的持久化，而是有一定的时间间隔。这时候如果想要手动进行一次持久化，可以使用save或者bgsave命令。

**SAVE**

基本命令：

```
redis 127.0.0.1:6379> SAVE 
OK
```

该命令将在 redis 安装目录中创建dump.rdb文件。

影响：

SAVE 直接调用 rdbSave函数 ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

如果数据量小，用此命令可能感觉不出有什么区别，但是当数据量很大的时候，就需要谨慎使用这个命令。

 

**BGSAVE**

基本命令：

```
127.0.0.1:6379> BGSAVE

Background saving started
```

客户端可以通过 LASTSAVE 命令查看相关信息，判断 BGSAVE 命令是否执行成功。

影响：

BGSAVE 命令执行之后立即返回 OK ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

BGSAVE方式比较适合线上的维护操作。

##### 恢复数据

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 **CONFIG** 命令，如下所示：

```
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
```

以上命令 **CONFIG GET dir** 输出的 redis 安装目录为 /usr/local/redis/bin。



#### 渐进式的rehash执行期间的哈希表操作？

​       上面说到了，将ht[0]中的元素全部copy到ht[1]中的过程并不是一次性完成的。

​       因为在进行渐进式rehash的过程中，字典会同时使用ht[0]和ht[1]两个哈希表，所以在渐进式rehash过程中，字典的删除(delete)，查找(find)，更新(update)等操作会在两个哈希表上进行。

​       例如，要在字典中查找一个键的话，程序会现在ht[0]里面进行查找，如果没找到的话，就会继续到ht[1]中进行查找，诸如此类。

​      另外，在渐进式rehash执行期间，新添加到字典的键值对一律会保存到ht[1]中，而ht[0]不在进行任何添加操作，这一系列操作会保证ht[0]中的键值对只减不增，并随着rehash操作的执行最终变成一个空表。





- ### set（集合）

   Redis的Set是string类型的无序集合。

   集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1).

- ### **zset**

  zset中的基本数据结构使用的是跳跃表(skiplist)，**用来维护一组有序的数据。**增删改查的时间复杂度均为O(lgN)。

  和红黑树的性能差不多，使用zset的原因有两个：
  
     1.跳表比红黑树的实现简单很多；
  
     2.zet有范围查询，所以使用跳表的效率更高。
  
  
  
  （跳表的结构）
  
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200614101604943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNzI4MDI4,size_16,color_FFFFFF,t_70)
  
  
  
  每次创建一个新跳跃表结点的时候，程序会根据幂次定律（power law，越大的数出现的概率越小）随机生成一个介于1和32之间的值作为level数组的大小，这个大小就是层的“高度‘。
  
  - 每个level结点不仅记录了指向下一个同层的指针，还记录同层之间的跨度span。span用于计算某个成员的排名rank。在查找某个元素的过程中，累加走过的跨度就可以计算出查找元素的rank。
  - zskiplistNode结点的obj指向一个sds对象，也就是成员的名称；
  - zskiplistNode结点的成员名不能重复，但是score可以相同，对于score相同的成员，按照成员名的字典序进行排序。
  
  