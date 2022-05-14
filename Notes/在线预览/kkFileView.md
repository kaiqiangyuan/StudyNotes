[toc]

### 1、简介

此项目为文件文档在线预览项目解决方案，对标业内付费产品有【[永中office](http://dcs.yozosoft.com/)】【[office365](http://www.officeweb365.com/)】【[idocv](https://www.idocv.com/)】等，在取得公司高层同意后以Apache协议开源出来反哺社区，在此特别感谢@唐老大的支持以及@端木详笑的贡献。该项目使用流行的spring boot搭建，易上手和部署，基本支持主流办公文档的在线预览，如doc,docx,Excel,pdf,txt,zip,rar,图片等等

### 2、安装部署

官方网址：https://kkfileview.keking.cn/zh-cn/index.html

码云：https://gitee.com/kekingcn/file-online-preview

> ubuntu中安装

使用docker进行安装部署

```sh
#使用docker安装
docker pull keking/kkfileview
#启动（这里设置重启自启动容器并且后台运行）
docker run -it -p 8012:8012 --restart=always -d keking/kkfileview
#访问 http://xxxxx:8012即可访问测试页面
```

![image-20210611195606328](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611195606.png)

修改配置文件

```sh
#获取application.properties文件位置
find / -name application.properties
#进入容器
docker exec -it 155a1c17efe4（容器id） bash
#修改完以后重启docker容器
docker restart 155a1c17efe4（容器id）
```

> mac中部署

先安装：https://www.openoffice.org/download/

![image-20210612213925437](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210612213925.png)

1. 第一种：使用docker进行安装部署
2. 第二种：在码云中下载已经编译好的安装包

![image-20210611195723174](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611195723.png)

进入`bin`目录下使用`startup.sh`

![Q](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611200937.png)

访问8012端口

![QQ20210611-200731@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611200747.png)

3. 第三种：下载最新版本的源码，自己进行编译安装

### 3、使用

> js中使用

```sh
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/js-base64@3.6.0/base64.min.js"></script>

var url = 'http://127.0.0.1:8080/file/test.txt'; //要预览文件的访问地址
window.open('http://127.0.0.1:8012/onlinePreview?url='+encodeURIComponent(Base64.encode(url)));
```

> SpringBoot中使用

使用==Thymeleaf==返回页面

1. 添加依赖

   ```xml
   		<dependency>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-thymeleaf</artifactId>
   		</dependency>
   ```

2. 配置properties

   ```properties
   spring.thymeleaf.prefix=classpath:/templates/
   
   spring.mvc.static-path-pattern=/**
   spring.resources.static-locations=classpath:/static/
   
   spring.thymeleaf.cache=false
   ```

3. 编写html页面（**==重要==**）

   ![image-20210611202436961](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611202437.png)

   ![image-20210611202005406](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611202005.png)

### 4、中文显示乱码

下载常用的字体：http://kkfileview.keking.cn/fonts.zip

解压放入到`/usr/share/fonts`目录中

![image-20210611202714694](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210611202714.png)

```sh
#使字体生效，若提示未安装相关命令，则安装即可
mkfontscale
mkfontdir
fc-cache
```





















