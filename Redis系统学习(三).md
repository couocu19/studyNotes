

# Redis系统学习(三)

## redis cluster集群机制

（ps： cluster：簇，团，束，聚集。）

redis cluster是Redis的分布式解决方案，在3.0版本推出之后有效的解决了redis分布式方面的需求。

**自动将数据进行分片，每个master上放一部分数据。**

**提供内置的高可用支持，部分master不可用时，还是可以继续工作的。**



支撑N个redis  master  node，每个master node都可以挂载多个slave node。

由多个Redis实例组成的整体，数据按照Slot存储分布在多个Redis实例上，**通过Gossip协议来进行节点之间的通信。**

高可用，因为每个master都有slave节点，如果master挂掉，redia cluster机制就会自动将某个slave切换成master。



### redis cluster vs replication + sentinal

  如果你的数据很少，主要是承载高并发高性能的场景，比如你的缓存只有几个G，单机就足够了。

 replication，一个master，多个slave，来几个slave跟你的要求的读吞吐量有关系，再搭建一个sentinal集群，保证高可用就可以了。



  而redis cluster，主要是针对**海量数据＋高并发+高可用**的场景，海量数据，如果数据量很大，那么就建议使用redis cluster机制。

### 概念

Redis 3.0之后，节点之间通过去中心化的方式，提供了完整的sharding，replication（复制机制仍使用原有机制，并且具备感知主备的能力），failover解决方案，称为redis cluster。

即：将proxy/sentinel的工作融合到了普通的redis节点里面。

Redis Cluster 一般由多个节点组成，节点数量至少为 6 个才能保证组成完整高可用的集群，其中三个为主节点，三个为从节点。三个主节点会分配槽，处理客户端的命令请求，而从节点可用在主节点故障后，顶替主节点。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTQ2MjA1Ny1kZmIxNDcwNGQ1NTMxNWYxLnBuZw?x-oss-process=image/format,png)

如上图。该急群众包含6个Redis节点，3主3从。

除了主从redis节点之间进行数据复制外，所有Redis节点之间采用Gossip协议进行通信，交换维护节点元数据的信息。

一般来讲，主Redis节点会吹Clients的读写操作，但是从节点只处理读操作。

### 数据分片

​       **分布式数据存储方案中最重要的一点就是数据分片，即所谓的sharding。**

​       为了使得集群能够水平扩展，首要解决的问题就是如何将整个数据集按照一定的规则分配到多个节点上，常用的数据分片的方法有：**范围分片，哈希分片，一致性哈希算法和虚拟槽等。**

### 拓扑结构

​     一个redis cluster由多个Redis节点组成。

​     不同的节点组服务的数据无交集，每个节点对应数据sharding的一个分片。

​    节点组内部分为主备两部分，即一个节点组包括一个master节点和多个slave节点。两者的数据实时一致，通过异步化的主备复制机制保证。

​     对于一个节点组的主备节点，只有master对外提供写服务，读服务可由master/slave提供。





