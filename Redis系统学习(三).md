

# Redis系统学习(三)

## redis cluster集群机制

（ps： cluster：簇，团，束，聚集。）

redis cluster是Redis的分布式解决方案，在3.0版本推出之后有效的解决了redis分布式方面的需求。

自动将数据进行分片，每个master上放一部分数据。

提供内置的高可用支持，部分master不可用时，还是可以继续工作的。



支撑N个redis  master  node，每个master node都可以挂载多个slave node。

由多个Redis实例组成的整体，数据按照Slot存储分布在多个Redis实例上，**通过Gossip协议来进行节点之间的通信。**

高可用，因为每个master都有slave节点，如果master挂掉，redia cluster机制就会自动将某个slave切换成master。



### redis cluster vs replication + sentinal

  如果你的数据很少，主要是承载高并发高性能的场景，比如你的缓存只有几个G，单机就足够了。

 replication，一个master，多个slave，来几个slave跟你的要求的读吞吐量有关系，再搭建一个sentinal集群，保证高可用就可以了。



  而redis cluster，主要是针对海量数据＋高并发+高可用的场景，海量数据，如果数据量很大，那么就建议使用redis cluster机制



### 概念

Redis 3.0之后，节点之间通过去中心化的方式，提供了完整的sharding，replication（复制机制仍使用原有机制，并且具备感知主备的能力），failover解决方案，称为redis cluster。



