---
title:  "国内镜像网站下载android源码并编译"
---
> [Android官网](https://source.android.google.cn/setup/build/initializing?hl=zh-cn)已经写的很详细了。此处记录一些要点。  

#### 下载android源码方法：
有两个下载源供选择，通过国内镜像网站下载要比从google下载快的多：
+ [清华大学镜像下载Android源码并编译源码](https://www.cnblogs.com/shenchanghui/p/8503623.html)
+ [中科大镜像下载Android源码并编译源码](https://mirrors.ustc.edu.cn/help/aosp.html)

>以上两个镜像源都提供了两种方式：初始化包和传统方法。我使用的是传统方法，使用初始化包没搞明白怎么切换分支，我是想编pixel4对应的版本，驱动程序需要根据对应的分支进行选择，之前使用初始化包，编出来的版本不能用，最后选择传统方法。

>注：编译的代码要刷真机，则还在编译时要添加对应的驱动文件[Google驱动程序](https://developers.google.cn/android/drivers?hl=zh-cn)。  

#### ubuntu18.04编译
**ubuntu18.04编译环境配置**：  
按照[android官网](https://source.android.google.cn/setup/build/initializing)要求：  
安装openjdk:
```
sudo apt-get install openjdk-8-jdk
```

安装所需的软件包:
```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip
```

**编译命令**：
```
source build/envsetup.sh
lunch aosp_arm-eng(也可以不带这个参数，在lunch执行后进行选择)
make -jn (n根据电脑配置选择，我用的是j4)
```

如果想删除之前的编译，可以：
```
make clobber
```
#### macOS 编译
系统版本：12.3.1  
安装openJDK8
```
brew install adoptopenjdk8
```
  
安装XCode  
AppStore安装，我安装时的版本是13.3.1。
  
以上即macOS的编译环境
>自 2021 年 6 月 22 日起,google不再支持macOS编译aosp,个人测试还是可以编的，今天2022.5.1，编译的Android12.  

#### 烧写版本
进入fastboot模式
```
adb reboot bootloader
```
刷机(-w会擦除设备上的 /data 分区)
```
fastboot flashall -w
```
  
#### 编译问题记录
[aosp 编译过程中Jack server SSL error 错误解决方法](https://segmentfault.com/a/1190000039970343)  
[MAC 下编译 ANDROID P 源码 提示 internal error: Could not find a supported mac sdk](https://www.cnblogs.com/larack/p/9646860.html)
