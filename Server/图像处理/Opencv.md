[toc]

### 一、安装（两种方式）

> 源码编译安装

[*主要参考*](https://blog.csdn.net/nlx1450161741/article/details/112262042?utm_medium=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-20.nonecase&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-20.nonecas)[^1]

mac下安装，使用源码进行安装时

1. **[*在官方上下载安装包*](https://opencv.org/releases/)**[^2]

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601130616.png)

下载后编译前需要下载各种依赖包啥的，版本的话下载可能有区别问题

使用brew安装的ant最新版本附带安装了openjdk15，为了避免在下面编译生成jar后时候用的是openjdk15进行编译，如果使用openjdk15进行编译，则jar包必须在jdk15的环境中才能运行，所以不建议使用brew进行安装ant，建议从官网上下载ant进行安装，[*网址*](https://ant.apache.org/bindownload.cgi)[^3]

2. **官网下载ant**下载了zip文件以后解压后将bin目录配置在系统的环境变量中即可，测试是否配置成功使用ant -version

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601130810.png)

```shell
brew install gcc git cmake pkg-config ffmpeg libgphoto2 libav libjpeg libpng libtiff libdc1394
```

3. **在opencv文件的根目录中创建build文件夹**

![111](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601130901.png)

4. **在build目录下执行**，这里可以自定义很多种方式，暂时没有了解那么多

```shell
cmake -DBUILD_SHARED_LIBS=OFF -DWITH_IPP=OFF -DCMAKE_INSTALL_PREFIX={/Users/yuankaiqiang/environment/opencv-4.5.2} ../
```

5. **然后再执行**，j后面写几根据自己机器的配置来写，一般写cpu的核数的两倍，我电脑是4核，所以是8

```shell
make -j8（等待，20分钟左右）
```

6. **然后再执行**

```shell
make install
```

以上命令都安装完成后的目录情况就算是安装完成了，jar包和动态链接的位置分别位于bin目录与lib目录下，如何需要配置环境变量的话则把build下的bin目录等配置到系统中就可以了

![123](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601131026.png)

![321](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601131042.png)

> 使用brew安装opencv（贼简单，但是有问题）

[*主要参考*](https://blog.csdn.net/enlangs/article/details/105291309)[^4]

```shell
1.查看系统中是否安装了xcode-select -version
没有则安装 xcode-select --install
2.安装 Ant（生成jar包必须工具！！！）
brew install ant
3.编译brew中的opencv，编译成jar包需要使用到的
brew edit opencv
-DBUILD_opencv_java=OFF 更改成 -DBUILD_opencv_java=ON
4.编译安装opencv
brew install --build-from-source opencv（等待，20分钟左右）
5.完成后再share中可以看到jar和动态链接库
```

==使用brew安装的问题，不能很好的进行版本控制，如何使用jar的话版本不好控制，因为编译安装opencv时候如果要生产jar包的话必须要使用brew安装的ant，官网上下载的编译的时候生成不了jar包，并且ant的最新的版本附带下载了openjdk15，在编译opencv的时候使用到了ant，因为编译生成的jar包是使用openjdk15进行编译的，也就是说这个jar包不能在低于jdk15以下的版本下进行使用==

```shell
因为使用jdk11进行编译，在jdk8的环境中运行将会报错
org/opencv/core/Core has been compiled by a more recent version of the Java Runtime (class file version 55.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

**==因此如果要在java中使用opencv，且使用的是jdk8的话，不推荐使用这种方式进行安装，如果是c++或者是别的语言进行开发则可以使用这种方式，xcode上使用的话需要导入lib库的文件，网上有教程==**

### 二、SpringBoot中使用

[*springboot中使用参考*](https://blog.csdn.net/qq_34814092/article/details/84990342)[^5]

[*rectangle函数与Rect函数的用法*](https://blog.csdn.net/Destiny_zc/article/details/106472118)[^6]

[*rectangle*](https://blog.csdn.net/wkk15903468980/article/details/52923216)[^7]

[*java_eclipse中添加外部动态链接库(dll文件)的三种方式*](https://blog.csdn.net/shyjhyp11/article/details/111874591)[^8]

> sts中opencv

1. 首先需要创建springboot项目或者是maven项目进行测试，测试代码如下，类的话自己创建

   ```shell
   	static {		
   		System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
   	}
   	public static void main(String[] args) {
   		System.out.println("Welcome to OpenCV " + Core.VERSION);
           //读取图像到矩阵中,取灰度图像
           Mat src = Imgcodecs.imread("/Users/yuankaiqiang/Downloads/b.png");
           //复制矩阵进入dst
           Mat dst = src.clone();
           Imgproc.resize(src, dst, new Size(src.width() * 0.1, src.height() * 0.1));
           Imgcodecs.imwrite("/Users/yuankaiqiang/Downloads/100000.png", dst);
   	}
   ```

2. 添加jar包

   ![888](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601131744.png)

3. 添加jar包与动态链接库

   ![999](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210601131809.png)

4. 启动程序

   ```shell
   Welcome to OpenCV 4.5.2 libpng warning: Application built with libpng-1.4.12 but running with 1.6.37 Exception in thread "main" CvException [org.opencv.core.CvException: cv::Exception: OpenCV(4.5.2) /Users/yuankaiqiang/environment/opencv-4.5.2/modules/imgproc/src/resize.cpp:4051: error: (-215:Assertion failed) !ssize.empty() in function 'resize' ] at org.opencv.imgproc.Imgproc.resize_3(Native Method) at org.opencv.imgproc.Imgproc.resize(Imgproc.java:4154) at com.yuankaiqiang.Sample.main(Sample.java:48)
   ```

   我的系统环境中可能是因为libpn版本与opencv中的libpng版本不一致的问题，所以如何使用png图片格式的话就会出现以下的错误，暂时还没有解决方案，目前使用源码编译安装opencv的时候就只有这一个问题，其余问题暂时还没有发现

### 参考网址

[^1]: https://blog.csdn.net/nlx1450161741/article/details/112262042?utm_medium=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-20.nonecase&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-20.nonecas
[^2]: https://opencv.org/releases/
[^3]: https://ant.apache.org/bindownload.cgi
[^4]: https://blog.csdn.net/enlangs/article/details/105291309
[^5]: https://blog.csdn.net/qq_34814092/article/details/84990342
[^6]: https://blog.csdn.net/Destiny_zc/article/details/106472118
[^7]: https://blog.csdn.net/wkk15903468980/article/details/52923216
[^8]: https://blog.csdn.net/shyjhyp11/article/details/111874591











