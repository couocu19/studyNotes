# Spring-Security框架

简介：是强大的，容易定制的实现认证，与授权的基于Spring开发的框架。

## 核心功能

![image-20200906093312568](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906093312568.png)

## 项目环境准备

基于springboot

```
1.login.html登录页面。
2.在登录页面登录之后，进入index.html首页（登录验证Authation）
3.首页看到syslog，sysuser，业务1，业务2四个页面选项。
4.我们希望syslog（日志管理）和sysuser（用户管理）只有admin管理员可以访问（权限管理Authorzation），当然管理员可以看到所有的业务。
5.biz1，biz2普通的用户操作auser就可以访问（权限管理Authorzation）。
```

```java
Using generated security password: c9c8d211-4f12-489e-9f10-2a25fc425974
```

![image-20200906102953228](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906102953228.png)



```java
@Configuration
public class SecurityConfig  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        //规则：所有请求必须都在登录之后访问
        //配置了之后，在访问login.html页面时，会弹出一个授权认证的框，需要输入提供的用户名和密码才可以进入到项目的登录中。
        //这个认证的框并不是自己开发的，而是spring-security提供的
        //
        http.httpBasic().and().authorizeRequests().anyRequest().authenticated();
    }
}
```

上述配置的要求为：

   项目启动之后必须首先完成登录认证才可以进行业务中的访问。是一种比较简陋的登录认证方式。

   这个功能的作用：

​         “防君子不防小人”的作用。

​          虽然这个认证方式比较简陋,但是也有一定的应用场景。比如当在一个公司级的项目的某一个部门作为开发人员实现了某一套接口，作为技术人员想要对这套接口进行暂时的保密工作，可以使用httpbasic进行一个登录认证的设置。

​        这个认证是可以被破解的。即登录认证的密码是可以被破解的。



### httpbasic认证的工作过程

​    客户端将password传到服务端，服务端再将收到的password回传给客户端，完成认证。

​    客户端怎样传给服务端？

​    在http请求头中会有一个Authorization的请求头，内容包含要传给服务端的key，其中这个key是通过base64来加密的。

​     解密之后就是我们针对认证所匹配的用户名和密码。



### form 表单登录认证模式

![image-20200906171752108](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906171752108.png)



#### 三要素：

- 登录认证逻辑（静态）
- 资源访问控制（动态）
- 用户角色权限（动态）

![image-20200906171449612](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906171449612.png)

![image-20200906171541171](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906171541171.png)

![image-20200906171642005](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906171642005.png)

### 表单授权登录的源码解析：

![image-20200906185553750](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906185553750.png)

在使用httpBasic登录认证时，需要经过两个过滤器：

​     BasicAuthenticFilter和UsernamePassword。。。。

​     在一个待验证身份的用户尝试通过验证时，首先会生成一个**登陆认证的主体（authentication，贯穿于登录认证始终）**，会像用户闯关一样的机制，会经过一系列的“filter”，在通过第二个栅的这些过滤器时，需要将自己的authentic对象进行填充，验证是哪一个过滤器， 然后会继续通过之后的过滤器，当通过了一系列的过滤器之后，才可以获得登录的api，如果没有认证通过，会通过处理异常的filter来处理。

​     在整个filter的流程走完之后，会将登陆认证对象填充到security context的session中，当相同的登录对象再次登录时不需要再次经过之后的过滤器。

​     

### 从数据库中动态加载用户信息实现授权登录

需要实现的接口：

​     UserDetails：（用户的信息）

​           password username 是否过期……

​     通过 用户名去数据库去加载用户信息



![image-20200906201116922](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200906201116922.png)







