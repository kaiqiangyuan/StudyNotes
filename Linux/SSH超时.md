> 连接服务器一段时间自动断开解决办法

参考：

<https://bin.zmide.com/?p=188>

<https://segmentfault.com/q/1010000009421113>

<https://blog.csdn.net/qq_38081870/article/details/102568417>

> 经常需要连接到Linux服务器，发现过一段时间不输入命令，服务器的ssh连接就会断开，又需要重新ssh登录

==ubuntu服务器== 

1. 修改 **/etc/ssh** 目录下的 **sshd_config** 文件  增加下面内容

```shell
ClientAliveInterval 60
ClientAliveCountMax 60
```

ClientAliveInterval表示服务器每隔多久向客户端发送一个“空包”，以保持连接，单位为秒，此处为60秒；

ClientAliveCountMax表示如果发现客户端没有响应，则判断一次超时，这个参数为允许超时的次数，此处为60次。相当于1小时无通信后断开ssh连接

2. 重启ssh服务

   ```shell
   systemctl restart ssh
   再重新ssh连接即可。
   ```

   
