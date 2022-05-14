[亿级用户的分布式数据存储解决方案](https://www.pianshen.com/article/51641527056/) [^3]



[toc]

### 1、简介

​		MySQL Cluster(MySQL集群)是一个高性能、可扩展、集群化数据库产品，其研发设计的初衷就是要满足许多行业里的最严酷应用要求。这些应用中经常要求数据库运行的可靠性要达到99.999%。
　　自从2004年开始MySQL Cluster发布以来，其新特性的变化就不断的被更新增强。这增加了MySQL Cluster在新的应用领域、市场、行业中的需求量。MySQL Cluster目前已经不仅仅应用于传统的传统的电信业务中，如HLR(Home Locator Registry)或 SLR( Subscriber Locator Registry)，它还被广泛的应用在VOIP、网络计费、会议管理、电子商务网站、搜索引擎，甚至是传统的后台应用中。

c?o;kd&5YLDT

vTw8&01niyek

ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

alter user user() identified by "123456789";

==特点：==

- 高可用性：主服务器故障后可自动切换到后备服务器
- 可伸缩性：可方便通过脚本增加DB服务器
- 负载均衡：支持手动把某公司的数据请求切换到另外的服务器，可配置哪些公司的数据服务访问哪个服务器

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701171756.png" alt="d833c895d143ad4b031769ca82025aafa40f06fa" style="zoom:75%;" />

### 2、搭建Mysql Cluster（基于阿里云内网部署，相同实例地区）

<u>*==主要参考==*</u>[^2][^5]

MySQL结构，由3类节点(计算机或进程)组成，分别是：

- 管理节点:用于给整个集群其他节点提供配置、管理、仲裁等功能。
- 数据节点:MySQL Cluster的核心，存储数据、日志，提供数据的各种管理服务。
- SQL节点(API):用于访问MySQL Cluster数据，提供对外应用服务。

==**各个节点中安装路径必须相同，配置环境一直！！！**==

1. 如果机器上安装了mysql-server，先卸载掉

   首先删除mysql-server:

   ```
   sudo apt-get remove mysql-*
   ```

   然后清理残留的数据

   ```
   dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
   ```

   它会跳出一个对话框，你选择yes就好了

2. 服务器上搭建时，同一个区域中的服务器（配置使用内网搭建，速度快）

修改 **vi /etc/hosts** 的文件，使用内网的ip

```shell
172.16.184.33	slave1
172.16.0.243	slave2
172.16.232.76	master
```

3. 在相应的主机中修改名称（或者是修改hosts的文件的名称，**为了便于后期使用，建议修改名称**）

```shell
hostnamectl set-hostname master
hostnamectl set-hostname slave1
hostnamectl set-hostname slave2
```

4. 设置主机之间的相互免密登录（设置公钥）

1. 下载安装包，<u>*==下载网址==*</u>[^1]

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701095735.png" alt="image-20210701095735494" style="zoom:50%;" />

2. 将压缩包移动到`/usr/local/mysql`下，解压完成后，修改路径，==每个节点都位于相同的路径下==

   ![image-20210701192501534](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701192501.png)

3. ==集群默认端口为1186==

#### 2.1 安装配置管理节点（GMG)

- 上传并解压文件

```sh
mkdir -p /usr/local/mysql （创建mysql目录）
cd /usr/local/mysql
tar -zxvf mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64.tar
mv mysql-cluster-8.0.25-linux-glibc2.12-x86_64 mysql-cluster （解压到目录下进行重命名）
```

- 创建管理目录

```sh
mkdir -p /var/log/mysql-cluster
mkdir -p /etc/mysql-cluster
```

- 配置目录

```sh
vi /etc/mysql-cluster/config.ini
```

```sh
[ndbd default]
NoOfReplicas=2                  #数据写入数量。2表示两份,2个数据节点
DataMemory=80M                  #配置数据存储可使用的内存
IndexMemory=18M                 #索引给100M


[ndb_mgmd]
nodeid=1
datadir=/var/log/mysql-cluster      #管理结点的日志
HostName=172.16.232.76              #管理结点的IP地址。本机IP


###### data node options:           #存储结点
[ndbd]
HostName=172.16.0.243
DataDir=/data/mysql                 #mysql数据存储路径
nodeid=2


[ndbd]
HostName=172.16.184.33
DataDir=/data/mysql                 #mysql数据存储路径
nodeid=3


# SQL node options:                 #关于SQL结点
[mysqld]
HostName=172.16.184.34
nodeid=4


[mysqld]
HostName=172.16.64.223
nodeid=5
```

- 初始化管理节点

```sh
/usr/local/mysql/mysql-cluster/bin/ndb_mgmd -f /etc/mysql-cluster/config.ini
```

- 查看集群的状态

```sh
/usr/local/mysql/mysql-cluster/bin/ndb_mgm
ndb_mgm> show
#或者使用
./ndb_mgm -e show
```

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701193206.png" alt="WX20210701-185520@2x" style="zoom: 50%;" />

发现管理节点已经启动，当数据节点以及SQL节点未启动时显示的是==not connected==

#### 2.2 安装配置数据节点（NDB)

==将下面配置同步到其余的配置数据节点中==

- 上传并解压文件

```sh
mkdir -p /usr/local/mysql （创建mysql目录）
cd /usr/local/mysql
tar -zxvf mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64.tar
mv mysql-cluster-8.0.25-linux-glibc2.12-x86_64 mysql-cluster （解压到目录下进行重命名）
```

- 创建mysql用户并运行

```sh
useradd -M -s /sbin/nologin mysql
```

- 创建mysql目录并授权

```sh
mkdir -p /etc/mysql
mkdir -p /data/mysql
chown -R mysql:mysql /data/mysql/
```

- 配置数据节点

```sh
vi /etc/mysql/my.cnf
[mysqld]
datadir=/data/mysql                     #mysql数据存储路径
ndbcluster                              #启动ndb引擎
ndb-connectstring=172.16.232.76         #管理节点IP地址  

[mysql_cluster]
ndb-connectstring=172.16.232.76         #管理节点IP地址
```

- 启动数据节点

```sh
#只有当第一次的时候才有加 --intital 否则数据会丢
/usr/local/mysql/mysql-cluster/bin/ndbd --initial
```

* 查看状态

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701193724.png" alt="WX20210701-185856@2x" style="zoom:50%;" />

![WX20210701-185909@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701193827.png)

![WX20210701-185917@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701193811.png)

#### 2.3 安装配置SQL节点（NDB)

==将下面配置同步到其余的安装配置SQL节点中==

- 上传并解压文件

```sh
mkdir -p /usr/local/mysql （创建mysql目录）
cd /usr/local/mysql
tar -zxvf mysql-cluster-gpl-7.6.12-linux-glibc2.12-x86_64.tar
mv mysql-cluster-8.0.25-linux-glibc2.12-x86_64 mysql-cluster （解压到目录下进行重命名）
```

- 创建mysql目录并授权

```sh
mkdir -p /etc/mysql
mkdir -p /data/mysql
chown -R mysql:mysql /data/mysql/
```

- 创建SQL节点配置文件

```sh
vi /etc/mysql/my.cnf
```

```sh
[mysqld]
user=mysql
ndbcluster                                                              #启动ndb引擎
ndb-connectstring=172.16.232.76                       #管理节点IP地址  

[mysql_cluster]
ndb-connectstring=172.16.232.76                     #管理节点IP地址
```

- 配置mysql服务

```sh
cp /usr/local/mysql/mysql-cluster/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
update-rc.d mysqld defaults
```

- 编辑mysqld服务

```sh
vi /etc/init.d/mysqld
```

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701194557.png" alt="image-20210701194557199" style="zoom:50%;" />

<center>替换为自己的安装目录</center>

```sh
  basedir=/usr/local/mysql/mysql-cluster
  bindir=/usr/local/mysql/mysql-cluster/bin
  if test -z "$datadir"
  then
    datadir=/data/mysql
  fi
  sbindir=/usr/local/mysql/mysql-cluster/bin
  libexecdir=/usr/local/mysql/mysql-cluster/bin
```

- 初始化mysql数据库

进入到 `/usr/local/mysql/mysql-cluster/bin`

```sh
#执行完命令 记得保存一下随机产生的密码
cd /usr/local/mysql/mysql-cluster/bin
./mysqld --initialize --user=mysql --basedir=/usr/local/mysql/mysql-cluster --datadir=/data/mysql/ 
```

Slave3（==保存随机产生的密码==）

![WX20210701-190549@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701194859.png)

Slave4（==保存随机产生的密码==）

![WX20210701-190639@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701194908.png)

- 启动mysql服务

```sh
systemctl daemon-reload
systemctl start mysqld
```

Slave3，发现已经启动了mysql的3306端口，出现红色的警告，可以使用`systemctl daemon-reload`尝试解决下，下面的截图是没有加`systemctl daemon-reload`的，但是还是可以正常启动

![WX20210701-190806@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701195302.png)

Slave4，发现已经启动了mysql的3306端口

![WX20210701-190818@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701195309.png)

#### 2.4 集群状态

进入到

```sh
cd /usr/local/mysql/mysql-cluster/bin
./ndb_mgm
```

进入下面界面使用`show`可==***查看到5个节点全部启动，说明搭建完成！！！***==

![WX20210701-191003@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701195652.png)

### 3、操作Mysql Cluster

* 在SQL节点（slave3，slave4）中使用操作数据库，输入前面保存的随机密码进入数据库中

```sh
cd /usr/local/mysql/mysql-cluster/bin
./mysql -uroot -p
```

- 操作数据库

  在使用命令之前需要先修改数据库的密码后进行操作，否则将会出现下面的错误

  ![WX20210701-191514@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701200526.png)

  使用下面命令修改密码为123456789

  ```mysql
  alter user user() identified by "123456789";
  ```

  <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701200649.png" alt="WX20210701-191530@2x" style="zoom:50%;" />

- 创建数据库

  在slave3中创建 `mytest` 数据库

  <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701200755.png" alt="WX20210701-191654@2x" style="zoom:50%;" />

  登录slave4中的数据库同时也修改密码，然后查看数据库时发现`mytest`==**数据库同步创建**==

  ![WX20210701-191741@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701200904.png)

- 创建数据表并添加数据

  添加 ==**engine=ndbcluster**== 或者是 ==**engine=ndb**==（mysql cluster集群==只能同步ndb引擎的表==）

  ```mysql
  create table sys_myfirst(id varchar(36) primary key, name varchar(100), memo varchar(255)) ENGINE=innodb DEFAULT CHARSET=utf8 engine=ndbcluster;
  Query OK, 0 rows affected, 1 warning (0.42 sec)
  ```

  slave3中创建数据表，并添加两条数据

  ![image-20210701205405177](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701205405.png)

  slave4中可查看到slave3中创建的表与数据

  ![image-20210701205442243](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701205442.png)

### 4、MySQL Cluster的关闭顺序与单点故障测试

[<u>==*关闭顺序与单点故障参考*==</u>](https://blog.csdn.net/mashuai720/article/details/79126360) [^4]

关闭顺序：SQL节点->数据节点->管理节点（在MySQL Cluster环境中，NDB节点和管理节点的关闭都可以在管理节点的管理程序中完成，也可以分节点关闭，但是SQL节点却没办法。所以，在关闭整个MySQL Cluster环境或者关闭某个SQL节点的时候，首先必须到SQL节点主机上来关闭SQL节点程序。关闭方法和MySQL Server的关闭一样。）

1. SQL节点关闭

   `systemctl stop mysqld`

   ```sh
   root@slave3:/usr/local/mysql/mysql-cluster/bin# lsof -i:3306
   COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   mysqld  1319 mysql   40u  IPv6  24436      0t0  TCP *:mysql (LISTEN)
   root@slave3:/usr/local/mysql/mysql-cluster/bin# systemctl stop mysqld
   Warning: The unit file, source configuration file or drop-ins of mysqld.service changed on disk. Run 'systemctl daemon-reload' to reload units.
   root@slave3:/usr/local/mysql/mysql-cluster/bin# lsof -i:3306
   root@slave3:/usr/local/mysql/mysql-cluster/bin# systemctl start mysqld
   Warning: The unit file, source configuration file or drop-ins of mysqld.service changed on disk. Run 'systemctl daemon-reload' to reload units.
   ###或者在/etc/init.d中使用
   root@slave4:/etc/init.d# ./mysqld stop
   ```

2. 数据节点（NDB）关闭

   ![image-20210702092053968](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210702092054.png)

   ```sh
   cd /usr/local/mysql/mysql-cluster/bin
   ./ndbd stop
   2018-01-16 18:30:51 [ndbd] INFO     -- Angel connected to '192.168.0.30:1186'
   2018-01-16 18:30:51 [ndbd] INFO     -- Angel allocated nodeid: 2
   #或者使用pkill -9 ndbd
   #启动使用（不需要加--initial）
   ./ndbd
   ```

3. 管理节点关闭（==关闭管理节点后会一起关闭数据节点==）

   ![image-20210702093643332](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210702093643.png)

   ```sh
   ndb_mgm> shutdown
   Node 2: Cluster shutdown initiated
   Node 3: Cluster shutdown initiated
   3 NDB Cluster node(s) have shutdown.
   Disconnecting to allow management server to shutdown.
   Node 3: Node shutdown completed.
   ndb_mgm> 
   #或者使用
   /usr/local/mysql/mysql-cluster/bin# ./ndb_mgm -e shutdown
   ```

### 5、Mysql Cluster 远程连接

1. 该表法

   ```mysql
   use mysql;
   select host, user from user;
   update user set host = '%' where user = 'root';
   select host, user from user;
   #修改完以后重启mysql
   ```

   <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210701235401.png" alt="image-20210701235401421" style="zoom:50%;" />

2. 授权法

   ```mysql
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456789' WITH GRANT OPTION;
   FLUSH PRIVILEGES  #使修改生效，就可以了
   ```

### 6、测试单点故障（高可用）

- 当停止掉两台数据节点中的一台节点时，使用SQL节点进行操作，可以正常操作
- 当停止掉两台SQL节点中的一台节点时，使用另一台SQL节点进行操作，也可以正常操作
- 当分别停止掉两台数据节点和两台SQL节点中的一台节点时，只留一台SQL节点与一台数据节点，同样可以正常服务

### 7、SpringBoot中使用

==*[springboot jpa连接mysql数据库集群(NDBCLUSTER)引擎设置](https://blog.csdn.net/xyzdwf/article/details/102482393)*==[^6]

主要参考：[*Spring Boot MyBatis 数据库集群访问实现*](https://github.com/smltq/spring-boot-demo/tree/master/mybatis-multi-datasource) [^7]

![image-20210702205711624](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210702205711.png)

代码路径[^8]：[语雀](https://www.yuque.com/yuankaiqiang/file/19961453)

### 参考网址

[^1]: https://dev.mysql.com/downloads/cluster/

[^2]: https://www.jianshu.com/p/37525ae8dee4
[^3]: https://www.pianshen.com/article/51641527056/
[^4]:https://blog.csdn.net/mashuai720/article/details/79126360
[^5]:https://www.cnblogs.com/zzdbullet/p/11475641.html
[^6]:https://blog.csdn.net/xyzdwf/article/details/102482393
[^7]:https://github.com/smltq/spring-boot-demo/tree/master/mybatis-multi-datasource
[^8]:https://www.yuque.com/yuankaiqiang/file/19961453



