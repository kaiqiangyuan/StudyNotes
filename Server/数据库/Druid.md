[toc]

### 1、简介

​		数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个；释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏。这项技术能明显提高对数据库操作的性能。==Druid==是目前最好的数据库连接池，在功能、性能、扩展性方面，都超过其他数据库连接池，包括DBCP、C3P0、BoneCP、Proxool、JBoss DataSource。

### 2、常用数据库连接池对此

![image-20210703100746886](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703100747.png)

|      功能       |        dbcp         |       druid        |                c3p0                |    tomcat-jdbc    |              HikariCP              |
| :-------------: | :-----------------: | :----------------: | :--------------------------------: | :---------------: | :--------------------------------: |
| 是否支持PSCache |         是          |         是         |                 是                 |        否         |                 否                 |
|      监控       |         jmx         |    jmx/log/http    |              jmx,log               |        jmx        |                jmx                 |
|     扩展性      |         弱          |         好         |                 弱                 |        弱         |                 弱                 |
|  sql拦截及解析  |         无          |        支持        |                 无                 |        无         |                 无                 |
|      代码       |        简单         |        中等        |                复杂                |       简单        |                简单                |
|    更新时间     |      2015.8.6       |     2015.10.10     |             2015.12.09             |                   |             2015.12.3              |
|      特点       |  依赖于common-pool  | 阿里开源，功能全面 | 历史久远，代码逻辑复杂，且不易维护 |                   | 优化力度大，功能简单，起源于boneCP |
|   连接池管理    | LinkedBlockingDeque |        数组        |                                    | FairBlockingQueue |  threadlocal+CopyOnWriteArrayList  |

### 3、SpringBoot中使用（含测试代码）

[*监控页面[^1]*](https://blog.csdn.net/u014209205/article/details/80625963)

[*主要参考[^2]*](https://blog.csdn.net/pengjunlee/article/details/80061797?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-18.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-18.control)

[*语雀代码路径[^3]*](https://www.yuque.com/yuankaiqiang/file/19964294)

浏览器打开：http://127.0.0.1:8080/druid/login.html，用户名及密码为代码中设置的密码

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703164232.png" alt="image-20210703164232261" style="zoom:50%;" />

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703164256.png" alt="image-20210703164256753" style="zoom:50%;" />

运行程序时执行创建表的SQL语句，当插入和查询时，也会出现对应的SQL语句的执行情况

![image-20210703164505569](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703164505.png)

同时也可以看到接口的调用情况信息，这里查询所有数据调用了上次，因为上面的查询SQL语句也相应的执行了三次

![image-20210703164655295](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703164655.png)

### 参考网址

[^1]:https://blog.csdn.net/u014209205/article/details/80625963
[^2]:https://blog.csdn.net/pengjunlee/article/details/80061797?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-18.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-18.control
[^3]:https://www.yuque.com/yuankaiqiang/file/19964294



















