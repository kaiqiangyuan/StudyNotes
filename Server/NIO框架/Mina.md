参考：

https://www.jianshu.com/p/ae9273a389c7

http://www.360doc.com/content/18/0814/13/52020206_778183718.shtml



查看mina网络通信框架，tcp部分的代码，并查询相关资料，

* IoServiece ：这个接口在一个线程上负责套接字的建立，拥有自己的 Selector，监听是否有连接被建立。

* IoProcessor ：这个接口在另一个线程上负责检查是否有数据在通道上读写，也就是说它也拥有自己的 Selector，这是与我们使用 JAVA NIO 编码时的一个不同之处，通常在 JAVA NIO 编码中，我们都是使用一个 Selector，也就是不区分 IoService与 IoProcessor 两个功能接口。另外，IoProcessor 负责调用注册在 IoService 上的过滤器，并在过滤器链之后调用 IoHandler。  

- IoAccepter ：相当于网络应用程序中的服务器端

+ IoConnector ：相当于客户端

- IoSession ：当前客户端到服务器端的一个连接实例

- IoHandler ：这个接口负责编写业务逻辑，也就是接收、发送数据的地方。这也是实际开发过程中需要用户自己编写的部分代码。

- IoFilter ：过滤器用于悬接通讯层接口与业务层接口，这个接口定义一组拦截器，这些拦截器可以包括日志输出、黑名单过滤、数据的编码（write 方向）与解码（read 方向）等功能，其中数据的 encode与 decode是最为重要的、也是你在使用 Mina时最主要关注的地方。



**负责接收发送数据的接口IoHandler有以下方法操作**

```java
•void exceptionCaught(IoSession session, Throwable cause)
当接口中其他方法抛出异常未被捕获时触发此方法
•void messageReceived(IoSession session, Object message)
当接收到客户端的请求信息后触发此方法
•void messageSent(IoSession session, Object message)
当信息已经传送给客户端后触发此方法
•void sessionClosed(IoSession session)
当连接被关闭时触发，例如客户端程序意外退出等等
•void sessionCreated(IoSession session)
当一个新客户端连接后触发此方法
•void sessionIdle(IoSession session, IdleStatus status)
当连接空闲时触发此方法
•void sessionOpened(IoSession session)
当连接后打开时触发此方法，一般此方法与 sessionCreated 会被同时触发
```



