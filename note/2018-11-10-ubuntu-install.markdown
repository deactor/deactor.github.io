---
sort: 5
title:  "vmware安装ubuntu18.04和shadowswork-qt5"
---
### vmware player12安装ubuntu18.04
#### 安装过程中碰到的问题：  
**问题1**：虚拟机创建后打开，自动启动ubuntu进行安装，进度条前进极度缓慢。  
该问题是我当时默认联网的，ubuntu安装时在访问网络下载东西，果断将虚拟机网断开连接。之后就快了很多了。  

**问题2**：上面的进度条走完后，ubuntu进入了黑屏状态，显示正在安装什么什么，正在删除临时目录之类的，相当的卡。我任由它黑了一晚上，也没安装成功，依旧卡着。之后google出了答案：  
>原因是：vmware安装ubuntu或者centos，会自作聪明提示“快速安装”，然后会使用一个autoinst.iso文件来快速安装，并且安装完系统后该重启了，它会插入一个操作：安装vm-tools，就是这个久久的卡住了。  
>解决方式就是：虚拟机创建后，不要勾选“创建后开启此虚拟机”，先不要安装系统，在设置的ubuntu安装目录中先把虚拟光驱加载的自动安装文件autoinst.iso找到，然后删除这个文件。此时启动虚拟机，进行安装系统就没有问题了。安装完系统后再手动的安装vmware-tools就可以了。

### ubuntu18.04 安装shadowsocks-qt5
**标准姿势**：
```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
但是在sudo apt-get update出现问题，提示:
```
错误:8 http://ppa.launchpad.net/hzwhuang/ss-qt5/ubuntu bionic Release          
  404  Not Found [IP: 91.189.95.83 80]
正在读取软件包列表... 完成                        
E: 仓库 “http://ppa.launchpad.net/hzwhuang/ss-qt5/ubuntu bionic Release” 没有 Release 文件。
```
上[ss-qt5官网](https://code.launchpad.net/~hzwhuang/+archive/ubuntu/ss-qt5)看一下，在"Adding this PPA to your system"下面点开"Technical details about this PPA"可以看到"Display sources.list entries for: "目前的源，没有ubuntu18的bionic。  
**解决方式**：  
在你的/etc/apt/sources.list.d目录(存放ppa源的地方)下，看这个文件(hzwhuang-ubuntu-ss-qt5-bionic.list)将里面的bionic改成上面提到的官网有的版本如artful，之后再sudo apt-get update就没有问题了。