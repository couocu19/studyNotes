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

​          可重入锁，也叫作递归锁，**指的是同一个线程的外层函数获得锁孩子后，其内层递归函数仍然可以获得该锁而不受到影响。**

​         可重入锁指的是同一个线程可以多次获取同一把锁。例如：

​        一个线程正在实行一个带锁的方法，该方法中又调用了另一个带锁的方法，则该线程可以直接执行调用的方法，而无需重新获取锁。

​         但是，获取多少次就要释放多少次，这样才能保证state回到0态。

​          

​         在java环境下ReentrantLock和Synchronized都是可重入锁。



- ### AQS（AbstractQueuedSynchronizer概述)

  AQS类如其名，抽象的队列式的同步器，通常被称为AQS类，是一个非常有用的父类，可用来定义锁以及依赖于排队阻塞线程的其他同步器。

  - #### 概述：

    AQS维护了一个**volatile int state（表示共享资源**）和一个FIFO**线程等待队列**（多线程争用资源被阻塞时会进入此队列）。

    - **AQS用一个32位的整型来表示同步状态，使用volatile来修饰**

      ```java
      private volatile int state;
      ```

      在互斥锁中它表示当前线程是否已经获取了锁**，0未获取，1表示已经获取了，大于1表示重入数。**

      ```java
      /**
        通过这三个方法来获取和修改state的值。
        所以state的值大于1表示该线程可能调用了多个需要当前锁的方法，或同一个线程调用了多次lock()
      方法。
       */
      /**
           * Returns the current value of synchronization state.
           * This operation has memory semantics of a {@code volatile} read.
           * @return current state value
           */
          protected final int getState() {
              return state;
          }
      
          /**
           * Sets the value of synchronization state.
           * This operation has memory semantics of a {@code volatile} write.
           * @param newState the new state value
           */
          protected final void setState(int newState) {
              state = newState;
          }
      
          /**
           * Atomically sets synchronization state to the given updated
           * value if the current state value equals the expected value.
           * This operation has memory semantics of a {@code volatile} read
           * and write.
           *
           * @param expect the expected value
           * @param update the new value
           * @return {@code true} if successful. False return indicates that the actual
           *         value was not equal to the expected value.
           */
          protected final boolean compareAndSetState(int expect, int update) {
              return STATE.compareAndSet(this, expect, update);
          }
      
      //这三个方法需要java.util.concurrent.atomic包的支持，采用cas操作，保证原子性和可见性。
      
      ```

    - ##### AQS内部维护着一个FIFO的CLH队列。

    

  - #### AQS类提供的一些方法：

    **ConditionObject类下提供的方法：**

     ConditionObject类的作用：
    
    ​    与ReentrantLock一起使用，是AQS中的一个内部类，为了支持条件队列的锁更加方便。
    
     **ConditionObject类中的条件队列和CLH队列之间的“配合”：**
    
    ```
    具体过程：
    step1:将该线程封装成node，新节点的状态为CONDITION，添加到队列尾部
    step2:尝试释放当前线程占有的锁，释放成功，则调用unparkSuccessor方法唤醒该节点在CLH队列中的后继节点。
    step3:在while循环中调用isOnSyncQueue方法检测node是否再次transfer到CLH队列中（其他线程调用signal或signalAll时，该线程可能从CONDITION队列中transfer到CLH队列中），如果没有，则park当前线程，等待唤醒，同时调用checkInterruptWhileWaiting检测当前线程在等待过程中是否发生中断，设置interruptMode的值来标志中断状态。如果检测到当前线程已经处于CLH队列中了，则跳出while循环。
    step4:调用acquireQueued阻塞方法来在CLH队列中获取锁。
    step5:检查interruptMode的状态，在最后调用reportInterruptAfterWait统一抛出异常或发生中断。
    
    基本流程：首先将node加入condition队列，然后释放锁，挂起当前线程等待唤醒，唤醒后重新在CLH队列中调用acquireQueued获取锁。（实现Object.wait方法的功能）
    ```
    
    - ConditionObject()
    
    - await()：void
    
    - await(long,TimeUnit):boolean
    
    - awaitNanos(long):long
    
      …………（还有很多方法）
    
    **AQS顶层实现的几个重要方法：**
    
    - isHeldExclusively():该线程是否正在独占资源。只有用到condition时才需要实现它。
    - tryAcquire(int)：独占方式。尝试获取资源，成功返回true，失败返回false。
    - tryRelease(int)：独占方式。尝试释放资源，成功返回true，失败返回false。
    - tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，并且没有剩余资源；整数表示成功，且有剩余资源。
    - tryReleaseShared(int)：共享方式。尝试释放资源。成功返回true，失败返回false。
    - tryReleaseShared(int)：共享方式。尝试释放资源

  

  

  在类结构上，AQS继承了AbstractOwnableSynchronizer，**AbstractOwnableSynchrinizer仅有的两个方法是提供对当前独占模式的线程设置：**

  - exclusiveOwnerThread**代表当前获得同步的线程，因为是独占模式**，在eQT持有同步的过程中其他线程的任何同步获取请求都不能获得满足。

  

   **注意：**

     AQS不仅支持独占模式下的同步实现，还支持共享模式下的同步实现。

     在java锁的实现上就有**共享锁和独占锁**的区别，而这些实现都是**基于AQS对于共享同步和独占同步**的支持。

  -   **AQS的API大致分为三类：**

    - 类似acquire(int) 的一类是基本的一类，不可中断
    
    - 类似acquireInterruptibly(int)的一类可以被中断
    
  - 类似tryAcquireNanos(int，long)的一类不仅可以被中断，而且可以设置阻塞时间。 
    
       
    
    上面的三种类型的API分为**独占和共享**两套，我们可以根据自己的需求来使用合适的API来做多线程同步。
    
  - **AQS中封装的node类:**

    ```java
        // 共享模式下等待的标记
        static final Node SHARED = new Node();
        // 独占模式下等待的标记
        static final Node EXCLUSIVE = null;
    
        // 线程的等待状态 表示线程已经被取消
        static final int CANCELLED =  1;
        // 线程的等待状态 表示后继线程需要被unpark(唤醒)
        //一般情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
    
        static final int SIGNAL    = -1;
        // 线程的等待状态 表示线程在Condtion上
        static final int CONDITION = -2;
    
        // 表示下一个acquireShared需要无条件的传播
       //(共享锁)其他线程获取到“共享锁”，对应的waitStatus的值
        static final int PROPAGATE = -3;
    
        //waitStatus为“CANCELLED,SIGNAL,CONDITION,PROPAGATE”时分别表示不同的状态
       //若waitStatus=0，则意味着当前线程不属于任何一种状态。
        volatile int waitStatus;
    
        //前一线程
        volatile Node prev;
    
        //后一线程
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
    
       //nextWaiter是区别当前CLH队列是“独占锁”队列还是“共享锁”队列的标记
       //若nextWaiter=SHARED,则CLH队列是“独占锁”队列。
       //若nextWaiter=EXCLUSIVE，则CLH是“共享锁”队列
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

    #### 说明：

    1. 每个Node都与一个线程对应。
    2. 每个Node会通过prev和next分别指向上一个节点和下一个节点，这分别代表上一个等待线程和下一个等待线程。
    3. Node通过waitStatus保存线程的等待状态。
    4. Node通过nextWaiter来区分线程是“独占锁”还是“共享锁”线程。如果是独占锁线程，nextWaiter的值为exclusive；如果是共享锁线程，nextWaiter的值是Shared。

    

  -   几个重要方法

      - void acquire(int arg)

        **以独占模式尝试获取锁，其中的tryAcquire()方法需要自己实现**

      ```java
        /**
           * 如果视图获取锁成功，则直接返回；
             如果获取失败，就执行addWaiter方法，将当前线程封装为一个node并使其入队；
             
           */
          public final void acquire(int arg) {
              if (!tryAcquire(arg) &&
                  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                  selfInterrupt();
          }
      ```
      
      - Node  addWaiter(Node node)
      
        将当前线程添加到CLH队列中，这就意味着将当前线程添加到**等待获取锁的等待线程队列中。**
      
      ```java
      /**
           * Creates and enqueues node for current thread and given mode.
       *
           * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
       * @return the new node
           */
      //入队
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
         * 作用：
                 如果CLH队列为空，则新建一个CLH表头；然后将node添加到CLH队尾。
                 否则，直接将node添加到CLH末尾。
             */
            private Node enq(Node node) {
                //无限循环("死循环")入队
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
                        
                         //如果队列为空，创建一个空的标志节点作为head结点，并将tail指向它。
                        initializeSyncQueue();
                    }
                }
            }
        
        //在上面代码中，两个方法都是通过一个CAS方法compareAndSetTail(Node expect, Node update)来设置尾节点，该方法可以确保节点是线程安全添加的。在enq(Node node)方法中，AQS通过“死循环”的方式来保证节点可以正确添加。
    //只有成功添加后，当前线程才会从该方法返回，否则会一直执行下去。
        
    ```
    
        #### 注意：
          
            1. addWaiter返回的是当前加入的尾结点；enq返回的是当前尾结点的前驱结点。
               2. 在网上查到的源码中，enq比addWaiter方法多了一个初始化头结点，并使尾结点指向头结点的过程，但是在我看到的源码中，两个方法均有这个过程，不同点就是返回的值不同，一个返回前驱结点一个返回当前尾结点，还有一个区别就是addWaiter方法会对传入方法的mode进行一个包装，包装为node之后才进行循环。
    
      ```java
      
       /**
           *新建一个表头并将tail指向指向它。
           */
          private final void initializeSyncQueue() {
              Node h;
              if (HEAD.compareAndSet(this, null, (h = new Node())))
                  tail = h;
          }
      ```
    
      
    
      - **boolean acquireQueued(final Node node, int arg)** 

  ```java
  /**
       * 作用：
           逐步的去执行CLH队列的线程，如果当前线程获得了锁，则返回；
           否则，当前线程进行休眠，知道唤醒并重新获取锁了之后才返回。
       */
  
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
  	           如果自己可以休息了，就进入waiting状态，直到被unpark（唤醒）
             **/
                  if (shouldParkAfterFailedAcquire(p, node))
                      //如果在等待过程中被中断过，就将interrupt标记为true
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

  #### 说明：

  ​       该方法返回的是在等待过程中有没有被中断过。

  

  - **shouldParkAfterFailedAcquire和parkAndCheckInterrupt**两个方法用来让当前节点找到一个合适的地方开始等待

    ![image-20200731104654581](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200731104654581.png)

    ```java
    
        /**
         *规则：
            1.如果前继结点状态为signal，表明当前节点需要被唤醒，此时返回true。如果规则一发生了，意味着前继结点是signal状态，意味着当前节点需要被阻塞。接下来会调用parkAndCheckInterrupt方法阻塞该线程，直到当前线程被唤醒才从parkAndCheckInterrupt中返回。
           
           2.如果前继结点状态为cancelled（ws>0）,说明前继结点已经被取消，则通过回溯找到一个有效的结点并返回false。
            3.如果前继结点状态为非signal，非cancelled，则设置前继的状态为signal，并返回false。
         
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

    **其中在尝试获取资源的过程中，首先会判断当前节点的前驱结点是不是头结点（判断当前节点是不是第二个节点），如果是，则尝试获取资源（同步状态）。**

  - 如果前驱结点不是头结点 ，则调用shouldParkAfterFailedAcquire（）方法，为当前结点找到合适的等待的位置。如果该方法返回true即如果找到合适的等待位置，那么当前线程被中断，如果找不到，就释放前驱结点继续往前找。

    - shouldParkAfterFailedAcquire（）：

      首先判断自己的前驱结点是不是等待状态（signal状态），如果是就返回true，代表已经找到了合适的位置。

      如果不是，代表前驱结点已经放弃，那么就一直往前找，直到找到一个正常等待状态的前驱结点，并排在他的后面。

    - parkAndCheckInterrupt()

      返回当前线程的中断状态。

  - 当前驱结点为头结点时，就可以尝试获取同步状态，如果获取成功，就将当前节点设为头结点，然后退出返回。

    #### **注意：**

    1. 如果在等待过程中线程被中断过，那么就不作出响应。只是在获取到资源后才进行一个自我中断，将中断补上。

    2. **在acquireQueued方法中需要完成两件事：第一，增加node节点；第二，挂起线程，使线程处于WAITING状态，等待被唤醒。**

    #### 问题：
    
       **1.线程阻塞之后如何被唤醒？**
    
    ​       答：1）.unpark()唤醒。当前继结点对应的线程使用完锁之后，通过unpark方式唤醒当前线程。
    
    ​               2）.中断唤醒。其他线程通过interrupt中断当前线程。
    
       **2.为什么要执行一个selfInterrupt（），即为什么要自己产生一个中断？**
    
    ​       答：这必须结合acquireQueued()进行分析，这个selfInterrupt方法只会在acquireQueued()时线程被中断过的情况下才会执行，否则不会执行。在这个方法中，即使是线程在阻塞状态被中断获取cpu执行的权利，但是他需要重新在循环中判断，**如果该线程的前面还有其他等待锁的线程，根据公平性原则，该线程依然无法获取到锁。他会再次阻塞。**
    
    ​         即在线程真正获取到锁并执行起来之前，他的中断会被忽略并且中断标记会被清除。在parkAnd。。中，线程的中断状态时调用了Thread.interrupted方法。该函数不同于Thread的isInterrupted函数，后者仅返回中断状态，而前者返回状态之后，会清除中断状态。正因为之前的中断状态被清除了，所以这里需要调用selfInterrupt重新产生一个中断，记录线程的中断状态。

- #### 锁的释放 release(int)

  以ReentrantLock，它释放锁的操作为unlock。unlock方法的底层实现即为release方法。

  此方法是独占模式下线程释放公共资源的顶层入口。它会释放指定量的资源，如果彻底释放了（state = 0），他会唤醒等待队列李的其他线程来获取资源。

  ```java
    /**
       * Releases in exclusive mode.  Implemented by unblocking one or
       * more threads if {@link #tryRelease} returns true.
       * This method can be used to implement method {@link Lock#unlock}.
       *
       * @param arg the release argument.  This value is conveyed to
       *        {@link #tryRelease} but is otherwise uninterpreted and
       *        can represent anything you like.
       * @return the value returned from {@link #tryRelease}
       */
      public final boolean release(int arg) {
          if (tryRelease(arg)) {
              Node h = head; //找到头结点
              if (h != null && h.waitStatus != 0)
                  unparkSuccessor(h); //唤醒等待队列里的下一个线程
                                     //用unpark唤醒等待队列中最前边的那个未放弃的线程
              return true;
          }
          return false;
      }
  ```

  - tryRelease()

    ```java
     //尝试释放锁
    /*
      1.如果当前线程不是锁的持有者，则抛出异常
      2.如果当前线程在本次释放锁操作之后，对锁的拥有状态时0，则设置锁的持有者为null，即锁是可获取状态。同时将当前线程的锁的状态设为0.
    
    */
     protected boolean tryRelease(int arg) {
            throw new UnsupportedOperationException();
        }
    ```

    #### 总结：

    ​      释放锁的过程要比获取锁的过程简单很多，主要做的工作就是跟新当前线程对应的锁的状态。

    ​     1.如果当前线程已经对该锁全部释放即state=0，则**设置锁的持有线程为null**，设置当前线程的状态为空。

    ​     2.**然后唤醒后继线程。**

  - #### 共享模式下过程分析

    **1.获取锁：**

    ​    和独占模式的过程大同小异。

    ​    不同点：当获取到资源之后，还会唤醒后继队友的操作，这体现了共享锁的定义。

    **2.释放锁：**

    ​    共享模式下不会有重入锁的现象出现，也就没有独占模式下，要求state=0时才可以唤醒后继线程的要求。
    
    
