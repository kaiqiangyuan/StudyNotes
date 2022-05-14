[toc]

### 一、简介

Kafka是由[Apache软件基金会](https://baike.baidu.com/item/Apache软件基金会)开发的一个开源流处理平台，由[Scala](https://baike.baidu.com/item/Scala/2462287)和[Java](https://baike.baidu.com/item/Java/85979)编写。Kafka是一种高吞吐量的[分布式](https://baike.baidu.com/item/分布式/19276232)发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像[Hadoop](https://baike.baidu.com/item/Hadoop)一样的[日志](https://baike.baidu.com/item/日志/2769135)数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过[Hadoop](https://baike.baidu.com/item/Hadoop)的并行加载机制来统一线上和离线的消息处理，也是为了通过[集群](https://baike.baidu.com/item/集群/5486962)来提供实时的消息。

==官方文档==：https://kafka.apache.org/documentation.html#quickstart

### 二、安装并启动

http://kafka.apache.org/downloads

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531212254.png)

单机版只需要修改confi目录下的 ==server.properties== ，在相应的目录下闯将kafka的日子文件目录

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531212330.png)

启动需要先启动zookeeper，可以使用kafka自带的zookeeper或者是使用自己安装的zookeeper启动，启动完zookeeper后再kafka的根目录下使用

```shell
#启动
bin/kafka-server-start.sh config/server.properties
```

出现下面两个则说明启动成功

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531212412.png)

### 三、发送消息

参考：

https://www.pianshen.com/article/9102902970/

https://blog.csdn.net/yitengtongweishi/article/details/82385663

```shell
创建主题  test
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
查看创建的主题
bin/kafka-topics.sh --list --zookeeper localhost:2181
启动生产者 发送message
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
启动消费者 消费message（建议另开一个ssh窗口，这样可以一边通过生产者命令行输入消息，一边观察消费者消费的数据）
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

![222](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602105931.png)

配置远程连接时需要在 server.properties 中增加

```shell
advertised.host.name = 外网的ip
```

### 四、Java测试发送消息

位移主题__consumer_offsets 主题参考：

https://blog.csdn.net/qq_43277087/article/details/111475523

https://blog.csdn.net/weixin_37150792/article/details/89851731

使用**kafka消费者组**的目的：https://blog.csdn.net/john1337/article/details/103060820

java操作主题发送消息参考：https://www.cnblogs.com/wuyongyin/p/12785095.html



java笔记代码：https://www.yuque.com/yuankaiqiang/file/19664959
