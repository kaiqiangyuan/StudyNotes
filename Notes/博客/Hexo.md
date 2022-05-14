[*官方网址[^1]*](https://hexo.io/zh-cn/)



[toc]

### 1、使用

```sh
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

### 2、本地预览

```sh
hexo clean  #清除缓存文件 
hexo g	#生成网站静态文件到默认设置的 public 文件夹
hexo s #启动本地服务器,用于预览主题
```

### 3、部署到Nginx

```sh
server {
        listen       9090;
        root         /root/blog/public;
        index index.html index.html;

        location / {
                try_files $uri $uri/ /index.html;
        }
}
```

```sh
# 清除缓存
hexo clean
# 生成静态页面
hexo generate
# 将本地静态页面目录部署到云服务器
hexo delopy


#如果nginx错误，ubuntu中，修改 
vim /etc/nginx/nginx.conf 
#将最上面的use用户设置为root
```

### 参考网址

[^1]:https://hexo.io/zh-cn/

