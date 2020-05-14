# SpringBoot项目中遇到的很多坑

## 4.16

- ### SpringBoot与Mybatis整合

  1.SpringBoot与mybatis项目整合时，有可能会遇到项目无法启动的问题。

     在使用mybatis-generator插件逆向生成配置文件时，运行多次插件可能会在一个配置文件中生成重复的代   码。

     生成重复代码会导致程序无法运行起来。

     可能会报的错误

  ```java
  Result Maps collection already contains value for com.dao.UserMapper.BaseResultMap
  ```

     意思就是id为BaseResultMap在一个xml文件中出现多次。

​        2.SpringBoot中的启动类的注解问题

```java
@SpringBootApplication(scanBasePackages = "com")
@MapperScan(basePackages = "com.dao")
```

 springboot中的启动类之上需要有这两个注解，第一个表示项目启动时自动扫描com包下的所有注解；第二个表示项目启动时自动给dao包下的所有类加上mapper注解（也可以一个一个加上mapper注解）

- ### 插入用户的一些小问题

  因为在设置密码的时候，数据库需要将密码进行MD5加密，所以！要将数据库中password的长度定的长一些，因为MD5加密后密码会变得很长！！！
  
- ### 关于jar包

  因为在上传头像或者是上传图片现在用的是七牛云存储，今天把jar包先复制粘贴到了pom文件里面，随之又把七牛云的类传到了项目中。。然后发现全是红色错误。。

  后来才知道是因为maven工程中并没有立即把七牛的jar包导入进去。

  解决方法：

  右上角的maven工程 可以选择reimport，反正就是重新导入一下！！

- ### 关电脑前的想法

  今天学习效率不高不低，但是因为是周四，脑子里有想出去玩又想学习。希望以后的周四改进8。

  目前觉得写学习笔记真的很好，希望以后写项目遇到bug可以继续记录，积累更多经验，防止以后入坑！！！

  

  ## 4.24更新

  ### 0:17
  
  最近开始写一个诗词的项目，跟之前的项目不同，这个项目需要提前录入大量的诗词以及使人的数据，但是经过一些挫折还是基本解决问题啦！
  
  #### 关于录入数据
  
  首先，可以在一个诗词库中获取所有的诗词资源，但是！数据太多了并不是所有数据都是有用数据，目前我只提取了唐诗宋词的诗句。
  
  首先可以将这个库下载下来供自己来使用。但是数据的形式全部都是json文件，这就需要用一个controller来将json文件中的数据解析到pojo中然后调用dao层的接口才传入数据库。
  
  但是会遇到很多奇奇怪怪的字符！！导致无法插入到数据库里面。因为数据庞大，所以可以用一个循环来控制不同的文件，然后每次遇到异常之后就打印该条记录的关键字并跳过该记录。
  
  对于这些有异常的数据可以单独插入。
  
  注意！这里用异常处理指的是在插入时编码出了问题，如果是sql语法的问题还需要重新找原因。
  
  #### 关于sql语法
  
  在插入作者信息的时候发现一直报语法错误但是，所有信息都无误，后来百度之后才知道是因为列名中出现了关键字。这时候要么就修改列名要么就将关键字用``引用起来才可以。
  
  注意！不是单引号是``
  
  
  
  明天继续加油吧
  
  晚安。
  
  
  
  ## 4.26更新
  
  ### 关于 mybatis-generator生成器
  
  刚才在写注册接口的时候，我本来是直接粘贴了上一个项目中的注册接口，然后发现插入数据失败啦。一看是dao层出了问题，我看着调用的方法时生成器写好的方法咋会失败……
  
  结果是这个生成器生成的pojo层的实体类是上一个数据库中同名的实体类。。。。迷惑
  
  然后百度解决啦！需要在生成器的配置文件中加一句配置
  
  **[generatorConfig.xml]**
  
  ```xml
    <!--jdbc的数据库连接 -->
          <jdbcConnection
                  driverClass="${db.driverClassName}"
                  connectionURL="${db.url}"
                  userId="${db.username}"
                  password="${db.password}">
               (ps:下面这一句是重点)
              <property name="nullCatalogMeansCurrent" value="true"/>
          </jdbcConnection>
  ```
  
  
  
  ## 4.28更新
  
  - ### 怎样确定(七牛云)的加速域名是否解析正确？
  
    windows系统下可以通过
  
      nslookup+加速域名 回车查看是否解析正确
  
    linux/mac系统下可以通过 dig+加速域名 回车查看

## 5.3更新

- ### 昨天晚上在centos下传项目遇到了一个问题：

   当我把修改好的项目打成jar包传上去的时候，启动项目出现了无法启动的项目，发现一直运行的还是之前的jar包对应的项目。

  这是因为一开始设置的后台一直运行这个项目，必须先要杀死这个进程(如果修改好的项目端口改变的话可以不管这个)，然后重新执行新的jar包。

  对应的centos命令为：

   **根据端口查看这个进程的pid** 

  ```
  netstat -lnp|grep 8080   #8080请换为你的apache需要的端口
  ```

   **查看进程的详细信息**

```
  ps 1777    #1777就是上一步得到的pid
```

​         **杀掉进程** 

```
 kill -9 [PID]  #-9 表示强迫进程立即停止
```

## 5.4更新

- ### springboot设置session过期时间

  ##### 第一种方式：在application.yml中修改

  ```yml
  server:
    port: 1111
    servlet:
      session:
        timeout: 10800s
  ```

  ##### 第二种方式：在启动类中写代码修改

  ```java
  //设置session过期时间
       @Bean
       public EmbeddedServletContainerCustomizer containerCustomizer() {
           return new EmbeddedServletContainerCustomizer() {
               @Override
               public void customize(ConfigurableEmbeddedServletContainer container) {
                   container.setSessionTimeout(60);// 单位为S
               }
           };
       }
  ```

- ### 关于websocket的使用

  ```jsva
  
  
  
  /**
   * @Auther: zj
   * @Date: 2018/8/16 17:55
   * @Description: websocket的具体实现类
   * 使用springboot的唯一区别是要@Component声明下，而使用独立容器是由容器自己管理websocket的，
   * 但在springboot中连容器都是spring管理的。
      虽然@Component默认是单例模式的，但springboot还是会为每个websocket连接初始化一个bean，
      所以可以用一个静态set保存起来。
   */
  @ServerEndpoint(value = "/websocket")
  @Component
  public class MyWebSocket {
      //用来存放每个客户端对应的MyWebSocket对象。
      private static CopyOnWriteArraySet<MyWebSocket> webSocketSet = new CopyOnWriteArraySet<MyWebSocket>();
      //与某个客户端的连接会话，需要通过它来给客户端发送数据
      private Session session;
      /**
       * 连接建立成功调用的方法
       */
      @OnOpen
      public void onOpen(Session session) {
          this.session = session;
          webSocketSet.add(this);     //加入set中
          System.out.println("有新连接加入！当前在线人数为" + webSocketSet.size());
          this.session.getAsyncRemote().sendText("恭喜您成功连接上WebSocket-->当前在线人数为："+webSocketSet.size());
      }
      /**
       * 连接关闭调用的方法
       */
      @OnClose
      public void onClose() {
          webSocketSet.remove(this);  //从set中删除
          System.out.println("有一连接关闭！当前在线人数为" + webSocketSet.size());
      }
      /**
       * 收到客户端消息后调用的方法
       *
       * @param message 客户端发送过来的消息*/
      @OnMessage
      public void onMessage(String message, Session session) {
          System.out.println("来自客户端的消息:" + message);
          //群发消息
          broadcast(message);
      }
      /**
       * 发生错误时调用
       *
       */
      @OnError
      public void onError(Session session, Throwable error) {
          System.out.println("发生错误");
          error.printStackTrace();
      }
      /**
       * 群发自定义消息
       * */
      public  void broadcast(String message){
          for (MyWebSocket item : webSocketSet) {
              //同步异步说明参考：http://blog.csdn.net/who_is_xiaoming/article/details/53287691
              //this.session.getBasicRemote().sendText(message);
              item.session.getAsyncRemote().sendText(message);//异步发送消息.
          }
      }
  }
  ```

  



```java

package com.example.socket.code; 
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.util.concurrent.ConcurrentHashMap;
 
/**
 * @Auther: liaoshiyao
 * @Date: 2019/1/11 11:48
 * @Description: websocket 服务类
 */
 
/**
 *
 * @ServerEndpoint 这个注解有什么作用？
 *
 * 这个注解用于标识作用在类上，它的主要功能是把当前类标识成一个WebSocket的服务端
 * 注解的值用户客户端连接访问的URL地址
 *
 */
 
@Slf4j
@Component
@ServerEndpoint("/websocket/{name}")
public class WebSocket {
 
    /**
     *  与某个客户端的连接对话，需要通过它来给客户端发送消息
     */
    private Session session;
 
     /**
     * 标识当前连接客户端的用户名
     */
    private String name;
 
    /**
     *  用于存所有的连接服务的客户端，这个对象存储是安全的
     */
    private static ConcurrentHashMap<String,WebSocket> webSocketSet = new ConcurrentHashMap<>();
 
 
    @OnOpen
    public void OnOpen(Session session, @PathParam(value = "name") String name){
        this.session = session;
        this.name = name;
        // name是用来表示唯一客户端，如果需要指定发送，需要指定发送通过name来区分
        webSocketSet.put(name,this);
        log.info("[WebSocket] 连接成功，当前连接人数为：={}",webSocketSet.size());
    }
 
 
    @OnClose
    public void OnClose(){
        webSocketSet.remove(this.name);
        log.info("[WebSocket] 退出成功，当前连接人数为：={}",webSocketSet.size());
    }
 
    @OnMessage
    public void OnMessage(String message){
        log.info("[WebSocket] 收到消息：{}",message);
        //判断是否需要指定发送，具体规则自定义
        if(message.indexOf("TOUSER") == 0){
            String name = message.substring(message.indexOf("TOUSER")+6,message.indexOf(";"));
            AppointSending(name,message.substring(message.indexOf(";")+1,message.length()));
        }else{
            GroupSending(message);
        }
 
    }
 
    /**
     * 群发
     * @param message
     */
    public void GroupSending(String message){
        for (String name : webSocketSet.keySet()){
            try {
                webSocketSet.get(name).session.getBasicRemote().sendText(message);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
 
    /**
     * 指定发送
     * @param name
     * @param message
     */
    public void AppointSending(String name,String message){
        try {
            webSocketSet.get(name).session.getBasicRemote().sendText(message);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```



```java


import org.springframework.stereotype.Component;

import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.util.concurrent.CopyOnWriteArraySet;

@ServerEndpoint(value = "/websocket")
@Component
public class MyWebSocket {
    //用来存放每个客户端对应的MyWebSocket对象。
    private static CopyOnWriteArraySet<MyWebSocket> webSocketSet = new CopyOnWriteArraySet<MyWebSocket>();
    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;

    /**
     *      * 连接建立成功调用的方法
     *     
     */
    @OnOpen
    public void onOpen(Session session) {
        this.session = session;
        webSocketSet.add(this);//加入set中
        System.out.println("有新连接加入！当前在线人数为" + webSocketSet.size());
        this.session.getAsyncRemote().sendText("恭喜您成功连接上WebSocket-->当前在线人数为：" + webSocketSet.size());
    }

    /**
     *      * 连接关闭调用的方法
     *     
     */
    @OnClose
    public void onClose() {
        webSocketSet.remove(this); //从set中删除
        System.out.println("有一连接关闭！当前在线人数为" + webSocketSet.size());
    }

    /**
     *      * 收到客户端消息后调用的方法
     *      *
     *      * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("来自客户端的消息:" + message);
//群发消息
        broadcast(message);
    }

    /**
     *      * 发生错误时调用
     *      *
     *     
     */
    @OnError
    public void onError(Session session, Throwable error) {
        System.out.println("发生错误");
        error.printStackTrace();
    }

    /**
     *      * 群发自定义消息
     *      *
     */
    public void broadcast(String message) {
        for (MyWebSocket item : webSocketSet) {
            //同步异步说明参考：http://blog.csdn.net/who_is_xiaoming/article/details/53287691
            //this.session.getBasicRemote().sendText(message);
            item.session.getAsyncRemote().sendText(message);//异步发送消息.
        }
    }
}
```



// https://blog.csdn.net/lyf_ldh/article/details/83181994 

## 5.8更新

- ### websocket个人理解

  刚刚理了一下websocket的使用思路，springboot+websocket的配置大概为：

  ##### 首先引入需要的jar包

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-websocket</artifactId>
  </dependency>
  
  <!--websocket作为客户端-->
  
  <dependency>
  	<groupId>org.java-websocket</groupId>
  	<artifactId>Java-WebSocket</artifactId>
  	<version>1.3.5</version>
  </dependency>
  
  </dependency>
  		<dependency>
  			<groupId>org.slf4j</groupId>
  			<artifactId>slf4j-api</artifactId>
  		</dependency>
  ```

  注意：

  1.对于第二个jar包，我查了一下大概是可以用java实现一个客户端，从而可以自己用客户端和服务端来进行测试，但是实际开发中作为后端只需要第一个，客户端的请求可以让前端来实现。

  2.对于第三个jar包，是在实现之后的server时打印日志所需要的

  ##### 然后需要一个配置文件来支持websocket()

  ```java
  
  /**
   * 开启WebSocket支持
   * Created by huiyunfei on 2019/5/31.
   */
  @Configuration
  public class WebSocketConfig {
      @Bean
      public ServerEndpointExporter serverEndpointExporter() {
          return new ServerEndpointExporter();
      }
  }
  ```

  最后需要自己实现服务端的websocketserver

  ```java
  @ServerEndpoint("/websocket/{sid}")
  @Component
  @Slf4j
  public class WebSocketServer {
      //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
      private static int onlineCount = 0;
      //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
      private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();
  
      //与某个客户端的连接会话，需要通过它来给客户端发送数据
      private Session session;
      //接收sid
      private String sid="" ;
      private Logger log = LoggerFactory.getLogger(WebSocketServer.class);
  /**
   * 连接建立成功调用的方法*/
      @OnOpen
      public void onOpen(Session session, @PathParam("sid") String sid) {
          this.session = session;
          webSocketSet.add(this);     //加入set中
          addOnlineCount();           //在线数加1
          log.info("有新窗口开始监听:"+sid+",当前在线人数为" + getOnlineCount());
          this.sid=sid;
          try {
              sendMessage("连接成功");
          } catch (IOException e) {
              log.error("websocket IO异常");
          }
      }
  
  /**
   * 连接关闭调用的方法
   */
      @OnClose
      public void onClose() {
          webSocketSet.remove(this);  //从set中删除
          subOnlineCount();           //在线数减1
          log.info("有一连接关闭！当前在线人数为" + getOnlineCount());
      }
  
  /**
   * 收到客户端消息后调用的方法
   *
   * @param message 客户端发送过来的消息*/
      @OnMessage
      public void onMessage(String message, Session session) {
          log.info("收到来自窗口"+sid+"的信息:"+message);
          //群发消息
          for (WebSocketServer item : webSocketSet) {
              try {
                  item.sendMessage(message);
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
  /**
   *
   * @param session
   * @param error
   */
      @OnError
      public void onError(Session session, Throwable error) {
          log.error("发生错误");
          error.printStackTrace();
      }
  
  /**
   * 实现服务器主动推送
   */
      public void sendMessage(String message) throws IOException {
          this.session.getBasicRemote().sendText(message);
      }
  
  /**
   * 群发自定义消息
   * */
      public static void sendInfo(String message,@PathParam("sid") String sid) throws IOException {
          WebSocketServer ws = new WebSocketServer();
  
          ws.log.info("推送消息到窗口"+sid+"，推送内容:"+message);
          System.out.println(message);
          for (WebSocketServer item : webSocketSet) {
              try {
                  //这里可以设定只推送给这个sid的，为null则全部推送
                  if(sid==null) {
                      item.sendMessage(message);
                      System.out.println("1ok");
                  }else if(item.sid.equals(sid)){
                      item.sendMessage(message);
                      System.out.println("2ok");
                  }
              } catch (IOException e) {
                  System.out.println("推送失败");
                  continue;
              }
          }
      }
      public static synchronized int getOnlineCount() {
          return onlineCount;
      }
      public static synchronized void addOnlineCount() {
          WebSocketServer.onlineCount++;
      }
      public static synchronized void subOnlineCount() {
          WebSocketServer.onlineCount--;
      }
      public static CopyOnWriteArraySet<WebSocketServer> getWebSocketSet() {
          return webSocketSet;
      }
  }
  ```

  ## 5.12更新
  
  ![1589289004508](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\1589289004508.png)

​        昨天整合了springboot和redis缓存数据，在连接redis成功之后发现自己设定的key可以缓存进redis，但是key之前出现了一堆很难看很奇怪的字符，今天学习的是怎样优化原pojo的序列化方式。

###  解决流程：

1.先把user的序列化删除

2.创建RedisConfiguration

3.flushdb 清空redis旧的数据，因为改了序列化，老数据已经不能兼容了，必须清空旧数据。

- ### 关于springboot+redis的连接问题

  其实昨天一直困扰了我很久的是springboot项目中集成了redis之后，在项目启动之后，一直无法连接成功redis，但是！通过我的努力，这个问题还是解决了。

  其实连接失败的原因我遇到了两种：

  1.连接超时。

  2.无法连接。主要原因就是报错信息为，客户端发起了密码验证，但是redis的配置文件中并没有配置密码。

  - #### 连接超时：

     1.找到redis.conf文件并打开修改。

    ```
    找到 protected-mode no，修改成yes
    protected-mode  no  →  protected-mode  yes
    ```
    
    2.linux下开放6379端口并且重启云服务器；同时阿里云中的安全组也要将6379端口规则添加进去。
    
    3.在redis.conf文件中找到bind 127.0.0.1,改成bind 0.0.0.0（或者直接将这个命令注释掉哦）
    
    4.保护模式
    
    ```
    将protected-mode yes改为      protected-mode no
    ```
    
     .....这是目前收集到过的问题，之后再遇到问题更新。
    
  - #### 无法连接
  
    问题描述：项目启动之后提示客户端发起了redis远程连接的密码验证，但是服务端redis并没有设置密码。
  
    解决：
    
    ​      如果原项目的配置文件中配置了redis的password，例如：
    
          ```yml
    spring:
      datasource:
        url: jdbc:mysql://118.31.12.175:3306/poets?serverTimezone=UTC
        username: root
        driver-class-name: com.mysql.cj.jdbc.Driver
      redis:
        database: 0
        host: 118.31.12.175
        timeout: 10000
        port: 6379  #可不配，因为底层默认值为6379
        password: 
          ```
    
    注意：如果redis并没有设置密码,那么原配置文件中就不要写password；
    
    ​            如果要设置不需要password，在redis.conf配置文件中要将 
    
    ```
    requirepass ...
    ```
    
    ​           这一项注释掉才可以。
    
    
    



​               