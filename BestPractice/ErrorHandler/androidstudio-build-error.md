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

----

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

----

#### android library编译打包，class.jar是空的，不能用
检查是否进行了混淆等，minifyEnabled需要为false，否则打包出来的就是空的，血的教训。

----

#### 编译报错：Manifest merger failed
```
Manifest merger failed : Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91
	is also present at [androidx.core:core:1.3.1] AndroidManifest.xml:24:18-86 value=(androidx.core.app.CoreComponentFactory).
	Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:5:5-19:19 to override.
```
就是AndroidSupport和Androidx包冲突,原build.gradle的依赖如下：
```
//noinspection GradleCompatible
implementation 'com.android.support:appcompat-v7:28.0.0'
implementation 'com.android.support.constraint:constraint-layout:1.1.3'
implementation 'com.google.android.material:material:1.3.0'
testImplementation 'junit:junit:4.+'
androidTestImplementation 'com.android.support.test:runner:1.0.2'
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
```
这个noinspection GradleCompatible(不检查gradle兼容)，实际就是意味着有兼容问题发生了。  
但是我的依赖中没有Androidx，那只可能是依赖包中有依赖Androidx。使用命令查看包依赖信息：
```
//在项目根目录下有gradlew命令文件，就在该目录下执行如下命令,moduleName写自己的模块名
./gradlew moduleName:dependencies
```
信息如下：
```
+--- com.android.support:appcompat-v7:28.0.0
|    +--- com.android.support:support-annotations:28.0.0
|    +--- com.android.support:support-compat:28.0.0
|    |    +--- com.android.support:support-annotations:28.0.0
|    |    +--- com.android.support:collections:28.0.0
|    |    |    \--- com.android.support:support-annotations:28.0.0
|    |    +--- android.arch.lifecycle:runtime:1.1.1
|    |    |    +--- android.arch.lifecycle:common:1.1.1
|    |    |    |    \--- com.android.support:support-annotations:26.1.0 -> 28.0.0
|    |    |    +--- android.arch.core:common:1.1.1
|    |    |    |    \--- com.android.support:support-annotations:26.1.0 -> 28.0.0
|    |    |    \--- com.android.support:support-annotations:26.1.0 -> 28.0.0
|    |    \--- com.android.support:versionedparcelable:28.0.0
|    |         +--- com.android.support:support-annotations:28.0.0
|    |         \--- com.android.support:collections:28.0.0 (*)
|    +--- com.android.support:collections:28.0.0 (*)
|    +--- com.android.support:cursoradapter:28.0.0
|    |    \--- com.android.support:support-annotations:28.0.0
|    +--- com.android.support:support-core-utils:28.0.0
|    |    +--- com.android.support:support-annotations:28.0.0
|    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    +--- com.android.support:documentfile:28.0.0
|    |    |    \--- com.android.support:support-annotations:28.0.0
|    |    +--- com.android.support:loader:28.0.0
|    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    +--- android.arch.lifecycle:livedata:1.1.1
|    |    |    |    +--- android.arch.core:runtime:1.1.1
|    |    |    |    |    +--- com.android.support:support-annotations:26.1.0 -> 28.0.0
|    |    |    |    |    \--- android.arch.core:common:1.1.1 (*)
|    |    |    |    +--- android.arch.lifecycle:livedata-core:1.1.1
|    |    |    |    |    +--- android.arch.lifecycle:common:1.1.1 (*)
|    |    |    |    |    +--- android.arch.core:common:1.1.1 (*)
|    |    |    |    |    \--- android.arch.core:runtime:1.1.1 (*)
|    |    |    |    \--- android.arch.core:common:1.1.1 (*)
|    |    |    \--- android.arch.lifecycle:viewmodel:1.1.1
|    |    |         \--- com.android.support:support-annotations:26.1.0 -> 28.0.0
|    |    +--- com.android.support:localbroadcastmanager:28.0.0
|    |    |    \--- com.android.support:support-annotations:28.0.0
|    |    \--- com.android.support:print:28.0.0
|    |         \--- com.android.support:support-annotations:28.0.0
|    +--- com.android.support:support-fragment:28.0.0
|    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    +--- com.android.support:support-core-ui:28.0.0
|    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    +--- com.android.support:support-core-utils:28.0.0 (*)
|    |    |    +--- com.android.support:customview:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    \--- com.android.support:support-compat:28.0.0 (*)
|    |    |    +--- com.android.support:viewpager:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    |    \--- com.android.support:customview:28.0.0 (*)
|    |    |    +--- com.android.support:coordinatorlayout:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    |    \--- com.android.support:customview:28.0.0 (*)
|    |    |    +--- com.android.support:drawerlayout:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    |    \--- com.android.support:customview:28.0.0 (*)
|    |    |    +--- com.android.support:slidingpanelayout:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    |    \--- com.android.support:customview:28.0.0 (*)
|    |    |    +--- com.android.support:interpolator:28.0.0
|    |    |    |    \--- com.android.support:support-annotations:28.0.0
|    |    |    +--- com.android.support:swiperefreshlayout:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    +--- com.android.support:support-compat:28.0.0 (*)
|    |    |    |    \--- com.android.support:interpolator:28.0.0 (*)
|    |    |    +--- com.android.support:asynclayoutinflater:28.0.0
|    |    |    |    +--- com.android.support:support-annotations:28.0.0
|    |    |    |    \--- com.android.support:support-compat:28.0.0 (*)
|    |    |    \--- com.android.support:cursoradapter:28.0.0 (*)
|    |    +--- com.android.support:support-core-utils:28.0.0 (*)
|    |    +--- com.android.support:support-annotations:28.0.0
|    |    +--- com.android.support:loader:28.0.0 (*)
|    |    \--- android.arch.lifecycle:viewmodel:1.1.1 (*)
|    +--- com.android.support:support-vector-drawable:28.0.0
|    |    +--- com.android.support:support-annotations:28.0.0
|    |    \--- com.android.support:support-compat:28.0.0 (*)
|    \--- com.android.support:animated-vector-drawable:28.0.0
|         +--- com.android.support:support-vector-drawable:28.0.0 (*)
|         \--- com.android.support:support-core-ui:28.0.0 (*)
+--- com.android.support.constraint:constraint-layout:1.1.3
|    \--- com.android.support.constraint:constraint-layout-solver:1.1.3
+--- com.google.android.material:material:1.3.0
|    +--- androidx.annotation:annotation:1.0.1 -> 1.1.0
|    +--- androidx.appcompat:appcompat:1.1.0 -> 1.2.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.3.0 -> 1.3.1
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.1.0
|    |    |    |    +--- androidx.lifecycle:lifecycle-common:2.1.0
|    |    |    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    |    |    +--- androidx.arch.core:core-common:2.1.0
|    |    |    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.versionedparcelable:versionedparcelable:1.1.0
|    |    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0
|    |    |    |         \--- androidx.annotation:annotation:1.1.0
|    |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    |    +--- androidx.cursoradapter:cursoradapter:1.0.0
|    |    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    +--- androidx.fragment:fragment:1.1.0
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|    |    |    +--- androidx.collection:collection:1.1.0 (*)
|    |    |    +--- androidx.viewpager:viewpager:1.0.0
|    |    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    |    +--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |    |    |    \--- androidx.customview:customview:1.0.0
|    |    |    |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    |         \--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |    |    +--- androidx.loader:loader:1.0.0
|    |    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    |    +--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |    |    |    +--- androidx.lifecycle:lifecycle-livedata:2.0.0
|    |    |    |    |    +--- androidx.arch.core:core-runtime:2.0.0
|    |    |    |    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    |    |    |    \--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    |    +--- androidx.lifecycle:lifecycle-livedata-core:2.0.0
|    |    |    |    |    |    +--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    |    |    +--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    |    |    \--- androidx.arch.core:core-runtime:2.0.0 (*)
|    |    |    |    |    \--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    \--- androidx.lifecycle:lifecycle-viewmodel:2.0.0 -> 2.1.0
|    |    |    |         \--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.activity:activity:1.0.0
|    |    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    |    +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|    |    |    |    +--- androidx.lifecycle:lifecycle-runtime:2.1.0 (*)
|    |    |    |    +--- androidx.lifecycle:lifecycle-viewmodel:2.1.0 (*)
|    |    |    |    \--- androidx.savedstate:savedstate:1.0.0
|    |    |    |         +--- androidx.annotation:annotation:1.1.0
|    |    |    |         +--- androidx.arch.core:core-common:2.0.1 -> 2.1.0 (*)
|    |    |    |         \--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.1.0 (*)
|    |    |    \--- androidx.lifecycle:lifecycle-viewmodel:2.0.0 -> 2.1.0 (*)
|    |    +--- androidx.appcompat:appcompat-resources:1.2.0
|    |    |    +--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.core:core:1.0.1 -> 1.3.1 (*)
|    |    |    +--- androidx.vectordrawable:vectordrawable:1.1.0
|    |    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    |    +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|    |    |    |    \--- androidx.collection:collection:1.1.0 (*)
|    |    |    \--- androidx.vectordrawable:vectordrawable-animated:1.1.0
|    |    |         +--- androidx.vectordrawable:vectordrawable:1.1.0 (*)
|    |    |         +--- androidx.interpolator:interpolator:1.0.0
|    |    |         |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |         \--- androidx.collection:collection:1.1.0 (*)
|    |    +--- androidx.drawerlayout:drawerlayout:1.0.0
|    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    +--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |    |    \--- androidx.customview:customview:1.0.0 (*)
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.cardview:cardview:1.0.0
|    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    +--- androidx.coordinatorlayout:coordinatorlayout:1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|    |    +--- androidx.customview:customview:1.0.0 (*)
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.constraintlayout:constraintlayout:2.0.1
|    |    +--- androidx.appcompat:appcompat:1.2.0 (*)
|    |    +--- androidx.core:core:1.3.1 (*)
|    |    \--- androidx.constraintlayout:constraintlayout-solver:2.0.1
|    +--- androidx.core:core:1.2.0 -> 1.3.1 (*)
|    +--- androidx.dynamicanimation:dynamicanimation:1.0.0
|    |    +--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |    +--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    |    \--- androidx.legacy:legacy-support-core-utils:1.0.0
|    |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |         +--- androidx.core:core:1.0.0 -> 1.3.1 (*)
|    |         +--- androidx.documentfile:documentfile:1.0.0
|    |         |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |         +--- androidx.loader:loader:1.0.0 (*)
|    |         +--- androidx.localbroadcastmanager:localbroadcastmanager:1.0.0
|    |         |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |         \--- androidx.print:print:1.0.0
|    |              \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    +--- androidx.annotation:annotation-experimental:1.0.0
|    +--- androidx.fragment:fragment:1.0.0 -> 1.1.0 (*)
|    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.1.0 (*)
|    +--- androidx.recyclerview:recyclerview:1.0.0 -> 1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|    |    +--- androidx.customview:customview:1.0.0 (*)
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.transition:transition:1.2.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.0.1 -> 1.3.1 (*)
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.vectordrawable:vectordrawable:1.1.0 (*)
|    \--- androidx.viewpager2:viewpager2:1.0.0
|         +--- androidx.annotation:annotation:1.1.0
|         +--- androidx.fragment:fragment:1.1.0 (*)
|         +--- androidx.recyclerview:recyclerview:1.1.0 (*)
|         +--- androidx.core:core:1.1.0 -> 1.3.1 (*)
|         \--- androidx.collection:collection:1.1.0 (*)
```
可以看到'com.google.android.material:material:1.3.0'这个包中依赖的都是androidx。将该包删除，及修改引用该包的代码，本报错中是theme.xml中引用了这个包内容，修改成androidsupport主题即可。
再此编译，成功。    
修改后依赖项：
```
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
//    implementation 'com.google.android.material:material:1.3.0'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
```

> 插曲：  
按照提示修改，添加'tools:replace="android:appComponentFactory"'，接续报错如下：
Manifest merger failed with multiple errors, see logs
并没有找到log在哪，这个merger fail信息最终在AndroidManifest.xml文件页面底部的Merged Manifest选项卡中。
能看到error的地方，但还没能直观的看出问题。

----

#### 编译报错：error: unknown element <overlay> found.
编译Overlay皮肤包，但是出现找不到overlay标签问题。  
**问题解决**：添加buildToolsVersion为对应sdk version  
修改前的build.gradle
```
compileSdkVersion 28

defaultConfig {
    applicationId "com.test.overlay2"
    minSdkVersion 27
    targetSdkVersion 28
    versionCode 1
    versionName "1.0"

    testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
}
```
修改后的build.gradle
```
compileSdkVersion 28
buildToolsVersion "28.0.3"

defaultConfig {
    applicationId "com.test.overlay2"
    minSdkVersion 27
    targetSdkVersion 28
    versionCode 1
    versionName "1.0"

    testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
}
```
**原因分析**：由于之前未指明buildToolsVersion，应该是使用了默认的buildTools，编译overlay时buildTools下的aapt2检查报的错。换成对应版本的buildTools后成功编译，应该就是buildTools下aapt2的版本问题。  
##### 问题来了：
**未指明buildToolsVersion时，如何确定默认用的是哪个版本？**  
看gradle插件版本，根build.gradle中：
```
dependencies {
    classpath "com.android.tools.build:gradle:3.1.2"
}
```
gradle插件3.0以上会默认指定buildToolsVersion，可以通过[官网](https://developer.android.com/studio/releases/gradle-plugin?hl=zh-cn) 查看对应关系。这3.1.2版本如果未指明则使用Build Tools 27.0.3版本。  
