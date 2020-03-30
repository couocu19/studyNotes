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
ApplicationContext会默认实例化所有singleton Bean。
Spring容器并不强制要求被管理组件是标准的javabean。
```

## Spring的核心机制:依赖注入/控制反转

说明:不管是依赖注入(Dependency Injection)还是控制反转(Inversion of Conctrol)，二者的含义相同。

正常情况下，一个实例(调用者)想要调用另一个实例(被调用者)时，调用者都需要new一个被调用者实例，然后进行调用。

但是依赖注入的模式下创建被调用实例的工作不由调用者来做，因此成为“控制反转”，而是交给Spring来完成，然后再将创建好的被调用实例注入调用者实例中，所以也称为“依赖注入”。

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

## Spring容器和被管理的bean

Spring的两个核心接口：**BeanFactory和ApplicationContext，其中ApplicationContext是BeanFactory的子接口。他们都可以代表Spring容器。**

Spring容器是生成Bean实例的工厂，并管理Spring中的bean，bean是Spring中的基本单位，在基于Spring的javaEE工程中，所有的组件都被当做bean来处理。

包括数据源，mybatis的sessionFactory，事务管理器。

- ### Spring容器 

- ### ApplicaionContext的事件机制

- ### 容器中bean的作用域

- ### 获取容器的引用

  

  ### 

  

  

  

  

  

  

  

  

  