#                        spring原理学习笔记(1)

## 关于spring容器

spring容器是Spring的核心，该容器负责管理spring中的java组件。

实例化方式:

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("bean.xml");
Person person = context.getBean("person1", Person.class);
```

```json
ApplicationContext正是Spring容器。
ApplicationContext会默认在容器创建时实例化所有singleton Bean。
Spring容器并不强制要求被管理组件是标准的javabean。
```

## Spring的核心机制:依赖注入/控制反转

### 什么是控制反转？

   Inversion Of Controller 即控制反转。

把创建对象的权利反转交给spring，这一过程叫做控制反转。

在spring中加载对象，就是通过反射来加载。

### 什么是依赖注入？

说明:不管是依赖注入(Dependency Injection)还是控制反转(Inversion of Conctrol)，二者的含义相同。

正常情况下，一个实例(调用者)想要调用另一个实例(被调用者)时，调用者都需要new一个被调用者实例，然后进行调用。

但是依赖注入的模式下创建被调用实例的工作不由调用者来做，因此成为“控制反转”，而是交给Spring来完成，然后再将创建好的被调用实例注入调用者实例中，所以也称为“依赖注入”。

### 依赖注入的目的是什么？

一般而言，在分层体系结构中，都是上层调用下层接口，上层依赖于下层的执行，即调用者依赖于被调用者。

但是通过IoC方式，使得上层不再依赖于下层的使用，二而是通过一定的机制来选择不同的下层实现，完成控制反转，使得调用者可以决定被调用者。

IoC通过注入一个实例化的对象来达到解耦的目的。

使用这种方法后，对象不会被显示的调用，而是根据需求通过IOC容器（例如spring）来提供。

## 依赖注入的主要两种方式

- ### 设置注入

​     **Ioc容器使用属性的setter方法来注入，其中可以简单的分为两类：**

​     1.基本类型值注入。-->标签：properties

​     2.引入类型值注入。 -->标签:ref （需要引入别的javabean作为属性时可以使用）

- ### 构造注入

  Ioc容器使用构造器来注入被依赖的实例。

  说明：单个有参构造方法注入时，会涉及到两个属性。

  1. index属性:当构造器中的属性和配置文件中的属性顺序不对应时，可以用index来标注构造器中属性的顺序来完成对应。
  2. type属性:当构造器中的属性和配置文件中的属性不对应时，可以用type标注属性对应的java数据类型来完成对应。

## 依赖注入的优缺点？

- ### 优点

  1.通过IOC容器，开发人员不需要关注对象时何时创建的，同时，增加新类型也比较方便，只需要修改配置文件即可实现对象的热插拔。

  2.IOC容器可以通过配置文件来确定需要注入的实例化对象，因此，非常便于进行单元测试。

- ### 缺点

  1.对象是通过反射机制实例化出来的，因此，会对系统的性能有一定的影响。

  2.创建对象的流程变得比较复杂。

## Spring容器和被管理的bean

Spring的两个核心接口：**BeanFactory和ApplicationContext，其中ApplicationContext是BeanFactory的子接口。他们都可以代表Spring容器。**

Spring容器是生成Bean实例的工厂，并管理Spring中的bean，bean是Spring中的基本单位，在基于Spring的javaEE工程中，所有的组件都被当做bean来处理。

包括数据源，mybatis的sessionFactory，事务管理器。

### BeanFactory和ApplicationContext有什么区别？

   答：BeanFactory是Spring框架的基础设置，面向Spring本身。是Spring中最原始的Factory，它的特点是只有在获取到对象时（在容器中拿bean时）才会实例化对象，功能比较单一，不能支持Aop等高级功能，只提供了容器的最简单的功能。

​           ApplicationContext是由BeanFactory派生而来的Factory。

应用上下文，继承BeanFactory接口，它是Spring的一各更高级的容器，提供了更多的有用的功能；

   1) 国际化（MessageSource）

   2) 访问资源，如URL和文件（ResourceLoader）

   3) 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  

   4) 消息发送、响应机制（ApplicationEventPublisher）

   5) AOP（拦截器）

特点是：每次容器创建时就会实例化容器中的所有对象， 它还可以为Bean配置lazy-init=true来让Bean延迟实例化，并拥有更多的功能。

（从类路径下加载配置文件ClassPathXmlApplicationContext
    从硬盘绝对路径下加载配置文件 FileSystemXmlApplicationContext）

#### 我们该用BeanFactory还是ApplicationContent？

**延迟实例化的优点：（BeanFactory）**

应用启动的时候占用资源很少；对资源要求较高的应用，比较有优势； 

**不延迟实例化的优点： （ApplicationContext）**

1. 所有的Bean在启动的时候都加载，系统运行的速度快； 

2. 在启动的时候所有的Bean都加载了，我们就能在系统启动的时候，尽早的发现系统中的配置问题 

3. 建议web应用，在启动的时候就把所有的Bean都加载了。（把费时的操作放到系统启动中完成） 

**结论：web开发过程中使用ApplicationContext，资源匮乏时使用BeanFactory**

- ### Spring容器 

  - Spring Core --->spring框架的核心容器。

    它提供了Spring框架的基本功能。这个模块中一个重要的组件为BeanFactory，他使用工厂模式来创建所需的对象。同时它使用了Spring中Ioc的思想，通过读取XML文件的方式来实例化对象，可以说BeanFactory提供了组建生命周期的管理，组建的创建，装配以及销毁功能。

  - Spring Context

    扩展核心容器，提供了Spring上下文环境，都给开发人员提供了很多非常有用的服务，例如国际化，Emali和JDNI访问等。

- ### ApplicaionContext的事件机制

- ### 容器中bean的作用域

- ### 获取容器的引用

  

  ### 

  

  

  

  

  

  

  

  

  