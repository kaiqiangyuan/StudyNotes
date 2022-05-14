[toc]

# 一、Linux安全

* 删除系统登录相关信息

  ```shell
  /etc/issue  /etc/issue.net  /etc/motd
  ```

* 更改shell命令历史记录

  ```shell
  # 编辑 vim /etc/bashrc   文件 
  HISTFILESIZE=4000 # 保存命令的记录行数（默认1000条）
  HISTSIZE=4000 #定义了history命令输出的记录总数
  HISTTIMEFORMAT='%F %F' # 时间格式
  export HISTTIMEFORMAT
  ```

* 锁定文件

  > root用户都无法操作

  ==chattr== 与 ==lsatter==

  ```shell
  # 查看文件属性等信息
  chattr -R 文件或者目录（递归修改文件属性信息）
  chattr -V 文件或者目录（显示修改内容）
  + 在原有参数设定基础上增加参数
  - 在原有参数设定基础上减少参数
  = 更新为指定参数
  a (append)只能向文件中添加数据，不能删除，常用于日志文件中（root用户设置这个属性）
  c (compress)文件经过压缩后再存储，读取时需要自动解压（耗时）
  i (immutable)文件不能被修改，删除，重命名，设定链接等操作，不能写入或新增内容
  s 删除文件或目录（不可恢复）
  u 与s相反，保留数据块（可以恢复）
  
  lsatter -R 文件或目录 （-R遍历查看所有的文件和目录属性）
  lsatter -V 文件或目录 （-V显示文件和目录版本）
  -a 列出目录中的所有文件，包括.开头
  -d 显示指定目录的属性
  ```


* 定时任务

  ==crontab== 命令

* 查看登录过系统的用户

  1. 直接使用 ==w== 命令 查看登陆过的用户，或者是 ==last==  命令
2. ![查看w命令内容](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/QQ20210321-150216@2x.png)
  3. 如果发现可疑用户则使用 ==passwd -l root== 将用户锁定
  4. 锁定后用户可能还处于登录状态，使用 ==/home# ps -ef|grep root== 获取 **pid** 后使用kill停止

* 查看TCP,UDP进程

  通过==fuser==指定的端口查看TCP,UDP进程PID

  ![根据端口查看TCP进程](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/QQ20210322-095301@2x.png)

  根据PID获取进程信息

  ![根据PID获取进程信息](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/QQ20210322-095703@2x.png)







