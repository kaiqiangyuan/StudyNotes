下载地址：http://visualvm.github.io/download.html

问题：安全打开后闪退问题

进入安装目录 `/Applications/VisualVM.app/Contents/Resources/visualvm/etc`

```sh
/Applications/VisualVM.app/Contents/Resources/visualvm/etc
yuankaiqiang@Mac etc % ls
visualvm.clusters visualvm.conf     visualvm.icns     visualvm.import
yuankaiqiang@Mac etc %   
```

修改 ==visualvm.conf==文件72行 修改为自己本地jdk路径

![image-20210722230205916](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210722230205.png)

下载插件Visual GC插件

![image-20210723155551773](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210723155551.png)

![image-20210723155533416](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210723155533.png)
