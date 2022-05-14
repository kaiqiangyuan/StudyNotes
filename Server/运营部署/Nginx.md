[toc]

### 1、Ubuntu服务器中的nginx

ubuntu中的nginx的配置文件

```sh
conf.d：用户自己定义的conf配置文件
sites-available：系统默认设置的配置文件
sites-enabled：由sites-available中的配置文件转换生成
nginx.conf：汇总以上三个配置文件的内容，同时配置我们所需要的参数
在部署需要的web服务时，我们可以拷贝sites-enabled中的default文件到conf.d并且修改名字为**.conf,然后进行配置
```

**==服务器使用nginx==**

```sh
nginx -t #查找配置文件在哪
```

ubuntu中的在 **/etc/nginx/nginx.conf**

![999](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602101158.png)

**之所以下面修改default文件可以使用是因为这里引入了sites-enable下面的文件，而sites-enable是sites-available拷贝而来，建议在conf.d中创建nginx.conf文件进行配置nginx**

![888](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602101243.png)

```
root@iZbp1fp1ljg8kt4kwm2nohZ:/# cd /etc/nginx/sites-available/
root@iZbp1fp1ljg8kt4kwm2nohZ:/etc/nginx/sites-available# ls
default
root@iZbp1fp1ljg8kt4kwm2nohZ:/etc/nginx/sites-available# vim default

启动: 直接使用命令: nginx
nginx
关闭1: 快速停止
nginx -s stop
关闭2: 完整有序停止
nginx -s quit
重启: 
nginx -s reload
```

### 2、Mac中使用nginx

```shell
安装
brew install nginx

启动
brew services start nginx

打开http://localhost:8080，看到这个页面说明启动成功


停止
brew services stop nginx

重启（会先stop，再start）
brew services restart nginx

重新加载配置（不会stop，只是重新加载配置）
nginx -s reload

验证nginx配置文件是否正确
nginx -t

配置文件位置
/usr/local/etc/nginx/nginx.conf
```

### 3、配置不同的域名访问项目不需要输入端口方式

第一种：直接配置nginx更改server_name

https://blog.csdn.net/qq_38571996/article/details/83089877

```nginx
server {
        listen 80 default_server; #监听端口
        root /mnt/aa; #访问根目录
        index index.html index.htm; #默认页,可以不设置
        server_name www.yanyusun.com; #根据域名跳转
        location / {
                try_files $uri $uri/ =404;
                proxy_pass http://127.0.0.1:8081;  #跳转的路径
        }
}
server {
        listen 80;#监听端口
        root /mnt/bb;
        index index.html index.htm;
        server_name bs.yanyusun.com;
        location / {
                try_files $uri $uri/ =404;
                proxy_pass http://127.0.0.1:8082; #另一个路径
        }
}
```

第二种：https://blog.csdn.net/weixin_42433970/article/details/108012788

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602102054.png)

通过http://abc.kaiqiang.top/访问

![222](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602102720.png)

### 4、ssl配置

```nginx
#以下属性中，以ssl开头的属性表示与证书配置有关。
server {
    listen 443 ssl;
    server_name www.kaiqiang.top; #需要将yourdomain.com替换成证书绑定的域名。
    
    ssl_certificate /etc/nginx/cert/5599006_www.kaiqiang.top.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key /etc/nginx/cert/5599006_www.kaiqiang.top.key; #需要将cert-file-name.key替换成已上传的证书密钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
    location / {
	root /home/poject/PersonalWebsite/build;   #站点目录
        try_files $uri $uri/ /index.html;
    }
}
```

### 5、nginx 禁止 ip 允许和阻止访问

nginx会根据配置文件中的语句，从上至下依次判断。因此，写在前面的语句可能会屏蔽后续的语句。所以deny和allow顺序需要设置好

https://www.cnblogs.com/niuben/p/11687638.html

1. 在相应的server中增加

   ```nginx
   allow 12.12.12.22  # 允许访问
   deny 223.104.123.12; # 禁止访问
   
   deny 123.0.0.0/8;    // 封 123.0.0.1~123.255.255.254 这个段的ip
   deny 123.1.0.0/16;   // 封 123.1.0.1~123.1.255.254 这个段的ip
   deny 123.1.1.0/24;   // 封 123.1.1.1~123.1.1.254 这个段的ip
   
   deny all;  // #除allow的IP地址外，其他的IP地址都禁止访问
   ```

2. 也可以使用文件的形式增加ip的访问访问权限

   创建 ==allow_deny_ip.conf== 文件，增加==deny all==

   server中增加

   ```nginx
   include /etc/nginx/allow_deny_ip.conf;
   ```

### 6、自定义403等页面

https://blog.csdn.net/weixin_34249367/article/details/92101696

在 ==/etc/nginx== 中创建 ==error_index== 文件夹

![3](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602103059.png)

在nginx的配置文件中增加

```nginx
error_page   403 = /403.html;  
location = /403.html {           
    root   /etc/nginx/error_index;
	allow all;
 }
```

或者是增加 error_page 403 http://39.103.149.176:9999/; 指定网页中的链接，可以在指定的页面中设置修改配置文件allow_deny_ip.conf中的ip，动态修改ip的访问权限，例如将禁止访问的ip去掉后就可以进行访问了

![4](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602103147.png)

==有无等号的区别，建议使用等号==: https://www.cnblogs.com/redirect/p/10066761.html

error_page 404 /404.html 可显示自定义404页面内容，正常返回404状态码。

error_page 404 = /404.html 可显示自定义404页面内容，但返回200状态码。

### 7、nginx413文件上传失败

参考：https://blog.csdn.net/z69183787/article/details/83070275

```nginx
client_max_body_size     50m; #文件大小限制，默认1m
client_header_timeout    1m;
client_body_timeout      1m;
proxy_connect_timeout     60s;
proxy_read_timeout      1m;
proxy_send_timeout      1m;
```

**client_max_body_size**

限制请求体的大小，若超过所设定的大小，返回413错误。

**client_header_timeout**

读取请求头的超时时间，若超过所设定的大小，返回408错误。

**client_body_timeout**

读取请求实体的超时时间，若超过所设定的大小，返回413错误。

**proxy_connect_timeout**

http请求无法立即被容器(tomcat, netty等)处理，被放在nginx的待处理池中等待被处理。此参数为等待的最长时间，默认为60秒，官方推荐最长不要超过75秒。

**proxy_read_timeout**

http请求被容器(tomcat, netty等)处理后，nginx会等待处理结果，也就是容器返回的response。此参数即为服务器响应时间，默认60秒。

**proxy_send_timeout**

http请求被服务器处理完后，把数据传返回给Nginx的用时，默认60秒。

### 8、负载均衡

​		当一台服务器的单位时间内的访问量越大时，服务器压力就越大，大到超过自身承受能力时，服务器就会崩溃。为了避免服务器崩溃，让用户有更好的体验，我们通过负载均衡的方式来分担服务器压力。我们可以建立很多很多服务器，组成一个服务器集群，当用户访问网站时，先访问一个中间服务器，在让这个中间服务器在服务器集群中选择一个压力较小的服务器，然后将该访问请求引入该服务器。如此以来，用户的每次访问，都会保证服务器集群中的每个服务器压力趋于平衡，分担了服务器压力，避免了服务器崩溃的情况。负载均衡配置一般都需要同时配置反向代理，通过反向代理跳转到负载均衡。

```nginx
upstream mysvr {   
    server 127.0.0.1:7878;
    server 192.168.10.121:3333 backup;  #热备
}
server {
    keepalive_requests 120; #单连接请求上限次数。
    listen       4545;   #监听端口
    server_name  127.0.0.1;   #监听地址       
    location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
        #root path;  #根目录
        #index vv.txt;  #设置默认页
        proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
        deny 127.0.0.1;  #拒绝的ip
        allow 172.18.5.54; #允许的ip           
    } 
}
```

> 五种实现方式

1. **轮询（默认）**

   轮询方式是Nginx负载默认的方式，顾名思义，所有请求都按照时间顺序分配到不同的服务上，如果服务Down掉，可以自动剔除，如下配置后轮训10001服务和10002服务。

   ```nginx
   upstream  dalaoyang-server {
          server    localhost:10001;
          server    localhost:10002;
   }
   ```

2. **权重weight**

   指定每个服务的权重比例，weight和访问比率成正比，通常用于后端服务机器性能不统一，将性能好的分配权重高来发挥服务器最大性能，如下配置后10002服务的访问比率会是10001服务的二倍。

   ```nginx
   upstream  dalaoyang-server {
          server    localhost:10001 weight=1;
          server    localhost:10002 weight=2;
   }
   ```

3. **ip_hash**（解决session问题）

   每个请求都根据访问ip的hash结果分配，经过这样的处理，每个访客固定访问一个后端服务，如下配置（ip_hash可以和weight配合使用）。

   上述方式存在一个问题就是说，在负载均衡系统中，假如用户在某台服务器上登录了，那么该用户第二次请求的时候，因为我们是负载均衡系统，每次请求都会重新定位到服务器集群中的某一个，那么***已经登录某一个服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的\***。
   我们可以采用***ip_hash\***指令解决这个问题，如果客户已经访问了某个服务器，当用户再次访问时，会将该请求通过***哈希算法，自动定位到该服务器\***。
   每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决***session的问题***。

   ```nginx
   upstream  dalaoyang-server {
          ip_hash; 
          server    localhost:10001 weight=1;
          server    localhost:10002 weight=2;
   }
   ```

4. **最少连接**

   将请求分配到连接数最少的服务上。

   ```nginx
   upstream  dalaoyang-server {
          least_conn;
          server    localhost:10001 weight=1;
          server    localhost:10002 weight=2;
   }
   ```

5. **fair**

   按后端服务器的响应时间来分配请求，响应时间短的优先分配。

   ```nginx
   upstream  dalaoyang-server {
          server    localhost:10001 weight=1;
          server    localhost:10002 weight=2;
          fair;  
   }
   ```

### 9、正向代理与反向代理

> 什么代理

代理其实就是一个中介，A和B本来可以直连，中间插入一个C，C就是中介。

> 正向代理

正向代理类似一个跳板机，代理访问外部资源

比如我们国内访问谷歌，直接访问访问不到，我们可以通过一个正向代理服务器，请求发到代理服，代理服务器能够访问谷歌，这样由代理去谷歌取到返回数据，再返回给我们，这样我们就能访问谷歌了

![下载](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210828231156.png)

![v2-922c89b735165f9f47db025bd77941c2_720w](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210828231736.png)

**正向代理的用途：**

1. 访问原来无法访问的资源，如google
2. 可以做缓存，加速访问资源
3. 对客户端访问授权，上网进行认证
4. 代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

总结一下，用最简单粗暴的说法：**「正向代理」指一对一或多对一，Server 不知道请求的 Client 都是哪些人。**

> 反向代理

反向代理隐藏了真实的服务端，当我们请求 [http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com) 的时候，背后可能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，[http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com) 就是我们的反向代理服务器，反向代理服务器会帮我们把请求转发到真实的服务器那里去。

![v2-102aad80326bbc9cd883c75a65bb147e_720w](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210828231625.jpg)

![v2-bf5255d84d73d10500cb9c717fac8b02_720w](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210828231702.png)

> 总结

|    区别    |                           正向代理                           |                           反向代理                           |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 代理的对象 |                            客户端                            |                            服务端                            |
|    关系    |                        一对一或一对多                        |                        一对多或多对多                        |
|    说明    | 正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端. | 反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端 |

### 10、常用配置

https://www.runoob.com/w3cnote/nginx-setup-intro.html

```nginx
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        charset      utf-8;	#中文
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

> gzip压缩

gzip压缩作用：将响应报⽂发送⾄客户端之前可以启⽤压缩功能，这能够有效地节约带宽，并提⾼响应⾄客户端的速度,压缩会消耗nginx的cpu性能

```nginx
server {
        listen 80;
        server_name test.com www.test.com;
        root /webroot/www;
        location ~ .*\.(jpg|gif|png|bmp)$ {
                gzip on;				#开启gzip压缩
                gzip_http_version 1.1;	#压缩协议版本
                gzip_comp_level 3;      # 压缩比率
                gzip_types text/plain application/json application/x-javascript application/css application/xml application/xml+rss text/javascript application/x-httpd-php image/jpeg image/gif image/png image/x-ms-bmp; 				#压缩类型，根据/usr/local/nginx/conf/mime.types中定义;
        }
}
```



### 11、Nginx与Tengine

Tengine是由淘宝网发起的Web服务器项目。它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如淘宝网，天猫商城等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。

原来是淘宝网发起的，这么了解也可以认为淘宝网在nginx的二次开发才是，那么淘宝网为什么要二次开发呢？原因是：针对大访问量网站的需求，并且更加的稳定，性能更加的强大！看到这么些话有没有心动的感觉？

区别在写一下：nginx和tengine的区别是：

1、tengine是在nginx上面开发的，包含了nginx的性能。

2、tengine更适合大访问量网站的需求，相比nginx更加的稳定，性能更加的强劲。

据网络测试：

Tengine相比Nginx默认配置，提升200%的处理能力。
Tengine相比Nginx优化配置，提升60%的处理能力。

![image-20210714083834793](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210714083834.png)

