# SpringBoot学习笔记(2)

## 一.配置文件

- ### yml配置文件学习

  YAML表示YAML Ain’t Markup Language，在百度百科的解释是：

  > YAML是"YAML Ain't a Markup Language"（YAML不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言），但为了强调这种语言以数据做为中心，而不是以标记语言为重点，而用反向缩略语重命名。

  所以，我们不用在意它是否是一种标记语言，我们只要记得它是一种以数据为中心的语言就可以，语法非常简洁，使用空白，缩进，分行组织数据，从而使得表示更加简洁易读。

  ## 1、YAML基本语法

  > 引用博客http://www.ruanyifeng.com/blog/2016/07/yaml.html
  >
  > - 大小写敏感
  > - 使用缩进表示层级关系
  > - 缩进时不允许使用Tab键，只允许使用空格。
  > - 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

  所以YAML基本语法其实就是key:(空格)value的形式，其中空格必须要有，以空格的缩进来控制层级关系，只要对齐的一列数据都是同一个层级的，比如：

  ```yaml
  server:
    port: 8081
    path: /example
  ```

  注意：属性key和值都是大小写敏感的

  ## 2、YAML支持的数据结构

  - 字面量：普通的值(整数、浮点数、字符串、布尔、Null值、时间、日期)
    key: value(字面值直接写上就可以)
     字符串也默认不需要加上单引号和双引号的
     单引号：会转义特殊字符，将特殊字符转为一个普通的字符串
     name: 'xiaowang \n' 打印 xiaowang \n (ps:这里的\n被转成字符串)
     双引号：不会转义特殊字符，特殊字符还是表达其本身想表示的意思
     name: 'xiaowang \n' 打印 xiaowang 换行 (ps:这里的\n执行换行操作)
  - 对象：也可以说是map，也就是键值对的形式
    key: value(以对象属性key:value的形式表示，在对象名下一行写属性:属性值,，同样注意空格缩进)
    example：

  ```yaml
  user:
    username: root
    password: rootpwd
    maps: {k1: v1,k2: v2}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 2
  ```

  也可以用行内写法表示：

  ```yaml
  user:{username: root,password: rootpwd}
  ```

  - 数组：也可以说是list或者序列的方式表示
    用"-"符合+值的方式数组中一个元素

  ```yaml
  pets:
    - cat
    - dog
  ```

  显然也有行内写法，用[]中括号表示

  ```yaml
  pets: [cat,dog]
  ```

## 3.springboot 配置文件处理器的依赖

```xml
<!‐‐导入配置文件处理器，配置文件进行绑定就会有提示‐‐>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐configuration‐processor</artifactId>
    <optional>true</optional>
</dependency>
```



## 二.重要注解

- ### @ConfigurationProperties 和 @Value比较

![1585746608931](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\1585746608931.png)

注：

1. 松散语法：

   ```
   - person.firstName:使用标准方式
   - person.first-name:大写用-         
   - person.first_name:大写用_
   - PERSON_FIRST_NAME: 系统属性推荐使用这种写法
   ```

2. Spel：即指EL表达式

   举例：

   ```
   userAge = 12   //可以
   userAge = #{2*6}  //EL表达式不支持
   ```

3. 数据校验：

    @ConfigurationProperties＋ @Validated 支持 

   举例：

   ```java
   import javax.validation.constraints.NotNull;
   
   import org.hibernate.validator.constraints.Email;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.validation.annotation.Validated;
   
   @ConfigurationProperties
   @Validated
   public class Properties {
   
       @NotNull
       private String userName;
       
       @Email
       private String email;
       
   }
   ```

   

- ### @PropertySource&@ImportResource

  **@PropertySource：**加载指定的配置文件

```java
@Component
@ConfigurationProperties(prefix = "person") //该注解定位的配置文件为全局配置文件
//@Validated
@PropertySource(value = {"classpath:person.properties"})
public class Person {
    private Integer id;
    //@Email 这个注解的意思是name的格式必须是邮箱的格式
    private String name;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```



**@ImportResource**

导入spring的配置，比如bean.xml文件，让配置文件里面的内容生效；

SpringBoot中没有spring的配置文件，我们自己编写的配置文件也不能自动识别。

想让生效，需要加这个注解。

但是，这并不是SpringBoot推荐的注解方式，比较麻烦。



> SpringBoot推荐的注解方式：

1.不去编写Spring配置文件。

2.编写一个配置类====代Spring配置文件

3.使用@Bean给容器中添加组件

```java
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中;容器中这个组件默认id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }

}
```



## 三.配置文件占位符

### 1.随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

### 2.占位符获取之前的值，如果之前没有可以用冒号来指定值。

还可以使用我们这个配置文件中自己在前面定义的值，例如：

```
person.last-name=张三
person.dog.name=${person.last-name}小狗
```

还有，我们如果前面没有这个值呢？

我们可以使用${app.name:金毛}来指定找不到属性时的默认值。



## 四.Profile文件的编写

- ### 多profile文件

  可以编写多个profile文件，然后动态切换，如动态切换端口的设置。

- ### yml文件支持多文档快方式

   “---”为分隔符，将配置文件分成不同的几个文档

- ### 激活profile

​       1.在(主)配置文件中指定 spring.profiles.active=XX

​       2.命令行:

​          (cmd命令窗口) java -jar XX(文件名) --spring.profiles.active=XX

​          可以直接在测试的时候，配置传入命令行参数。

​       3.虚拟机参数：

​          -Dspring.profiles.active=dev



## 五.配置文件加载位置

- springboot 启动会扫描一下位置的application.properties或者application.yml文件作为Springboot的默认配置文件。

  -- file../config/  (项目根路径下的config文件夹下的配置文件)

  -- file../      (项目根路径下的配置文件)

  -- classpath:/config/  (resources文件夹下的config文件夹下的配置文件)

  -- classpath:/   （resources文件夹下的配置文件）

​       (注意:**以上四个文件路径的优先级从高到低依次减小**，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置内容)

​        --也可以通过配置spring.config.location来改变默认配置



## 六.springboot日志框架

springboot默认已经为我们配置好了日志，怎样使用？

下面是测试代码

```java
@SpringBootTest
class Day02ConfigApplicationTests {

    //记录器
    Logger logger  = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
        //日志的级别
        //级别由低到高 trace<debug<info<warn<error
        //可以调整输出的日志级别，可以在配置文件中调整
        logger.trace("trace");
        logger.debug("debug");

        //springboot默认输出日志级别,没有调整就使用默认级别
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }

}
```

