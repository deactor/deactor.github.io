---
title:  "Manjaro配置Android11编译环境"
---

1. 参考[Manjaro下编译环境搭建](https://www.jianshu.com/p/6612d7966210)进行环境配置，主要以下几点：
   1. 换软件源
   2. 安装编译依赖包：`yay -S lineageos-devel`,如果没成功，则安装一下基础包`sudo pacman -S base-devel`
2. 安装openjdk-8, 最新manjaro版本自带的是jdk18，并按切换到jdk8。
3. 安装python3.7或3.8.自带的3.10太高了，repo运行不了。
   1. 安装3.7版本的python：yay -S python37
   2. 切换python到python3.7（将/usr/bin/python的软链接删除，重新创建对应的软连接，或者从一开始就用pyenv进行python的安装配置）：ln -s python3 /usr/bin/python3.7

>如果使用ProxyCommand nc报找不到nc，需要安装nc即：sudo pacman -S openbsd-netcat

>对于ubuntu，如果openssl版本过低或过高也是不行的，可以参考[openssl更新版本](https://www.johngo689.com/28206/)更换版本

编译错误解决：
1. 碰到一个错误，报找不到cpio指令，解决方案就是安装cpio：yay -S cpio