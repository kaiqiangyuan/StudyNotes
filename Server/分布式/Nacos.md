[toc]

### 1、简介

> 动态配置服务

动态配置服务让您能够以中心化、外部化和动态化的方式管理所有环境的配置。动态配置消除了配置变更时重新部署应用和服务的需要。配置中心化管理让实现无状态服务更简单，也让按需弹性扩展服务更容易。

> 服务发现及管理

动态服务发现对以服务为中心的（例如微服务和云原生）应用架构方式非常关键。Nacos支持DNS-Based和RPC-Based（Dubbo、gRPC）模式的服务发现。Nacos也提供实时健康检查，以防止将请求发往不健康的主机或服务实例。借助Nacos，您可以更容易地为您的服务实现断路器。

> 动态DNS服务

通过支持权重路由，动态DNS服务能让您轻松实现中间层负载均衡、更灵活的路由策略、流量控制以及简单数据中心内网的简单DNS解析服务。动态DNS服务还能让您更容易地实现以DNS协议为基础的服务发现，以消除耦合到厂商私有服务发现API上的风险。

### 2、安装

[官网](https://nacos.io/zh-cn/docs/quick-start.html) [^1]

1. [下载地址](https://github.com/alibaba/nacos/releases) [^2]

2. 启动

   ```sh
   cd nacos/bin
   bash startup.sh -m standalone(ubuntu，standalone非集群模式启动)
   ```

   若提示内存不足，则修改启动内存 `bin/startup.sh` 

   * Xms 是指设定程序启动时占用内存大小

   * Xmx 是指设定程序运行期间最大可占用的内存大小

   * -Xmn 新生代的大小

   ![image-20210609192625760](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210609192625.png)

3. 使用 http://localhost:8848/nacos 访问，账号密码初始为nacos

   ![image-20210609195125981](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210609195126.png)

4. 修改为mysql数据库，数据库版本需为5版本（application.properties文件）

   ![image-20210624134345889](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624134352.png)

   创建nacos数据库，将config文件下的==nacos-mysql.sql==文件导入到nacos数据库中，修改为自己myslq的用户名和密码

   ![image-20210624134538608](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624134538.png)

5. 重启nacos服务

### 3、配置

> 命令空间，隔离多个环境

![image-20210624211408315](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624211408.png)

在不同的命令空间中创建相应的配置信息

![image-20210624211535233](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624211535.png)

> 回滚

在历史版本中可以进行配置版本的回滚操作

![image-20210624211702040](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624211702.png)

> 无需账号密码登录

注释掉application.properties配置文件中的这一行

![image-20210624215542651](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624215542.png)

加上下面后重启服务

```properties
spring.security.enabled=false
management.security=false
security.basic.enabled=false
nacos.security.ignore.urls=/**
```

### 4、Springboot中使用

> 获取配置，第一种方式

导入依赖

```xml
		<dependency>
			<groupId>com.alibaba.nacos</groupId>
			<artifactId>nacos-client</artifactId>
			<version>2.0.1</version>
		</dependency>
```

```java
package com.yuankaiqiang.nacos;

import java.util.Properties;
import java.util.concurrent.Executor;

import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;

public class NacosConifig {

	public static void main(String[] args) throws NacosException, InterruptedException {
		String serverAddr = "114.55.26.230:8848";
		String dataId = "spiritmark.nacos.demo.properties";
		String group = "DEFAULT_GROUP";
//		String namespace = "";
		Properties properties = new Properties();
		properties.put("serverAddr", serverAddr);
//		properties.put("namespace", namespace);
		
		ConfigService configService = NacosFactory.createConfigService(properties);
		String content = configService.getConfig(dataId, group, 5000);
		System.out.println(content);
		
        //添加配置监听
        configService.addListener(dataId, group, new Listener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                System.out.println("receive:" + configInfo);
            }

            @Override
            public Executor getExecutor() {
                return null;
            }
        });
        
//        //发布配置
//        boolean isPublishOk = configService.publishConfig(dataId, group, "content");
//        System.out.println(isPublishOk);
//
//        //获取配置
//        Thread.sleep(3000);
//        content = configService.getConfig(dataId, group, 5000);
//        System.out.println(content);
//
//        //删除配置
//        boolean isRemoveOk = configService.removeConfig(dataId, group);
//        System.out.println(isRemoveOk);
//        Thread.sleep(3000);
//
//        //再次获取配置
//        content = configService.getConfig(dataId, group, 5000);
//        System.out.println(content);
//        Thread.sleep(300000);       
	}
}
```

> 获取配置，第二种方式

导入依赖（这个依赖包括第一种方式的jar包）

```xml
<!-- nacos-配置管理功能依赖 -->
<dependency>
	<groupId>com.alibaba.boot</groupId>
  <artifactId>nacos-config-spring-boot-starter</artifactId>					<version>0.2.1</version>
</dependency>
```

启动类上增加@NacosPropertySource，多个配置增加多个注解，==只支持properties==

![image-20210625190446948](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625190446.png)

```java
@RestController
@RequestMapping("/user")
public class Controller {

	@NacosValue(value = "${username:null}", autoRefreshed = true)
	private String username;

	@RequestMapping(value = "/getUser")
	public String getUser() {
		return username;
	}
}
```

原配置

![image-20210625190649606](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625190649.png)

![image-20210625190705864](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625190705.png)

修改为zhangsan后

![image-20210625190803773](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625190803.png)

![image-20210625190816255](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625190816.png)

> 注册服务

启动项目后

```properties
server.port = 8891
spring.application.name=provider-server
nacos.discovery.server-addr=114.55.26.230:8848
nacos.config.server-addr=114.55.26.230:8848
```

```java
@Configuration
public class NacosRegisterConfiguration {

	@Value("${server.port}")
	private int serverPort;

	@Value("${spring.application.name}")
	private String applicationName;

	@NacosInjected
	private NamingService namingService;

	@PostConstruct
	public void registerInstance() throws NacosException {
		namingService.registerInstance(applicationName, "127.0.0.1", serverPort, "DEFAULT");
	}	
}
```

服务列表中增加了一个服务

![image-20210625204010182](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625204010.png)

![image-20210625204030596](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625204030.png)

> 订阅注册服务者

订阅者订阅某个服务

![image-20210625224619567](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625224619.png)

![image-20210625224553199](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625224553.png)

### 5、搭建集群

需要依赖mysql

伪分布式（下面这种方式适用1.4.2使用，2.x启动有问题）

1. 复制三份nacos文件夹，重命名为nacos1，nacos2，nacos3，相当于三个集群

![image-20210624224631980](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210624224632.png)

2. 修改每个文件中集群中的端口号，application.properties  中增加 `server.port=8848`

   | Nacos1 | 8848 |
   | :----: | :--: |
   | Nacos2 | 8849 |
   | Nacos3 | 8850 |

3. 修改每个文件，集群配置文件(cluster.conf.example)中修改成对应的ip和端口，因为都是本地所以都为192.168.x.x（使用192），并将文件名称修改为 `cluster.conf`

   ```sh
   #it is ip
   #example
   192.168.185.116:8848
   192.168.185.116:8849
   192.168.185.116:8850
   ```

4. 分别三个集群启动 `bash startup.sh -m cluster` cluster为集群模式启动

5. 查看状态

   8848为leader

   ![image-20210625142014286](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625142014.png)

   关掉8848服务器，leader为8849

   ![image-20210625142207853](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210625142207.png)

> 测试Demo

测试代码：**关注公众号==原棵哈==回复==nacos==获取**

![扫码_搜索联合传播样式-标准色版](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210720080705.png)

### 6、常见问题

```sh
nacos 进程占用
ps -ef | grep nacos
kill -9 xx
```

### 参考网址

[^1]: https://nacos.io/zh-cn/docs/quick-start.html
[^2]: https://github.com/alibaba/nacos/releases











