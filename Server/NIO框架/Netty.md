[toc]

### 一、简介

​		Netty是由JBOSS提供的一个java开源框架，现为 Github上的独立项目。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。

​		也就是说，Netty 是一个基于NIO的客户、服务器端的编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户、服务端应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。

​		“快速”和“简单”并不用产生维护性或性能上的问题。Netty 是一个吸收了多种协议（包括FTP、SMTP、HTTP等各种二进制文本协议）的实现经验，并经过相当精心设计的项目。最终，Netty 成功的找到了一种方式，在保证易于开发的同时还保证了其应用的性能，稳定性和伸缩性。

### 二、对比Mina

这两个NIO框架的创作者是同一个人Trustin Lee （韩国人），github地址：https://github.com/trustin

> Netty的优点可以总结如下

1. API使用简单，开发门槛低；

2. 功能强大，预置了==多种编解码==功能，支持==多种主流协议==；

3. 定制能力强，可以通过ChannelHandler对通信框架进行灵活地扩展；

4. ==性能高==，通过与其他业界主流的NIO框架对比，Netty的综合性能最优；

5. ==成熟、稳定==，Netty修复了已经发现的所有JDK NIO BUG，业务开发人员不需要再为NIO的BUG而烦恼；

6. ==社区活跃==，版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会加入；

7. 经历了大规模的商业应用考验，质量得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它已经完全能够满足不同行业的商业应用了。

> 与Mina相比有什么优势？

1. 都是Trustin Lee的作品，Netty更晚；
2. Mina将内核和一些特性的联系过于紧密，使得用户在不需要这些特性的时候无法脱离，相比下性能会有所下降，Netty解决了这个设计问题；
3. Netty的==文档更清晰==，很多Mina的特性在Netty里都有；
4. Netty更新周期更短，新版本的发布比较快；
5. 它们的架构差别不大，==Mina靠apache生存==，而==Netty靠jboss==，和jboss的结合度非常高，Netty有对google protocal buf的支持，有更完整的ioc容器支持(spring,guice,jbossmc和osgi)；
6. Netty比Mina使用起来更简单，Netty里你可以自定义的处理upstream events或/和downstream events，可以使用decoder和encoder来解码和编码发送内容；
7. Netty和Mina在处理UDP时有一些不同，Netty将UDP无连接的特性暴露出来；而Mina对UDP进行了高级层次的抽象，可以把UDP当成”面向连接”的协议，而要Netty做到这一点比较困难；

### 三、优势

1. Reactor线程模型：高性能多线程设计思路
2. Netty中自己定义的channel概念：增强版的NIOchannel
3. ChannelPipeline责任链设计模式：事件处理机制
4. 内存管理：增强型byteBuf缓冲区

### 四、SpringToolSuite4中使用

导入依赖

```xml
		<dependency>
			<groupId>io.netty</groupId>
			<artifactId>netty-all</artifactId>
		</dependency>
```

#### 1、服务端

Netty启动网上很多示例，核心代码如下，主要是两个线程组==bossGroup==，==workGroup==

```java
public class NettyServerRun {

	private static final Logger log = LoggerFactory.getLogger(NettyServerRun.class);

	private final EventLoopGroup bossGroup = new NioEventLoopGroup();
	private final EventLoopGroup workGroup = new NioEventLoopGroup();
	private static Channel channel;

	public ChannelFuture start(InetSocketAddress socketAddress) throws Exception {

		ChannelFuture future = null;

		ServerBootstrap bootstrap = new ServerBootstrap()
		         //第1步  定义两个线程组，用来处理客户端通道的accept和读写事件
		         //bossGroup用来处理accept事件，workGroup用来处理通道的读写事件
		         //bossGroup获取客户端连接，连接接收到之后再将连接转发给workGroup去处理
				.group(bossGroup, workGroup)
				// 第2步  绑定服务端通道
				.channel(NioServerSocketChannel.class)
				// 第3步  处理读写事件，ChannelInitializer是给通道初始化
				.childHandler(new MyServerCodec())
				.localAddress(socketAddress)
				/**
				 * 设置队列大小
		         * 用于构造服务端套接字ServerSocket对象，标识当服务器请求处理线程全满时，用于临时存放已完成三次握手的请求的队列的最大长度。
		         * 用来初始化服务端可连接队列
		         * 服务端处理客户端连接请求是按顺序处理的，所以同一时间只能处理一个客户端连接，多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理
				 */
				.option(ChannelOption.SO_BACKLOG, 1024);
				// 两小时内没有数据的通信时,TCP会自动发送一个活动探测数据报文
//				.childOption(ChannelOption.SO_KEEPALIVE, true);
		// 绑定端口,开始接收进来的连接
		try {
			future = bootstrap.bind(socketAddress).sync();

			channel = future.channel();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (future != null && future.isSuccess()) {
				log.info(">>>>>> Netty 服务端启动成功，IP：" + socketAddress.getAddress() + "，端口：" + socketAddress.getPort() + " ...");
			} else {
				log.error(">>>>> Netty 服务端启动失败!");
			}
		}
		return future;
	}

	/**
	 * 停止服务
	 */
	public void destroy() {
		if (channel != null) {
			channel.close();
			workGroup.shutdownGracefully();
			bossGroup.shutdownGracefully();
			log.info("Netty 服务端关闭!");
		}
	}
}
```

<u>ChannelHandlerd的生命周期</u>[^4]

==MyServerCodec==里面主要是定义了一些==编解码的方式==、==心跳机制==、==消息处理==等

![image-20210628150830432](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628150830.png)

当没有设置心跳时，加上这个后默认为2小时自动检测一次

![image-20210628151123553](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628151123.png)

当程序启动后，开启==另外一个线程==进行处理消息，因为这里阻塞了，因此在后面使用接口动态启动服务端与客户端的时候都是另外启动一个线程防止阻塞

![image-20210628151338980](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628151339.png)

> MyServerCodec（==编解码的方式==、==心跳机制==、==消息处理==等，**==*重点*==**）

```java
public class MyServerCodec extends ChannelInitializer<SocketChannel> {
	@Override
	protected void initChannel(SocketChannel socketChannel) throws Exception {
    xxx
	}
}
```

1. 心跳（心跳部分下面那行代码==必须放在方法中的第一行==，放入后面可能不起作用）

   dleStateHandler的readerIdleTime参数指定超过时间还没收到客户端的连接， 会触发IdleStateEvent事件并且交给下一个handler处理，下一个handler必须实现userEventTriggered方法处理对应事件，

   三个参数：

   * 表示多久没有读（客户端没发送消息），会发送一个心跳检测包检测是否连接
   * 表示多久没有写（服务端没发送消息），会发送一个心跳检测包检测是否连接
   * 多长时间没有读写，会发送一个心跳检测包检测是否连接

   ```java
   socketChannel.pipeline().addLast(new IdleStateHandler(6, 0, 0, TimeUnit.SECONDS));
   ```

   设置上面的时间以后，在消息处理中将会每6秒钟调用下面的userEventTriggered方法，具体实现可以自己根据自己业务情况编写。

   ```java
   	/**
   	 * 超过空闲时间调用，过了次数后关闭链接
   	 */
   	@Override
   	public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
   		String ip = ((InetSocketAddress) ctx.channel().remoteAddress()).getAddress().getHostAddress();
   		IdleStateEvent event = (IdleStateEvent) evt;
   
   		String eventType;
   		switch (event.state()) {
   		case READER_IDLE:
   			eventType = "读空闲"; // 如果客户端未发送信息到服务端,那么会触发channel的读空闲
   			// 读空闲的计数加1
   			readIdleTimes++;
   			break;
   		case WRITER_IDLE:
   			eventType = "写空闲"; // 如果服务端未发送信息到客户端,那么会触发channel的写空闲
   			// 不处理
   			break;
   		case ALL_IDLE:
   			eventType = "读写空闲";
   			// 不处理
   			break;
   		default:
   			throw new IllegalStateException("非法状态！");
   		}
   		log.info(">>>>>> " + ctx.channel().remoteAddress() + " 超时事件：" + eventType);
   		// 超过3次超时则断开连接
   		if (readIdleTimes > 3) {
   			log.info(">>>>> Netty 客户端 IP：" + ip + " 连接超时，关闭连接！");
   			ctx.channel().close();
   		}
   	}
   ```

2. 使用Netty自带的编解码

   <u>Netty4自带编解码器详解</u>[^1]

   <u>Protobuf编解码</u>[^2]

   1. 字符串编解码

      ```java
      socketChannel.pipeline().addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
      socketChannel.pipeline().addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
      ```

   2. Base64编解码 base64的使用需要在String的基础上，不然消息是无法直接传递

      ```java
      socketChannel.pipeline().addLast("base64Decoder", new Base64Decoder());
      socketChannel.pipeline().addLast("base64Encoder", new Base64Encoder());
      ```

   3. Object编解码，==图片文件传输==可以使用这种方式，当文件传输时，客户端和服务端都为对象传输，开业直接传递File

      ```java
      socketChannel.pipeline().addLast("decoder", new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())));
      socketChannel.pipeline().addLast("encoder", new ObjectEncoder());
      ```

      发送方式

      ![image-20210628152952809](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628152952.png)

      接收方式

      ![image-20210628153116512](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628153116.png)

   4. 字节编解码

      ```java
      socketChannel.pipeline().addLast("decoder", new ByteArrayDecoder());
      socketChannel.pipeline().addLast("encoder", new ByteArrayEncoder());
      ```

   5. Protobuf编解码（暂未了解）

3. 使用自定义的编解码方式

   ```java
   socketChannel.pipeline().addLast("decoder", new MyByteToMessageDecoder());
   socketChannel.pipeline().addLast("encoder", new MyMessageToByteEncoder());
   ```

   ```java
   /**
    * @ClassName: MyMessageToByteEncoder
    * @Description: 自定义编码器
    * @Author yuankaiqiang
    * @DateTime 2021-06-27 15:02:26
    */
   public class MyMessageToByteEncoder extends MessageToByteEncoder<ByteBuf>{
   
   	@Override
   	protected void encode(ChannelHandlerContext ctx, ByteBuf msg, ByteBuf out) throws Exception {
   		out.writeBytes(msg);
   	}
   
   }
   ```

   ```java
   /**
    * @ClassName: MyByteToMessageDecoder
    * @Description: 自定义解码器
    * @Author yuankaiqiang
    * @DateTime 2020-11-30 00:14:02
    */
   public class MyByteToMessageDecoder extends ByteToMessageDecoder {
   
   	@Override
   	protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
   		
   		byte[] bytes = new byte[in.readableBytes()];
   		// 复制内容到字节数组bytes
   		in.readBytes(bytes);
   		
   		// 转换到对应的类型
   		String ms = ConvertFactory.bytesToHexString(bytes);
   		
   		out.add(ms);
   	}
   	
   }
   ```

4. 消息处理

   ```java
   socketChannel.pipeline().addLast(new NettyServerHandler());
   ```

   主要是继承ChannelInboundHandlerAdapter，具体实现看文章后面的代码链接

   ![image-20210628153603016](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628153603.png)

5. 粘包拆包处理

   <u>粘包拆包处理参考</u>[^3]

   1. FixedLengthFrameDecoder

      **固定长度**的粘包和拆包场景，指定长度为20，长度为20时才会接收一次，也可以自定义的包处理，未满20的长度补空格

      ```java
      socketChannel.pipeline().addLast(new FixedLengthFrameDecoder(20));
      ```

      发送第四次才成功

      <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628164217.png" alt="image-20210628164217480" style="zoom: 50%;" />

      ![image-20210628164240791](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628164240.png)

   2. LineBasedFrameDecoder

      通过**分隔符**进行粘包和拆包问题的处理，LineBasedFrameDecoder的作用主要是通过换行符，即\n或者\r\n对数据进行处理

      ```java
      socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
      ```

      第一次请求发送消息，没有加换行的情况，消息没发送出去

      <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628164615.png" alt="image-20210628164615657" style="zoom: 50%;" />

      加了换行以后发送，可以进行发送，并且，上一次的数据也进行了发送，因此打印了两次，上一次的数据存在缓存中

      <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628164716.png" alt="image-20210628164716493" style="zoom: 50%;" />

      ![image-20210628164900037](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628164900.png)

   3. DelimiterBasedFrameDecoder

      将delimiter设置到DelimiterBasedFrameDecoder中，经过该解码一器进行处理之后，源数据将会被按照==\$\_\$==进行分隔，这里1024指的是分隔的最大长度，即当读取到1024个字节的数据之后，若还是未读取到分隔符，则舍弃当前数据段，因为其很有可能是由于码流紊乱造成的。

      ```java
      socketChannel.pipeline().addLast(new DelimiterBasedFrameDecoder(5,
      	    Unpooled.wrappedBuffer("$_$".getBytes())));
      ```

      当发送的数据中以==\$\_\$==结尾才可以读取到数据，且数据包的内容是剔除调数据包的分隔符==\$\_\$==

      <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628165149.png" alt="image-20210628165149281" style="zoom: 50%;" />

      ![image-20210628165342402](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628165342.png)

      若你设置的大小为20个且结尾为 ==\$_\$==，当你每一次发送的数据大小小于20个时会存在缓存中等待你下次输入的是否是 ==\$_\$==，若还不是，则丢弃掉这个包，断开连接（低版本抛出异常），若是则接收之前每次发送的数据包。

      抛出 TooLongFrameException 异常防止由于异常码流缺失分隔符导致内存溢出（亲测 Netty 4.1 版本，服务器并未抛出异常，而是客户端被强制断开连接了）

      <img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628170317.png" alt="image-20210628170317208" style="zoom:50%;" />

      ![image-20210628170329865](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210628170329.png)

#### 2、客户端

启动方法

```java
package com.yuankaiqiang.netty.client;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import com.yuankaiqiang.netty.client.filter.MyClientCodec;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * @ClassName: NettyClientRun
 * @Description: 客户端启动
 * @Author yuankaiqiang
 * @DateTime 2021-06-27 16:43:03
 */
@Component
public class NettyClientRun {
	
	private static final Logger log = LoggerFactory.getLogger(NettyClientRun.class);

	private static ChannelFuture future = null;
	private EventLoopGroup group = new NioEventLoopGroup();
    
    public ChannelFuture start(String ip, int port) {
    	
    	
        Bootstrap bootstrap = new Bootstrap()
                .group(group)
                //该参数的作用就是禁止使用Nagle算法，使用于小数据即时传输
                .option(ChannelOption.TCP_NODELAY, true)
                .channel(NioSocketChannel.class)
                .handler(new MyClientCodec());
        try {
            future = bootstrap.connect(ip, port).sync(); 
			// 第一次连接发送消息
			future.channel().writeAndFlush("第一次连接发送消息");
            	
            return future;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return future;
        }
    }

	public static ChannelFuture getFuture() {
		return future;
	}
	
	/**
	 * 停止服务
	 */
	public void destroy() {
		if (future != null) {
			future.channel().close();
			group.shutdownGracefully();
			log.info("Netty 客户端端关闭!");
		}
	}
}
```

==MyClientCodec与服务端类似==

### 参考文章链接：

[^1]: https://blog.csdn.net/u010889990/article/details/79891585
[^2]: https://blog.csdn.net/qq_37598011/article/details/83352572
[^3]: https://my.oschina.net/zhangxufeng/blog/3023794
[^4]:https://blog.csdn.net/qq_28497823/article/details/106191187

