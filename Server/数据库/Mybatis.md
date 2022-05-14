[toc]

|                             #{}                              |                         ${}                          |
| :----------------------------------------------------------: | :--------------------------------------------------: |
| ==不能写到字符串中==的，如果写到字符串表达字串内容一部分。Mybatis是无法给这个占位符赋值。 | 是==可以写到字符串中==的，表达取中表描述的字段的值。 |
|        使用jdbc的预编译处理，可以有==防止sql注入==。         |          取字符的值，==有sql注入的可能==。           |



```xml
resultType：主要针对于从数据库中提取相应的数据出来
parameterType：主要针对于将信息存入到数据库中 如： insert ， Update等
resultMap与parameterMap（可以自定义）
不推荐使用parameterMap
```

1. namespace

namespace中的包名要和接口的包名一致！

![QQ20200521-171132@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602084350.jpg)

2. select

id：就是对应的namespace中的方法名；

resultType：Sql语句执行的返回值！

parameterType：参数类型

### 一、XML使用

> XML(不使用注解操作数据库，推荐使用map)，核心配置文件必须按顺序添加，不然会报错

xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<!--   引入外部配置文件
	<properties resource="xxx.properties">
		<property name="" value=""/>
	</properties>   -->

	<!-- 日志 -->
	<settings>
		<setting name="logImpl" value="STDOUT_LOGGING"/>
	</settings>

	<!-- 可以给实体类起别名 -->
	<typeAliases>
		<typeAlias type="pojo.User" alias="User"/>
	</typeAliases>



  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>   <!-- &使用&amp;才有效 -->
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/test?useSSL=true&amp;serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="123456789"/>
      </dataSource>
    </environment>
  </environments>
  
  <mappers>
  <!-- 用 / 不用 . 
  resource 绑定mapper需要使用路径
  -->
   	<mapper resource="dao/UserMapper.xml"/>
  </mappers>
  
</configuration>
```

1. 编写接口

```java
public interface UserDao {
	//查询全部用户
	List<User> getUserList();
	
	//根据ID查询用户
	User getUserById(int id);
	
	//插入数据
	int addUser(User user);
	
	//修改数据
	int updateUser(User user);
	
	//删除用户
	int deleteUser(int id);
}
```

2. 编写对应的mapper中的sql语句

   ```xml
   <mapper namespace="dao.UserDao">
   
   	<!-- select查询语句 -->
     <select id="getUserList" resultType="pojo.User">
       select * from test.user
     </select>
     
     <select id="getUserById" resultType="pojo.User" parameterType="int">
       select * from test.user where id=#{id}
     </select>
     
     <!-- 对象中的属性，User中的属性，可以直接取出来  -->
     <insert id="addUser" parameterType="pojo.User">
     	insert into test.user values(#{id},#{name},#{pwd})
     </insert>
     
     <update id="updateUser" parameterType="pojo.User">
     	update test.user set id = #{id}, pwd=#{pwd} where name=#{name}
     </update>
     
     <delete id="deleteUser" parameterType="int">
     	delete from test.user where id=#{id}
     </delete>
     
   </mapper>
   ```

3. 测试

   ```java
   	@Test
   	public void getUserList() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		List<User> userList = userDao.getUserList();
   		
   		for (User user : userList) {
   			System.out.println(user);
   		}
   		
   		sqlSession.close();
   	}
   	
   	@Test
   	public void getUserById() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		
   		User user = userDao.getUserById(1);
   		System.out.println(user);
   		
   		sqlSession.close();
   		
   	}
   
   	//需要提交事务，才可以将数据插入到数据库中
   	//  sqlSession.commit();
   	@Test
   	public void addUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		
   		int temp = userDao.addUser(new User(4,"凯","123456"));
   		if(temp>0) {
   			System.out.println("成功");
   			//需要提交事务，才可以将数据插入到数据库中
   			sqlSession.commit();
   		}
   		sqlSession.close();
   	}
   	
   	@Test
   	public void upadateUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		
   		int temp = userDao.updateUser(new User(4,"凯","123123"));
   		if(temp>0) {
   			System.out.println("成功");
   			//需要提交事务，才可以将数据插入到数据库中
   			sqlSession.commit();
   		}
   		sqlSession.close();
   	}
   	
   	@Test
   	public void deleteUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		
   		int temp = userDao.deleteUser(4);
   		if(temp>0) {
   			System.out.println("成功");
   			//需要提交事务，才可以将数据插入到数据库中
   			sqlSession.commit();
   		}
   		sqlSession.close();
   	}
   ```

   利用mybatis进行模糊插叙时，要么使用如下方式要么使用 **like concat(concat('%',#{xxx}),'%')**

   ![QQ20200521-182944@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602084805.jpg)

> XML配置文件

引入外部配置文件

```xml
<!--   引入外部配置文件
	<properties resource="xxx.properties">
		<property name="" value=""/>
	</properties>   -->
```

* 可以在其中增加一些属性配置

- 如果两个文件有同一个字段，优先使用外部配置文件的信息。

==**类型别名**==

* 第一种

```xml
<!-- 可以给实体类起别名   -->
<typeAliases>
    <typeAlias type="pojo.User" alias="User"/>
</typeAliases>
```

* 第二种

也可以指定一个包名，扫描实体类的包名，，默认的别名就是这个类的类名，==首字母小写（建议）==

```xml
<!-- 可以给实体类起别名   包名-->
<typeAliases>
    <!-- type这个地方为包名的，实体类的包名-->
    <typeAlias type="pojo"/>
</typeAliases>
```

实体类较少时可以使用第一种，可以自定义别名

实体类多时使用第二种，不能自定义别名

* 第三种

在实体类上加上注解**@Alias("XXX")**也可以实现

以下的别名为hello

![QQ20200521-194847@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602085348.jpg)

==其它的数据类型的默认别名==

![QQ20200521-195242@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602085409.jpg)

![QQ20200522-174310@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602085435.jpg)

> 结果集映射

如果实体类中的属性与数据库中的字段不对应，可以使用结果集映射

```xml
  <select id="getUserById" resultMap="UserMap">
    select * from test.user where id=#{id}
  </select>
  <!-- 结果集映射 -->
  <resultMap type="User" id="UserMap">
  	<!-- colum数据库中的字段  ， property实体类中的属性 -->
  	<result column="password" property="pwd"/>
  </resultMap>
```

> 日志log4j

```xml
	<!-- 日志 -->
	<settings>
		<setting name="logImpl" value="STDOUT_LOGGING"/>
	</settings>
```

### 二、注解使用

在工具类创建的时候实现自动提交事务

```java
	public static SqlSession getSqlSession() {
		//  加上true后增删改不需要提交事务，不需要commit
		return sqlSessionFactory.openSession(true);
	}
```

1. 编写接口，添加注解

   ```java
   public interface UserDao {
   	
   	@Select("select * from user")
   	List<User> getUsers();
   	
   	// 当参数为基本数据类型或String类型时，所有的参数前必须加上@Param("xxx")注解
   	//引用类型不需要写@Param("xxx")注解
   	@Select("select * from user where id=#{id}")
   	User getUserById(@Param("id") int id);
   	
   	@Insert("insert into user values (#{id},#{name},#{pwd})")
   	void addUser(User user);
   	
   	@Update("update user set name=#{name} where id=#{id}")
   	void upadateUser(@Param("id") int id,@Param("name") String name);
   	
   	@Delete("delete from user where id=#{id}")
   	void deleteUser(@Param("id") int id);
   	
   }
   ```

2. 测试

   ```java
   public class AppTest {
   	
   	//查询全部数据
   	@Test
   	public void getUserList() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		List<User> user = userDao.getUsers();
   		for (User user2 : user) {
   			System.out.println(user2);
   		}
   		sqlSession.close();
   	}
   	
   	//根据ID查询
   	@Test
   	public void getUserById() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		User user = userDao.getUserById(1);
   		System.out.println(user);
   		
   		sqlSession.close();
   	}
   	
   	//插入数据
   	@Test
   	public void addUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		userDao.addUser(new User(4,"袁","123123"));
   		
   		sqlSession.close();
   	}
   	
   	//修改数据
   	@Test
   	public void updateUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		userDao.upadateUser(4, "凯");
   		
   		sqlSession.close();
   	}
   	
   	//删除数据
   	@Test
   	public void deleteUser() {
   		SqlSession sqlSession = MybatisUtils.getSqlSession();
   		
   		UserDao userDao = sqlSession.getMapper(UserDao.class);
   		userDao.deleteUser(4);
   		
   		sqlSession.close();
   	}
   	
   }
   ```

   ==【注意：必须要将接口注册绑定到我们的核心配置文件中】==

![QQ20200522-174001@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602085845.jpg)

==@Param()==注解

- 基本类型的参数或者String类型，需要加上
- 引用类型不需要加
- 如果一只有一个基本类型的话，可以忽略，但是建议加上
- SQL中的引用就是@Param()中设定的属性名

> 按照查询嵌套处理（多张表有关系时）

子查询、联表查询时

![QQ20200522-220542@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602085958.jpg)

![QQ20200522-220727@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090010.jpg)

![QQ20200522-220713@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090024.jpg)

![QQ20200522-221206@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090040.jpg)

![QQ20200522-220934@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090055.jpg)

对象：associatio  【多对一】

集合：collection【一对多】

javaType="x"  指定实体类中属性的类型

ofType 之低昂映射到List或者集合中的pojo类型，泛型中的约束类型。

集合中的泛型信息，使用ofType获取

![QQ20200522-221010@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090127.jpg)

![QQ20200522-221053@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090208.jpg)

当java中实体类的属性名与数据库中的字段名不一致时（数据库中存在"_"）

==mapUnderscoreToCamelCase==

是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。

数据库中的的名字为**test_Test**时java中可以使用 **testTest**

![QQ20200523-111945@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090509.jpg)

> UUID

生成唯一的32位的随机数，在数据库中设定id时可以使用，以便形成唯一的用户

```java
System.out.println(UUID.randomUUID().toString().replaceAll("-", ""));
```

> 动态SQL语句

==在SQL中增加逻辑代码==

在select中使用**if条件**进行查询时，方便进行判断查询操作，动态查询数据库中的信息。

例如以下例子，当传递过来的属性不为空时，则在其sql语句中增加sql语句

![QQ20200523-114219@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602090650.jpg)

上面这种方式where后面需要加上1=1，不好

**==where==**

**where** 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

```xml
  <select id="queryData" parameterType="map" resultType="user">
  	select * from user
  	<where>
	  	<if test="name != null">
	  		name = #{name}
	  	</if>
	  	<if test="pwd != null">
	  		and pwd = #{pwd}
	  	</if>
  	</where>
  </select>
```

**==choose==**

使用**choose**时，则只能执行一条语句，都都不符合**when**的条件则执行**otherwise**

```xml
	<where>
		<choose>
			<when test="name != null ">
				name = #{name}
			</when>
			<when test="pwd != null ">
				pwd = #{pwd}
			</when>
			<otherwise>
				id = #{id}
			</otherwise>
		</choose>
	</where>
```

==**Set**==

用于动态更新语句的类似解决方案叫做 *set*。*set* 元素可以用于动态包含需要更新的列，忽略其它不更新的列，元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）

```xml
  <!-- 动态sql 更新 -->
  <update id="updateData">
  	update user
  	<set>
		<if test="name != null">
			name = #{name},
		</if>
		<if test="pwd != null">
			pwd = #{pwd},
		</if>
  	</set>
  	where id = #{id}
  </update>
```

> SQL片段

**将一些功能相同部分抽取出来，方便复用**

1. 使用SQL标签抽取公共的部分

   ```xml
     <sql id="sql-name-pwd">
   	  	<if test="name != null">
   	  		name = #{name}
   	  	</if>
   	  	<if test="pwd != null">
   	  		and pwd = #{pwd}
   	  	</if>
     </sql>
   ```

2. 在需要使用的地方使用include标签引入

   ```xml
     <select id="queryData" parameterType="map" resultType="user">
    	select * from user
     	<where>
   		<include refid="sql-name-pwd"></include>
     	</where>
     </select>
   ```

注意：

- 最好是单表查询来定义SQL片段，多表效率会低，
- 抽取的SQL片段中不要有where标签

> foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602091139.png)

```xml
  <select id="queryData" parameterType="map" resultType="user">
	select * from user
	<where>
		<foreach collection="ids" item="id" open="id in(" separator="," close=")">
			#{id}
		</foreach>
	</where>
  </select>
```

```java
		Map<String,Object> map = new HashMap<String, Object>();
		ArrayList<Integer> list2 = new ArrayList<Integer>();
		list2.add(1);
		list2.add(2);
		map.put("ids", list2);
		for (User user : userDao.queryData(map)) {
			System.out.println(user);
		}
```

> 一级缓存（默认开启）

1. 一级缓存也叫本地缓存，SqlSession

2. 与数据库相同操作期间查询到的数据会放在本地缓存中。

3. 以后如果需要获取相同的数据，直接从缓存中拿，不必再去操作数据库。

**一级缓存只在如下部分有效。**

![QQ20200523-162305@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602091307.jpg)

缓存失效的情况：

- 查询不同的东西 。
- **增删改查操作**，可能会改变原来的数据，所以必定会刷新缓存。
- 查询不同的Mapper.xml。
- 手动清理缓存   使用  **sqlSession.clearCache();   //手动清理缓存。**

总结：一级缓存是默认开启的，只在一次SqlSession中有效，也就是拿到这个连接到关闭这个连接区间中 。

> 二级缓存

1. 开启全局缓存

   cacheEnabled：全局性地开启或关闭所有映射器配置文件中已配置的任何缓存

   ```xml
   	<!-- 设置 -->
   	<settings>
   		<!-- 日志 -->
   		<setting name="logImpl" value="STDOUT_LOGGING"/>
   		<!-- 运行java使用驼峰属性名匹配数据库中的带"_"的字段名 -->
   		<setting name="mapUnderscoreToCamelCase" value="true"/>
   		<!-- 显示开启全局缓存 -->
   		<setting name="cacheEnabled" value="true"/>
   	</settings>
   ```

2. 在xml文件中使用二级缓存`<cache/>`

   需要将实体类序列化，否则报错：Caused by: java.io.NoSerializableException:com.ykq

   解决：在实体类中实现接口Serializable

   ![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602091528.png)

   也可以使用如下，可自定义

   ```xml
   <cache
     eviction="FIFO"
     flushInterval="60000"
     size="512"
     readOnly="true"/>
   ```

==**缓存顺序：**==

==**二级缓存》一级缓存》查询数据库==**















