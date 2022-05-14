[toc]

### 一、安装

1. 将存储库密钥添加到系统

   ```shell
   wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   ```

2. 将Debian包存储库地址附加到服务器的sources.list

   ```shell
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/  > /etc/apt/sources.list.d/jenkins.list'
   ```

3. 更新apt-get

   ```shell
   apt-get update
   ```

4. 安装Jenkins

   ```
   sudo apt-get install jenkins
   ```

### 二、启动

1. 使用命令启动

   默认启动端口为8080

   ```shell
   指定端口： java -jar jenkins.war --httpPort=8787
   修改 /etc/default/jenkins 中的8080端口修改为8787 如果 不行就继续修改/etc/init.d/jenkins8080 端口修改为8787
   重启 sudo service jenkins restart 或 systemctl restart jenkins 
   启动 sudo service jenkins start 或 systemctl start jenkins 
   停止 sudo service jenkins stop 或 systemctl stop jenkins 
   systemctl status jenkins 查看状态
   
   # 在开机时启用服务
   systemctl enable jenkins
   # 在开机时禁用服务
   systemctl disable jenkins
   # 查看服务是否开机启动
   systemctl is-enabled jenkins
   # 查看开机自启动的服务列表
   systemctl list-unit-files | grep enabled
   # 查看启动失败的服务列表
   systemctl --failed
   ```

2. 使用war包启动

   或者用下面这张方式启动（默认是8080，加上--httpPort=8787启动就可以了）

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531205802.png)

114服务器 账号admin密码yuankaiqiang

39服务器 账号admin密码yuankaiqiang



39服务器上用war包启动的里面已经配置好了一个可以使用，用命令启动的是全新的

* /usr/lib/jenkins/jenkins.war  WAR包 

* /var/log/jenkins/jenkins.log  Jenkins日志文件



新服务器报错参考 https://blog.csdn.net/laiwenjian/article/details/88813071 

创建java软链接，ln -s <javaPath> /usr/bin/java，javaPath是java路径

### 三、部署

安装public over ssh(方便构建后上传到服务器中)

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531210003.png)

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531210017.png)

参考：https://blog.csdn.net/qq_35663625/article/details/101533691

![QQ20210402-165543@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531210204.png)

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531210221.png)

参考https://blog.csdn.net/weixin_44251399/article/details/88660019

![11](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531210253.png)

