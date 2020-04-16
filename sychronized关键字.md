# sychronized关键字

> > synchronized（同步的）关键字是Java多线程的关键字，主要用来给对象，(静态)方法以及代码块加锁，当它锁定一个方法或者代码块时，同一时间最多有一个线程执行这段代码块；当两个并发线程访问同一个对象中的这个加锁的代码块时，同一时间只能有一个线程执行。



## 修饰对象

- ### 当前的两个线程访问的是同一个对象

  对于加锁的代码块，同一时间只能有一个线程执行，而另一个线程将进入阻塞状态，必须等第一个线程之行结束之后才可以执行下一个线程中的代码。

  ```java
  public class ThreadRunnable implements Runnable {
  
      public static void main(String[] args) {
          ThreadRunnable tr = new ThreadRunnable();
          ThreadRunnable tr1 = new ThreadRunnable();
          Thread thread1 = new Thread(tr);
          Thread thread2 = new Thread(tr);
          thread1.start();
          thread2.start();
  //        tr.fun();
  //        tr1.fun();
      }
  
      public void run(){
          synchronized (this){
              for(int i = 10;i<50;i++){
                  System.out.print(i+" ");
              }
          }
      }
  
  }
  
  ```

  ```java
  1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 
  1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 
  ```

  > 根据结果可看出这时产生了互斥现象，因为两个线程访问的是同一个对象的资源，当修饰了synchronized关键字之后实现了线程同步。

- ### 当前的两个线程访问的是不同的两个对象

  对于加锁的代码块，同一时间两个代码块交叉进行。

  ```java
  public class ThreadRunnable implements Runnable {
  
      public static void main(String[] args) {
          ThreadRunnable tr = new ThreadRunnable();
          ThreadRunnable tr1 = new ThreadRunnable();
  
          Thread thread1 = new Thread(tr);
          Thread thread2 = new Thread(tr1);
          thread1.start();
          thread2.start();
  //        tr.fun();
  //        tr1.fun();
  
      }
  
      public void run(){
          synchronized (this){
              for(int i = 1;i<30;i++){
                  System.out.print(i+" ");
              }
              System.out.println();
  
          }
      }
  
  }
  
  ```

  ```java
  1 1 2 2 3 3 4 4 5 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 
  6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 
  
  ```

  #### 注意：

  #### 1.上述例子是通过修饰代码块来“锁对象的”，如果修饰的是静态的方法，产生互斥的条件与上述例子相同。

  #### 2.相同情况下同步代码快的效率比同步方法的效率高。

## 修饰类

> >当关键字修饰的是静态方法或直接修饰类时，产生的锁为类锁。

- #### 类锁的两种形式

  - synchronized static void test();
  - synchronized(Person.class);

- #### 类锁怎样产生互斥现象？(类锁怎样实现同步)

  

  ```java
  public class ThreadRunnable implements Runnable {
  
      public static void main(String[] args) {
          ThreadRunnable tr = new ThreadRunnable();
          ThreadRunnable tr1 = new ThreadRunnable();
          Thread thread1 = new Thread(tr);
          Thread thread2 = new Thread(tr1);
          thread1.start();
          thread2.start();
  
      }
      public void run(){
          testStatic();
  
      }
      public void testStatic(){
          synchronized (ThreadRunnable.class){
              for(int i = 1;i<10;i++){
                  System.out.print(i+"->");
              }
              System.out.println();
          }
      }
  
  }
  
  ```

  ####        与上述的对象锁有所不同的是，对象锁在实例化多个对象时不会产生互斥。但是类锁不论正在进行的几个线程访问的是否是同一个对象，都会产生互斥现象。原因是synchronized修饰的是整个类Class对象，即这个类中实例化出来的所有对象都只能共用一把锁。所以都会产生互斥。

- #### 当类锁和对象锁同时存在会不会产生互斥？

  > >不会。因为类锁对应的是整个类，如一个类中的静态资源，它们不属于任何一个对象，在加载类的时候便加载了这些静态资源。而对象锁对应的是实例化出来的某一个对象，这个类中的静态资源无关。所以当两个线程，一个访问类中的上锁的静态方法，一个访问类中上锁的非静态方法时，是不会产生互斥现象的。

  

  

  

  

  

  

  

  

