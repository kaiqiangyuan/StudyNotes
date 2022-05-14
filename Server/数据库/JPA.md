> 事务范围（单Session即一级缓存） 

​		事务范围的缓存**只能被当前事务访问**，**每个事务都有各自的缓存**，缓存内的数据通常采用相互关联的对象形式。**缓存的生命周期依赖于事务的生命周期**，只有当事务结束时，缓存的生命周期才会结束。**事务范围的缓存使用内存作为存储介质**，一级缓存就属于事务范围。

　　1、使用一级缓存的目的是为了**减少对数据库的访问次数**，从而提升hibernate的执行效率；（当执行一次查询操作的时候，执行第二次查询操作，先检查缓存中是否有数据，如果有数据就不查询数据库，直接从缓存中获取数据）；

　　 2、Hibernate中的一级缓存，也叫做session的缓存，它可以在session范围内减少数据库的访问次数，**只在session范围内有效，session关闭，一级缓存失败；**

　　3、一级缓存的特点，只在session范围有效，作用时间短，效果不是特别明显，在**短时间内多次操作数据库，效果比较明显**。

　　4、当调用session的**save/saveOrUpdate/get/load/list/iterator方法的时候，都会把对象放入session缓存中；**

　　5、session的缓存是由hibernate维护的，用户不能操作缓存内容；如果想操作缓存内容，必须通过hibernate提供的evict/clear方法操作

　　6、缓存相关的方法（在什么情况下使用上面方法呢？批量操作情况下使用，如Session.flush();先与数据库同步，Session.clear();再清空一级缓存内容）：

　　　　session.flush();让一级缓存与数据库同步；

　　　　session.evict();清空一级缓存中指定的对象；

　　　　session.clear();清空一级缓存中所有的对象；

> 应用范围（单SessionFactory即二级缓存） 

​		应用程序的缓存**可以被应用范围内的所有事务共享访问**。**缓存的生命周期依赖于应用的生命周期**，只有当应用结束时，缓存的生命周期才会结束。**应用范围的缓存可以使用内存或硬盘作为存储介质**，二级缓存就属于应用范围。

  1、在执行各种条件查询时，如果所获得的结果集为实体对象的集合，那么就会把所有的数据对象根据ID放入到二级缓存中。 

   2、当Hibernate根据ID访问数据对象的时候，首先会从Session一级缓存中查找，如果查不到并且配置了二级缓存，那么会从二级缓存中查找，如果还查不到，就会查询数据库，把结果按照ID放入到缓存中。 

   3、删除、更新、增加数据的时候，同时更新缓存。

**注意：**

 在通常情况下会将具有以下特征的数据放入到二级缓存中： 

   ● 很少被修改的数据。 

   ● 不是很重要的数据，允许出现偶尔并发的数据。 

   ● 不会被并发访问的数据。 

   ● 常量数据。 

   ● 不会被第三方修改的数据 

 而对于具有以下特征的数据则不适合放在二级缓存中： 

  ● 经常被修改的数据。 

  ● 财务数据，绝对不允许出现并发。 

  ● 与其他应用共享的数据。 

![20191012192942748](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210704193847.png)

![image-20210704193519142](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210704193519.png)

| **Keyword**       | **Sample**                                              | **JPQL snippet**                                             |
| ----------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| And               | findByLastnameAndFirstname                              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname                               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstname,findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                                  | … where x.age <= ?1                                          |
| GreaterThan       | findByAgeGreaterThan                                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual                               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                                  | … where x.firstname not like ?1                              |
| StartingWith      | findByFirstnameStartingWith                             | … where x.firstname like ?1(parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith                               | … where x.firstname like ?1(parameter bound with prepended %) |
| Containing        | findByFirstnameContaining                               | … where x.firstname like ?1(parameter bound wrapped in %)    |
| OrderBy           | findByAgeOrderByLastnameDesc                            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection<Age> ages)                       | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection<Age> ages)                    | … where x.age not in ?1                                      |
| True              | findByActiveTrue()                                      | … where x.active = true                                      |
| False             | findByActiveFalse()                                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase                               | … where UPPER(x.firstame) = UPPER(?1)                        |

In和NotIn可以使用任何Collection的子类作为参数。
