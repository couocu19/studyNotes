#                   Mybatis原理笔记(1)

## Mybatis的初始化

- ### Mybatis的配置信息/高层级结构

  **（Configuration）**

  - properties 属性

  - settings  设置

  - type Alias  类型命名

  - type Handler 类型处理器

  - Object Factory 对象工厂

  - environments 环境

    1. environment 环境变量

    2. transactionManager 事务管理器

    3. dataSource 数据源

       (1包含2和3)

  - 映射器

  **说明:Mybatis的初始化过程也就是创建Configuration对象的过程。**

- ### Mybatis初始化的基本过程

  SqlSessionFactoryBuilder根据传入的数据生成Configuration对象，根据Configuration对象生成默认的SqlSessionFactory对象。

- ### 详细过程