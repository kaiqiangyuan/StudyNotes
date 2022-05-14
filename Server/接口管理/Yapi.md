[toc]

### 一、简介

https://hellosean1025.github.io/yapi/index.html

旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API

**测试网页**：https://yapi.baidu.com/

39服务器 `/home/poject/my-yapi` 已经部署了一个，查看mongodb是否安装，是否启动默认端口27017

网址：http://39.103.149.176:3000/  账号名："2210774904@qq.com"，密码："ymfe.org"

### 二、部署Yapi

1. 安装需要的环境

- nodejs（7.6+)，最好为node12，node14部署时候会出现异常
- mongodb（2.6+）

>mongodb

```sh
sudo apt update
sudo apt-get install libcurl4 openssl
sudo apt install mongodb
```

修改 `/etc/mongodb.conf`  

```sh
bind_ip = 0.0.0.0
```

```sh
systemctl start mongodb（启动）
systemctl stop mongodb（停止）
systemctl restart mongodb（重启）
```

> Node

若node的版本为14则部署时可能会出现以下问题

![image-20210604191214209](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604191214.png)

切换node的版本为12重新启动

![image-20210604191832132](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604191832.png)

2. 安装并启动

   ```shell
   npm install -g yapi-cli --registry https://registry.npm.taobao.org
   yapi server
   ```

   访问9090端口

   ![image-20210604191538146](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604191538.png)

   ![QQ20210604-162236@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604191325.png)

   进入 `/root/my-yapi` 目录执行

   ```sh
   node vendors/server/app.js
   ```

   ![QQ20210604-162528@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604192107.png)

### 三、使用

设置接口请求的ip地址信息

![image-20210604212014190](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604212014.png)

添加接口

![image-20210604212136261](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604212136.png)

设置请求体或请求头中的各个信息

![image-20210604212213809](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604212213.png)

发送请求，**请求需要增加插件**

![FireShot Capture 014 - 获取区域-YApi-高效、易用、功能强大的可视化接口管理平台 - 114.55.26.230](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210604212350.png)





















