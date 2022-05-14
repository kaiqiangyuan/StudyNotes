[toc]

为什么要使用消息队列?

https://www.cnblogs.com/xiapu5150/p/9927323.html

https://www.cnblogs.com/yanfei1819/p/10615605.html

https://blog.csdn.net/qq_43652509/article/details/83926758

使用消息队列主要应用于三个场景：==**解耦、异步、削峰**==

==**使用场景**==

1. 用户注册，重点使用用户信息数据库保存，而发短信、发邮件，增加业务处理复杂度，这时候使用MQ， 将发短信、发邮箱，通知MQ，由另外服务平台完成。解决了代码的耦合问题
2.  当系统使用搜索平台、缓存平台的时候
3. 订单

查询数据，建立缓存、索引 ，当再次查询相同数据的时候，不从数据库查询，从缓存或者索引库查询

当增加、修改、删除数据时，发送消息给MQ， 缓存平台、索引平台 从MQ获取到这个信息，更新缓存或者索引

### 一、安装ActiveMQ

下载：https://activemq.apache.org/activemq-5014000-release

1. 在 **bin** 目录下使用 **./activemq start** 启动项目

2. 查看 8161(web管理页面端口）、61616（activemq服务监控端口） 两个端口是否存在

3. 登录web，初始账号密码都为 admin 或 user

4. 修改 **config** 目录下的 vim jetty-realm.properties （admin: admin, admin依次为：用户名：密码，角色）

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531211123.png)

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531211138.png)

- **Name** 表示队列名称，可在应用程序中自动创建，也可在 ActiveMQ 控制台中手动创建
- **Number Of Pending Messages** 表示阻塞在队列中未经消费的消息条数
- **Number Of Consumers** 表示正在与 ActiveMQ 建立连接的消费者数量
- **Messages Enqueued** 表示进入队列的消息数量
- **Messages Dequeued** 表示离开队列的消息数量

### 二、SpringBoot中使用

**maven**

```shell
 	<dependency>
  		<groupId>org.apache.activemq</groupId>
  		<artifactId>activemq-all</artifactId>
  		<version>5.14.0</version>
  </dependency>
```

参考：

https://blog.csdn.net/fanxing1964/article/details/81913681/

https://www.cnblogs.com/HigginCui/p/8465760.html

> 消息确认机制

消息只有被确认之后，才认为是被成功消费，然后消息才会从队列或主题中删除。

客户端接收消息 = 》 消息处理 =》 消息确认

==四种确认机制：==

* Session.AUTO_ACKNOWLEDGE；客户（消费者）成功从receive方法返回时，或者从MessageListener.onMessage方法成功返回时，会话自动确认消息,然后自动删除消息.

* Session.CLIENT_ACKNOWLEDGE；客户通过显式调用消息的acknowledge方法确认消息,。 即在接收端调用message.acknowledge();方法,否则,消息是不会被删除的.

* Session. DUPS_OK_ACKNOWLEDGE ；不是必须确认，是一种“懒散的”消息确认，消息可能会重复发送，在第二次重新传送消息时，消息头的JMSRedelivered会被置为true标识当前消息已经传送过一次，客户端需要进行消息的重复处理控制。

* Session.SESSION_TRANSACTED；事务提交并确认

> 丢消息怎么办？

可以设置持久化消息，messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);使用acknowledge（）方法，客户端返回确认信息。

>  事务消息

创建回话Session使用transacted = true，事务消息在发送和接收后必须显示的调用session.commit(),事务消息会自动确认。与设置的确认机制无关

> 消息接收方式

receive方法接收，一次即结束；使用监听器接收，实现**onMessage**方法，作为一个服务实时监测。

> 定时清理queue或者是topic

参考：https://blog.csdn.net/weixin_30847865/article/details/98047557?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-10.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-10.control

设置 ==conf/activemq.xml==

增加 `schedulePeriodForDestinationPurge="3600000"`

增加`<policyEntry queue=">" gcInactiveDestinations="true" inactiveTimoutBeforeGC="30000"/>`

schedulePeriodForDestinationPurge = 3600000，表示每一小时检查一次，默认为 0，此功能关闭

gcInactiveDestinations，true 表示删除回收闲置的队列，默认为 false

inactiveTimoutBeforeGC = 600000，表示当队列或主题闲置 10 分钟后被删除，默认为 60 秒。

```shell
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" 
        schedulePeriodForDestinationPurge="3600000">
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry topic=">">
                    <pendingMessageLimitStrategy>
                        <constantPendingMessageLimitStrategy limit="1000"/>
                    </pendingMessageLimitStrategy>
                </policyEntry>
                <policyEntry queue=">" gcInactiveDestinations="true" inactiveTimoutBeforeGC="30000"/>
            </policyEntries>
        </policyMap>
    </destinationPolicy>    
</broker>
```

每个队列中的数据一般是无限制的，如何想要设置队列中的数据大小

可以设置：==producerFlowControl="true" memoryLimit="20mb"==

参考：http://www.voidcn.com/article/p-nfkwwqva-bwp.html

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531211906.png)

### 3、如何存储数据

> ActiveMQ 是怎么存储消息的？

理解 ActiveMQ 存储消息的存储机制的基本知识是非常重要的。队列和主题中的消息存储是不同的，因为有些可以在主题上优化的地方并不适合队列。
队列存储消息是非常直接的——即最基本的先进先出（FIFO）。
主题存储消息是有点复杂的，它为每一个消费者维持一个指向消息队列的指针。

> KahaDB 消息存储

自从 5.3 版本之后，ActiveMQ 推荐的消息存储一般是使用 KahaDB。它是一种基于文件形式的消息存储，集合了便于恢复的事务日志和良好的性能以及可伸缩性。

> AMQ 消息存储

跟 KahaDB 差不多的东西吧，两者有啥区别？反正我是没看明白！

> JDBC 消息存储

就是利用关系型数据库存储消息啦！感觉有点怪怪的。。。

> 内存消息存储

这个就不是持久化了，就是单纯的把消息保存在内存中，配置简单，使用方便！







