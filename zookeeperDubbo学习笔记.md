# zookeeper/Dubbo学习笔记

## 引入

### 传统互联网到分布式架构的演变

   （单机应用和分布式应用架构演进基础知识）

​        互联网时代演变过程：

​              **pc时代-->移动互联网时代（增加了手机端）-->物联网时代（万物互联，手机可以控制机器等现象）**



### 几种不同的架构：

- #### 传统单体应用

  **缺点：** 系统耦合度高，开发效率随着时间的增长而降低，启动应用的时间长,依赖庞大(如果一个单体应用的功能很多，那么需要依赖的jar包就越多)。

  **适用场景：**初创公司，业务场景简单，功能单一，研发人员少。

- #### 微服务应用

  将一个应用中不同的功能分成不同模块的服务，分别部署到不同的地方。

  **优点**：易开发，理解和维护；独立部署和启动

  **缺点**：需要管理多个服务，在架构层面变得复杂。服务切分之后，服务之间的可能需要解决分布式的问题。

  

## 微服务核心基础知识

### 1.网关

​     作用：根据客户端的请求决定使用哪一个服务；并根据具体的请求操作过滤某些响应，比如未登录状态下付款，就要过滤掉付款页面，先要求让用户登录。 

​     （  路由转发+过滤器 ）       

### 2.服务注册中心

   作用： 将程序中存在的服务在服务注册中心注册，网关层在根据请求调用某个服务的时候可以直接去服务注册中心调用。

​       （调用和被调用方信息的维护）

### 3.配置中心

​    管理配置， 动态更新application.poperities

​    

### 4.链路追踪

​     分析调用链路耗时

​     **例： 用户下单-->查询商品价格-->查询用户信息-->保存数据库**

### 5.负载均衡器

​    分发负载

### 6.熔断：

​    当某一个服务发生错误，为了不影响别的相关服务，就将出错的服务删掉。

   （保护自己和被调用方）

## dubbo发展过程

2011年阿里发布 dubbo-2.0.7

然后更新维护



2014年再次发布，之后的几年停止更新

同期SpringBoot和SpringCloud出现了



消沉了很久之后，17年捐给了apache

### dubbo和spring-cloud的比较

  相同点：都是微服务的框架

 1.dubbo ：zookeeper + dubbo + springmvc/springboot

​    配套：rpc 性能高 

​    注册中心： zookeeper/redis

​    配置中心：diamond（企业很少用）

2.springcloud ：全家桶+轻松嵌入第三方组件（NetFlix 奈飞）

   配套：

​    通信方式： http restful

​    注册中心： eruka/consul

​    配置中心：config

​    断路器：hystrix

​    网关：zuul

​    分布式追踪系统：sleuth+zipkin

## CAP定理

指的是在一个分布式系统中，Consistency（一致性），Availability（可用性），Partition tolerance（分区容错性），**三者不可同时获得。**

### 一致性（C）：

​      在分布式系统中的所有数据备份，在同一时刻是否同样的值。（所有节点在同一时间的数据完全一致，越多节点，数据同步越耗时）

####       强一致性

####       单调一致性

####       会话一致性

####       最终一致性

####       弱一致性

### 可用性（A）：

​     负载过大后，集群整体是否还能响应客户端的读写请求。（服务一直可用，而且是正常响应时间）

### 分区容错性（P）：

​    分区容忍性，就是高可用行，一个结点崩了，并不影响其他的结点（100个结点，挂了几个，不影响服务，机器越多越好）

（一个分布式系统必须能够保证分区容错性）

**CAP理论就是说在分布式的存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题，分区容忍性使我们必须要实现的。所以我们之只能在一致性和可用性之间进行权衡。**         

![image-20200819173653872](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200819173653872.png)

   



## zookeeper在win,linux下的安装

- ### linux下的安装路径：

     /usr/local

- ### 查看zookeeper的运行状态

  ```
   在准备好相应的配置之后，可以直接通过zkServer.sh 这个脚本进行服务的相关操作
  1. 启动ZK服务:       sh bin/zkServer.sh start
  2. 查看ZK服务状态: sh bin/zkServer.sh status
  3. 停止ZK服务:       sh bin/zkServer.sh stop
  4. 重启ZK服务:       sh bin/zkServer.sh restart
  ```

  ![image-20200819211957520](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200819211957520.png)

  

```
tickTime=2000  #心跳时间,zookeeper中最基本的时间单位，毫秒级别，2000毫秒=2秒
initLimit=10  #两个服务器之间连接能容忍的最多的心跳数 #结合tickTime，最多容忍的时间为tickTime*initLimit=20秒左右
syncLimit=5  #两个服务器能容忍的连接时的最大失败数
dataDir=/tmp/zookeeper  #这个路径要修改成自己创建的data文件夹的目录的路径
clientPort=2181

```



## zookeeper在windows端和linux端的连接

   1.在win和linx端安装好zookeeper

   2.各自修改配置文件

   3.在win端打开windows powershall命令框进入zookeeper的bin目录。

   4.使用命令：

     ```
 .\zkCli.cmd -timeout 5000 -server 118.31.12.175:2181
     ```

ps:我在连接的时候一直报错，一直显示无法定位登录配置，无语……

![image-20200819211321521](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200819211321521.png)



## 8.29学习更新

### zookeeper在centos下启动时的注意事项

在启动时，首先需要切换一下用户，需要由root用户切换到自定义的用户上。

```
 chown -R zookeeper:zookeper zookeeper-3.4.14/
 su zookeeper
 cd zookeeper-3.4.14/bin
 ./zkServer.sh start ---->zookeeper启动命令
```



- ### zookeeper常用命令值zkCli

  ####   zkCli.sh

     不填后面的参数，默认连接的就是localhost：2181

     **win下面运行的是 .cmd结尾的文件， linux下运行的是.sh结尾的文件。**

  - ####  连接远程服务器

      ```
    zkCli.sh -timeout 0 -r -server ip:port
    ```

  - **zkcli.sh h 出现相应的帮助信息**

    ```
    ZooKeeper -server host:port cmd args
            stat path [watch]
            set path data [version]
            ls path [watch]
            delquota [-n|-b] path
            ls2 path [watch]
            setAcl path acl
            setquota -n|-b val path
            history
            redo cmdno
            printwatches on|off
            delete path [version]
            sync path
            listquota path
            rmr path
            get path [watch]
            create [-s] [-e] path data acl   //创建节点
            addauth scheme auth
            quit
            getAcl path
            close
            connect host:port
    ```

  - #### zookeeper 节点 

    zookeeper下创建节点:

     **create [-s] [-e] path data acl   //创建节点**

    ​     **-s表示创建顺序节点**

    ​     **-e表示创建临时节点**

    ​     **data表示创建的结点的数据内容**

    ```
    [zk: localhost:2181(CONNECTED) 0] create -s /couclass cclass
    Created /couclass0000000000
    [zk: localhost:2181(CONNECTED) 1] create -s /couclass1 cclass1
    Created /couclass10000000001
    ```

    注意：在创建节点时，节点的路径必须是相对于根节点的。即结点的路径前必须加“/”

    

    - **查看**:

       1.获取结点的子节点： 

       ```
      ls path
      ```

       2.获取结点的状态:

      ```
      [zk: localhost:2181(CONNECTED) 1] stat /couclass0000000000
      cZxid = 0x4  --事务的id
      ctime = Sat Aug 29 21:03:40 CST 2020  --结点创建的时候的时间
      mZxid = 0x4  --最后一次更新时事务的id
      mtime = Sat Aug 29 21:03:40 CST 2020  --最后一次更新的时间
      pZxid = 0x4   --该节点的子节点列表最后一次被修改的事务id
      cversion = 0  --子节点列表的版本  
      dataVersion = 0  --数据内容的版本
      aclVersion = 0  --acl版本
      ephemeralOwner = 0x0  --用于临时节点，表示创建该临时结点的事务id，如果当前节点不是一个临时结点，该字段的值就是0
      dataLength = 6  --数据内容的长度
      numChildren = 0  --子节点的数量
      ```

      

       3.查看结点的数据:

      ```
      [zk: localhost:2181(CONNECTED) 0] get /couclass0000000000
      cclass
      cZxid = 0x4
      ctime = Sat Aug 29 21:03:40 CST 2020
      mZxid = 0x4
      mtime = Sat Aug 29 21:03:40 CST 2020
      pZxid = 0x4
      cversion = 0
      dataVersion = 0
      aclVersion = 0
      ephemeralOwner = 0x0
      dataLength = 6
      numChildren = 0
      ```

      

       4.获取结点的子节点以及当前结点的状态:

        **(可以先为某一个节点创建一个子节点，然后查看状态)**

      ```
      [zk: localhost:2181(CONNECTED) 17] ls2 /couclass0000000000
      [wiggin]
      cZxid = 0x4
      ctime = Sat Aug 29 21:03:40 CST 2020
      mZxid = 0x4
      mtime = Sat Aug 29 21:03:40 CST 2020
      pZxid = 0xa
      cversion = 1
      dataVersion = 0
      aclVersion = 0
      ephemeralOwner = 0x0
      dataLength = 6
      numChildren = 1
      ```

      

      

      # 9.2更新
  
      ## zookeeper session机制
    
    ​          用于客户端和服务端之间的连接，可设置超时时间，**通过心跳包的机制(客户端向服务端ping包请求)**检查心跳结束，session就过期。
    
    ​          session过期的时候，该session创建的所有临时结点都会被抛弃。
    
    ##     zookeeper watcher 机制
    
     **对节点的watcher操作 get/set stat:**
    
    ​      针对每一个节点的操作，都可以有一个监控者，当结点发生变化，会触发watcher事件；
    
    ​      但是 get stat set命令只能针对当前的结点，如果当前节点下创建了子节点，是不会触发watcher事件的
    
    ​      **zk中watcher是一次性的，触发后立即销毁;**
    
    ​      所有有监控者的节点的变更操作都能触发watcher事件。
    
     **子节点的watcher操作**
    
    （监控父节点，当父节点对应的子节点发生变更的时候，父节点上的watcher事件都会被触发）
    
       **ls ls2**---**增删会触发，修改不会，如果子节点再去新增子节点，不会触发**（**也就是说，触发watcher事件也定是直系子节点**）
    
    
    
    ## zookeeper的acl权限控制
    
     （ps: access control lists）
    
    ​    ![image-20200902210644695](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200902210644695.png)
    
    
    
    ## zookeeper中的选举机制
    
    ![image-20200902211919922](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200902211919922.png)
    
    ​        

​      











