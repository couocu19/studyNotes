#                    SpringBoot学习笔记()

## 1.SpringBoot简介

> 简化spring应用开发的一个框架；
>
> 整个Spring技术栈的一个大整合；
>
> J2EE开发的一站式解决方案；

## 2.微服务

2.14，martin fowler

微服务：架构风格（服务微化）

一个应用应该i是一组小型服务；可以通过http的方式进行互通；

每一个功能元素最终都是一个可独立替换和升级的软件单元；

## 3.环境准备

1. maven设置
2. IDEA设置

## 4.第一个程序

1. 创建一个maven工程;(jar)

2. 导入springboot的相关依赖。

   ```xml
   <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>1.5.4.RELEASE</version>
       </parent>
       <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       </dependencies>
   ```

3. 编写第一个主程序：启动Spring

   ```java
   /**
    * @SpringBootApplication 用来标注一个主程序类,说明这是一个springboot应用
    */
   @SpringBootApplication
   public class HelloWorld {
       public static void main(String[] args) {
           //Spring应用启动起来
           SpringApplication.run(HelloWorld.class,args);
   
       }
   }
   ```

   4.编写相关的Controller以及Service

   5.运行测试

   6.简化部署

   部署配置：

   ```xml
   <!-- 这个插件,可以将应用打包成一个可执行的jar包  -->
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   ```

   

## 5.第一个程序结构分析

- ### pom文件分析

  1.parent项目

  ```xml
  <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>1.5.4.RELEASE</version>
  </parent>
  
  他的父项目是
  <parent>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-dependencies</artifactId>
  		<version>1.5.4.RELEASE</version>
  		<relativePath>../../spring-boot-dependencies</relativePath>
  </parent>
  他来真正管理Springboot应用里面的所有依赖版本；
  
  ```

  SpringBoot的版本仲裁中心；

  以后我们导入依赖默认不需要写版本；(不在dependencies中管理的依赖自然需要声明版本号)

  2.导入的依赖

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    </dependencies>
```

**spring-boot-starter-web：**

  spring-boot 场景启动器；帮我们导入了web模块正常运行所依赖的组件；

总结：

Springboot将所有的功能场景抽取出来，做成一个个的starters(启动器)，只需要在项目里面引入这些starter，相关场景到的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。



## 6.快速创建springboot项目

idea：使用Spring Initializer快速创建Spring Boot项目；

选择需要的模块:向导会联网创建springboot项目；

默认生成的项目：

- 主程序已经生成好了,我们只需要编写自己项目中的逻辑；

- resources文件夹中的目录结构

  - static：保存所有的静态资源: js,css,images;

  - templates:保存所有模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持jsp页面）；可以使用模板引擎(freemarker，thymeleaf);

  - application.properties:SpringBoot应用的配置文件；可以修改一些默认配置；

     比如默认本地服务器访问的端口号 :server.port=8081

    