---
sort: 1
layout: post
title:  "jekyll搭建gitpage"
date:   2018-10-13 23:35:28 +0800
categories: jekyll for gitpage
---
#### 1.搭建gitpage  
参考gitpage官网，按照官网的步骤搭建： [gitpage官网](https://pages.github.com/) 
gitpage本质上还是一个github仓库，按照github官网的要求建立一个仓库之后，
github会读取你仓库中的html文件构建网页。
#### 2.使用github推荐的jekyll来搭建blog  
同样也是按照官网的指示操作，在gitpage的下方有Blogging with Jekyll，点击
Learn how to set up Jekyll.按照步骤进行搭建。如果是mac系统，安装起来是十
分轻松的，只要按照官网提示和mac系统提示操作即可，但是对于windows系统
这里就有几个坑要注意了。  
首先安装jekyll的第一步就是配置Ruby开发环境。要安装Ruby，RubyGems，gcc
和make，Ruby官网提供了一个RubyInstaller来在windows上安装Ruby，不得不
说，这个installer真的是出奇的不好用，反正我是死都安装不上或者说安装不完整
以上的Ruby开发环境，下载速度慢的要死，各种下载不下来。  
这个RubyInstaller安装Ruby还好，主要是卡在安装gcc和make上。下载msys2有问
题。既然没法一次安装到位，那就缺什么，手动安装什么。Ruby -v，gem -v都ok
之后，就剩下gcc -v，g++ -v，make -v了，这基本就靠msys2安装了。
>注意如果安装成功了，bin也有，但是命令 -v不成功，记得先配置下环境变量。

msys2也是直接上官网，按提示安装:[msys2官网](http://www.msys2.org/)。  
msys2有点像linux下的apt-get，安装后可以使用pacman -S 来安装工具和软件。可
能也需要像Linux一样修改下软件源，我是用了3个中科大的镜像。具体可参考这篇
[msys2总结](http://www.360doc.com/content/16/0514/16/496343_559090195.shtml)。  
然后jekyll是使用mingw来编译，我原本直接安装过mingw，但是没有用，执行gem 
install jekyll bundler会报错。所以就把原先装的mingw删除后使用msys2重新安装了
一遍mingw。在安装mingw时出现了如下之类的错误:  
>错误：无法提交处理 (有冲突的文件)  
mingw-w64-x86_64-libiconv: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-gmp: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-libwinpthread-git: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-gcc-libgfortran: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-gcc-libs: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-bzip2: 文件系统中已存在 /mingw64  
mingw-w64-x86_64-zlib: 文件系统中已存在 /mingw64  

解决方法是：需要手动创建mingw64目录，在执行msys2安装mingw就没问题了。
安装完mingw，gcc -v，g++ -v，make -v都ok就可以下一步了。执行gem install 
jekyll bundler正常，按照步骤操作下去，就能看到jekyll的项目了。_posts目录下的
markdown文件就是你的blog内容了。具体可以看jekyll的官网介绍。
这里有个中文的：[jekyll中文](https://www.jekyll.com.cn/)。  
此外在我完全安装正确Ruby开发环境之时，执行gem install jekyll bundler报错，如：
>ERROR:  Error installing jekyll:  
ERROR: Failed to build gem native extension.  
具体的错误点有：  
1.setjmpex.h no such file or directory  
2.cannot find -lgmp  
等等.

这些问题都可以根据提示，查看log文件，找到原因，就是因为gcc编译环境不对，用
msys2正确安装上mingw之后就正常了。
所以遇到报错还是要根据log判断出错原因，来完善安装和配置。