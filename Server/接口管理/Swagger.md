[toc]

#### 一、简介

Swagger是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。

在项目开发中，根据业务代码自动生成API文档，给前端提供在线测试，自动显示JSON格式，方便了后端与前端的沟通与调试成本。

#### 二、SpringBoot中使用

1. 导入依赖

   ```xml
   		<!-- swagger -->
   		<dependency>
   			<groupId>io.springfox</groupId>
   			<artifactId>springfox-swagger2</artifactId>
   			<version>2.9.2</version>
   		</dependency>
   		<dependency>
   			<groupId>io.springfox</groupId>
   			<artifactId>springfox-swagger-ui</artifactId>
   			<version>2.9.2</version>
   		</dependency>
   ```

2. 创建配置类

   ```java
   @Configuration
   @EnableSwagger2
   ```

   打开：http://localhost:8080/swagger-ui.html

![image-20210602222647769](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602222647.png)

3. 配置类信息

   ```java
   @Configuration
   @EnableSwagger2		//开启swagger2
   public class SwaggerConfig {
   
   	/**
   	 * 创建该API的基本信息（这些基本信息会展现在文档页面中） 访问地址：http://项目实际地址/swagger-ui.html
   	 * 
   	 * @return
   	 */
   	private ApiInfo apiInfo() {
   		// 作者信息
   		Contact contact = new Contact("袁凯强", "https://www.kaiqiang.top/", "2210774904@qq.com");
   
   		return new ApiInfo(
   				"学习笔记", 	//标题
   				"坚持！！！", 	// 描述
   				"v1.0", 					//版本
   				"https://www.kaiqiang.top/", //
   				contact, 	//作者信息
   				"Apache 2.0",
   				"http://www.apache.org/licenses/LICENSE-2.0", new ArrayList<VendorExtension>());
   	}
   
   	/**
   	 * 创建API应用 apiInfo() 增加API相关信息
   	 * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
   	 * 本例采用指定扫描的包路径来定义指定要建立API的目录。
   	 *
   	 * @return
   	 */
   	@Bean
   	public Docket docket(Environment environment) {
   		// 设置要显示的Swagger环境，可配置多个，设置只在dev环境中启动swagger
   		Profiles profiles = Profiles.of("dev");
   		// 判断自己是否处理当前设置的环境中
   		boolean flag = environment.acceptsProfiles(profiles);
   		
   		return new Docket(DocumentationType.SWAGGER_2)
   				.apiInfo(apiInfo())
   				// 分组
   				.groupName("测试1")
   				// 是否启动swagger，false不启动，true启动
   				.enable(flag)
   				.select()
   				// RequestHandlerSelectors配置扫描接口方式
   				// basePackage指定要扫描的包
   				// any()扫描全部,none()都不扫描
   				// withClassAnnotation扫描类上的注解,withClassAnnotation(RestController.class)
   				// withMethodAnnotation扫描方法上的注解,withMethodAnnotation(GetMapping.class)
   				.apis(RequestHandlerSelectors.basePackage("com.yuankaiqiang.rest")) // 这里写的是API接口所在的包位置
   				// 过滤路径 
   				.paths(PathSelectors.any())
   				.build();
   	}
   }
   ```

4. 不同环境中才启动swagger

   Docket中设置

   ```java
   		// 设置要显示的Swagger环境，可配置多个，设置只在dev环境中启动swagger
   		Profiles profiles = Profiles.of("dev");
   		// 判断自己是否处理当前设置的环境中
   		boolean flag = environment.acceptsProfiles(profiles);
   ```

   当使用生成环境中时不启动swagger

   ![image-20210603002632989](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603002633.png)

   ![image-20210603002815693](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603002815.png)

5. 配置API文档的分组

   ```java
   groupName("测试2")
   ```

   可以分别配置多个@Bean

   ![image-20210603003137491](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603003137.png)

   **==不同组可以单独的配置==**

   ![image-20210603003243709](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603003243.png)

   ![image-20210603003313354](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603003313.png)

6. 实体类配置

   ```java
   //等价@Api(value = "用户实体类")
   @ApiModel("用户实体类")		//给生成的文档加注释
   public class User {
   	
   	@ApiModelProperty("用户名")	//给生成的文档加注释
   	private String username;
   	
   	@ApiModelProperty("密码")
   	private String password;
   	
   	public String getUsername() {
   		return username;
   	}
   	public void setUsername(String username) {
   		this.username = username;
   	}
   	public String getPassword() {
   		return password;
   	}
   	public void setPassword(String password) {
   		this.password = password;
   	}
   	
   }
   ```

7. 若想在swaggerui中显示实体信息

   ![image-20210603155333997](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603155334.png)

   需在接口中增加接口返回实体类的信息

   ![image-20210603155404760](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210603155404.png)

8. 接口信息

   ```java
   @RestController
   @RequestMapping("/user")
   @Api(tags = "用户信息管理", description = "提供增删查改") 	// 给类上加注释
   public class UserController {
   	
   	@GetMapping("getUser")
   	public User getUser() {
   		return new User();
   	}
   	
   	@PostMapping("getUserName")
   	@ApiOperation(value = "获取用户名", notes = "通过username获取用户名")		//给方法加注释
   	public String getUserName(@ApiParam(value = "用户名", required = true) @RequestBody String str) {
   		return "用户名为===》"+str;
   	}
   
   	@PostMapping("getUserId/{id}")
   	@ApiOperation(value = "获取Id", notes = "通过用户ID获取")
   	public String findById(@ApiParam(value = "用户ID", required = true) @PathVariable("id") Long id) {
   		return "id为===》" + id;
   	}
   	
   	/**
   	 *  defaultValue 如果本次请求没有携带这个参数，或者参数为空，那么就会启用默认值
   		name 绑定本次参数的名称，要跟URL上面的一样
   		required 这个参数是不是必须的
   		value 跟name一样的作用，是name属性的一个别名
   	*/
   	@PostMapping("getUserId2")
   	@ApiOperation(value = "获取Id2", notes = "通过用户ID获取")
   	public String findById2(@ApiParam(value = "用户ID", required = true) @RequestParam("id") Long id) {
   		return "id为===》" + id;
   	}
   
   }
   ```

   

































