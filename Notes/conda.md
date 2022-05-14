[系统pip问题](https://blog.csdn.net/meifannao789456/article/details/112171566)

使用：curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | python

```shell
opencv 添加环境变量
export PATH="/usr/local/opt/opencv@3/bin:$PATH"
export OpenCV_LIBS=/usr/local/opt/opencv@3/lib
export OpenCV_INCLUDE_DIRS=/usr/local/opt/opencv@3/include
```

```shell
查看环境        conda info -e

退出base环境    conda deactivate

创建环境        conda create -n tensorflow python=3.5 # test为环境名，3.5为python的版本号

激活conda环境   source activate tensorflow

切换conda环境  conda activate base

在condo环境中农安装TensorFlow    pip/conda install tensor flow/conda install tensorflow-gpu==2.1.0

删除环境        conda remove -n tensorflow —all

环境重命名      conda create -n tf --clone rcnn #把环境 rcnn 重命名成 tf
               conda remove -n rcnn --all
#conda 其实没有重命名指令，实现重命名是通过 clone 完成的，分两步：1.先 clone 一份 new name 的环境。2.删除 old name 的环境。

添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
设置搜索时显示通道地址 conda config --set show_channel_urls yes

查看当前的config     conda config —show

查看添加的镜像        conda config --get channels

删除源        conda config --remove-key channels
 conda config --remove-key channels
```

