# Redis学习(1)

- ## 为什么要用redis？它解决了什么问题？

  > 是一个高性能的ky-value内存数据库，他支持常用的5种数据结构：String，Hash哈希表，List列表，Set集合，Zset有序集合等数据类型。

  > > 解决的问题？
  > >
  > > 1.性能
  > >
  > > 通常数据库的操作，一般都需要十几秒，而redis的读操作仅需要不到一毫秒。通常只要把数据库的数据缓存进redis，就能得到几十倍甚至上百倍的性能提升。
  > >
  > > 2.并发
  > >
  > > 在大并发的情况下，所有的请求直接访问数据库，数据库会出现连接异常，甚至卡死在数据库中。为了解决卡死的问题，一般的做法是采用redis的一个缓冲操作，让请求先访问到redis，而不是直接访问数据库。
  > >
  > > 3.什么是并发？
  > >
  > > （。。。）

- ## 安装(linux)

  ```
  $ wget http://download.redis.io/releases/redis-5.0.8.tar.gz
  $ tar xzf redis-5.0.8.tar.gz //解压
  $ cd redis-5.0.8  //进入目录
  $ make  //编译
  $ make install //
  ```

- ## 启动命令

   ./src/redis-server

- ## 测试体验

   客户端连接服务器

  ```
  [root@izbp15p4bkvr39g2jyaa9qz redis-5.0.8]# ./src/redis-cli
  127.0.0.1:6379> set user1 coucou
  OK
  127.0.0.1:6379> get user1
  "coucou"
  ```

  注意:

  如果出现连接失败需要修改一下redis.conf文件

  修改的部分为:

  ```
  (守护进程)
  将daemonize no 修改为 daemonize yes
  ```

  然后保存

  执行命令

  ```
  [root@izbp15p4bkvr39g2jyaa9qz redis-5.0.8]# redis-server redis.conf
  28401:C 08 Apr 2020 22:13:50.611 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  28401:C 08 Apr 2020 22:13:50.612 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=28401, just started
  28401:C 08 Apr 2020 22:13:50.612 # Configuration loaded
  ```

  如果提示redis-server命令不存在 

  执行make install命令

  ### 以后台进程方式启动redis

  > >为什么？
  > >
  > >因为 ./redis-server 以这种方式启动redis需要一直打开窗口，不能进行其他操作，不太方便。
  > >
  > >按ctrl+c可以关闭窗口

  ```
  (守护进程)
  将daemonize no 修改为 daemonize yes
  ```

  

