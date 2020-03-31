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

