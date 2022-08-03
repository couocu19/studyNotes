# SQL相关问题总结

- 添加索引有什么缺点？
- 数据库拆分的两种方式？分别有什么优缺点？
- sql优化的5中方式？
- InnoDB和MyISAM存储引擎的比较

```
1、InnoDB支持事务，MyISAM不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了。
2、MyISAM适合查询以及插入为主的应用，InnoDB适合频繁修改以及涉及到安全性较高的应用
3、InnoDB支持外键，MyISAM不支持
4、MyISAM是默认引擎，InnoDB需要指定
5、InnoDB不支持FULLTEXT类型的索引
6、InnoDB中不保存表的行数，如select count(*) from table时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含where条件时MyISAM也需要扫描整个表
7、对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引
8、清空整个表时，InnoDB是一行一行的删除，效率非常慢。MyISAM则会重建表
9、InnoDB支持行锁（某些情况下还是锁整表，如 update table set a=1 where user like '%lee%'

二：存储引擎：InnoDB和MyISAM的比较
MyISAM存储引擎：
优点：查询数据相对较快，适合大量的select，可以全文索引。
缺点：不支持事务，不支持外键，并发量较小，不适合大量update

InnoDB存储引擎：
优点：支持事务，支持外键，并发量较大，适合大量update
缺点：查询数据相对较慢，不适合大量的select【影响速度的主要原因是AUTOCOMMIT默认设置是打开的】
```

- 模糊查询什么时候会导致索引失效?

```
    使用左或左右模糊查询的时候，也就是like "%张"或like "%张%"这两种模糊查询方式都会导致索引失效
因为B+树是根据索引值进行排列的，前缀不确定的时候可能是，“小张”，"二张"之类的所有的情况，就只能通过全表扫描的方式来查询
```

- 哪些情况会使索引失效?

- sql面试题

  ```
  1. 现在一张打卡记录表(假设为table1)，字段有id，name，time（时间戳）；现在要写一条sql，查询每个月每个员工有效打卡的天数。
  (假设查询8月份每个员工有效打卡的天数)
  select id ,name,datediff('2022-07-31-0-0',time) as days_count
  from table1 
  where  datediff(time,'2022-07-31-0-0') < 30
  group by id
  order by time desc
  
  2.日志合并：现有一条用户操作记录表，描述用户购物的流程，包含uid，content（指当前记录的操作是什么：收藏、加购、购买）,time；
    现在需要将一分钟之内进行的连续行为日志合并成会话级别日志。提示：使用窗口函数，string_agg.
    select uid,STRING_AGG(content,',') as contents from table2 
    group by uid
    where date_sub(now(),interval 1 minute) < time;
   
  ```

  