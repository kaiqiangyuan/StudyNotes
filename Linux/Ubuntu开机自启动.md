> Ubuntu18.0.4

[参考1](https://www.r9it.com/20180613/ubuntu-18.04-auto-start.html) [参考2](https://www.liangzl.com/get-article-detail-29559.html) [参考3](https://blog.csdn.net/D15306354614/article/details/112743649)

1. 查看==/lib/systemd/system==的==rc.local.service==文件（没有则创建）

2. 默认

   ```shell
   #  This file is part of systemd.
   #
   #  systemd is free software; you can redistribute it and/or modify it
   #  under the terms of the GNU Lesser General Public License as published by
   #  the Free Software Foundation; either version 2.1 of the License, or
   #  (at your option) any later version.
   
   # This unit gets pulled automatically into multi-user.target by
   # systemd-rc-local-generator if /etc/rc.local is executable.
   [Unit]
   Description=/etc/rc.local Compatibility
   ConditionFileIsExecutable=/etc/rc.local
   After=network.target
   
   [Service]
   Type=forking
   ExecStart=/etc/rc.local start
   TimeoutSec=0
   RemainAfterExit=yes
   ```

增加

```shell
[Install]  
WantedBy=multi-user.target  
Alias=rc-local.service
```

* [Unit] 段: 启动顺序与依赖关系

* [Service] 段: 启动行为,如何启动，启动类型

* Install] 段: 定义如何安装这个配置文件，即怎样做到开机启动

3. 创建

   ```shell
   sudo touch /etc/rc.local
   #加上权限
   sudo chmod +x /etc/rc.local
   #建立连接
   ln -s /lib/systemd/system/rc.local.service /etc/systemd/system/
   
   #里面放测试脚本，运行/home/script  目录下的start.sh脚本
   #!/bin/bash
   
   # test.sh
   
   cd /home/script/
   
   sh start.sh
   
   exit 0
   ```

   4. 在==/home/script/==的start.sh脚本中编写sh脚本即可

   5. 自启动jar包（加上环境）

      ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531101729.png)

> ubuntu20.0.4
>
> 跟上面一样，修改的是==/lib/systemd/system==的rc-local.service文件

