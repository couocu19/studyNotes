# Redis+mysql怎么使用

> > >​      首先要知道mysql存储在磁盘里，redis存储在内存里，redis既可以用来做持久存储，也可以做缓存，而目前大多数公司的存储都是mysql + redis，mysql作为主存储，redis作为辅助存储被用作缓存，加快访问读取的速度，提高性能
> > >​    那么为什么不直接全部用redis存储呢？
> > >​    我的看法是：因为redis存储在内存中，如果存储在内存中，存储容量肯定要比磁盘少很多，那么要存储大量数据，只能花更多的钱去购买内存，造成在一些不需要高性能的地方是相对比较浪费的，所以目前基本都是mysql(主) + redis(辅)，在需要性能的地方使用redis，在不需要高性能的地方使用mysql，好钢用在刀刃上



## [Redis和MySQL的结合方案](https://www.cnblogs.com/lipengsheng-javaweb/p/11433453.html)

方案一：

程序同时写Redis和MySQL
读Redis

![img](https://img-blog.csdn.net/20160126164208799?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 

方案二：

程序写MySQL， 使用Gearman调用MySQL的UDF，完成对Redis的写
读Redis
参考 [《利用Gearman进行Mysql到Redis的复制》](http://blog.csdn.net/stubborn_cow/article/details/50460349)

![img](https://img-blog.csdn.net/20160126164918036?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 

方案三：

程序写MySQL， 解析binlog，数据放入队列写Redis
读Redis
参考 [《利用Canal完成Mysql数据同步Redis](http://blog.csdn.net/stubborn_cow/article/details/50371405)》

![img](https://img-blog.csdn.net/20160126171202576?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

为了保证数据的一致性，可以将写到redis的操作，和mysql的操作放到一个事务里面进行处理。

虽然这是操作两个数据库，每个数据库都有自己的事务，但是可以把它们放到同一个java进程中，形成一个事务，然后进行处理。

 

方案四：

程序写Redis，并将写放入MQ写MySQL
读Redis

![img](https://img-blog.csdn.net/20160126173637114?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 插入：磁盘和内存有什么区别？

磁盘和内存：

程序运行时，内存和磁盘的作用及相互关系。

计算机在运行程序时，必须将磁盘中的内容加载到内存中，不加载是不能运行程序的。
在内存中有一部分数据存的是磁盘的缓存，这样做可以加速磁盘访问速度。就跟我们开发程序中使用的缓存作用一样。

虚拟内存：

虚拟内存，是指把磁盘中的一部分作为假想的内存使用，windows通过分页式虚拟内存机制：即在不考虑程序构造的情况下将程序按照一定大小的页进行划分，并以页为单位在内存和磁盘间进行置换。一般来说自己计算机的实际内存大小即为当前页文件的大小。这个是可以在电脑中进行设定的。
