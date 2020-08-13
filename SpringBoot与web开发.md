#                     SpringBoot与web开发

## 一.怎样使用SpringBoot

1. 创建springboot应用，选中需要的模块。
2. 在Springboot的默认场景配置下，在配置文件中指定少量的配置可以运行。
3. 编写业务代码。

**注意：需要掌握springboot的自动配置原理**



## 二.springboot对静态资源的映射

。。。

## 三.springboot与swagger整合

4.Restful接口（注解个别解释如下）
@ApiOperation：
@ApiOperation不是spring自带的注解是swagger里的
@ApiOperation(value = “接口说明”, httpMethod = “接口请求方式”, response = “接口返回参数类型”, notes = “接口发布说明”；其他参数可参考源码；
@ApiImplicitParam：
@ApiImplicitParam(value = “备注输入参数名称（中文）”, name = “备注输入参数名称（英文）”, required = （true、false）该入参是否必填, dataType = “（String）该入参的数据类型”, paramType = “（query）前台接口调用时url 参数形式”)
如：query 的形式：getUser?user =admin
path的形式：getUser/user/admin

@RestController
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下，可去除
public class UserController {

    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());
    
    @ApiOperation(value="获取用户列表", notes="")
    @RequestMapping(value={""}, method=RequestMethod.GET)
    public List<User> getUserList() {
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }
    
    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestBody User user) {
        users.put(user.getId(), user);
        return "success";
    }
    
    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long id) {
        return users.get(id);
    }
    
    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @RequestBody User user) {
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }
    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }



## 5.2更新

- ### mybatis实现当插入数据之后返回刚刚插入的id

  实现方法：在sql语句的属性中加入<keyProperty="id" useGeneratedKeys="true">这两个属性

  ```xml
    <!-- MySQL中获取主键并插入1 -->
      <insert id="insertUser" parameterType="user" keyProperty="userId" useGeneratedKeys="true">
          insert into USER(u_id, u_name, u_pwd) values(#{userId}, #{userName}, #{userPassword})
      </insert>
  ```


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



- ### springboot中的监听器的分类

  1.监听Servlet上下文对象

  2.监听HTTP会话 Session对象

  3.监听客户端请求Servlet Request对象

  4.在SpringBoot中自定义监听事件

  前三种不详细写了，下面说一下自定义事件监听器的使用方法

  **自定义spring监听器和自定义spring事件**

  **自定义spring事件**

  ```java
  package com.hzt.listener;
  import org.springframework.context.ApplicationEvent;
  public class MySpringEvent extends ApplicationEvent {
      public MySpringEvent(Object source) {
          super(source);
      }
  ```

  自定义spring监听器,监听自定义事件

  ```java
  package com.hzt.listener;
  
  import org.springframework.context.ApplicationListener;
  import org.springframework.stereotype.Component;
  
  @Component
  public class MySpringListener implements ApplicationListener<MySpringEvent> {
      @Override
      public void onApplicationEvent(MySpringEvent mySpringEvent) {
          System.out.println("spring监听自定义的事件======"+mySpringEvent.getClass());
      }
  }
  ```

  同时还有spring监听器监听本身事件

  ```java
  @Component
  public class SpringSelfListener implements ApplicationListener<ApplicationEvent> {
      @Override
      public void onApplicationEvent(ApplicationEvent ApplicationEvent) {
          System.out.println("spring监听本身事件========="+ApplicationEvent);
      }
  ```

  springboot启动

  ```java
  @SpringBootApplication
  @ServletComponentScan("com.hzt")//servlet监听器的使用
  public class Application {
      /*public static void main(String[] args) {
          SpringApplication.run(Application.class,args);
      }*/
      
  public static void main(String[] args) {
      ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
      context.publishEvent(new MySpringEvent(new Object()));//发布事件
  }
  ```

  ```
  spring监听本身事件=========org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@8940838: startup date [Wed Jul 24 15:23:15 CST 2019]; root of context hierarchy]
  2019-07-24 15:23:19.282  INFO 12464 --- [  restartedMain] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
  spring监听本身事件=========org.springframework.boot.context.embedded.EmbeddedServletContainerInitializedEvent[source=org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer@16a7f909]
  spring监听本身事件=========org.springframework.boot.context.event.ApplicationReadyEvent[source=org.springframework.boot.SpringApplication@6717ce81]
  2019-07-24 15:23:19.289  INFO 12464 --- [  restartedMain] com.hzt.Application                      : Started Application in 4.485 seconds (JVM running for 5.503)
  spring监听自定义的事件======class com.hzt.listener.MySpringEvent
  spring监听本身事件=========com.hzt.listener.MySpringEvent[source=java.lang.Object@1492c960]
  ```


  可知,springboot发布事件是找到所有相关的监听器,调用其监听方法,把事件作为参数传输
  如果在监听方法里,传输的事件是ApplicationEvent,那么当前该spring监听器可以监听所有spring事件,因为spring事件都继承ApplicationEvent

  当自定义spring监听器时,一定要想好这个监听器是监听什么事件,而不是直接在监听方法里面写ApplicationEvent参数
  如果写ApplicationEvent参数,就是代表所有的只要继承spring事件ApplicationEvent的都监听.

  从事件里一定是可以拿到事件源的



## 8.13学习更新

### SpringBoot中随机数的设置方法

可以在下一个项目中使用到:

**1.添加config/random.properties文件，添加以下内容：**

```java
#随机32位MD5字符串
user.random.secret=${random.value}

#随机int数字
user.random.intNumber=${random.int}

#随机long数字
user.random.longNumber=${random.long}

#随便uuid
user.random.uuid=${random.uuid}

#随机10以内的数字
user.random.lessTen=${random.int(10)}

#随机1024~65536之内的数字
user.random.range=${random.int[1024,65536]}

```



**2.添加一个配置文件的绑定类**

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;
@Component
@ConfigurationProperties(prefix = "user.random")
@PropertySource(value = { "config/random.properties" })
public class RandomConfig {

    private String secret;
    private int intNumber;
    private int lessTen;
    private int range;
    private long longNumber;
    private String uuid;

    public String getSecret() {
        return secret;
    }

    public void setSecret(String secret) {
        this.secret = secret;
    }

    public int getIntNumber() {
        return intNumber;
    }

    public void setIntNumber(int intNumber) {
        this.intNumber = intNumber;
    }

    public int getLessTen() {
        return lessTen;
    }

    public void setLessTen(int lessTen) {
        this.lessTen = lessTen;
    }

    public int getRange() {
        return range;
    }

    public void setRange(int range) {
        this.range = range;
    }

    public long getLongNumber() {
        return longNumber;
    }

    public void setLongNumber(long longNumber) {
        this.longNumber = longNumber;
    }

    public String getUuid() {
        return uuid;
    }

    public void setUuid(String uuid) {
        this.uuid = uuid;
    }
    
}
```

