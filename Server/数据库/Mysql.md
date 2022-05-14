[toc]

第一范式：1NF 原子性（列不可再分：每一列属性都是不可再分的属性值，确保每一列的原子性）
第二范式：2NF 唯一性（在第一范式的基础上建立，确保表中的每列都和主键相关）
第三范式：3NF 冗余性（每一列数据都和主键直接相关，而不能间接相关）

# 1、安装数据库

直接apt-get安装数据：http://www.linuxboy.net/ubuntujc/140519.html

# 2、修改密码或操作sql文件

```sh
#修改密码
mysqladmin -u root -p password 123456789
#导出数据库中的sql文件
mysqldump -u 用户名 -p 数据库名 > 导出的文件名.sql
mysqldump -uroot -p123456789 --databases DIAN_TI >> DIAN_TI.sql
#将sql文件导入数据库中
source sql文件
```

# 3、中文乱码

Ubuntu20版本的mysql5.7的配置文件：my.conf这个文件应该是在/etc/mysql/mysql.conf.d/mysqld.cnf这个文件

* 在创建数据库时指定默认的编码格式为utf8

create database 数据库名 CHARACTER SET utf8 COLLATE utf8_general_ci;

* 修改已创建的数据库的编码

alter database 数据库表名 CHARACTER SET utf8 COLLATE utf8_general_ci;

* 修改表的默认编码格式，将表和字段的编码都更改为utf8编码格式

alter table 表名 character set utf8 COLLATE utf8_general_ci;

# 4、8版本导入5版本数据问题

Mysql8的sql文件导入到Mysql5中出现的问题

`utf8mb4`替换为`utf8`

`utf8mb4_0900_ai_ci`替换为`utf8_general_ci`

`utf8_croatian_ci`替换为`utf8_general_ci`

`utf8mb4_general_ci`替换为`utf8_general_ci`

# 5、JDBC连接

参考网址：https://www.runoob.com/java/java-mysql-connect.html

jdbc:mysql://127.0.0.1:3306/book?useSSL=true

```java
package test;

import java.sql.*;

public class Main {

	public static void main(String[] args) throws Exception{
				Class.forName("com.mysql.jdbc.Driver");
				try(
					Connection conn = DriverManager.getConnection(
						"jdbc:mysql://114.55.26.230:3306/book?autoReconnect=true&useUnicode=true&useSSL=false"
						, "root" , "1314520ok");
					Statement stmt = conn.createStatement();
					ResultSet rs = stmt.executeQuery("select * from book_system"))
				{
					while(rs.next())
					{
						System.out.println(rs.getInt(1) + "\t"
							+ rs.getInt(2) + "\t"
							+ rs.getString(3) + "\t"
							+ rs.getString(4) + "\t"
							+ rs.getString(5));
					}
				}

	}

}
```

# 6、内连接、左连接（左外连接）、右连接（右外连接）、全连接（全外连接）

参考：https://blog.csdn.net/plg17/article/details/78758593

```mysql
CREATE TABLE `a_table` (
  `a_id` int(11) DEFAULT NULL,
  `a_name` varchar(10) DEFAULT NULL,
  `a_part` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `b_table` (
  `b_id` int(11) DEFAULT NULL,
  `b_name` varchar(10) DEFAULT NULL,
  `b_part` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

![20171209135012639](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826095328.png)

> 内连接

```mysql
#关键字
inner join on
#语句
select * from a_table a inner join b_table b on a.a_id = b.b_id;
```

![20171209133941291](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826095523.png)

说明：组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。

![20171209135846780](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826095548.png)

> 左连接（左外连接）

```mysql
#关键字
left join on / left outer join on
#语句
select * from a_table a left join b_table b on a.a_id = b.b_id;
```

![20171209141445680](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826095939.png)

说明：

left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。

左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

![20171209142610819](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826100010.png)

如果要获取a_table中的阴影部分且不包括与b_table中的数据则可以使用

```mysql
select * from a_table a left join b_table b on a.a_id = b.b_id WHERE a.id NOT IN (SELECT b_table.id FROM b_table);
```

> 右连接（右外连接）

```sql
#关键字
right join on / right outer join on
#语句
select * from a_table a  right outer join b_table b  on a.a_id = b.b_id;
```

![20171209143426953](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826102003.png)

说明：
right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。
与左(外)连接相反，右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。

![20171209144056668](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210826102023.png)

> 全外连接

**【注意】：Oracle数据库支持`full join`，mysql是不支持`full join`的，但仍然可以同过*左外连接+ UNION+右外连接*实现**

![505dbbcd46d08c04ebd69312f167edc1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210903111528.png)

```mysql
#关键字
UNION
#语句
select * from a_table a left join b_table b on a.a_id = b.b_id
UNION
select * from a_table a  right outer join b_table b  on a.a_id = b.b_id;
```

# 7、索引

参考：https://www.bilibili.com/video/BV19y4y127h4?p=3&spm_id_from=pageDriver

> 素引的优点

- 大大加快数据查询速度

> 素引的缺点

- 维护素引需要耗费数据库资源 
- 素引需要占用磁盘空间
- 当对表的数据进行增删改的时候,因为要维护索引,速度会受到影响

> 索引的分类

InnoDB

- **主键索引**：设定为主键后数据库会自动連立素引，innodb为聚族索引，索引列值不能为空
- **单值（单例/普通）索引**：一个索引只包含单个列，一个表可以有多个单列素引
- **唯一索引**：索引列的值必须唯一，但允许有空值，允许为null但是只能有一个null 
- **复合索引**：一个索引包含多个列

MyISAM

- Full Text（全文索引）：MySQL5.7版本之前，只能用于MyISAM引擎，MySQL8也支持全文索引。在定义索引的列上支持值的全文查找，允许在这些索引列中插入重复值和空值。全文索引可以在**CHAR、 VARCHAR、TEXT类型**列上创建。

> 索引的基本操作

**主键索引**

主键索引是一种特殊的唯一索引，不允许值重复或者值为空。创建表时自动进行创建。

创建主键索引通常使用 PRIMARY KEY 关键字。不能使用 CREATE INDEX 语句创建主键索引。

**普通索引**

两种方式进行创建：

给name列创建索引

1. 建表时创建（==KEY关键字==）（不可以指定索引的名称，名称默认为列的名称）

   ```mysql
   CREATE TABLE t_user (
   	id INTEGER PRIMARY KEY,
   	name VARCHAR(20),
       KEY(name)
   );
   ```

2. 建表后创建（name_index索引名称，t_user为表名，name为哪个列创建索引）

   ```mysql
   CREATE INDEX name_index ON t_user(name);
   ```

**唯一索引**

1. 建表时创建（==UNIQUE关键字==）（不可以指定索引的名称，名称默认为列的名称）

   ```mysql
   CREATE TABLE t_user (
   	id INTEGER PRIMARY KEY,
   	name VARCHAR(20),
       UNIQUE(name)
   );
   ```

2. 建表后创建

   ```mysql
   CREATE UNIQUE INDEX name_index ON t_user(name);
   ```

**复合索引**

1. 建表时创建

   ```mysql
   CREATE TABLE t_user (
   	id INTEGER PRIMARY KEY,
   	name VARCHAR(20),
       age INTEGER
       KEY(name,age)
   );
   ```

2. 建表后创建

   ```mysql
   CREATE INDEX name_index ON t_user(name,age);
   ```

==左前缀原则==

索引index1:(a,b,c)有三个字段，我们在使用sql语句来查询的时候，会发现很多情况下不按照我们想象的来走索引。

- 会走索引

  ```mysql
  select * from table where a = '1'  
  select * from table where a = '1' and b = ‘2’ 
  select * from table where a = '1' and b = ‘2’  and c='3'
  ```

  ```mysql
  select * from table where a = '1' and c= ‘2’
  -- a,c也走，但是只走a字段索引，不会走c字段
  ```

> 什么情况索引会失效

1. 条件中有or，**要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引**。

2. 复合索引未用左前缀字段;

3. like以%开头;

   like '张三%'，实际你要找的是'张三XXX'，只要把所有'张三'开头的那部分内容返回即可，这部分是连续的，不需要全表扫描。
   like '%张三'，实际你要找的是'XXX张三'，这部分在索引里是不连续的，如果要返回需要的结果，只能全表扫描。

4. 需要类型转换;where中索引列有运算;

5. where中索引列使用了函数;

6. 使用不等于（!= 、<>）

> 索引底层数据结构

**B树（B-Tree）**

每个节点都存储key和data，所有节点组成这棵树，并且叶子节点指针为null。

![20170920132504569](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830125314.png)

**B加树（B+Tree）**

只有叶子节点存储data，叶子节点包含了这棵树的所有键值，在B+树上增加了==顺序访问指针==，也就是每个叶子节点增加一个指向相邻叶子节点的指针

![9e31f41f0d93fe3e79d962cb9998d73a2845e805.png@942w_339h_progressive](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830121745.webp)

1、B+Tree非叶子节点不存储data，只存储key；
2、所有的关键字全部存储在叶子节点上；
3、每个叶子节点含有一个指向相邻叶子节点的指针，带顺序访问指针的B+树提高了区间查找能力；
4、非叶子节点可以看成索引部分，节点中仅含有其子树（根节点）中的最大（或最小）关键字；

**为什么Mysql索引要用B+树不是B树？**

​		用B+树不用B树考虑的是IO对性能的影响，B树的每个节点都存储数据，而B+树只有叶子节点才存储数据，所以查找相同数据量的情况下，B树的高度更高，IO更频繁。数据库索引是存储在磁盘上的，当数据量大时，就不能把整个索引全部 加载到内存了，只能逐一加载每一个磁盘页（对应索引树的节点）。

**==mysql的InnoDB存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要1~3次磁盘I/O操作。==**

![image-20210830131455218](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830131455.png)

![image-20210830135118451](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830135118.png)

# 8、InnoDB与MyISAM

- 聚簇索引：==将数据存储与索引放到了一块，素引结构的叶子节点保存了行数据==（聚簇索引不一定是主键索引，**主键索引一定是聚簇索引** ），一个表只能有一个聚簇索引，聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。
- 非聚簇索引：==将数据与索引分开存储，索引结构的叶子节点指向了数据对应的位置==

参考：https://www.runoob.com/w3cnote/mysql-different-nnodb-myisam.html

|  区别  |                 InnoDB                 |       MyISAM       |
| :----: | :------------------------------------: | :----------------: |
|  事务  |                  支持                  |       不支持       |
|  外键  |                  支持                  |       不支持       |
| 锁级别 |               支持行级锁               |      表级锁定      |
|  适合  | 适合频繁修改以及涉及到安全性较高的应用 | 适合查询为主的应用 |

**MYISAM索引实现（非聚集）**

所以MYISAM这个存储引擎他的查找的一个大致过程就是，先看条件字段有没有用到索引，是索引字段就先去到索引文件去查找这个索引所在的那一行的磁盘文件地址，就**借助B+Tree的特点从根节点顺藤摸瓜找到磁盘文件地址指**针，然后从MYD文件一次性定位到所找的数据，也就是说MYISAM会垮两个文件。

![image-20210830134250693](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830134250.png)

![a](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830122642.png)

**InnoDB索引实现（聚集）**==默认数据页大小为1 6kb==

![image-20210830134035196](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830134035.png)

它也是一个B+Tree，但是它的叶子节点和MYISAM有点区别，它存储的是索引所在行的所有字段。

![aH](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830122731.png)

![image-20210830133846337](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210830133846.png)

# 9、关系型与非关系型数据库区别

参考：https://blog.csdn.net/qq_33472765/article/details/81515251 

> 非关系型数据库与关系型数据库各自的优势

非关系型数据库的优势：

1. **性能NOSQL是基于键值对的**，nosql的存储格式是key,value形式、文档形式、图片形式等等，可以想象成表中的主键和值的对应关系，而且不需要经过SQL层的解析，所以**性能非常高。**

2. **可扩展性**同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平扩展。

关系型数据库的优势：

1. 复杂查询可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。

2. 事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。 

> 关系型数据库的优势和劣势

关系型数据库把所有的数据都通过行和列的二元表现形式表示出来。

关系型数据库的优势：

1. 保持**数据的一致性**（**事务处理**）
2. 由于以标准化为前提，数据更新的开销很小（相同的字段基本上都只有一处）
3. 可以进行Join等**复杂查询**

关系型数据库的不足：

1. 大量数据的写入处理

2. 为有数据更新的表做索引或表结构（schema）变更

3. 字段不固定时应用

4. 对简单查询需要快速返回结果的处理

# 10、Mysql order by与limit混用陷阱

参考：https://blog.csdn.net/qiubabin/article/details/70135556

​		在Mysql中我们常常用order by来进行排序，使用limit来进行分页，当需要先排序后分页时我们往往使用类似的写法select * from 表名 order by 排序字段 limt M,N。但是这种写法却隐藏着较深的使用陷阱。在排序字段有数据重复的情况下，会很容易出现排序结果与预期不一致的问题。

比如现在有一张user表，表结构及数据如下：

![aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzMyMDQ3LWVhN2VhMzYyMzc2Y2JjNzQucG5n](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210903112722.png)

![aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzMyMDQ3LTYyZTgyYWU1NzFhMWNjNjUucG5n](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210903112725.png)

现在想根据创建时间升序查询user表，并且分页查询，每页2条

```mysql
select * from user order by create_time limit pageNo,2;
```

1、查询第一页数据时：

![aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzMyMDQ3LWY4NDY3MGNlNzVmMDcwMmQucG5n](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210903112929.png)

2、查询第四页数据时：

![aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzMyMDQ3LTNlYzQ0YzE1NzA5YjVjNzAucG5n](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210903112942.png)

**user表共有8条数据，有4页数据，但是实际查询过程中第一页与第四页竟然出现了相同的数据。**

​		如果order by的字段有多个行都有相同的值，mysql是会随机的顺序返回查询结果的，具体依赖对应的执行计划。也就是说如果排序的列是无序的，那么排序的结果行的顺序也是不确定的。

基于这个我们就基本知道为什么分页会不准了，因为我们排序的字段是create_time，正好又有几个相同的值的行，在实际执行时返回结果对应的行的顺序是不确定的。对应上面的情况，第一页返回的name为8的数据行，可能正好排在前面，而第四页查询时name为8的数据行正好排在后面，所以第四页又出现了。

​		如果想在Limit存在或不存在的情况下，都保证排序结果相同，**可以额外加一个排序条件**。例如id字段是唯一的，可以考虑在排序字段中额外加个id排序去确保顺序稳定。

所以上面的情况下可以在SQL再添加个排序字段，比如fund_flow的id字段，这样分页的问题就解决了。

```mysql
SELECT * FROM user ORDER BY create_time,id LIMIT 6,2;
```

# 11、SELECT 语句的执行过程

![13](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210916094148.png)

> 客户端

​		客户端的作用是访问Mysql 的server层,相信大家第一次接触到Mysql, 然后…就啥都没了. 客户端就是用界面来与Mysql做交互的.
常用链接Mysql server层的方式有两种 :

- 图形化界面工具:
  - [NAVICAT](https://www.navicat.com.cn/) **界面好看 功能强大, 但是收费**
  - [SQLYOG](https://www.webyog.com/) **刚开始学Mysql 的时候用的, 满足日常使用,收费!**
  - [DBEAVER](https://dbeaver.io/) **挺棒的, 公司就是用的好这个, 因为免费**
- 命令行 (Linux/ Windows 都有)

> 连接器

在 JDBC 中获取的 Connection 连接对象。连接器的作用是通过TCP握手获取登录信息后验证权限, 以及连接的管理。

> 缓存

你执行一条SQL(查询)，如果你没有手动禁用缓存的话，Mysql会先到缓存中查询之前是否执行过这条SQL，有则视为命中缓存并且返回数据，没有命中则交由，分析器处理。
**需要注意的是增 删 改** 都会刷新Mysql的缓存，所以经常修改的的数据，不建议使用缓存。比如城市交通的线路，站名，城市名，省名，这些长期不变的数据 则建议使用缓存，提高效率。

MySQL 8.0 的版本中，已经不在支持查询缓存。

> 分析器

分析器的作用是对你执行的SQL的解析, 检查你的语法是否能正常被Mysql的执行器执行.
分析器又分为: **词法分析**和**语法分析**

> 优化器

经历了分析器，那么 MySQL 就知道你要干啥了，那么怎么做？优化器完成了这一步，判断需要**用哪个索引去查询**、先执行哪个条件、或者决定 join 表的连接顺序等等，**把前面的解析出来的换成执行的计划**，进行最优评估。

> 执行器

执行器首先要做的是检查**是否有权限访问这张表**，然后再根据建表时的选用的**储存引擎**去调用该储存引擎的读写(IO)接口。

查询：有索引走索引, 无索引走全表 .命中一条就返回一条到结果集中. 返回的同时也会放到缓存中方便下一次使用。

# 12、Mysql四种存储引擎

**InnoDB存储引擎**：是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键，上图也看到了，InnoDB是默认的MySQL引擎。

**MyISAM存储引擎**：MyISAM基于ISAM存储引擎，并对其进行扩展。它是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但不支持事物。

**MEMORY存储引擎**：MEMORY存储引擎将表中的数据存储到内存中，未查询和引用其他表数据提供快速访问。

如何选择存储引擎？

- 如果要提供提交、回滚、崩溃恢复能力的事物安全（ACID兼容）能力，并要求实现并发控制，InnoDB是一个好的选择
- 如果数据表主要用来插入和查询记录，则MyISAM引擎能提供较高的处理效率如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存中的Memory引擎。
- MySQL中使用该引擎作为临时表，存放查询的中间结果如果只有INSERT和SELECT操作，可以选择Archive，Archive支持高并发的插入操作，但是本身不是事务安全的。Archive非常适合存储归档数据，如记录日志信息可以使用Archive使用哪一种引擎需要灵活选择，一个数据库中多个表可以使用不同引擎以满足各种性能和实际需求。

# 13、SQL去重的三种方式

**==distinct、group by、row_number over()==**

![image-20210924131040949](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210924131040.png)

> distinct

```sql
select distinct name from userinfo;    # 现在需要当前用户表不重复的用户名
name 
xiaogang
xiaohei
xiaoli
xiaoming
select distinct name,id from userinfo; # 此时distinct同时作用了两个字段，即必须得id与name都相同的才会被排除
xiaogang 10
xiaoli 11
xiaohei 12
xiaogang 13
xiaoming 14
```

>group by

```sql
select name from userinfo groub by name; # 与上面的一致 
select name from userinfo groub by name, id; # 与上面的一致 
```

>row_number over()

语法：`ROW_NUMBER() OVER(PARTITION BY COLUMN1 ORDER BY COLUMN2)`；

**1：Partition BY 用来分组**

**2：Order by 用来排序**

```sql
SELECT * FROM (
select *,ROW_NUMBER() over(partition by name order by id desc) AS rn from userinfo ) AS u WHERE u.rn=1;

id  name   age height rn
13 xiaogang 26 172 1
12 xiaohei 22 152 1
11 xiaoli 31 176 1
14 xiaoming 31 176 1
```

**distinct 和group by 的区别：**

（1）distinct常用来查询不重复记录的条数：count(distinct name)，group by 常用它来返回不重记录的所有值。

（2）在使用group by 分组后，在select中可以选择分组字段，和非分组字段的函数值，如 max()、min()、sum、count()等。

**distinct 和row_number over()区别:**

（1）distinct 和 row_number over 都可以实现去重功能，而distinct 作用于当行的时候，其"去重" 是去掉表中字段所有重复的数据，作用于多行的时候是，其"去重"所有字段都相同的数据。

（2）在使用row_number over 子句时候是先分组，然后进行排序，再取出每组的第一条记录"去重"。

# 14、Expain

|     信息      |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
|      id       | ==查询的序号==，包含一组数字，表示查询中执行select子句或操作表的顺序<br/>**两种情况**<br/>id相同，执行顺序从上往下<br/>id不同，id值越大，优先级越高，越先执行 |
|  select_type  | ==查询类型==，主要用于区别普通查询，联合查询，子查询等的复杂查询<br/>1、simple ——简单的select查询，查询中不包含子查询或者UNION<br/>2、primary ——查询中若包含任何复杂的子部分，最外层查询被标记<br/>3、subquery——在select或where列表中包含了子查询<br/>4、derived——在from列表中包含的子查询被标记为derived（衍生），MySQL会递归执行这些子查询，把结果放到临时表中<br/>5、union——如果第二个select出现在UNION之后，则被标记为UNION，如果union包含在from子句的子查询中，外层select被标记为derived<br/>6、union result:UNION 的结果<br/> |
|     table     |                      ==输出结果集的表==                      |
|  partitions   |                          匹配的分区                          |
|     type      | ==连接类型==，显示查询使用了何种类型，按照从最佳到最坏类型排序<br/>1、system：表中仅有一行（=系统表）这是const联结类型的一个特例。<br/>2、const：表示通过索引一次就找到，const用于比较primary key或者unique索引。因为只匹配一行数据，所以如果将主键置于where列表中，mysql能将该查询转换为一个常量<br/>3、eq_ref:唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于唯一索引或者主键扫描<br/>4、ref:非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，可能会找多个符合条件的行，属于查找和扫描的混合体<br/>5、range:只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是where语句中出现了between,in等范围的查询。这种范围扫描索引扫描比全表扫描要好，因为它开始于索引的某一个点，而结束另一个点，不用全表扫描<br/>6、index:index 与all区别为index类型只遍历索引树。通常比all快，因为索引文件比数据文件小很多。<br/>7、all：遍历全表以找到匹配的行<br/>注意:一般保证查询至少达到range级别，最好能达到ref。<br/> |
| possible_keys |                  表示查询时，可能使用的索引                  |
|      key      | ==表示实际使用的索引==。如果没有选择索引,键是NULL。查询中如果使用覆盖索引，则该索引和查询的select字段重叠。 |
|    key_len    | ==索引字段的长度。==表示索引中使用的字节数，该列计算查询中使用的索引的长度在不损失精度的情况下，长度越短越好。如果键是NULL,则长度为NULL。该字段显示为索引字段的最大可能长度，并非实际使用长度。 |
|      ref      |                        列与索引的比较                        |
|     rows      |                   扫描出的行数(估算的行数)                   |
|   filtered    |                    按表条件过滤的行百分比                    |
|     Extra     | ==执行情况的描述和说明==。包含不适合在其他列中显示，但是十分重要的额外信息<br/>1、Using filesort：说明mysql会对数据适用一个外部的索引排序。而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成排序操作称为“文件排序”<br/>2、Using temporary:使用了临时表保存中间结果，mysql在查询结果排序时使用临时表。常见于排序order by和分组查询group by。<br/>3、Using index:表示相应的select操作用使用覆盖索引，避免访问了表的数据行。如果同时出现using where，表名索引被用来执行索引键值的查找；如果没有同时出现using where，表名索引用来读取数据而非执行查询动作。<br/>4、Using where :表明使用where过滤<br/>5、using join buffer:使用了连接缓存<br/>6、impossible where:where子句的值总是false，不能用来获取任何元组<br/>7、select tables optimized away：在没有group by子句的情况下，基于索引优化Min、max操作或者对于MyISAM存储引擎优化count（*），不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。<br/>8、distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作。<br/> |

# 15、SQL执行的顺序

1. from 
2. join 
3. on 
4. 添加外部行
5. where 
6. group by(开始使用select中的别名，后面的语句中都可以使用)
7. 函数count、avg、sum.... 
8. having 
9. select 
10. distinct 
11. order by
12. limit 

# 16、mysql锁

**全局锁、表级锁和行锁**

- 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
- 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

**MyISAM锁**

MySQL表级锁有两种模式：表共享锁（Table Read Lock）和表独占写锁（Table Write Lock）。

- 对MyISAM的读操作，不会阻塞其他用户对同一表请求，但会阻塞对同一表的写请求；
- 对MyISAM的写操作，则会阻塞其他用户对同一表的读和写操作；

**INNODB锁**

- 行锁

适用：从锁的角度来说，==表级锁更适合于以查询为主==，只有少量按索引条件更新数据的应用，如Web应用；而==行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，如一些在线事务处理==









