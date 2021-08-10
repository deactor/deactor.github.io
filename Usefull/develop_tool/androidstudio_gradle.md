---
title:  "gradle，gradle插件，sdkbuildtool的作用及相互关系"
---
## gradle:
> 项目构建工具。可以管理项目中的差异，依赖，编译，打包等。

## sdk build tool:
> 编译工具，包含aapt，d8等，用来编译android项目。每个sdk都有对应的build tool。

## gradle plugin: 
>针对Gradle发行版和sdk build tool封装的一个工具，主要两大功能：
>1. 调用Gradle本身的代码和批处理工具来构建项目  
>2. 调用Android SDK的编译、打包功能

## 三者间的关系：
gradle plugin对应一个或多个gradle和sdk build tool的版本。三者的依赖版本不一致就会出现编译构建问题。依赖关系可查[Android Gradle 插件版本说明](https://developer.android.com/studio/releases/gradle-plugin?hl=zh-cn)


## 参考：
[详解Gradle发行版本、Gradle插件版本以及Android SDK Build Tools版本之间的关系](http://www.blogdaren.com/post-2418.html)