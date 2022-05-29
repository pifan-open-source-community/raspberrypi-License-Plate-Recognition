

此开源项目由树莓派爱好者基地的谢远伦、赖乾恩、高强，共同协作完成。在此感谢各位成员的付出与努力。正是有各位的付出，树莓派生态才能越来越丰富！

```bash
代码仓库
1、码云Gitee：https://gitee.com/yangkun_monster/raspberrypi-Garbage-classification
2、Github：https://github.com/pifan-open-source-community/garbage-Classification
视频教程地址：
哔哩哔哩bilibili：树莓派爱好者基地
```

## 一、项目概述
该项目主要用车牌识别算法对图像或视频中的车牌进行识别。首先进行车牌区域检测，然后将检测后的图像区域进行文本识别操作，反馈车牌的识别结果的同时将车牌区域进行框选，并将识别结果显示在对应车辆上方，后期移植在树莓派上进行实时视频流的车牌检测和识别。

前期：主要考虑PC端性能，并尽可能优化模型大小，训练可采用GPU，但调用模型测试的时候用CPU运行，测试帧率和准确性（测试10张左右图像的运行时间取平均值或实时视频流的帧率）。

后期：部署在树莓派端，在本地进行USB摄像头实时视频流的车牌识别。

编程语言： python。

Demo展示：输入单张图像或USB摄像头实时视频流，在显示器上显示出识别的结果（区域框选+识别结果）。

## 二、车牌识别系统简介
EasyPR是一个中文的开源车牌识别系统，其目标是成为一个简单、灵活、准确的车牌识别引擎。
相比于其他的车牌识别系统，EasyPR有如下特点：

 - 它基于openCV这个开源库，这意味着所有它的代码都可以轻易的获取。
 - 它能够识别中文，例如车牌为苏EUK722的图片，它可以准确地输出std:string类型的"苏EUK722"的结果。
 - 它的识别率较高。目前情况下，字符识别已经可以达到90%以上的精度。

## 三、树莓派端部署
### 1、烧录系统
下载树莓派官方镜像

```bash
http://raspberrypi.org/software/operating-systems/
```

版本Raspberry Pi OS with desktop and recommended software
下载后并解压缩

然后烧录镜像烧录系统。这里如果不会可以查看树莓派爱好者基地之前的视频教程，哈哈哈哈哈。

### 2、树莓派换源
输入命令


```powershell
sudo nano /etc/apt/sources.list
```


将其他源 前面加上 # 注释掉
下面粘贴科大源


```powershell
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```


输入 ctrl+o 回车保存，再ctrl+x 退出
输入命令


```powershell
sudo apt-get update
```

### 3、安装cmake


```powershell
sudo apt-get install cmake
```

检查版本

```powershell
cmake -version
```


出现版本信息则安装成功！
下载pkg-config
```powershell
wget http://pkgconfig.freedesktop.org/releases/pkg-config-0.29.2.tar.gz
```
解压

```powershell
tar -zxvf pkg-config-0.29.2.tar.gz
```
进入解压目录

```powershell
cd pkg-config-0.29.2
```

依次执行下面的命令

```powershell
./configure --with-internal-glib
make
make check
sudo make install
```

如果第三条报错，把路径改为全英文并且重新解压执行，并在第三条命令前加上sudo再执行。）

### 4、安装其它依赖
通过软件源安装libopenexr-dev

```powershell
sudo apt-get install libopenexr-dev
```
### 5、配置opencv
下载OpenCV - 3.2.0

```powershell
wget https://github.com/Itseez/opencv/archive/3.2.0.zip
```

使用unzip解压，没有unzip的先安装unzip。

```powershell
sudo apt-get install unzip
```

解压

```powershell
unzip 3.2.0.zip
```

进入opencv目录

```powershell
cd opencv-3.2.0
```

创建release目录

```powershell
mkdir release
```

进入release目录，安装OpenCV时，所有的文件都会被放到这个release目录下

```powershell
cd release
```

使用cmake编译OpenCV的源码，安装到/usr/local/目录下

```powershell
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local/ ..
```

编译安装

```powershell
make 
sudo make install
```

如果出现此错误：

```powershell
make[2]: *** [modules/python3/CMakeFiles/opencv_python3.dir/build.make:180：modules/python3/CMakeFiles/opencv_python3.dir/__/src2/cv2.cpp.o] 错误 1  
make[1]: *** [CMakeFiles/Makefile2:6687：modules/python3/CMakeFiles/opencv_python3.dir/all] 错误 2  
make: *** [Makefile:163：all] 错误 2
```

解决方法:
编辑 opencv-3.2.0/modules/python/src2/cv2.cpp 文件，更改第730行：

```powershell
bool pyopencv_to(PyObject* obj, String& value, const char* name)  
{  
    (void)name;  
    if(!obj || obj == Py_None)  
        return true;  
    char* str = (char *)PyString_AsString(obj);//这是文件的第730行，更改这行，在=后面加(char *)
    if(!str)  
        return false;  
    value = String(str);  
    return true;  
```

再重新编译安装就好了。
### 6、EasyPR安装
在树莓派爱好者基地微信公众号发送【EasyPR】就可以获得压缩包。
解压后上传到/home/pi目录下。
进入到EasyPR的目录：

```powershell
cd EasyPR
```

直接执行命令:

```powershell
./build.sh
```

如果出现此错误：

```powershell
make[2]: *** [CMakeFiles/easypr.dir/build.make:141：CMakeFiles/easypr.dir/src/core/plate_judge.cpp.o] 错误 1  
make[1]: *** [CMakeFiles/Makefile2:73：CMakeFiles/easypr.dir/all] 错误 2  
make[1]: *** 正在等待未完成的任务....  
[ 48%] Linking CXX static library libthirdparty.a  
[ 48%] Built target thirdparty  
make: *** [Makefile:84：all] 错误 2
```

解决方法：
修改EasyPR/include/easypr/config.h文件的第四行：

```powershell
#ifndef EASYPR_CONFIG_H_  
#define EASYPR_CONFIG_H_  
  
#define CV_VERSION_THREE_TWO//修改这一行，将#define CV_VERSION_THREE_ZERO改为#define CV_VERSION_THREE_TWO
```

再编译，出现[100%] Built target demo之后即编译成功！
测试直接运行dome

```powershell
./demo
```

输出以下信息：

```powershell
  EasyPR Option:  
1. 测试;  
2. 批量测试(推荐);  
3. SVM训练;  
4. ANN训练;  
5. 中文字符训练;  
6. 生成字符;  
7. 感谢名单;  
8. 退出;  
```

请选择一项操作:
选择“2批量测试”，可进行多张照片识别。
可以在EasyPR/resources/image目录下对general_test和native_test这两个文件夹中的照片进行替换，换成自己的车牌照片。
