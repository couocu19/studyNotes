# java并发编程(二)

## 7.30学习更新

- ### 可重入锁

  ```java
  /**
   * 可重入锁的实现：
   *    1.java内置锁(synchronize)
   *    2.Lock(ReentrantLock)
   *
   * 为什么会有可重入锁的出现？
   *    为了避免死锁的情况出现。
   *    当m1方法调用m2方法的时候，如果不可重入，
   *    那么调用b方法时当前线程就会等待a方法对应的锁的释放，
   *    但是该锁还被当前线程所占用，所以就出现了死锁。
   *
   */
  public class DuplicateLock implements Runnable {
  
      public synchronized void m1(){
          System.out.println("m1获得锁,正常运行！");
          m2();
      }
  
      public synchronized void m2(){
          System.out.println("m2获得锁,正常运行！");
      }
  
      @Override
      public void run() {
          m1();
      }
  
      public static void main(String[] args) {
          DuplicateLock dl = new DuplicateLock();
          new Thread(dl).start();
          new Thread(dl).start();
      }
  }
  
  ```
  
  #### 前言

  ​     在java中，锁是实现并发的关键组件，多个线程之间的同步关系需要锁来保证。

  ​     所谓锁，其语义就是资源获取到资源并释放的一系列过程。

  ​     **使用lock：**

  ​         想要进入临界区对共享资源执行操作。

  ​     **使用unlock：**

  ​         说明线程已经完成了相关工作或者发生了异常从而离开临界区释放共享资源。

  ​         可以说，在多线程环境下，锁是一个必不可少的组件。

  ​    我们最常用的并发锁是synchronized关键字，在最新的jdk中，synchronized性能已经有了很大的提升，而且未来还会对他做进一步优化。

  ​     在大多数情况下，我们写并发的代码使用synchronized就足够了，而且使用synchronized是首选，不过如果我们希望更加灵活地使用锁来开发，java还提供了一个接口Lock。

####     什么叫可重入锁？

​          可重入锁，也叫作递归锁，指的是同一个线程的外层函数获得锁孩子后，其内层递归函数仍然可以获得该锁而不受到影响。

​          在java环境下ReentrantLock和Synchronized都是可重入锁。



- ### AQS（AbstractQueuedSynchronizer概述)

  - #### AQS类提供的一些方法：

    ConditionObject类下提供的方法：

    - ConditionObject()
    
    - await()：void
    
    - await(long,TimeUnit):boolean
    
    - awaitNanos(long):long
    
      …………（还有很多方法）

  在类结构上，AQS继承了AbstractOwnableSynchronizer，**AbstractOwnableSynchrinizer仅有的两个方法是提供对当前独占模式的线程设置：**

  - exclusiveOwnerThread**代表当前获得同步的线程，因为是独占模式**，在eQT持有同步的过程中其他线程的任何同步获取请求都不能获得满足。

  

   **注意：**

     AQS不仅支持独占模式下的同步实现，还支持共享模式下的同步实现。在java锁的实现上就有**共享锁和独占锁**的区别，而这些实现都是**基于AQS对于共享同步和独占同步**的支持。

  -   **AQS的API大致分为三类：**

    - 类似acquire(int) 的一类是基本的一类，不可中断
    
    - 类似acquireInterruptibly(int)的一类可以被中断
    
  - 类似tryAcquireNanos(int，long)的一类不仅可以被中断，而且可以设置阻塞时间。 
    
       
    
    上面的三种类型的API分为**独占和共享**两套，我们可以根据自己的需求来使用合适的API来做多线程同步。
    
  -   **AQS中封装的node类:**

      ```java
          // 共享模式下等待的标记
          static final Node SHARED = new Node();
          // 独占模式下等待的标记
          static final Node EXCLUSIVE = null;
      
          // 线程的等待状态 表示线程已经被取消
          static final int CANCELLED =  1;
          // 线程的等待状态 表示后继线程需要被唤醒
          static final int SIGNAL    = -1;
          // 线程的等待状态 表示线程在Condtion上
          static final int CONDITION = -2;
      
          // 表示下一个acquireShared需要无条件的传播
          static final int PROPAGATE = -3;
      
      
          volatile int waitStatus;
      
      
          volatile Node prev;
      
      
          volatile Node next;
      
          /**
           * 当前节点的线程,初始化后使用,在使用后失效 
           */
          volatile Thread thread;
      
          /**
           * 链接到等在等待条件上的下一个节点,或特殊的值SHARED,因为条件队列只有在独占模式时才能被访问,
           * 所以我们只需要一个简单的连接队列在等待的时候保存节点,然后把它们转移到队列中重新获取
           * 因为条件只能是独占性的,我们通过使用特殊的值来表示共享模式
           */
          Node nextWaiter;
      
          /**
           * 如果节点处于共享模式下等待直接返回true
           */
          final boolean isShared() {
              return nextWaiter == SHARED;
          }
      
          /**
           * 返回当前节点的前驱节点,如果为空,直接抛出空指针异常
           */
          final Node predecessor() throws NullPointerException {
              Node p = prev;
              if (p == null)
                  throw new NullPointerException();
              else
                  return p;
          }
      
          Node() {    // 用来建立初始化的head 或 SHARED的标记
          }
      
          Node(Thread thread, Node mode) {     // 指定线程和模式的构造方法
              this.nextWaiter = mode;
              this.thread = thread;
          }
      
          Node(Thread thread, int waitStatus) { // 指定线程和节点状态的构造方法
              this.waitStatus = waitStatus;
              this.thread = thread;
          }
      }
      
      ```

  -   几个重要方法

      - void acquire(int arg)

        以独占模式尝试获取锁，其中的tryAcquire()方法需要自己实现

      ```java
        /**
           * Acquires in exclusive mode, ignoring interrupts.  Implemented
           * by invoking at least once {@link #tryAcquire},
           * returning on success.  Otherwise the thread is queued, possibly
           * repeatedly blocking and unblocking, invoking {@link
           * #tryAcquire} until success.  This method can be used
           * to implement method {@link Lock#lock}.
           *
           * @param arg the acquire argument.  This value is conveyed to
           *        {@link #tryAcquire} but is otherwise uninterpreted and
           *        can represent anything you like.
           */
          public final void acquire(int arg) {
              //如果tryAcquire方法失败并且成功入入队且被中断过
              if (!tryAcquire(arg) &&
                  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                  selfInterrupt();
          }
      ```

      - Node  addWaiter(Node node)

      ```java
      /**
           * Creates and enqueues node for current thread and given mode.
           *
           * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
           * @return the new node
           */
          private Node addWaiter(Node mode) {
             //将当前线程和模式(可以是共享或者独占模式)包装成一个node
              Node node = new Node(mode);
      
              //尝试将当前节点快速入队，之后的enq会进行无限循环入队，太费时间，所以需要先尝试将节点入队
              for (;;) {
                  Node oldTail = tail;
                  //判断尾结点是否为空
                  if (oldTail != null) {
                      node.setPrevRelaxed(oldTail);
                      //cas替换尾结点，node成功入队         
                      if (compareAndSetTail(oldTail, node)) {
                          oldTail.next = node;
                          return node;
                      }
                  } else {
                      initializeSyncQueue();
                  }
              }
          }
      ```

      - Node enq(Node node)

        ```java
        /**
             * Inserts node into queue, initializing if necessary. See picture above.
             * @param node the node to insert
             * @return node's predecessor
             */
            private Node enq(Node node) {
                //无限循环入队
                for (;;) {
                    Node oldTail = tail;
                    if (oldTail != null) {
                        node.setPrevRelaxed(oldTail);
                       //cas操作入队
                        if (compareAndSetTail(oldTail, node)) {
                            oldTail.next = node;
                            return oldTail;
                        }
                        //如果获取到的尾结点为空的话就重新进行初始化，然后再次循环进来
                    } else {
                        initializeSyncQueue();
                    }
                }
            }
        ```

        

      - boolean acquireQueued(final Node node, int arg) 

  ```java
  /**
       * Acquires in exclusive uninterruptible mode for thread already in
       * queue. Used by condition wait methods as well as acquire.
       *
       * @param node the node
       * @param arg the acquire argument
       * @return {@code true} if interrupted while waiting
       */
  //判断是否入队成功/失败
      final boolean acquireQueued(final Node node, int arg) {
      //判断是否中断
          boolean interrupted = false;
          try {
              //又是一个自旋，说明会一直尝试
              for (;;) {
                  final Node p = node.predecessor();
                  //判断前驱结点是否为头结点并且tryAcquire是否成功
                  if (p == head && tryAcquire(arg)) {
                      //将自己设为前驱结点
                      setHead(node);
                      //将前驱结点释放，前驱结点拜拜
                     
                      p.next = null; // help GC
                      return interrupted;
                  }
                  
             /**
  	         * 会去检查前面的Node的状态，当满足一定条件后才会将线程Park住，注意这时没有跳出循环
             **/
                  if (shouldParkAfterFailedAcquire(p, node))
                      interrupted |= parkAndCheckInterrupt();
              }
          } catch (Throwable t) {
              cancelAcquire(node);
              //获取到资源后进行自我中断
              if (interrupted)
                  selfInterrupt();
              throw t;
          }
      }
  ```

  - **shouldParkAfterFailedAcquire和parkAndCheckInterrupt**两个方法用来让当前节点找到一个合适的地方开始等待

    ![image-20200731104654581](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200731104654581.png)

    ```java
    
        /**
         * Checks and updates status for a node that failed to acquire.
         * Returns true if thread should block. This is the main signal
         * control in all acquire loops.  Requires that pred == node.prev.
         *
         * @param pred node's predecessor holding status
         * @param node the node
         * @return {@code true} if thread should block
         */
        private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
            int ws = pred.waitStatus;
            if (ws == Node.SIGNAL) //状态已经是signal，可以挂起，直接返回true
                /*
                 * This node has already set status asking a release
                 * to signal it, so it can safely park.
                 */
                return true;
            //表示结点被取消，需要被移除
            if (ws > 0) {
                /*
                 * Predecessor was cancelled. Skip over predecessors and
                 * indicate retry.
                 */
                do {
                    node.prev = pred = pred.prev;
                } while (pred.waitStatus > 0);
                pred.next = node;
            } else {
               //如果没有取消，也不是SIGNAL状态，则尝试修改为SIGNAL，
            //node是prevNode的nextNode，说明prevNode存在后续节点，也就表明prevNode需要是SIGNAL状态，在prevNode执行完之后，
    		//需要去唤醒nextNode。因此，如果prevNode不是SIGNAL状态的话，需要修改为SIGNAL状态
    
                pred.compareAndSetWaitStatus(ws, Node.SIGNAL);
            }
            return false;
        }
    
     /**
         * Convenience method to park and then check if interrupted.
         *
         * @return {@code true} if interrupted
         */
        private final boolean parkAndCheckInterrupt() {
            LockSupport.park(this);
            return Thread.interrupted();
        }
    
    ```

  #### 总结流程：

  - **acquire()中调用tryAcquire()方法，尝试获取资源(获取当前的锁)。**

  - **如果获取成功，就退出返回。**

  - **如果获取失败，调用addWaiter(Node node)方法，将当前线程和模式封装成一个node，并入队，添加到当前尾结点之后。**

  - **acquireQueued()使线程在等待队列中休息，有机会时（轮到自己，会被unpark()）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。**

    调用acquireQueued(final Node node, int arg)方法，这里参数中的node即为addWaiter中的标记为独占模式的node，使当前线程进入“等待”状态，进行自旋，尝试获取资源。

    其中在尝试获取资源的过程中，首先会判断当前节点的前驱结点是不是头结点，如果是，则尝试获取资源（同步状态）。

  - 如果前驱结点不是头结点 ，则调用shouldParkAfterFailedAcquire（）方法，为当前结点找到合适的等待的位置。如果该方法返回true即如果找到合适的等待位置，那么当前线程被中断，如果找不到，就释放前驱结点继续往前找。

    - shouldParkAfterFailedAcquire（）：

      首先判断自己的前驱结点是不是等待状态（signal状态），如果是就返回true，代表已经找到了合适的位置。

      如果不是，代表前驱结点已经放弃，那么就一直往前找，直到找到一个正常等待状态的前驱结点，并排在他的后面。

    - parkAndCheckInterrupt()

      返回当前线程的中断状态。

  - 当前驱结点为头结点时，就可以尝试获取同步状态，如果获取成功，就将当前节点设为头结点，然后退出返回。

    #### **注意：**

    1. 如果在等待过程中线程被中断过，那么就不作出响应。

       如果没有中断过，那么就在获取到资源之后进行一个自我中断。

    2. 在acquireQueued方法中需要完成两件事：第一，增加node节点；第二，挂起线程，使线程处于WAITING状态，等待被唤醒。

