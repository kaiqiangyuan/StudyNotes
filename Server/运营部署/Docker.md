[toc]

### 一、简介

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的[Linux](https://baike.baidu.com/item/Linux)机器或Windows 机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。

### 二、安装

> 使用官方安装ubuntu18.04

```sh
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

```
sudo service docker start（启动）
sudo service docker stop（停止）
sudo service docker restart（重启）
```

> Ubuntu18.04系统docker换源

修改 `/etc/docker/daemon.json` 文件，一般不存在，新建一个就可以了

```sh
{"registry-mirrors":["https://reg-mirror.qiniu.com/"]}
```

重启服务

```sh
#重新加载某个服务的配置文件，如果新安装了一个服务，归属于 systemctl 管理，要是新服务的服务程序配置文件生效，需重新加载
sudo systemctl daemon-reload
sudo systemctl restart docker
```

> Mac下

安装地址：https://docs.docker.com/desktop/mac/install/

切换镜像，先到阿里云地址上复制：[地址](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```json
"registry-mirrors": [
	"https://tkcnyb3h.mirror.aliyuncs.com"
]
```

![image-20220312112855984](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/1647055736.png)

### 三、镜像与容器

- 镜像：用来启动容器的只读模板，是容器启动所需的rootfs，类似于虚拟机所使用的镜像。
- 容器：Docker 容器是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

镜像是容器的基础，可以简单的理解为镜像是我们启动虚拟机时需要的镜像，容器时虚拟机成功启动后，运行的服务。

### 四、镜像命令(images)

```sh
docker version # 显示 Docker 版本信息。
docker info # 显示 Docker 系统信息，包括镜像和容器数
docker --help # 帮助
#查看已下载的Docker镜像latest具体版本
docker image inspect (docker image名称):latest|grep -i version
```

**docker images**

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602091930.png)

```sh
REPOSITORY 镜像的仓库源
TAG 镜像的标签
IMAGE ID 镜像的ID
CREATED 镜像创建时间
SIZE 镜像大小
# 同一个仓库源可以有多个 TAG，代表这个仓库源的不同版本，我们使用REPOSITORY：TAG 定义不同
的镜像，如果你不定义镜像的标签版本，docker将默认使用 lastest 镜像！
# 可选项
-a： 列出本地所有镜像
-q： 只显示镜像id
--digests： 显示镜像的摘要信息
```

**docker search**

```sh
docker search 某个镜像的名称 对应DockerHub仓库中的镜像
# 可选项
--filter=stars=50 ： 列出收藏数不小于指定值的镜像。
```

**docker pull**

```sh
# 下载镜像
docker pull mysql
# 指定版本下载
docker pull mysql:5.7
# 不写tag，默认是latest
docker.io/library/mysql:latest # 真实位置
```

**docker rmi**

```sh
# 删除镜像
docker rmi -f 镜像id # 删除单个
docker rmi -f 镜像名:tag 镜像名:tag # 删除多个
docker rmi -f $(docker images -qa) # 删除全部
```

**docker save（将一个镜像导出为文件）**

将指定镜像保存成 tar 归档文件，**-o :**输出到的文件

```sh
root@Ubuntu:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
centos       latest    300e315adb2f   6 months ago   209MB
root@Ubuntu:~# 
root@Ubuntu:~#  docker save -o centos_new.tar centos 
root@Ubuntu:~# ls
centos_new.tar
```

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703170620.png" alt="image-20210703170620802" style="zoom:50%;" />

**docker load（将文件导入为一个镜像，会保存该镜像的的所有历史记录，文件大）**

导入使用 [docker save](https://www.runoob.com/docker/docker-save-command.html) 命令导出的镜像，**--input , -i :** 指定导入的文件，代替 STDIN，**--quiet , -q :** 精简输出信息

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210703170846.png" alt="image-20210703170846147" style="zoom:50%;" />

**docker push**

上传镜像到仓库，用户在DockerHub网站注册后，即可上传自制的镜像，第一次使用时，会提示输入登录信息或进行注册

**创建镜像**

1、基于已有镜像创建  2、基于本地模板导入  3、基于DockerFile创建

​	1、docker commit 从容器创建一个新的镜像（==停止容器后创建==）

```sh
-a：作者信息
-m：提交信息
-p：提交时暂停容器运行
docker commit 提交容器副本使之成为一个新的镜像！ # 语法 docker commit -m="提交的描述信息" -a="作者" 容器id 要创建的目标镜像名:[标签名]
```

​	2、基于本地模板导入

推荐使用OpenVZ提供的模板创建，[OpenVZ模板下载地址](https://download.openvz.org/template/precreated/)

```sh
cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04
```

### 五、容器命令(Container)

有镜像才能创建容器

docker pull centos

**启动**

```sh
# 命令
docker run [OPTIONS] IMAGE [COMMAND][ARG...]
# 常用参数说明
--name="Name" # 给容器指定一个名字,或者使用--name xxx
-d # 后台方式运行容器，并返回容器的id！
-i # 以交互模式运行容器，通过和 -t 一起使用
-t # 给容器重新分配一个终端，通常和 -i 一起使用
-P # 随机端口映射（范围49000-49900）（大写）
-p # 指定端口映射（小结），一般可以有四种写法
    ip:hostPort:containerPort
    ip::containerPort
    hostPort:containerPort (常用)
    containerPort
# 使用centos进行用交互模式启动容器，在容器内执行/bin/bash命令！
docker run -it centos /bin/bash
```

**列出所有运行的容器**

```sh
# 命令
docker ps [OPTIONS]
# 常用参数说明
-a # 列出当前所有正在运行的容器 + 历史运行过的容器
-l # 显示最近创建的容器
-n=? # 显示最近n个创建的容器
-q # 静默模式，只显示容器编号
```

**退出容器**

```sh
exit # 容器停止退出
ctrl+P+Q # 容器不停止退出
```

**启动停止容器**

```sh
docker start (容器id or 容器名) # 启动容器
docker restart (容器id or 容器名) # 重启容器
docker stop (容器id or 容器名) # 停止容器
docker kill (容器id or 容器名) # 强制停止容器
```

**删除容器**

```sh
docker rm 容器id # 删除指定容器
docker rm -f $(docker ps -a -q) # 删除所有容器
docker ps -a -q|xargs docker rm # 删除所有容器
```

**后台启动容器**

`--restart=always`自启动容器

--restart=always参数能够使我们在重启docker时，自动启动相关容器

```sh
#第一次启动时添加
docker run -it -p 8012:8012 --restart=always -d keking/kkfileview
#已经启动的镜像没有加上自动启动容器的参数时可以进行修改使用
docker container update --restart=always 容器名字
```

```sh
# 命令
docker run -d 容器名
# 例子
docker run -d centos # 启动centos，使用后台方式启动
# 问题： 使用docker ps 查看，发现容器已经退出了！
# 解释：Docker容器后台运行，就必须有一个前台进程，容器运行的命令如果不是那些一直挂起的命令，就会自动退出。
# 比如，你运行了nginx服务，但是docker前台没有运行应用，这种情况下，容器启动后，会立即自杀，因为他觉得没有程序了，所以最好的情况是，将你的应用使用前台进程的方式运行启动。
```

**查看日志**

```sh
# 命令
docker logs -f -t --tail 容器id
# -t 显示时间戳
# -f 打印最新的日志
# --tail 数字 显示多少条！
```

**查看容器中运行的进程信息**

```sh
# 命令
docker top 容器id
```

**查看容器/镜像的元数据**

```sh
# 命令
docker inspect 容器id
```

**进入正在运行的容器**

```sh
# 命令1
docker exec -it 容器id xx(nginx中是zsh)
# 命令2
docker attach 容器id
# 区别
# exec 是在容器中打开新的终端，并且可以启动新的进程（推荐使用）
# attach 直接进入容器启动命令的终端，不会启动新的进程
```

**从容器内拷贝文件到主机上**

```sh
# 命令，可以互相拷贝
docker cp 容器id:容器内路径 目的主机路径
```

**更新容器状态（update）**

参考：https://www.lwhweb.com/posts/26195/

```sh
#更新容器的重启策略
docker update --restart=always f361b7d8465（启动镜像时自动启动容器）
docker update --restart=no f361b7d8465（启动镜像时不自动启动容器）
#更新容器内存
docker update -m 500M f361b7d8465
#更新 CPU 共享数量
docker update --cpu-shares 512 f361b7d8465
```

**导入导出容器，迁移镜像**

导出镜像 **export（将一个容器导出为文件）**

```sh
#方式一
docker export 5da872843d36 > new_kkfile.tar
#方式二
docker export -o new_kkfile_two.tar 5da872843d36
```

```sh
root@Ubuntu:~# docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                    NAMES
5da872843d36   keking/kkfileview   "java -Dfile.encodin…"   17 seconds ago   Up 16 seconds   0.0.0.0:8012->8012/tcp   cool_hellman
root@Ubuntu:~# docker export 5da872843d36 > new_kkfile.tar
root@Ubuntu:~# ls
new_kkfile.tar
root@Ubuntu:~# 
```

导入镜像 **import（将容器导入成为一个新的镜像，丢失所有元数据和历史记录，仅保存容器当时的状态，相当于虚拟机快照）**

**-m :**提交时的说明文字

```sh
root@Ubuntu:~# ls
new_kkfile.tar  new_kkfile_two.tar
root@Ubuntu:~# cat new_kkfile_two.tar | docker import - test/kkfile:v1.0
sha256:d2e526f9d6c749682f83312934c990d4e92c85b4d490e5931b97ff59f92070f9
root@Ubuntu:~# docker images
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
test/kkfile         v1.0      d2e526f9d6c7   22 seconds ago   1.14GB
keking/kkfileview   latest    894100e865a8   2 months ago     1.16GB
root@Ubuntu:~# 
```

​		实际上，既可以使用==docker load==命令来导人镜像存储文件到本地的镜像库，又可以使用==docker import==命令来导人一个容器快照到本地镜像库。这两者的区别在于**容器快照文件将丢弃所有的历史记录和元数据信息(即仅保存容器当时的快照状态)**，而**镜像存储文件将保存完整记录，体积也要大**。此外，从容器快照文件导人时可以重新指定标签等元数据有低。

### 六、DockerFile

**构建步骤：**

1. 编写DockerFile文件

2. docker build 构建镜像

3. docker run

**基础知识：**

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数

2. 指令按照从上到下，顺序执行

3. #表示注释

4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

**流程：**

1. docker从基础镜像运行一个容器

2. 执行一条指令并对容器做出修改

3. 执行类似 docker commit 的操作提交一个新的镜像层

4. Docker再基于刚提交的镜像运行一个新容器

5. 执行dockerfifile中的下一条指令直到所有指令都执行完成！

**指令**

```sh
FROM # 基础镜像，当前新镜像是基于哪个镜像的
MAINTAINER # 镜像维护者的姓名混合邮箱地址
RUN # 容器构建时需要运行的命令
EXPOSE # 当前容器对外保留出的端口
WORKDIR # 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
ENV # 用来在构建镜像过程中设置环境变量
ADD # 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
COPY # 类似ADD，拷贝文件和目录到镜像中！
VOLUME # 容器数据卷，用于数据保存和持久化工作
CMD # 指定一个容器启动时要运行的命令，dockerFile中可以有多个CMD指令，但只有最后一个生效！
ENTRYPOINT # 指定一个容器启动时要运行的命令！和CMD一样
ONBUILD # 当构建一个被继承的DockerFile时运行命令，父镜像在被子镜像继承后，父镜像的ONBUILD被触发
```

自己的镜像：登陆后的默认路径、vim编辑器、查看网络配置ifconfifig支持

1. 准备编写DockerFlie文件

   ```dockerfile
   FROM ubuntu
   MAINTAINER kaiqiang<kaiqiang>
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   RUN apt-get install vim
   RUN apt-get install net-tools
   EXPOSE 80
   CMD echo $MYPATH
   CMD echo "----------end--------"
   CMD /bin/bash
   ```

2. 构建

   ```sh
   docker build -f dockerfile地址 -t 新镜像名字:TAG . #docker build 命令最后有一个 . . 表示当前目录
   ```

3. 运行

   ```sh
   docker run -it 新镜像名字:TAG
   ```

### 七、Springboot整合Docker

1. 创建一个简单的Sringboot项目并打包

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello,yuankaiqiang";
    }
}
```

2. 在jar包的同级目录下编写DockerFile文件，并且一起上传到服务器

```java
FROM java:8
# 服务器只有dockerfile和jar在同级目录
COPY *.jar /app.jar
CMD ["--server.port=8080"]
# 指定容器内要暴露的端口
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```

3. 然后构建镜像  运行就可以访问到项目了

```sh
docker build -t kaiqiang .
```

4. 运行

```sh
docker run -d -P --name idea-ks kaiqiang kaiqiang
```

### 八、部署nginx

```sh
docker pull nginx
docker run -d -p 80:80 --name nginx nginx
#这样就简单的把nginx启动了，但是我们想要改变配置文件nginx.conf ，进入容器,命令：
#nginx.conf配置文件在 /etc/nginx/ 
docker exec -it nginx bash
```

### 九、常见错误

> 当进入容器中使用vim命令提示错误

![QQ20210609-112414@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210609112543.png)

```sh
apt-get update
apt-get install vim
```

### 十、使用容器构建自定义的镜像（commit），修改端口映射

参考：https://zhuanlan.zhihu.com/p/94949253

> 方式一

1. 查看容器

   ![image-20210612150010225](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612150010.png)

2. 创建新的镜像

   ![image-20210612150050432](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612150050.png)

3. 得到新的镜像

   ![image-20210612150154925](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612150154.png)

4. 重新使用新的镜像运行一个容器，重新配置镜像的端口等信息

   ![image-20210612150256306](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612150256.png)

> 方式二

1. 先取消镜像的docker启动后容器的自启动，没有设置容器自启动不需要这一步

   ```sh
   docker update --restart=no xxx
   ```

2. 停止容器

   ```sh
   docker stop xxx
   ```

3. 停止主机docker服务

   ```sh
   systemctl stop docker
   ```

4. 进入到`/var/lib/docker/containers`相应的容器中，修改`hostconfig.json`

   ![image-20210612151818683](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612151818.png)

   8012/tcp对应的是容器内部的8012端口，HostPort对应的是映射到宿主机的端口9090

5. 修改完成重启docker与容器

   ```sh
   systemctl start docker
   docker start xxx
   ```

> 方式三

端口映射实际是通过主机的`iptables`来实现

```sh
#查看映射
sudo iptables -t nat -vnL
```

```sh
Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8012 to:172.17.0.2:8012
```

docker创建了一个名为DOKCER的自定义的链条Chain。而我开放8012端口的容器的ip是172.17.0.2

也可以通过inspect命令查看容器ip

```sh
#通过inspect命令查看容器ip
docker inspect [containerId] |grep IPAddress
```

增加一个端口映射，比如`8081->81`

```sh
sudo iptables -t nat -A  DOCKER -p tcp --dport 8081 -j DNAT --to-destination 172.17.0.2:81
```

加错了或者想修改：先显示行号查看

```sh
sudo iptables -t nat -vnL DOCKER --line-number
```

删除规则3

```sh
sudo iptables -t nat -D DOCKER 3
```

### 十一、Docker Hub

> 注册

注册网址：https://registry.hub.docker.com/

```sh
root@Ubuntu:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: yuankaiqiang
Password: 
Error response from daemon: Get https://registry-1.docker.io/v2/: unauthorized: incorrect username or password
root@Ubuntu:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: yuankaiqiang
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
root@Ubuntu:~# 
```

### 十二、实践

#### 1、挂载容器目录

作用：挂载宿主机的一个目录

-v 本地目录:容器目录 或 -v 容器目录

```sh
docker run -it -v /宿主机目录:/容器目录 镜像名 /bin/bash
```

#### 2、启动主从模式

-e 传递环境变量

--link可以用来链接2个容器

```sh
#创建一个命令为mysql的主容器
docker run -d -e REPLICATION_MASTER=true -P --name mysql mysql
#创建从容器，并连接到刚刚创建的主容器
docker run -d -e REPLICATION_SLAVE=true -P --link mysql:mysql mysql
```

#### 3、使用Mysql

1. 下载镜像
2. 启动mysql容器

容器内的Mysql提供了root账号和admin账号，其中root账号无需密码，但只允许本地访问

```mysql
#mysql命令行中使用如下命令查看
select host, user, password from mysql.user
```

使用 `docker logs eef `查看adamin密码

启动容器时指定admin账号和密码

```sh
docker run -d -P -e MYSQL_PASS="123456789" mysql
```















