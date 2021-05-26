---
title:  "AndroidStudio编译错误处理"
---
#### 编译报错65535
解决：更新gradle build tools版本解决，尝试添加muiltdex并没有卵用，因为minSdkVersion配置为26，已经默认开启muiltdex了。
解决后的gradle版本：
```
//顶层build.gradle
classpath 'com.android.tools.build:gradle:4.1.3'
```
```
//gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-6.5-bin.zip
```
```
//模块build.gradle
android {
    compileSdkVersion 29

    defaultConfig {
        applicationId "xx.xx.xx"
        minSdkVersion 26
        targetSdkVersion 29
        versionCode 112
        versionName gitVersionName
    }

    buildTypes {
        release {
            postprocessing {
                removeUnusedCode false
                removeUnusedResources false
                obfuscate false
                optimizeCode false
                proguardFile 'proguard-rules.pro'
            }
        }
    }
}
```

#### NullPointerException(no error messge)
问题背景：之前一直可以正常编译，把代码copy到另一台电脑上，编译就出现了这个问题。唯一区别是这台电脑的as是4.2.1最新版本，之前那台电脑as是4.1版本。
解决：更新gradle build tools
```
报错：
Caused by: java.lang.NullPointerException 	at com.google.common.base.Preconditions.checkNotNull(Preconditions.java:782)
```
更新gradle build tools
```
//顶层build.gradle
classpath 'com.android.tools.build:gradle:4.2.1'
```
```
//gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-6.7.1-bin.zip
```
删除.gradle文件夹，重启as。问题解决。
