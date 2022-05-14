[toc]

### 一、简介

ZooKeeper是一个[分布式](https://baike.baidu.com/item/分布式/19276232)的，开放源码的[分布式应用程序](https://baike.baidu.com/item/分布式应用程序/9854429)协调服务，是[Google](https://baike.baidu.com/item/Google)的Chubby一个[开源](https://baike.baidu.com/item/开源/246339)的实现，是Hadoop和[Hbase](https://baike.baidu.com/item/Hbase/7670213)的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

> 使用场景

[*Zookeeper常见使用场景*](https://blog.csdn.net/hc1329964732/article/details/114404945) [^3]

### 二、安装

[*Zookeeper集群搭建（基于外网ip的云服务器搭建）*](https://blog.csdn.net/qq_41051923/article/details/108567148?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-1&spm=1001.2101.3001.4242) [^1]

[*菜鸟教程*](https://www.runoob.com/w3cnote/zookeeper-linux-cluster.html) [^2]

服务器上搭建集群时，需要开放端口，且在==zoo.conf==加上`quorumListenOnAllIPs=true`

1. 安装包地址[^4]

2. 配置环境变量

   ```shell
   #set zookeeper environment
   export ZK_HOME=/root/zookeeper-3.4.14
   export PATH=$PATH:$ZK_HOME/bin
   source /etc/profile
   ```

3. 配置文件将在根目录下的conf文件夹下，需要先将其重命名为zoo.cfg

   ```shell
   cd zookeeper-3.4.14/conf   
   cp zoo_sample.cfg zoo.cfg
   ```

4. 修改配置文件 **zoo.cfg**，在安装目录下创建data和log文件夹

   ```shell
   dataDir=/home/environment/zookeeper-3.6.2/data
   dataLogDir=/home/environment/zookeeper-3.6.2/log
   #添加集群配置
   server.1=114.55.26.230:2888:3888
   #2181 : 对 client 端提供服务
   #2888 : 集群内机器通信使用
   #3888 : 选举 leader 使用
   # 修改默认的8080端口
   admin.serverPort=8081
   #如果出现配置集群时,集群启动失败,配置的阿里云外网地址,显示地址无法请求问题
   #因为配置的结点不是在同一内网中，设置的是公网ip。所以需要在配置文件中添加设置
   quorumListenOnAllIPs=true
   ```

5. 在data文件夹下**创建myid文件**，touch myid

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601133230.png)

   myid的内容与zoo.cfg的内容ip一致

6. 启动节点

   ```shell
   启动命令=》zkServer.sh start
   停止命令=》zkServer.sh stop
   重启命令=》zkServer.sh restart
   查看集群节点状态=》zkServer.sh status
   ```

### 三、使用

```shell
查看启动日志./zkServer.sh start-foreground

get /runoob 获取数据点数据
set path data 命令用于修改节点存储的数据(设置一次后dataVersion版本加1)
delete path 命令用于删除某节点(只能一层一层删除)
stat /runoob  查看节点状态信息
create [-s] [-e] path data acl 命令用于创建节点并赋值  (-s 代表顺序节点， -e 代表临时节点)创建节点时，必须要带上全路径
```

四字命令

```shell
#开启四字命令（修改config的zoo.cfg文件）
4lw.commands.whitelist=*

stat 命令用于查看 zk 的状态信息
    echo stat | nc 127.0.0.1 2181
ruok 命令用于查看当前 zkserver 是否启动，若返回 imok 表示正常
    echo ruok | nc 127.0.0.1 2181
dump 命令用于列出未经处理的会话和临时节点
    echo dump | nc 127.0.0.1 2181
conf 命令用于查看服务器配置
    echo conf | nc 192.168.3.38 2181
cons 命令用于展示连接到服务器的客户端信息
    echo cons | nc 127.0.0.1 2181
envi 命令用于查看环境变量
    echo envi | nc 127.0.0.1 2181
```

[permissions]：有五种类型（==create、read、write、delete、admin==），含义如下：

==create==：创建节点的权限，简写为c；

==read==：读节点的权限，简写为r；

==write==：写节点的权限，简写为w；

==delete==：删除节点的权限，简写为d；

==admin==：为节点分配权限的权限，简写为a。

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601133454.png)

==auth== 用于授予权限

```shell
setAcl /runoob/child auth:user1:123456:cdrwa
addauth digest user1:123456
setAcl /runoob/child auth:user1:123456:cdrwa
getAcl /runoob/child
```

==digest== 退出当前用户，重新连接终端，digest 可用于账号密码登录和验证

```shell
ls /runoob
create /runoob/child01 runoob
getAcl /runoob/child01
setAcl /runoob/child01 digest:user1:HYGa7IZRm2PUBFiFFu8xY2pPP/s=:cdra
getAcl /runoob/child01
addauth digest user1:123456
getAcl /runoob/child01
```

==ip== 限制 IP 地址的访问权限，把权限设置给 IP 地址为 192.168.3.7 后，IP 为 192.168.3.38 已经没有访问权限

```shell
create /runoob/ip 0
getAcl /runoob/ip
setAcl /runoob/ip ip:192.168.3.7:cdrwa
get /runoob/ip
```

### 四、常用使用场景

#### 1、分布式锁

对某一个数据连续发出两个修改操作，两台机器同时收到了请求，但是只能一台机器先执行完另外一个机器再执行。那么此时就可以使用 zookeeper 分布式锁，一个机器接收到了请求之后先获取 zookeeper 上的一把分布式锁，就是可以去创建一个 znode，接着执行操作；然后另外一个机器也尝试去创建那个 znode，结果发现自己创建不了，因为被别人创建了，那只能等着，等第一个机器执行完了自己再执行。

![image-20210712201128884](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210712201128.png)

[*分布式锁参考*](https://www.jianshu.com/p/38b648df2448) [^5]

![20210712153726](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210712153747.jpg)

分布式锁代码-阿里云盘地址：**关注公众号==原棵哈==回复==zookeeper==获取**

![扫码_搜索联合传播样式-标准色版](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210720080705.png)

#### 2、分布式协调

这个其实是 zookeeper 很经典的一个用法，简单来说，就好比，你 A 系统发送个请求到 mq，然后 B 系统消息消费之后处理了。那 A 系统如何知道 B 系统的处理结果？用 zookeeper 就可以实现分布式系统之间的协调工作。A 系统发送请求之后可以在 zookeeper 上对某个节点的值注册个监听器，一旦 B 系统处理完了就修改 zookeeper 那个节点的值，A 系统立马就可以收到通知，完美解决。

![image-20210712201011608](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210712201011.png)

#### 3、元数据/配置信息管理

![image-20210712201245100](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210712201245.png)

#### 4、HA 高可用性

hadoop、hdfs、yarn 等很多大数据系统，都选择基于 zookeeper 来开发 HA 高可用机制，就是一个**重要进程一般会做主备**两个，主进程挂了立马通过 zookeeper 感知到切换到备用进程。

![image-20210712201332221](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210712201332.png)

### 参考网址

[^1]: https://blog.csdn.net/qq_41051923/article/details/108567148?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-1&spm=1001.2101.3001.4242
[^2]: https://www.runoob.com/w3cnote/zookeeper-linux-cluster.html
[^3]:https://blog.csdn.net/hc1329964732/article/details/114404945
[^4]: https://downloads.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz
[^5]: https://www.jianshu.com/p/38b648df2448
[^6]: https://www.aliyundrive.com/s/hTsDosQMuJP

