[toc]

# 1、Spring

==Spring== 框架就像一个家族，有众多衍生产品例如 ==boot、security、jpa==等等；但他们的基础都是Spring 的==ioc==和 ==aop==，==ioc== 提供了依赖注入的容器， ==aop==解决了面向切面编程，然后在此两者的基础上实现了其他延伸产品的高级功能。

核心：

- ==控制反转（IOC）==

  基于 工厂模式+反射

  **将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。在我们以往的编程中如果需要一个bean往往需要去手动去new一个出来。而spring帮我们解决了这个问题，在spring中我们只需要去定义bean，spring就会自动的帮我们实例化并管理Bean。而这些Bean就管理在spring容器中。**

  1. **谁控制谁，控制什么：**以前我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而==IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建==；**谁控制谁？**当然是IoC 容器控制了对象；**控制什么？**==主要控制了外部资源获取（不只是对象包括比如文件等）==。
  2. **为何是反转，哪些方面反转了：**正转，**传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象；** **而反转则是由容器来帮忙创建及注入依赖对象；**为何是反转？**因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；**哪些方面反转了？**依赖对象的获取被反转了。**

- ==面向切面（AOP）==

  基于动态代理

  主要是通过切面类来提高代码的复用，降低业务代码的耦合性，从而提高开发效率。主要的功能是：==**日志记录，性能统计，安全控制，事务处理，异常处理**==等等。
  
  - oop 是面向对象编程

# 2、Spring MVC

Spring MVC提供了一种轻度耦合的方式来开发==web==应用；它是Spring的一个==模块==，是一个==web框架==；通过==DispatcherServlet, ModelAndView 和 View Resolver==，开发web应用变得很容易；解决的问题领域是网站应用程序或者服务开发——==URL路由、Session、模板引擎、静态Web资源==等等。

<img src="https://images2015.cnblogs.com/blog/811883/201704/811883-20170423150019101-1710764799.jpg" alt="img" style="zoom:60%;" />

# 3、SpringBoot

> Spring Boot不是一门新技术。从本质上来说，Spring Boot就是Spring，它做了一些对Spring Bean的默认配置。

它使用“==习惯优于配置==”（项目中存在大量的配置，此外还内置了一个习惯性的配置，让你无需手动进行配置）的理念让你的项目快速运行起来。使用Spring Boot很容易创建一个独立运行（运行jar,内嵌Servlet容器）、准生产级别的基于Spring框架的项目，使用Spring Boot你可以不用或者只需要很少的Spring配置。

1. ==springboot自动装载==：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置

   https://blog.csdn.net/dfBeautifulLive/article/details/106428263

   https://www.cnblogs.com/hhcode520/p/9450933.html

   ![img](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202112041233116.png)

   @SpringBootApplication等同于下面三个注解：

   - @SpringBootConfiguration
   - @EnableAutoConfiguration
   - @ComponentScan

   其中==@EnableAutoConfiguration==是关键(启用自动配置)，内部实际上就去加载META-INF/spring.factories文件的信息，然后筛选出以EnableAutoConfiguration为key的数据，加载到IOC容器中，实现自动配置功能！

2. ==内嵌了如Tomcat容器==，也就是说可以直接跑起来，用不着再做部署工作了；

3. 无需再像Spring那样搞一堆繁琐的xml文件的配置；

4. 可以自动配置(核心)Spring。SpringBoot将原有的XML配置改为Java配置，将==bean注入改为使用注解注入的方式(@Autowire)==，

5. ==将多个xml、properties配置浓缩在一个appliaction.yml配置文件中==。

6. ==整合常用依赖==（开发库，例如web、jpa、security和test等），提供的POM可以简化Maven的配置。当我们引入核心依赖时，SpringBoot会自引入其他依赖。

------

https://www.jb51.net/article/169300.htm

==**@Resource和@Autowired**==都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。

**@Autowired**为Spring提供的注解，注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。如下：

```java
public class TestServiceImpl {
  @Autowired
  @Qualifier("userDao")
  private UserDao userDao; 
}
```

**@Resource**默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。

**启动流程**：https://blog.csdn.net/weixin_44235601/article/details/108320844 

# 4、面向对象特性

1. 抽象：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面，抽象只关注对象的哪些属性和行为，并不关注这此行为的细节是什么

2. 继承：继承是从已有类得到继承信息创建新类的过程，继承让变化中的软件系统有了一定的延续性，同时继承也是封装程序中可变因素的重要手段。子类继承父类属性(静态特征)和方法(动态特征)，继承必须遵循封装当中的控制访问（对父类和方法的复用）

3. 封装：**封装是把过程和数据包围起来，对数据的访问只能通过已定义的界面。封装隐藏了类的内部实现机制，从而可以在不影响使用者的前提下改变类的内部结构，同时保护了数据。**

   ![20190122202121761](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210831184833.png)

4. 多态性：多态是同一个行为具有多个不同表现形式或形态的能力。

   **优点**： 消除类型之间的耦合关系、可替换性、可扩充性、接口性、灵活性、简化性

   - 重载：同一个动作作用在同一个对象上拥有不同的解释 overload

   - 重写：同一个动作作用在不同的对象上拥有不同的解释 override

   **存在的必要条件**：继承、重写、父类引用指向子类对象：**Parent p = new Child();**

   **实现方式**：重写、接口、抽象类和抽象方法

   **多态有什么好处？**

   1. 应用程序不必为每一个派生类编写功能调用，只需要对抽象基类进行处理即可。大大提高程序的可复用性。//继承 
   2.  派生类的功能可以被基类的方法或引用变量所调用，这叫向后兼容，可以提高可扩充性和可维护性。 //多态的真正作用，

# 5、面向对象与面向过程区别

- 面向过程：分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了；
  - 优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源，比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发，性能是最重要的因素。 
  - 缺点：没有面向对象易维护、易复用、易扩展

- 面向对象：把构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为。
  - 优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护 
  - 缺点：性能比面向过程低

# 6、Java类加载机制

**过程**

java编译器将 .java 文件编译成扩展名为 .class 的文件。.class 文件中保存着java转换后，虚拟机将要执行的指令。当需要某个类的时候，java虚拟机会加载 .class 文件，并创建对应的class对象，将class文件加载到虚拟机的内存，这个过程被称为类的加载。

![20190302102035338](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210831203339.png)

1. 加载
   1. 通过类的全限定名来获取定义此类的二进制字节流
   2. 将这个类字节流代表的静态存储结构转为方法区的运行时数据结构
   3. 在堆中生成一个代表此类的java.lang.Class对象，作为访问方法区这些数据结构的入口。
2. 验证
   目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身的安全。

   - 文件格式验证：基于字节流验证。

   - 元数据验证：基于方法区的存储结构验证。

   - 字节码验证：基于方法区的存储结构验证。

   - 符号引用验证：基于方法区的存储结构验证。
3. 准备
   **为类变量（static修饰的字段变量）分配内存并且设置该类变量的初始值**，（如static int i = 5 这里只是将 i 赋值为0，在初始化的阶段再把 i 赋值为5)，这里不包含final修饰的static ，因为final在编译的时候就已经分配了。这里不会为实例变量分配初始化，类变量会分配在方法区中，实例变量会随着对象分配到Java堆中。
4. 解析
   这里主要的任务是把常量池中的符号引用替换成直接引用

   - 类或接口的解析

   - 字段解析

   - 类方法解析

   - 接口方法解析
5. 初始化
   如果该类具有父类就进行对父类进行初始化，执行其静态初始化器（静态代码块）和静态初始化成员变量。

**父类的静态字段——>父类静态代码块——>子类静态字段——>子类静态代码块——>**

**父类成员变量（非静态字段）——>父类非静态代码块——>父类构造器——>子类成员变量——>子类非静态代码块——>子类构造器**

# 7、IOC-Bean的注入过程

Bean的定义：**三个步骤**

​    	一旦找到了资源，他就开始解析，并将信息保存起来。注意此时还没有初始化Bean，也就没有Bean实例，它有的仅仅是Bean的定义。然后将Bean定义发送到IOC容器中。**三个步骤**

1. 第一步，资源定位，就是Spring根据我们定义的注解(@Component)或者是XML，找到相应的类。
2. 找到了资源就开始解析，并将定义的信息保存起来，将定位到的资源保存到BeanDefinition中。并没有初始化bean。
3. 然后将BeanDefinition的定义发布到SpringIoc的容器中，这个过程就是将前面的BeanDefition保存到HashMap中的过程。

![new_20150724202916051](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210911143516.jpeg)

> 动态注入：

基于ApplicationContextAware来获取ApplicationContext的引用，然后基于ApplicationContext进行对象的动态获取。

实现代码如下：

```java
@Component
public class SpringContextUtil implements ApplicationContextAware {
	// Spring应用上下文环境
	private static ApplicationContext applicationContext;
    
    // 实现ApplicationContextAware接口的回调方法，设置上下文环境
    public void setApplicationContext(ApplicationContext applicationContext) {
    	SpringContextUtil.applicationContext = applicationContext;
    }
    public static ApplicationContext getApplicationContext() {
    	return applicationContext;
    }

	//获取对象
	public static Object getBean(String name) throws BeansException {
    	return applicationContext.getBean(name);
    }
}
// 接在代码中动态获取所需要的Bean实例
BeanType bean = SpringContextUtil.getBean("beanName")
```

# 8、Spring-bean的循环依赖以及解决方式

> 什么是循环依赖？

循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。如下图：

![20170912082357749](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210901200754.jpeg)

注意，这里不是函数的循环调用，是对象的相互依赖关系。循环调用其实就是一个死循环，除非有终结条件。

Spring中循环依赖场景有：
（1）构造器的 循环依赖
（2）field属性的循环依赖。

> 怎么检测是否存在循环依赖

检测循环依赖相对比较容易，Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明了循环依赖了。

>Spring怎么解决循环依赖

 ①：构造器的循环依赖。【这个Spring解决不了】

 ②【setter循环依赖】field属性的循环依赖【setter方式 单例，默认方式-->通过递归方法找出当前Bean所依赖的Bean，然后提前缓存【会放入Cache中】起来。通过提前暴露 -->暴露一个exposedObject用于返回提前暴露的Bean。】

https://blog.csdn.net/lkforce/article/details/97183065

| **步骤** |                           **操作**                           |                     **三层列表中的内容**                     |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    1     |                       开始初始化对象A                        | singletonFactories：<br>earlySingletonObjects：<br>singletonObjects： |
|    2     |            调用A的构造，把A放入singletonFactories            | singletonFactories：A<br/>earlySingletonObjects：<br/>singletonObjects： |
|    3     |               开始注入A的依赖，发现A依赖对象B                | singletonFactories：A<br/>earlySingletonObjects：<br/>singletonObjects： |
|    4     |                       开始初始化对象B                        | singletonFactories：A,B<br/>earlySingletonObjects：<br/>singletonObjects： |
|    5     |            调用B的构造，把B放入singletonFactories            | singletonFactories：A,B<br/>earlySingletonObjects：<br/>singletonObjects： |
|    6     |               开始注入B的依赖，发现B依赖对象A                | singletonFactories：A,B<br/>earlySingletonObjects：<br/>singletonObjects： |
|    7     | 开始初始化对象A，发现A在singletonFactories里有，则直接获取A，把A放入earlySingletonObjects，把A从singletonFactories删除 | singletonFactories：B<br/>earlySingletonObjects：A<br/>singletonObjects： |
|    8     |                     对象B的依赖注入完成                      | singletonFactories：B<br/>earlySingletonObjects：A<br/>singletonObjects： |
|    9     | 对象B创建完成，把B放入singletonObjects，把B从earlySingletonObjects和singletonFactories中删除 | singletonFactories：<br/>earlySingletonObjects：A<br/>singletonObjects：B |
|    10    |       对象B注入给A，继续注入A的其他依赖，直到A注入完成       | singletonFactories：<br/>earlySingletonObjects：A<br/>singletonObjects：B |
|    11    | 对象A创建完成，把A放入singletonObjects，把A从earlySingletonObjects和singletonFactories中删除 | singletonFactories：<br/>earlySingletonObjects：<br/>singletonObjects：A,B |
|    12    |           循环依赖处理结束，A和B都初始化和注入完成           | singletonFactories：<br/>earlySingletonObjects：<br/>singletonObjects：A,B |

# 9、int与Integer区别

- Integer是int的包装类，int则是java的一种基本的数据类型；
- Integer变量必须实例化之后才能使用，而int变量不需要实例化；
- Integer实际是对象的引用，当new一个Integer时，实际上生成一个指针指向对象，而int则直接存储数值
- Integer的默认值是null，而int的默认值是0，不能为null。
- 包装类型需要占用更大的空间

# 10、接口（Interface）与抽象类（Abstract Class）的区别？

|      区别      |                             接口                             |                            抽象类                            |
| :------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|      定义      |                       interface关键字                        |                        abstract class                        |
|   访问修饰符   |                          只有public                          |                  public、protected和default                  |
|      继承      |     接口只可以继承接口(一个或多个)；子类可以实现多个接口     | 抽象类可以继承一个类和实现多个接口；子类只可以继承一个抽象类 |
| 是否有构造函数 |                              否                              |                              是                              |
|      实现      | 子类使用关键字**implements**来实现接口，需要实现接口中声明的**所有方法** | 子类使用**extends**关键字来继承抽象类。子类如果不是抽象类，需要实现抽象类中声明的**所有抽象方法** |
|    方法实现    | 接口完全是抽象的，没构造方法，且方法都是抽象的，不存在方法的实现 |           可定义构造方法，可以有抽象方法和具体方法           |
|      作用      |       为了把程序模块进行固化的契约，是为了**降低偶合**       |              了把相同的东西提取出来，即**重用**              |

# 11、重写与重载

> **重写**

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。**即外壳不变，核心重写！**

重写的好处在于子类可以根据需要，定义特定于自己的行为。 也就是说子类能够根据需要实现父类的方法。

重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常。例如： 父类的一个方法申明了一个检查异常 IOException，但是在重写这个方法的时候不能抛出 Exception 异常，因为 Exception 是 IOException 的父类，只能抛出 IOException 的子类异常。

在面向对象原则里，重写意味着可以重写任何现有方法。实例如下：

- 参数列表与被重写方法的参数列表必须完全相同。
- 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的派生类（java5 及更早版本返回类型要一样，java7 及更高版本可以不同）。
- **访问权限不能比父类中被重写的方法的访问权限更低。例如：如果父类的一个方法被声明为 public，那么在子类中重写该方法就不能声明为 protected。**
- 父类的成员方法只能被它的子类重写。
- 声明为 final 的方法不能被重写。
- 声明为 static 的方法不能被重写，但是能够被再次声明。
- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为 private 和 final 的方法。
- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为 public 和 protected 的非 final 方法。
- 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。但是，重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以。
- 构造方法不能被重写。
- 如果不能继承一个类，则不能重写该类的方法。

> **重载**

- **被重载的方法必须改变参数列表(参数个数或类型不一样)；**
- **被重载的方法可以改变返回类型**；
- **被重载的方法可以改变访问修饰符**；
- **被重载的方法可以声明新的或更广的检查异常**；
- **方法能够在同一个类中或者在一个子类中被重载**。
- **无法以返回值类型作为重载函数的区分标准。**

|  区别点  |                             重载                             |                             重写                             |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 参数列表 |                           必须修改                           |                         一定不能修改                         |
| 返回类型 |                           可以修改                           |                         一定不能修改                         |
|   异常   |                           可以修改                           |        可以减少或删除，一定不能抛出新的或者更广的异常        |
|   访问   |                           可以修改                           |            一定不能做更严格的限制（可以降低限制）            |
|  类区别  | **一个类**中定义了多个方法名相同,而他们的参数的数量不同或数量相同而类型和次序不同,则称为方法的重载 | 在**子类**存在方法与**父类**的方法的名字相同,而且参数的个数与类型一样,返回值也一样的方法,就称为重写(Overriding)。 |
| 多态表现 |                           一个类中                           |                  子类与父类的一种多态性表现                  |

# 12、常见加密算法及实现

**对称加密**：采用单钥密码系统的加密方法，同一个密钥可以同时**用作信息的加密和解密**，这种加密方法称为对称加密，也称为单密钥加密。

**非对称加密**：如果用公开密钥对数据进行加密，**只有用对应的私有密钥才能解密**。 如果用私有密钥对数据进行加密，只有用对应的公开密钥才能解密。

**MD5**：MD5用的是 **哈希函数**，它的典型应用是对一段信息产生 **信息摘要**，以 **防止被篡改**。严格来说，MD5不是一种 **加密算法** 而是 **摘要算法**。无论是多长的输入，MD5 都会输出长度为 `128bits` 的一个串 (通常用 `16` **进制** 表示为 `32` 个字符)。

**SHA1算法**：SHA1 是和 MD5 一样流行的 消息摘要算法，然而 SHA1 比 MD5 的 安全性更强。对于长度小于 2 ^ 64 位的消息，SHA1 会产生一个 160 位的 消息摘要。基于 MD5、SHA1 的信息摘要特性以及 不可逆 (一般而言)，可以被应用在检查 文件完整性 以及 数字签名 等场景。

**HMAC算法**：HMAC 是密钥相关的 哈希运算消息认证码（Hash-based Message Authentication Code），HMAC 运算利用 哈希算法 (MD5、SHA1 等)，以 一个密钥 和 一个消息 为输入，生成一个 消息摘要 作为 输出。HMAC 发送方 和 接收方 都有的 key 进行计算，而没有这把 key 的第三方，则是 无法计算 出正确的 散列值的，这样就可以 防止数据被篡改。

# 13、JDK1.8新特性

- **Lambda表达式**
- **Stream API**
- **新时间日期API**
- **Base64**
- **Optional 类**
- **方法引用 "::"**
- **默认方法（接口修饰符加了default后不需要实现）**
- **函数式接口**

> **Lambda表达式**

lambda表达式本质上是一段匿名内部类，也可以是一段可以传递的代码。

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

```java
  //匿名内部类
  Comparator<Integer> cpt = new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
          return Integer.compare(o1,o2);
      }
  };

  TreeSet<Integer> set = new TreeSet<>(cpt);

  System.out.println("=========================");

  //使用lambda表达式
  Comparator<Integer> cpt2 = (x,y) -> Integer.compare(x,y);
  TreeSet<Integer> set2 = new TreeSet<>(cpt2);
```

> **Stream API**

- 流在管道中传输， 并且可以在管道的节点上进行处理， 比如**筛选， 排序，聚合**等。

- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

**生成流**

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

**forEach**：Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

```java
List<Integer> list1 = new ArrayList<>();
list1.add(1);
list1.add(3);
list1.add(2);
list1.stream().forEach(System.out::println);
```

**map**：map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```

**filter**：filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
```

**limit**：limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 2条数据

```java
List<Integer> list1 = new ArrayList<>();
list1.add(1);
list1.add(3);
list1.add(2);
list1.stream().limit(2).forEach(System.out::println);
```

**sorted**：sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序

```java
List<Integer> list1 = new ArrayList<>();
list1.add(1);
list1.add(3);
list1.add(2);
// 降序
list1.stream().sorted(Comparator.reverseOrder()).forEach(System.out::println);
// 升序
list1.stream().sorted().forEach(System.out::println);
// 根据对象里面的某个属性排序-降序
list.stream().sorted(Comparator.comparing(Student::getAge).reversed()).collect(Collectors.toList());
// 根据对象里面的某个属性排序-升序
list.stream().sorted(Comparator.comparing(Student::getAge)).collect(Collectors.toList());
```

**IntSummaryStatistics**：一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

> **新时间日期API**

在旧版的 Java 中，日期时间 API 存在诸多问题，其中有：

- **非线程安全** − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
- **设计很差** − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
- **时区处理麻烦** − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

Java 8 在 **java.time** 包下提供了很多新的 API。以下为两个比较重要的 API：

- **Local(本地)** − 简化了日期时间的处理，没有时区的问题。
- **Zoned(时区)** − 通过制定的时区处理日期时间。

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

![20210716092541](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210716092824.jpg)

>**Base64**（Encoder编码、Decoder解码）

在Java 8中，Base64编码已经成为Java类库的标准。

Java 8 内置了 Base64 编码的编码器和解码器。

Base64工具类提供了一套静态方法获取下面三种BASE64编解码器：

- **基本：**输出被映射到一组字符A-Za-z0-9+/，编码不添加任何行标，输出的解码仅支持A-Za-z0-9+/。
- **URL：**输出映射到一组字符A-Za-z0-9+_，输出是URL和文件。
- **MIME：**输出隐射到MIME友好格式。输出每行不超过76字符，并且使用'\r'并跟随'\n'作为分割。编码输出最后没有行分割。

> **Optional 类**

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

```java
import java.util.Optional;
 
public class Java8Tester {
   public static void main(String args[]){
   
      Java8Tester java8Tester = new Java8Tester();
      Integer value1 = null;
      Integer value2 = new Integer(10);
        
      // Optional.ofNullable - 允许传递为 null 参数
      Optional<Integer> a = Optional.ofNullable(value1);
        
      // Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
      Optional<Integer> b = Optional.of(value2);
      System.out.println(java8Tester.sum(a,b));
   }
    
   public Integer sum(Optional<Integer> a, Optional<Integer> b){
    
      // Optional.isPresent - 判断值是否存在
        
      System.out.println("第一个参数值存在: " + a.isPresent());
      System.out.println("第二个参数值存在: " + b.isPresent());
        
      // Optional.orElse - 如果值存在，返回它，否则返回默认值
      Integer value1 = a.orElse(new Integer(0));
        
      //Optional.get - 获取值，值需要存在
      Integer value2 = b.get();
      return value1 + value2;
   }
}
$ javac Java8Tester.java 
$ java Java8Tester
第一个参数值存在: false
第二个参数值存在: true
10
```

> **方法引用**

方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

方法引用使用一对冒号 **::** 。

```java
import java.util.List;
import java.util.ArrayList;
 
public class Java8Tester {
   public static void main(String args[]){
      List<String> names = new ArrayList();
        
      names.add("Google");
      names.add("Runoob");
      names.add("Taobao");
      names.add("Baidu");
      names.add("Sina");
        
      names.forEach(System.out::println);
   }
}
```

>**默认方法**

Java 8 新增了接口的默认方法。

简单说，默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法。

我们只需在方法名前面加个 **default** 关键字即可实现默认方法。

```java
public class Test implements Vehicle {
    // 不需要进行实现
}
interface Vehicle {
    default void print() {
        System.out.println("我是一辆车!");
    }
    static void blowHorn() {
        System.out.println("按喇叭!!!");
    }
}
```

> **函数式接口**

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。

函数式接口可以被隐式转换为 lambda 表达式。

Lambda 表达式和方法引用（实际上也可认为是Lambda表达式）上。

# 14、二叉树，平衡二叉树，红黑树，特点

> **二叉树binary tree是指每个节点最多含有两个子树的树结构。**

1. 所有节点最多拥有两个子节点
2. 左子树的键值小于根的键值，右子树的键值大于根的键值。
3. 二叉树第i层上的结点数目最多为 2{i-1} (i≥1)
4. 深度为k的二叉树至多有2{k}-1个结点(k≥1)
5. 在任意一棵二叉树中，若终端结点的个数为n0，度为2的结点数为n2，则n0=n2+1
6. 包含n个结点的二叉树的高度至少为log2 (n+1)

> **平衡二叉树**

1. 符合二叉树的条件下
2. 任何节点的两个子树的高度最大差为1

> **红黑树**

**特征**

1. 每个节点或者是黑色，或者是红色。
2. 根节点是黑色。
3. 每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
4. 如果一个节点是红色的，则它的子节点必须是黑色的。
5. 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

**为什么能保证平衡？**

三种操作：==左旋、右旋和变色==。

==右旋==：以某个结点作为支点(旋转结点)，其左子结点变为旋转结点的父结点，左子结点的右子结点变为旋转结点的左子结点，右子结点保持不变。（**右旋**只影响旋转结点和其**左子树**的结构，把左子树的结点往右子树挪了）

![20210716092541](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210924103216.gif)

==左旋==：以某个结点作为支点(旋转结点)，其右子结点变为旋转结点的父结点，右子结点的左子结点变为旋转结点的右子结点，左子结点保持不变。（**左旋**只影响旋转结点和其**右子树**的结构，把右子树的结点往左子树挪了）

![20210716092541](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210924103200.gif)

==变色==：结点的颜色由红变黑或由黑变红。

> 过程

**问题描述：逐个增加【10 70 32 34 13 56 30 66 21 3 62 4】 到红黑树中？**

![1765333-20200831141054778-475231711](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210924115210.png)

![1765333-20200831141102217-1098310422](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210924115226.png)

# 15、创建对象几种方式

|           使用new关键字            |  } → 调用了构造函数  |
| :--------------------------------: | :------------------: |
|    使用Class类的newInstance方法    |  } → 调用了构造函数  |
| 使用Constructor类的newInstance方法 |  } → 调用了构造函数  |
|           使用clone方法            | } → 没有调用构造函数 |
|            使用反序列化            | } → 没有调用构造函数 |

# 16、序列化作用

序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

> 作用

1. 对象随着程序的运行而被创建，然后在不可达时被回收，生命周期是短暂的。但是如果我们想长久地把对象的内容保存起来怎么办呢？把它转化为字节序列**保存在存储介质上**即可。那就需要序列化。
2. 所有可在**网络上传输**的对象都必须是可序列化的，比如RMI（remote method invoke,即远程方法调用），传入的参数或返回的对象都是可序列化的，否则会出错；所有需要保存到磁盘的java对象都必须是可序列化的。通常建议：程序创建的每个JavaBean类都实现Serializeable接口
3. **进程间传递对象**，Android是基于Linux系统，不同进程之间的java对象是无法传输，所以我们此处要对对象进行序列化，从而实现对象在 **应用程序进程 和 ActivityManagerService进程** 之间传输。

> 序列化的实现

1. ==实现Sericalizable接口==，实现Sericalizable接口的时候还要写一个==serialVersionUID==，这个是版本号**，JVM会把传进来的字节流中的serialVersionUID与本地实体类中的serialVersionUID进行比较，如果相同则认为是一致的，便可以进行反序列化，否则就会报序列化版本不一致的异常。**如果实现接口的时候，没有给定UID，就会使用默认的UID，当使用默认的UID的时候，jvm每次编译的时候会生成一个UID，当后面程序改了一些代码，再次编译的时候会生成不同的UID，会导致反序列化失败！所以在实现Sericalizable接口的时候，我们自己给定一个固定的UID值，这样就能保证编译完再 反序列化的时候的版本一致性。所以j能不能成功反序列化，就是看对象中的UID和实体中UID是否一致。
2. ==实现Externalnalizable接口==，在类中实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法，在方法中定义类对象自定义的序列化和反序列化操作。这样通过对象输出流和对象输入流的输入输出方法序列化和反序列化对象时会自动调用类中定义的readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法。

# 17、面向对象的五大原则

**1、模块化**

面向对象开发方法很自然地支持了把系统分解成模块的设计原则：对象就是模块。它是把数据结构和操作这些数据的方法紧密地结合在一起所构成的模块。分解系统为一组具有高内聚和松耦合的模块是模块化的属性。

**2、抽象**

面向对象方法不仅支持过程抽象，而且支持数据抽象。

**3、信息隐藏**

在面向对象方法中，信息隐藏通过对象的封装性来实现。

**4、低耦合**

在面向对象方法中，对象是最基本的模块，因此，耦合主要指不同对象之间相互关联的紧密程度。低耦合是设计的一个重要标准，因为这有助于使得系统中某一部分的变化对其它部分的影响降到最低程度。

**5、高内聚**

操作内聚；类内聚；具体内聚。

# 18、八大数据结构

数组、栈、队列、链表、树、散列表、堆、图

1、**数组**

​		数组是可以再==内存中连续存储多个元素的结构，在内存中的分配也是连续的==，数组中的元素通过数组下标进行访问，数组下标从0开始。

2、**栈**

​		栈是一种特殊的线性表，仅能在线性表的一端操作，栈顶允许操作，栈底不允许操作。 栈的特点是：==先进后出==，或者说是后进先出，从栈顶放入元素的操作叫入栈，取出元素叫出栈。

![20180903195046375](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109282241912.jpeg)

3、**队列**

​		队列与栈一样，也是一种线性表，不同的是，队列可以在一端添加元素，在另一端取出元素，也就是：==先进先出==。从一端放入元素的操作称为入队，取出元素为出队。

4、**链表**

​		链表是==物理存储单元上非连续的、非顺序的存储结构==，数据元素的逻辑顺序是通过链表的指针地址实现，每个元素包含两个结点，一个是存储元素的数据域 (内存空间)，另一个是指向下一个结点地址的指针域。根据指针的指向，链表能形成不同的结构，例如单链表，双向链表，循环链表等。

5、**树**

​		树是一种数据结构，它是由n（n>=1）个有限节点组成一个具有层次关系的集合。把它叫做 “树” 是因为它看起来像一棵倒挂的树，也就是说它是根朝上，而叶朝下的。它具有以下的特点：

- 每个节点有零个或多个子节点；
- 没有父节点的节点称为根节点；
- 每一个非根节点有且只有一个父节点；
- 除了根节点外，每个子节点可以分为多个不相交的子树；

6、**散列表**

​		散列表，也叫哈希表，是根据关键码和值 (key和value) 直接进行访问的数据结构，通过key和value来映射到集合中的一个位置，这样就可以很快找到集合中的对应元素。

![20180903195448444](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109282245459.png)

7、**堆**

堆是一种比较特殊的数据结构，可以被看做一棵树的数组对象，具有以下的性质：

- 堆中某个节点的值总是不大于或不小于其父节点的值；
- 堆总是一棵完全二叉树。

将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。常见的堆有二叉堆、斐波那契堆等。

![20180903195606329](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109282246796.png)

8、**图**

图是由结点的有穷集合V和边的集合E组成。其中，为了与树形结构加以区别，在图结构中常常将结点称为顶点，边是顶点的有序偶对，若两个顶点之间存在一条边，就表示这两个顶点具有相邻关系。

按照顶点指向的方向可分为无向图和有向图：

<center class="half">
    <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109282247453.png" width="300"/>
    <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109282247388.png" width="300"/>
</center>
# 19、Error与Exception的区别

|                Error                 |                          Exception                           |
| :----------------------------------: | :----------------------------------------------------------: |
|         都是继承Throwable类          |                     都是继承Throwable类                      |
| 是系统中的错误，程序编译时出现的错误 |          表示程序可以处理的异常，可以捕获且可能恢复          |
|                                      | Exception又分为两类：<br>CheckedException：（**编译时异常**） 需要用try——catch显示的捕获，对于可恢复的异常使用CheckedException。<br>UnCheckedException（RuntimeException）：（**运行时异常**）不需要捕获，对于程序错误（不可恢复）的异常使用RuntimeException。 |

# 20、面向切面（Spring Aop）、拦截器、过滤器的区别

http://www.javashuo.com/article/p-rnhpzjrq-dz.html

> 过滤器

==过滤器拦截的是URL后端==

Spring中自定义过滤器（Filter）通常**只有一个方法**，返回值是void，当请求到达web容器时，会探测当前请求地址是否配置有过滤器，有则调用该过滤器的方法（可能会有多个过滤器），而后才调用真实的业务逻辑，至此过滤器任务完成。过滤器并无定义业务逻辑执行前、后等，仅仅是请求到达就执行。svg

特别注意：过滤器方法的入参有request，response，FilterChain，其中FilterChain是过滤器链，使用比较简单，而request，response则关联到请求流程，所以能够对请求参数作过滤和修改，同时FilterChain过滤链执行完，而且完成业务流程后，会返回到过滤器，此时也能够对请求的返回数据作处理。post

>拦截器

==拦截器拦截的是URL性能==

**拦截器有三个方法**，相对于过滤器更加细致，有被拦截逻辑执行前、后等。Spring中拦截器有三个方法：**preHandle，postHandle，afterCompletion**。分别表示以下日志

public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o)表示被拦截的URL对应的方法执行前的自定义处理xml

public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView)表示此时还未将modelAndView进行渲染，被拦截的URL对应的方法执行后的自定义处理，。blog

public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e)表示此时modelAndView已被渲染，执行拦截器的自定义处理。图片

> AOP（面向切面）

==面向切面拦截的是类的元数据（包、类、方法名、参数等）==

相对于拦截器更加细致，并且很是灵活，**拦截器只能针对URL作拦截**，而**AOP针对具体的代码，可以实现更加复杂的业务逻辑。**

三者使用场景 三者功能相似，但各有优点，从过滤器–》拦截器–》切面，拦截规则愈来愈细致，==执行顺序依次是过滤器、拦截器、切面==。通常状况下数据被过滤的时机越早对服务的性能影响越小，所以咱们在编写==相对比较公用的代码时，优先考虑过滤器，而后是拦截器，最后是aop==。好比权限校验，通常状况下，全部的请求都须要作登录校验，此时就应该使用过滤器在最顶层作校验；日志记录，通常日志只会针对部分逻辑作日志记录，并且牵扯到业务逻辑完成先后的日志记录，所以使用过滤器不能细致地划分模块，此时应该考虑拦截器，然而拦截器也是依据URL作规则匹配，所以相对来讲不够细致，所以咱们会考虑到使用AOP实现，AOP能够针对代码的方法级别作拦截，很适合日志功能。

# 21、动态代理和静态代理

https://blog.csdn.net/fox_bert/article/details/80891148

代理模式是常用的Java设计模式，它的特征是**代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等**。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。按照代理类的创建时期，代理类可分为两种。

| 静态代理 | 由程序员创建或由特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了静态代理通常只代理一个类静态代理事先知道要代理的是什么 |
| :------- | ------------------------------------------------------------ |
| 动态代理 | 在程序运行时，运用反射机制动态创建而成动态代理是代理一个接口下的多个实现类动态代理不知道要代理什么东西，只有在运行时才知道 |

- **动态代理是实现JDK里的InvocationHandler接口的invoke方法，但注意的是代理的是接口，也就是你的业务类必须要实现接口，通过Proxy里的newProxyInstance得到代理对象。**
- **还有一种动态代理CGLIB，代理的是类，不需要业务类继承接口，通过派生的子类来实现代理。通过在运行时，动态修改字节码达到修改类的目的。**

由Proxy类的静态方法创建的动态代理类具有以下特点：

-  动态代理类是public、final和非抽象类型的；
-  动态代理类继承了java.lang.reflect.Proxy类；
-  动态代理类的名字以“$Proxy”开头；
-  动态代理类实现getProxyClass()和newProxyInstance()方法中参数interfaces指定的所有接口；

# 22、Final特点

1、final修饰的类不可以被继承，但是可以继承其他类。

2、final修饰的方法不可以被覆盖，但是可以被继承使用。

3、final修饰的基本数据类型变量称为常量，只能赋值一次。

4、final修饰的引用数据类型变量值为地址值，地址值不能改变，但是地址内的属性对象可以该改变。

5、final修饰的成员变量，需要在创建对象前赋值，否则报错（定义时直接赋值，如果在构造方法赋值，多个构造方法的均需赋值）

# 23、泛型原理

https://www.cnblogs.com/robothy/p/13949788.html

​		Java的泛型是==伪泛型==。在编译期间，所有的泛型信息都会被擦除掉。正确理解泛型概念的首要前提是理解类型擦出（type erasure）。

 		==Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。==

​		泛型本质是将数据类型参数化，它通过擦除的方式来实现。声明了泛型的 .java 源代码，在编译生成 .class 文件之后，泛型相关的信息就消失了。源代码中泛型相关的信息，就是提供给编译器用的。泛型信息对 Java 编译器可以见，对 Java 虚拟机不可见。

Java 编译器通过如下方式实现擦除：

- 用 Object 或者界定类型替代泛型，产生的字节码中只包含了原始的类，接口和方法；
- 在恰当的位置插入强制转换代码来确保类型安全；
- 在继承了泛型类或接口的类中插入桥接方法来保留多态性。

