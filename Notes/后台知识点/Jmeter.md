[toc]

### 一、简介

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。 它可以用于测试静态和动态资源，例如静态文件、Java [小服务程序](https://baike.baidu.com/item/小服务程序/4148836)、CGI 脚本、Java 对象、数据库、[FTP](https://baike.baidu.com/item/FTP/13839) 服务器， 等等。JMeter 可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试它们的强度和分析整体性能。另外，JMeter能够对应用程序做功能/[回归测试](https://baike.baidu.com/item/回归测试/1925732)，通过创建带有断言的脚本来验证你的程序返回了你期望的结果。为了最大限度的灵活性，JMeter允许[使用正则表达式](https://baike.baidu.com/item/使用正则表达式/6555484)创建断言。

### 二、安装

http://jmeter.apache.org/download_jmeter.cgi

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601130248.png)

参考：https://www.jianshu.com/p/bce9077d883c

解压后进入 bin 目录执行 jmeter 即可启动

**修改语言为中文**修改 bin 目录下的 jmeter.properties 文件

```shell
#Preferred GUI language. Comment out to use the JVM default locale's language.
#language=en
修改为
language=zh_CN
```

