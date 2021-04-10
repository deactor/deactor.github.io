---
sort: 4
title:  "清华镜像网站下载android源码并编译"
---
#### 下载android源码方法：
我的系统是ubuntu 18.04，主要参考了blog[通过清华大学镜像下载Android源码并编译源码](https://www.cnblogs.com/shenchanghui/p/8503623.html)，通过国内镜像网站下载要比从google下载快的多  
按照一下步骤操作即可：
1. 终端输入一下命令:
    ```
    mkdir ~/bin
    PATH=~/bin:$PATH
    curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo   #使用tuna的git-repo镜像
    chmod a+x ~/bin/repo
    ``` 
1. 打开bin文件夹下的repo文件,将
    ```
    REPO_URL = 'https://gerrit.googlesource.com/git-repo'
    ```
    改为
    ```
    REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
    ```
3. 使用每月更新的初始化包，使用方法如下(repo sync的时候使用-j进行多线程下载时参考下清华镜像的说明，不要太高，我用的是加j4)：
    ```
    wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
    tar xvf aosp-latest.tar
    cd aosp# 解压得到的 aosp工程目录
    # 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
    repo sync # 正常同步一遍即可得到完整目录
    # 或 repo sync -l 仅checkout代码
    ```
    >如果repo sync时提示"/usr/bin/env: ‘python’: No such file or directory"则去/usr/bin目录下看下有没有python，我的是有python3且是python3.6没有python,因此只能执行python3,而执行不了python。  
    所以执行该条命令"sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.6 1"后就可以执行python了。
    
我下载到的是此时最新的android P的源码。  
参考blog中提到的[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)的确是个好地方。我同时也在该网站中找到了ubuntu的软件源，并按提示修改了我ubuntu的软件源。

#### ubuntu18.04编译android9.0
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

整个编译流程就是上面这样。但是在编译时报错了。  
**第一个问题是内存不足。**  
我用的虚拟机，配置是4G内存+2G的SWAP分区。从ubuntu自带的系统监视器可以看到在编译时，内存和SWAP满了，不够用。  
解决方法就是扩展SWAP分区。ubuntu18.04的交换分区是用文件做的，默认是/swapfile。我们可以自己创建。如下：
1. 创建交换文件(在当前目录创建16G的swap文件)
    ```
    sudo fallocate -l 16G swap
    ``` 
2. 设置swap为swap文件系统
    ``` 
    sudo mkswap -f swap
    ``` 
3. 开启swap
    ``` 
    sudo swapon swap
    ``` 
4. 关闭和删除原来的swapfile(也可以只关闭不删除)
    ``` 
    sudo swapoff  /swapfile
    sudo rm /swapfile
    ``` 
5. 设置开机启动
    ``` 
    sudo vim /etc/fstab
    将里面的swapfile改为swap
    ``` 

**第二个问题是存储空间不足。**  
我用的虚拟机一开始分配的是150G，装完系统和一些软件，拉完android9.0代码，最后可用空间剩余50G。但是依然不够阿。只能进行分区扩展。(这个网上详细教程一堆)
1. 先设置虚拟机，对虚拟机进行硬盘扩展。
2. 启动ubuntu，对ubuntu进行分区扩展，这里使用ubuntu18.04自带的磁盘工具即可，操作方便。直接选择磁盘，选择调整大小，进行扩展(这里会清除磁盘，可能造成数据丢失，没有关系，虚拟机没有影响，尽管扩展。)瞬间完成。

**第三个问题是OutOfMemoryError**  
编译到“//frameworks/base:api-stubs-docs Metalava”时报Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
这个问题我暂时没解决掉，网上有设置jack-server的，但是android9.0代码里没看到jack-server。
