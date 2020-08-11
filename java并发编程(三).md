# java并发编程(三)

## 信号量(Semaphore)的使用

- ### 概念：

     Semaphore是java并发包中一个非常有用的类，是一个技术信号量。从概念上讲，信号量维护了一个**许可的集。**

- ### 实质：

     实质上是一个“共享锁”。

- ### 作用：

     **信号量实际是用来限制并发访问的某些资源的线程数目。**也就是我们通俗所说的限流

     比如：现在一共有100个线程在等待状态，但是信号量只允许同时有5个线程使用，则必须等到这5个许可线程中的一个线程执行完毕之后，剩下的线程才可以开始准备执行，

- ### 执行过程：

  1.  **void accquire()**

       **从信号量获取一个许可**，是可中断方式，**在提供一个许可当前线程会阻塞**，如果成功了许可并立即返回，将信号量中的有用许可减一。

  2.  **accquireUninterruptibly()**

     **与acquire方法相同**，使用的是不可中断方式，如果尝试获取许可失败，那么他会进入AQS的队列中排队。

  3.   **tryAcquire()**

      **尝试获取一个许可**

       仅仅调用此信号量存在一个可用的许可，如果存在返回true，不逊在放回false并立即返回。

  4.  **tryAcquire(0,YimeUnit.SECONDS)**

       在单位等待时间内获取许可，这样希望是遵守公平的设置，如果已经炒熟等待时间会立即返回。

  5. **release()**

     释放一个许可，将其归还给指定的信号量。

  #### 总结：

  ​     信号量的执行过程可以总结为：**获取许可--执行任务--归还许可**。

  ​     信号量，保存了一系列的许可，每次调用acquire都会消耗一个许可，每次release都会归还一个许可。

  - #### 源码解析

    Semaphore类中包含了主要两部分：

    1. 一个继承了AQS的抽象类Syc。

    2. Syc的两个子类 FairSyc和NonFairSyc。一个对应公平模式，一个对应非公平模式。

       - **syc内部类源码：**

       ```java
       /**
            * Synchronization implementation for semaphore.  Uses AQS state
            * to represent permits. Subclassed into fair and nonfair
            * versions.
            */
           abstract static class Sync extends AbstractQueuedSynchronizer {
               private static final long serialVersionUID = 1192457210091910933L;
       
               //构造方法，传入许可次数，放入states中
               Sync(int permits) {
                   setState(permits);
               }
       
               //获取许可次数
               final int getPermits() {
                   return getState();
               }
       
               //非公平模式尝试获取许可
               final int nonfairTryAcquireShared(int acquires) {
                   for (;;) {
                       //看看还有几个许可
                       int available = getState();
                       //减去这次需要获取的许可还剩下几个许可
                       int remaining = available - acquires;
                       //如果剩余许可小于0就直接返回，
                       //如果剩余许可不小于0，则尝试原子更新state的值,成功了返回剩余的许可。
                       if (remaining < 0 ||
                           compareAndSetState(available, remaining))
                           return remaining;
                   }
               }
       
               //释放许可
               protected final boolean tryReleaseShared(int releases) {
                   for (;;) {
                       //看看还剩几个许可
                       int current = getState();
                       //加上这次释放的许可
                       int next = current + releases;
                       //检测溢出
                       if (next < current) // overflow
                           throw new Error("Maximum permit count exceeded");
                       //如果原子更新state的值成功，就说明释放许可成功，则返回true
                       if (compareAndSetState(current, next))
                           return true;
                   }
               }
       
               //减少许可
               final void reducePermits(int reductions) {
                   for (;;) {
                       //看看还有几个许可
                       int current = getState();
                       //减去将要减少的许可
                       int next = current - reductions;
                       //检测溢出
                       if (next > current) // underflow
                           throw new Error("Permit count underflow");
                       if (compareAndSetState(current, next))
                           return;
                   }
               }
       
               //销毁许可
               final int drainPermits() {
                   for (;;) {
                       //看看还有几个许可
                       int current = getState();
                       //如果为0，直接返回，如果不为0，将state的原子更新为0
                       if (current == 0 || compareAndSetState(current, 0))
                           return current;
                   }
               }
           }
       ```

    通过以上的源码分析，可以获取到几个信息。

    1. 许可是在构造方法时传入的。
    2. 许可存放在状态变量的state中。
    3. 尝试获取一个许可时，将state的值减一。
    4. 尝试释放一个许可时，将state的值加一。
    5. 当state的值为0时，则无法获取许可。
    6. 许可的数量可以动态的改变。

    - **FairSyc和NonFairSyc源码解析：**

      ```java
       /**
           *非公平模式
           */
          static final class NonfairSync extends Sync {
              private static final long serialVersionUID = -2694183684443567898L;
      
              NonfairSync(int permits) {
                  super(permits);
              }
      
              protected int tryAcquireShared(int acquires) {
                  //直接调用父类中的尝试获取许可的方法即可
                  return nonfairTryAcquireShared(acquires);
              }
          }
      
          /**
           * 公平模式
           */
          static final class FairSync extends Sync {
              private static final long serialVersionUID = 2014338818796000944L;
      
              FairSync(int permits) {
                  super(permits);
              }
      
              //尝试获取许可
              protected int tryAcquireShared(int acquires) {
                  for (;;) {
                      //公平模式需要检测是否前面有排队的
                      //如果有排队的直接返回失败
                      if (hasQueuedPredecessors())
                          return -1;
                      //没有排队的在尝试更新state的值
                      int available = getState();
                      int remaining = available - acquires;
                      if (remaining < 0 ||
                          compareAndSetState(available, remaining))
                          return remaining;
                  }
              }
          }
      
      
      ```

      **注意：**公平模式下，先检测前面是否有排队的，如果有排队的则获取许可失败，进入队列进行排队，否则尝试原子更新state的值。

    - #### 构造方法

      ```java
      
      //构造方法，创建时要传入的许可次数，默认使用非公平模式
        public Semaphore(int permits) {
              sync = new NonfairSync(permits);
          }
      //构造方法，需要传入许可次数，及时否为公平模式
          public Semaphore(int permits, boolean fair) {
              sync = fair ? new FairSync(permits) : new NonfairSync(permits);
          }
      ```

  - ### Q/A

    -  **如何动态增加n个许可？**

      调用release方法即可，我们知道释放许可的时候state的值会增加，再回头看看释放许可的源码，发现和ReentrantLock的释放锁还是有区别的，Semaphore在释放许可的时候并不会检查当前线程有没有获取过许可，所以可以调用释放许可的方法动态增加一些许可。

    - ##### 如何实现限流?

      限流，即在流量突然增大的时候，上层要能够限制住突然的大流量对下游服务的冲击，在分布式系统中限流一般做在网关层，当然在个别功能中也可以自己简单的来限流，比如秒杀场景，假如只有10个商品要秒杀，那么服务本身可以限制同时只进来100个请求，其他请求全部作废，这样服务压力也不会太大。

  ## 独占锁和共享锁的比较

  在阅读AQS源码的时候知道了AQS类支持两种模式下，一种是独占模式，一种是共享模式。这两种模式也就分别对应了独占锁和共享锁两种锁。

  - ### 独占锁

    也叫悲观锁。

     举例：synchronized和ReentrantLock

  - ### 共享锁

    指的是一个锁可以同时被多个线程所拥有的。

    例如：ReadWriteLock接口中的读锁，他是可以被共享的，但是他的写锁每次只能被独占。

​                           Semaphore类，继承自AQS类。



