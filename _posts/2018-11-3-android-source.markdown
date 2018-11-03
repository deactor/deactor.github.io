---
layout: post
title:  "清华镜像网站下载android源码"
date:   2018-11-03 20:12:28 +0800
categories: android
---
#### 下载android源码方法：
我的系统是ubuntu 14.04，主要参考了blog[通过清华大学镜像下载Android源码并编译源码](https://www.cnblogs.com/shenchanghui/p/8503623.html)，通过国内镜像网站下载要比从google下载快的多  
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
我下载到的是此时最新的android P的源码。  
参考blog中提到的[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)的确是个好地方。我同时也在该网站中找到了ubuntu的软件源，并按提示修改了我ubuntu的软件源。