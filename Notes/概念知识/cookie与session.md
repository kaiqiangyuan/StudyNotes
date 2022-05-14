HTTP是无状态协议，无法记得上一次连接的信息，比如记不得用户是否已经登陆等信息。

而Session、Cookie、Token便是对HTTP无状态的一种补充；

![image-20210629210056257](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210629210056.png)

1. **Cookie**

   Cookie是浏览器对于一些信息的键值对形式保存，当浏览器关闭，Cookie也就删除了；也可以设置Cookie的存活时间，关闭浏览器后不会断开会话。

2. **Session**

   Session是服务器中保存的对象（Tomcat保存在ConcurrentHashMap\<Session\>中），生成之后在返回Http response时，会发送sesionId给浏览器（响应头里会有Set-Cookie：JSESSIONID=XXXXXXX ），浏览器收到response后会在Cookie里保存session id。之后浏览器发送请求时，便可以通过Cookie中的sessionId找到响应的用户。

   也就是说，==Cookie和Session组合使用==（会员Cookie）实现网站对于用户的身份认证。

   缺点： 

   ==1. 如果web服务器做了负载均衡，那么下一个操作请求到了另一台服务器的时候session会丢失。==

   ==2. 数据量多时服务器压力大。==

3. **Token**

   1. ==Token可以解决会员Cookie方式存在的当浏览器请求另一台服务器时找不到Session的问题==（因为Cookie保存在上一台服务器上）。

   2. token在服务器时可以==不用存储用户信息==的，token传递的方式也不限于cookie传递，token也可以保存起来。

   3. Token采用通过一个==只有服务器知道的密钥和相应的密钥算法（比如SHA256）对信息加密==，生成一个secret,然后和header(指定了加密算法)和payload(一些信息，包括用户信息)组成token;之后浏览器请求服务器时，服务器通过指定的算法和保存的密钥计算secret，若secret和token中的signature相同，则认证成功，因此成功地记录了用户的登录状态。由于不存在需要在浏览器中保存session相似的操作，因此解决了session在不同服务器间不可见的问题（时间换空间）

   特点：

   1. 无状态、可扩展
   2. 支持移动设备
   3. 跨程序调用
   4. 安全

- Token与Cookie/Session的相同点：

  都是为了解决HTTP协议无状态的问题

- 不同点：
  1. 认证成功后，会对当前用户数据进行加密，生成一个加密字符串token，返还给客户端（==服务器端并不进行保存，减少了服务器压力==）
  2. 通常浏览器会将接收到的==token值存储在Local Storage中==，（通过js代码写入Local Storage，通过js获取，并不会像cookie一样自动携带）
  3. 再次访问时服务器端对token值的处理：==服务器对浏览器传来的token值进行解密==，解密完成后进行用户数据的查询，如果查询成功，则通过认证，实现状态保持，所以，即时有了多台服务器，服务器也只是做了token的解密和用户数据的查询，它不需要在服务端去保留用户的认证信息或者会话信息，这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利，解决了session扩展性的弊端。



1. [三种方式的区别](https://blog.csdn.net/fubicheng208/article/details/106685077)

2. [Token和Session 原理及优缺点](https://www.jianshu.com/p/0e77beb3c1e5)
