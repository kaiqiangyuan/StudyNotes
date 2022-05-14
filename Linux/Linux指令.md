[toc]

# 一、常用指令

```shell
cd - 是返回到上一次的工作目录
df -h 查看磁盘空间大小
disabledu -sh  查看当前目录大小，在当前目录下使用
du -h --max-depth=1 查看当前目录下所有一级子目录文件夹大小
du -m --max-depth=1|sort -nr 按照目录大小进行倒序排序（以M为单位显示）
stat xxx  查看文件或目录的状态
ls -lh 查看当前文件下的所有文件大小（不能看到文件夹）
du -h 文件夹名称 | sort -hr 查看文件夹里面文件的大小并排序（不能看到文件）
ls -lS 查看目录下的文件大小（字节形式显示b）/1024k/1024m = 多少m
find / -name img.jpg 	查找名称为img.jpg文件

ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:" 获取ip
jps 显示当前所有java进程pid的命令
ps -aux 获取终端上所有用户的有关进程的所有信息
netstat -ntlp 查看当前所有tcp端口
netstat -lnp|grep 6379   命令可以看到端口
ps -ef|grep redis 查看进程信息
lsof -i:3000 (端口号)           查看端口的详细端口占用情况
lsof -i tcp:8099 查看TCP端口信息
kill -9 4324  //强制杀死PID为4324的进程
pkill -9 java //结束所有的 java 进程
ps aux | grep PID 根据进程ID获取对应进程的信息
ps -u --pid 8012(pid) 根据进程ID获取对应进程的信息 
pwdx 633（pid）根据进程id所在的位置
top -p pid 查看进程信息

nohup 你的shell命令 & 使程序在Linux下后台运行
unzip 解压zip文件
reboot 重启
curl cip.cc 查看服务器外网ip

#参考：https://blog.csdn.net/skh2015java/article/details/94012643
systemctl start xxx （启动）
systemctl stop xxx （停止）
systemctl restart xxx （重启）
systemctl reload xxx （不关闭的情况下重启载入配置文件）
systemctl enable xxx （设置开机自启动）
systemctl disable xxx （取消开启自启动）
systemctl status xxx （查看状态）
systemctl list-unit-files | grep enabled（查看开机自启动列表）
systemctl is-active xxx （查看有没有正在运行中）
systemctl is-enable xxx （查看开机时有没有默认要启用）
systemctl kill xxx （向运行 unit 的进程发送信号，不是结束进程）
systemctl show xxx （列出 unit 的配置）
systemctl mask xxx （注销 unit，注销后你就无法启动这个 unit 了）
systemctl unmask xxx （取消对 unit 的注销）
```

> 日志

```sh
#日志操作 tail、cat、head、sed、grep
tail notes.log         # 默认显示最后 10 行
tail -f notes.log       # 实时更新查看日志情况，显示最后十行
tail -n +20 notes.log   # 显示文件 notes.log 的内容，从第 20 行至文件末尾
cat -n notes.log        # 日志的每一行加上行号后打印输出，也可以输出到一个文件中，cat -n notes.log > new.log
cat -b notes.log		# 与-n一致，对于空白行不编号
cat -s notes.log		# 当遇到有连续两行以上的空白行，就代换为一行的空白行。
head notes.log         # 默认显示开头 10 行
head -n 5 notes.log    # 默认显示开头5行 
cat notes.log | head -n 10 | tail -n +5  # 显示5行到10行
sed -n '5,10p' notes.log       			 # 显示5行到10行，sed命令可以用来更改一下文件的操作
grep -i "hello" notes.log　　				# notes.log中包含hello的所有行，i为忽略大小写
cat -n notes.log | grep -i "你好"  		# 显示查询到的字符在第几行
cat -n notes.log | grep "你好" -C 10		# 根据关键字查看前后10行日志，并显示出行号，-A为前几行，-B为后几行
less [选项] 文件     # 分页显示命令
```

| less参数 | https://www.runoob.com/linux/linux-comm-less.html |
| :------: | :-----------------------------------------------: |
|    -e    |            当文件显示结束后，自动离开             |
|    -i    |                忽略搜索时的大小写                 |
|    -N    |                  显示每行的行号                   |
|    -s    |                显示连续空行为一行                 |

|  符号   |                 描述                 |
| :-----: | :----------------------------------: |
| /字符串 |        向下搜索“字符串”的功能        |
| ?字符串 |        向上搜索“字符串”的功能        |
|    n    |   重复前一个搜索（与 / 或 ? 有关）   |
|    N    | 反向重复前一个搜索（与 / 或 ? 有关） |
|    b    |              向前翻一页              |
|    d    |              向后翻半页              |
|    q    |            退出 less 命令            |
| 空格键  |              向后翻一页              |
| 向上键  |             向上翻动一行             |
| 向下键  |             向下翻动一行             |

```sh
nmap -p 1-10000 114.55.26.230 #查看ip开放端口信息
```

![nmap信息](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531095341.png)

# 二、vim或默认终端操作

```shell
# 显示当前行行号，在VI的命令模式下输入
:nu
# 显示所有行号，在VI的命令模式下输入(number)
:set nu
# vim下搜索xxx，命令模式下使用，n下一个匹配，N上一个匹配
:/xxx

#vim 未编辑情况中
vim移动光标至=》开头：gg
vim移动光标至=》末尾：shift+g
vim跳转到指定行数：xxgg
0 所行行首
$ 所行行尾
dd快速删除一行
# 终端中
ctrl+a可以快速跳转到终端首端
ctrl+e可以快速跳转到终端尾端
ctrl+u删除光标至行首的所有命令
ctrl+k删除光标至命令尾的所有命令
ctrl+l清屏
ctrl+c删除整行
# 进程中
ctrl+c 发送 SIGINT 信号给前台进程组中的所有进程。常用于终止正在运行的程序。
ctrl+z 发送 SIGTSTP 信号给前台进程组中的所有进程，常用于挂起一个进程。
ctrl+d 不是发送信号，而是表示一个特殊的二进制值，表示 EOF。
ctrl+\ 发送 SIGQUIT 信号给前台进程组中的所有进程，终止前台进程并生成 core 文件。
```

# 三、设置服务器中文显示

[参考](https://blog.csdn.net/weixin_44129085/article/details/105176429)

```shell
sudo apt install language-pack-zh-hans-base language-pack-zh-hans    # 下载安装相关语言包
sudo update-locale LANG=zh_CN.UTF-8 LANGUAGE="zh_CN:zh"    # 设置语言为简体中文
# 设置语言为英文  sudo update-locale LANG=en_US.UTF-8 LANGUAGE="en_US:en"
# 实际修改的是 cat /etc/default/locale
source /etc/default/locale    # 应用修改
date # 查看是否修改成功 2021年 05月 13日 星期四 21:07:19 CST
```

# 四、上下传文件

```sh
#文件
scp 本地文件路径 root@114.55.26.230:/home/ 
#文件夹
scp -r 本地文件夹路径 root@114.55.26.230:/home/

# 从服务端发送文件到客户端： 
apt install lrzsz
sz 
# 从客户端上传文件到服务端： 
rz 
# 在弹出的框中选择文件，上传文件的用户和组是当前登录的用户
```

# 五、服务器定时开机自启动

修改 `vi /etc/crontab`
增加`00 1    * * *   root    init 6`

每天凌晨一点自动重启

![image-20210606195202714](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210606195202.png)

```sh
init命令用于改变操作系统的运行级别。
Init 6是重新启动机器。
reboot也是重新启动机器。
"init 6" 基于一系列/etc/inittab文件，并且每个应用都会有一个相应shutdown脚本。
'init 6' 调用一系列shutdown脚本(/etc/rc0.d/K*)来使系统优雅关机;
'reboot'并不执行这些过程，reboot更是一个kernel级别的命令，不对应用使用shutdown脚本。 
在出问题的状况下或强制重启时使用reboot
```

# 六、查看文件目录结构

tree 命令：https://www.runoob.com/linux/linux-comm-tree.html

- -a 显示所有文件和目录。
- -A 使用ASNI绘图字符显示树状图而非以ASCII字符组合。
- -C 在文件和目录清单加上色彩，便于区分各种类型。
- -d 显示目录名称而非内容。
- -D 列出文件或目录的更改时间。
- -f 在每个文件或目录之前，显示完整的相对路径名称。
- -F 在执行文件，目录，Socket，符号连接，管道名称名称，各自加上"*","/","=","@","|"号。
- -i 不以阶梯状列出文件或目录名称。
- -L level 限制目录显示层级。
- -l 如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。
- -n 不在文件和目录清单加上色彩。
- -N 直接列出文件和目录名称，包括控制字符。
- ==-p 列出权限标示。==
- -P<范本样式> 只显示符合范本样式的文件或目录名称。
- -q 用"?"号取代控制字符，列出文件和目录名称。
- ==-s 列出文件或目录大小。==
- -t 用文件和目录的更改时间排序。
- -u 列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码。
- -x 将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该子目录予以排除在寻找范围外。

```sh
#ubuntu安装
apt install tree
root@Ubuntu:/home/file# tree /home/file/
/home/file/
├── img.jpg
└── PersonalWebsiteFile
    ├── BasicUserPicture
    └── CacheFile
#mac安装
brew install tree
```

# 七、进程快捷键（ctrl+c等）

![20210902204904](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210902205634.jpg)

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210902205824.jpg" alt="20210818100201" style="zoom:50%;" />





