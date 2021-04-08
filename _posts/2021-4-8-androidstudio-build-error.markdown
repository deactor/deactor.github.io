---
layout: post
title:  "AndroidStudio编译错误处理"
date:   2021-4-8 17:35:28 +0800
categories: Practical Memo
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