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





