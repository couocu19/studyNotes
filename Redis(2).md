# Redis(2)

- ## String数据结构的字符串，整数，浮点操作（入门级）

  #### 例子：把java对象转为json，然后作为字符串存储
  
  ```json
  {"id":1,"username":"cocou","pass":"1234"}
  ```
  
  - ### 字符串操作
  
  #### set命令：
  
  ```
  
  127.0.0.1:6379> set user1 '{"id":1,"username":"cocou","pass":"1234"}'
  OK
  127.0.0.1:6379> get user1
  "{\"id\":1,\"username\":\"cocou\",\"pass\":\"1234\"}"
  
  ```
  
  ###### NX：表示key不存在才设置，如果存在则返回nil(失败)
  
  ###### XX：表示key存在时才设置，如果不存在返回nil(失败)
  
  ###### EX seconds：设置过期时间，过期时间精确为秒。 //采用ttl查看剩余时间
  
  ###### PX millsecond：设置过期时间，过期时间精确为毫秒 
  
  ###### （一旦超过过期时间就会把这个key删除）
  
  #### SETNX命令：
  
  所有参数为必选参数，设置一对key value。
  
  如果key存在则设置失败，等同于 set key value NX
  
  #### SETEX （和set中的ex命令等价）
  
  #### SETPX（和set中的px命令等价）

####      mset命令   :

       ```
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
       ```

####     GETSET

作用：先查出key value的值，然后再修改新的值

语法：GETSET key value

所有参数为必选参数，获取指定key的value，并设置key的值为value

```

127.0.0.1:6379> set user1 coucou
OK
127.0.0.1:6379> getset user1 coucou1
"coucou"
127.0.0.1:6379> get user1
"coucou1"
```

####    SETRANGE

作用：修改某个key对应的value的偏移量

```
127.0.0.1:6379> set user3 123456
OK
127.0.0.1:6379> setrange user3 3 coucou
(integer) 9
127.0.0.1:6379> get user3
"123coucou"
127.0.0.1:6379>
```

####    GETRANGE

作用：截取字符串

```
127.0.0.1:6379> getrange user3 2 5
"3cou"
```

####  APPEND

```
127.0.0.1:6379>append user3 abc
(integer) 12
127.0.0.1:6379> get user3
"123coucouabc"

```

#### SUBSTR

作用：字符串的截取

语法：

````
127.0.0.1:6379> substr user3 3 6
"couc"
````

- ### 数字操作

  - #### incr

    ```
    incr pro01
    (integer) 1
    127.0.0.1:6379> get pro01
    "1"
    127.0.0.1:6379> incr pro01
    (integer) 2
    127.0.0.1:6379>
    ```

  - #### decr

    (和incr同理)

  - #### incrby

    ```
    incrby pro01 50
    (integer) 150
    127.0.0.1:6379> get pro01
    "150"
    127.0.0.1:6379> decrby pro01 60
    (integer) 90
    ```

  - #### decrby

  - #### incrbyfloat key num

    作用：在原有的key上加上浮点数

    ```
    set money 10.5
    OK
    127.0.0.1:6379> incrbyfloat money 2.2
    "12.7"
    ```

    

