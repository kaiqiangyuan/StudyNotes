[toc]

### 一、简介

​		redis是一个key-value[存储系统](https://baike.baidu.com/item/存储系统)。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list([链表](https://baike.baidu.com/item/链表))、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些[数据类型](https://baike.baidu.com/item/数据类型)都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

​		Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了[memcached](https://baike.baidu.com/item/memcached)这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。 [1] 

​		Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

### 二、安装

> Ubuntu安装

参考：https://www.cnblogs.com/zongfa/p/7808807.html

使用`apt-get install redis`安装

使用`netstat -nlt|grep 6379`命令可以看到redis服务器状态

ubuntu中的配置文件在 `/etc/redis` 中

**设置密码**

```sh
bind 127.0.0.1  # 注释掉绑定的ip
protected-mode yes # 保护模式设置为no(无需密码)
requirepass 123456789 #设置密码
redis-cli 进入redis客户端
ping不通
auth "密码" 后进行操作，否则ping不通
输入密码后可以ping通
```

> Mac安装

1. 使用brew安装

```sh
brew install redis
brew services start redis（启动）
brew services stop redis（停止）
brew services restart redis（重启）
#退出客户端
quit
```

- Homebrew安装的软件会默认在`/usr/local/Cellar/`路径下
- redis的配置文件`redis.conf`存放在`/usr/local/etc`路径下

查看版本

```sh
redis-server -v
```

2. 设置密码

   ```sh
   cd /usr/local/etc
   vim redis.conf
   protected-mode yes  //保护模式，默认就是yess
   daemonize yes //设置后台启动，默认前台启动(设置了这个后不能用brew方式进行关闭redis)
   requirepass 123456789 //设置密码
   ```

### 三、使用

> 启动

```shell
#ubuntu中使用
启动服务: service-server redis start
停止服务: service-server redis stop
重启服务: service-server redis restart
#mac中(参考https://blog.csdn.net/realize_dream/article/details/106227622)
启动 brew services start redis 或者是 redis-server
停止 brew services stop redis 或者是 在redis-cli客户端中执行shutdown
重启 brew services restart redis 
```

> 命令

```shell
默认有16个数据库  第一个数据库为0（数组一样）   默认使用第一个数据库
select 3  切换到第四个数据库
DESIZE    查看当前数据库的大小
keys *      查看当前数据库所有的key
flushdb    清除当前数据库
flushall     清除全部数据库的内容
exists  key   判断这个key是否存在      
move key 1   删除key   1为当前数据库
expire  key  10  设置这个key的过期时间  10秒后这个key就自动删除  可以用ttl key检测
用户ip实时控制
type key     查看key的类型
```

- 当key不存在时，设置新的key/value。

  NX命令: **仅当key不存在时，set才会生效**

  ```sh
  127.0.0.1:6379> set hello world
  OK
  127.0.0.1:6379> set hello newval nx   # 由于hello存在，所以set命令不会生效，返回nil表示失败。
  (nil)
  127.0.0.1:6379> get hello   # 此时还是原来的值。
  "world"
  
  127.0.0.1:6379> set newkey value nx    # newkey不存在，set命令成功。
  OK
  127.0.0.1:6379> get newkey
  "value"
  ```

- 当key存在时，覆盖原有的key/value。

  XX命令：**仅当key存在时，set才会生效**

  ```sh
  127.0.0.1:6379> set hello world
  OK
  127.0.0.1:6379> set hello newval xx    # 由于hello存在，所以set命令会设置成功。
  OK
  127.0.0.1:6379> get hello              # 可以获取到新值
  "newval"
  
  127.0.0.1:6379> set newkey val xx      # 由于newkey不存在，所以不会设置成功
  (nil)
  127.0.0.1:6379> get newkey
  (nil)
  
  ```

### 四、类型操作

Redis支持五种数据结构，分别是String，List，Hash，Set，Zset。即字符串，列表，哈希，集合，有序集合。

> String字符串

```shell
append key “xxx”   在字符串后面加xxx（如果key不存在则创建key）

strlen key  获取字符串的长度

********************************************************

\# i++

\#步长  i+=

incr key   自增1（将key的value加1）

decr  key    减1（将key的value减1）

incrby key  10 （将key的value增加10）

decrby key  10（将key的value减10）

********************************************************

字符串范围    range

getrange key 0 3     截取字符串 [0,3]闭区间

getrange key 0 -1  获取全部字符串  和get key一样

替换

setrange key 1 999   替换指定位置开始的字符串（例如key的value为abcdefg，替换为a999efg）

********************************************************

setex（set with expire ） 设置过期时间

setnx（set if not exist）   不存在设置（在分布式锁中会常常使用！）

setex key  30  "hello"	设置key的值为hello，30秒后过期

setnx key value 将 key 的值设为 value （当且仅当 key 不存在若给定的 key 已经存在，则 SETNX 不做任何动作。）

getset key value1 将key的value值设置为value1 ，返回key旧的value（更新操作）

********************************************************

同时设置多个值 	mset	

同时获取多个值	mget

mset key1 value1 key2 value2 key3 value3  

mget value1 value2 value3

msetnx key1 key1 key4 key4       #会创建失败，msetnx是一个原子性的操作，要么一起成功，要么一起失败

\#	对象	

set user:1{name:zhangsan,age:3}    // 设置一个user:1对象  值为json字符来保存一个对象（user:{id}:{filed}）

127.0.0.1:6379> set user:1:name zhangsan

OK

127.0.0.1:6379> get user:1:name

"zhangsan"

127.0.0.1:6379> 
```

********************************************************

> List列表

可以当成==栈、队列、阻塞队列==使用

==所有的List命令都是以L开头==

```shell
lpush  将一个值或多个值插入到列表头部（左）
通过lenage 0 -1获取全部 也可以区间  
rpush   将一个值或多个值插入到列表尾部（右）
```

![QQ20200530-094340@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602075746.jpg)



```shell
lpop list    移除list的第一个元素
rpop list    移除lsitt的最后一个元素
********************************************************
lindex list 1	通过下标获得list的某个值 下标从0开始
********************************************************
llen list	获取list的长度
********************************************************
移除指定的值
lrem list  x x
```

移除list集合中指定个数的value

可以删除一个

可以删除多个重复的value

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602080048.png)

```shell
********************************************************
ltrim list 1 2   通过下标截取list中的value ，截取后的list将被修改
********************************************************
rpoplpush   移除列表的最后一个元素，将他移动到新的列表中
rpoplpush list list1   将list中的最后一个value删除并移动到list1
********************************************************
lset	将列表下标中指定下标的值替换为另外一个值，更新操作
lset list 0 good	将list下标为0的值替换为good，若list不存在则会报错
********************************************************
```

linsert

linsert list before|after xx yy将在lsit的xx前或者后插入yy

![QQ20200530-104713@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602080242.jpg)

list小结：

- list相当于是一个链表，before Node after ，left，right都可以插入值
- 如果key不存在，创建新的链表
- 如果存在，新增内容
- 如果移除了所有值，空链表，也代表不存在

> Set集合

值不能重复

```shell
********************************************************
sadd set hello	set集合中添加值
smembers set	查看set的所有值
sismember set hello	判断某一个值是不是在set集合中 ，在则返回1
 ********************************************************
scard set	获取set集合中的元素个数
********************************************************
srem set xx	移除set中的指定元素
********************************************************
set无序不重复集合，随机抽
srandmember list    随机抽选set中的一个元素
srandmember list  2  随机抽选set中的指定个数元素
********************************************************
spop set		随机删除一个set集合中的元素
********************************************************
smove set set1 xx		将set中指定的元素移动到set1中
********************************************************
```

两个集合的   差集  交集  并集

```shell
sdiff  set1  set2    查看不同的元素（set中查找与set1不同元素）
sinter  set1  set2  交集：共同元素
sunion set set1    并集
```

![QQ20200530-120408@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602080418.jpg)

> Hash哈希

```shell
hset map key value   	set一个具体key-value
hget map key			获取一个字段值
hmset map key1 value1 key2 value2		set多个key-value
hmget map key1 key2		获取多个字段
hgetall map		获取全部数据
hdel map key 	删除指定的key-value
hlen map 		获得map中的长度，有多少个key-value
hexists map key 	判断map中的指定字段是否存在
hkeys map 		获得所有的 key
hvals map	获得所有的value
hincrby map key 1 	将key对应的value加1
hincrby map key -1 	将key对应的value减1
hsetnx map key value		如果存在key则不能设置，不存在可以设置
hset user:1 key value 		对象
```

> Zset有序集合

```sh
在set基础上加上了一个值
zadd set 1 one		添加一个值 
zadd set 1 one set2 2 one2  	添加多个值   
```

```shell
zrangebyscore num -inf +inf显示全部用户信息从小到大
zrangebyscore num -inf +inf withscores显示信息并排序
zrangebyscore num -inf 2500 withscores  具体范围
zrevrange num 0 -1   从大到小排序
```

![QQ20200530-141305@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602080605.jpg)

```shell
zrem set xx 		移除有序集合中的指定元素
zcard set		获取有序集合的个数
zcount set 1 3		获取指定区间的成员数量
```

> geospatial

版本3.2以上可用

地理位置，距离等信息，经纬度

官方文档：https://www.redis.net.cn/order/3685.html

getadd 添加地理位置

> Hyperloglog

```
统计数量（数量大时有一定的误差）
PFadd mykey a b c d e f g h i j
PFCOUNT mykey		数量10
PFadd mykey2 i j z x c v b n m
PFCOUNT mykey2	数量9
PFMERGE mykey3 mykey mykey2       合并数量15
```

> Bitmap

操作二进制位来进行储存，只有0和1两个状态

> 事务

Redis单条命令式保存原子性的，但是事务不保证原子性！

> 乐观锁

### 五、整合Springboot

连接远程Redis

```sh
 redis-cli -h 114.55.26.230 -p 6379 -a 123456789
```

springboot2.x以后将原来的jedis替换为了lettuce

jedis : 采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全的，使用 jedis pool 连接

池！ 更像 BIO 模式

lettuce : 采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况！可以减少线程数据

了，更像 NIO 模式

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602081721.png)

所有的对象需要实例化才能传输

在以后的开发中，实体类都需要进行序列化

```java
User user = new User("袁凯强",3);
String jsonUser = new ObjectMapper().writeValueAsString(user);//使用jackjson序列化
```

传输中文，在控制台中无法显示，将被转义

因为默认使用的是jdk序列化

![QQ20200531-153242@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602081829.jpg)

```java
@Qualifier注解了，qualifier的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的，我们修改调用代码，添加@Qualifier注解，需要注意的是@Qualifier的参数名称必须为我们之前定义@Service注解的名称之一！
@Qualifier解释https://www.cnblogs.com/chenxiaoxian/p/9760032.html
```

代码参考：https://www.yuque.com/yuankaiqiang/file/19676353

### 六、Redis持久化（备份）

> RDB(Redis默认使用)

**优点：适合大规模的数据恢复，备份**

redis为内存数据库，断电即失，因此提供了持久化

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602082211.png)

修改配置文件，60秒内修改了5个key的话就持久化一次生成一个dump.rdb文件

可以根据自己的需要设置配置文件

![QQ20200601-141645@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602082244.jpg)

触发机制

1. svae的规则满足情况下，会自动触发rdb规则
2. 执行flushall操作时，也会触发rdb规则
3. 退出redis时（相当于断电），也会产生rdb

备份自动生成一个rdb文件

**恢复rdb文件的数据**

1. 只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb 恢复其中

的数据！

2. 查看需要存放的位置，若果在这个目录下存在dump.rdb文件，启动就会自动恢复其中的数据。

例如我启动redis-server /usr/local/bin/myconfig/redis.conf时，dump.rdb就会生成在/usr/local/bin/目录下  

恢复时需要要在这个redis-server /usr/local/bin/myconfig/redis.conf启动

![4](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602082338.png)

![3](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602082557.png)

> AOF（Append Only File）

将我们的所有命令都记录下来，history，恢复的时候就把这个文件全部在执行一遍！

手动开启  设置为yes

![5](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602082648.png)

优点：

1. 每一次修改都同步，文件的完整会更加好！

2. 每秒同步一次，可能会丢失一秒的数据

3. 从不同步，效率最高的！

缺点：

1. 相对于数据文件来说，aof远远大于 rdb，修复的速度也比 rdb慢！

2. Aof 运行效率也要比 rdb 慢，所以我们redis默认的配置就是rdb持久化

> 对比

1. RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储

2. AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重

写，使得AOF文件的体积不至于过大。

3. 只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化

4. 同时开启两种持久化方式

在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。

RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

5. 性能建议

因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。如果Enable AOF 好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。

### 七、Redis发布订阅

菜鸟教程：https://www.runoob.com/redis/redis-pub-sub.html

![9](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602083642.png)

![QQ20200601-160633@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602083701.jpg)

### 八、Redis主从复制

参考：https://www.cnblogs.com/kismetv/p/9236731.html

### 九、哨兵模式

参考：https://www.cnblogs.com/kismetv/p/9236731.html

### 十、分布式锁

- 当在分布式模型下，数据只有一份（或有限制），此时需要利用锁的技术控制某一时刻修改数据的进程数。
- 用一个状态值表示锁，对锁的占用和释放通过状态值来标识

1. 加锁

   加锁实际上就是在redis中，给Key键设置一个值，为避免死锁，并给定一个过期时间。**当任务时间大于设置的过期时间时，检测剩余生存时间，然后重新设置过期的时间即可（重新设计时间时不需要加上NX）。**

   ```sh
   SET lock_key random_value NX PX 5000
   TTL lock_key  #当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。
   ```

   `random_value` 是客户端生成的唯一的字符串。
   `NX` 代表只在键不存在时，才对键进行设置操作。
   `PX 5000` 设置键的过期时间为5000毫秒。

   - 在沙滩上踩一脚，留下自己的脚印，就对应了加锁操作。其他进程或者线程，看到沙滩上已经有脚印，证明锁已被别人持有，则等待。

2. 解锁

   解锁的过程就是将Key键删除。但也不能乱删，不能说客户端1的请求将客户端2的锁给删除掉。这时候`random_value`的作用就体现出来，为了保证解锁操作的原子性，我们用LUA脚本完成这一操作。先判断当前锁的字符串是否与传入的值相等，是的话就删除Key，解锁成功。

   ```sh
   if redis.call('get',KEYS[1]) == ARGV[1] then 
      return redis.call('del',KEYS[1]) 
   else
      return 0 
   end
   ```
   
   - 把脚印从沙滩上抹去，就是解锁的过程。
   
3. 锁超时

   - 为了避免死锁，我们可以设置一阵风，在单位时间后刮起，将脚印自动抹去。

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210705220127.png" alt="image-20210705220126916" style="zoom:50%;" />

```properties
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=123456789
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=20
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=10
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=1000
```

```java
/**
 * @ClassName: RedisLock
 * @Description: 加锁解锁操作
 * @Author yuankaiqiang
 * @DateTime 2021-07-06 23:24:08
 */
@Service
public class RedisLock {
	
	private static final Logger log = LoggerFactory.getLogger(RedisLock.class);

	private String lock_key = "redis_lock"; // 锁键

	protected long internalLockLeaseTime = 30000;// 锁过期时间

	private long timeout = 999999; // 获取锁的超时时间

	@Autowired
	private RedisTemplate<String, Object> redisTemplate;

	/**
	 * 加锁
	 * 
	 * @param id
	 * @return
	 */
	public boolean lock(String id, String threadName) {
		Long start = System.currentTimeMillis();
		try {
			while (true) {
				boolean flag = redisTemplate.opsForValue().setIfAbsent(lock_key, id, internalLockLeaseTime,
						TimeUnit.MILLISECONDS);
				// true，则证明获取锁成功，处理相关数据
				if (flag) {
					IndexController.setCount(IndexController.getCount() + 1);
					log.info(threadName + "->线程获取锁成功，当前count值为：{1}" + IndexController.getCount());
					return true;
				}
				// 否则循环等待，在timeout时间内仍未获取到锁，则获取失败
				long l = System.currentTimeMillis() - start;
				if (l >= timeout) {
					log.info(threadName + "->线程获取锁超时！");
					return false;
				}
				try {
					Thread.sleep(1000);
					log.info(threadName + "->锁被占用，正在等待！");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		} catch (Exception e) {
			return false;
		}
	}

	/**
	 * 解锁
	 * 
	 * @param id
	 * @return
	 */
	public boolean unlock(String id, String threadName) {
		String script = "if redis.call('get',KEYS[1]) == ARGV[1] then" + "   return redis.call('del',KEYS[1]) " + "else"
				+ "   return 0 " + "end";
		try {
			// 执行 lua 脚本
			DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
			// 指定 lua 脚本
			redisScript.setScriptText(script);
			// 指定返回类型
			redisScript.setResultType(Long.class);
			// 参数一：redisScript，参数二：key列表，参数三：arg（可多个）
			Long result = redisTemplate.execute(redisScript, Collections.singletonList(lock_key), id);

			if (1 == result) {
				log.info(threadName + "->线程任务完成，解锁成功！");
				return true;
			}
			return false;
		} catch (Exception e) {
			return false;
		}
	}
}
```

```java
/**
 * @ClassName: IndexController
 * @Description: 分布式锁实现
 * @Author yuankaiqiang
 * @DateTime 2021-07-06 23:23:53
 */
@RestController
public class IndexController {
	
	private static final Logger log = LoggerFactory.getLogger(IndexController.class);

    @Autowired
    RedisLock redisLock;
    
    static int count = 0;
    
    /**
     * @Title: index
     * @Description: 增加了redis分布式锁，每次都一致，只是简单实现
     * @Author yuankaiqiang
     * @DateTime 2021-07-06 23:23:10
     * @return
     * @throws InterruptedException
     */
    @RequestMapping("/index")
    public int index() throws InterruptedException {
        int clientcount = 100;

        for (int i = 0;i < clientcount;i++){
			String key = i + "_" + UUID.randomUUID().toString().replaceAll("-","").toLowerCase();
			new Thread( ()-> {
        		try {
        			redisLock.lock(key, Thread.currentThread().getName());
        		} finally {
                    redisLock.unlock(key, Thread.currentThread().getName());
        		}
        	}, key).start();
        }
        return count;
    }
    
    /**
     * @Title: index1
     * @Description: 不使用分布式锁，每次count的值都不一样，很少情况才为正确的值
     * @Author yuankaiqiang
     * @DateTime 2021-07-06 22:51:15
     * @return
     * @throws InterruptedException
     */
    @RequestMapping("/index1")
    public int index1() throws InterruptedException {
    	count = 0;
        int clientcount = 100;
        
        for (int i = 0;i < clientcount;i++){
			new Thread( ()-> {
        		try {
        			try {
						Thread.sleep(1);
						count++;
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
        		} catch (Exception e) {
        			e.printStackTrace();
				}
        	}).start();
        }
        
        Thread.sleep(5000);
        log.info("count值：" + count);
        return count;
    }
    
	public static int getCount() {
		return count;
	}

	public static void setCount(int count) {
		IndexController.count = count;
	}
    
}
```

增加锁后，每次变量都为100

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210707173859.png" alt="image-20210707173859280" style="zoom:50%;" />

### 十一、常见问题

redis-cli中文显示问题

```sh
#查看value值，对象中含中文
39.103.149.176:6379> get user
"[\"com.redis.pojo.User\",{\"name\":\"\xe8\xa2\x81\xe5\x87\xaf\xe5\xbc\xba\",\"age\":3}]"
#使用客户端时加上 --raw
redis-cli --raw -h 39.103.149.176

yuankaiqiang@Mac ~ % redis-cli --raw -h 39.103.149.176
39.103.149.176:6379> auth 123456789
OK
39.103.149.176:6379> ping
PONG
39.103.149.176:6379> get user
["com.redis.pojo.User",{"name":"袁凯强","age":3}]
```

































