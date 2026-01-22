---
title:  "jni报错解决合集"
---
###1.java.lang.UnsatisfiedLinkError: JNI_ERR returned from JNI_OnLoad in "/system/lib64/libnvwriter_jni.so"
在这个错误上面有这样一段信息：
```
01-03 08:34:32.720 4369 4369 E art : Failed to register native method com.wind.emode.utils.NvWriter.n_readflag_NV()Ljava/lang/String in /system/app/Emode/Emode.apk
01-03 08:34:32.720 4369 4369 E art : ----- class 'Lcom/wind/emode/utils/NvWriter;' cl=0x12c52f40 -----
01-03 08:34:32.720 4369 4369 E art : objectSize=252 (232 from super)
01-03 08:34:32.720 4369 4369 E art : access=0x0008.0001
01-03 08:34:32.720 4369 4369 E art : super='java.lang.Class<java.lang.Object>' (cl=0x0)
01-03 08:34:32.720 4369 4369 E art : vtable (2 entries, 11 in super):
01-03 08:34:32.720 4369 4369 E art : 0: java.lang.String com.wind.emode.utils.NvWriter.readFlagNV()
01-03 08:34:32.720 4369 4369 E art : 1: void com.wind.emode.utils.NvWriter.writeFlagNV(int, char)
01-03 08:34:32.720 4369 4369 E art : direct methods (5 entries):
01-03 08:34:32.720 4369 4369 E art : 0: void com.wind.emode.utils.NvWriter.<clinit>()
01-03 08:34:32.720 4369 4369 E art : 1: void com.wind.emode.utils.NvWriter.<init>()
01-03 08:34:32.720 4369 4369 E art : 2: com.wind.emode.utils.NvWriter com.wind.emode.utils.NvWriter.getInstance()
01-03 08:34:32.720 4369 4369 E art : 3: java.lang.String com.wind.emode.utils.NvWriter.n_readflag_NV()
01-03 08:34:32.720 4369 4369 E art : 4: void com.wind.emode.utils.NvWriter.n_writeflag_NV(int, char)
01-03 08:34:32.720 4369 4369 E art : static fields (1 entries):
01-03 08:34:32.720 4369 4369 E art : 0: com.wind.emode.utils.NvWriter com.wind.emode.utils.NvWriter.sInstance
01-03 08:34:32.720 4369 4369 E art :
01-03 08:34:32.727 4369 4369 D AndroidRuntime: Shutting down VM
--------- beginning of crash
01-03 08:34:32.728 4369 4369 E AndroidRuntime: FATAL EXCEPTION: main
01-03 08:34:32.728 4369 4369 E AndroidRuntime: Process: com.wind.emode, PID: 4369
01-03 08:34:32.728 4369 4369 E AndroidRuntime: java.lang.UnsatisfiedLinkError: JNI_ERR returned from JNI_OnLoad in "/system/lib64/libnvwriter_jni.so"
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at java.lang.Runtime.loadLibrary0(Runtime.java:989)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at java.lang.System.loadLibrary(System.java:1562)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at com.wind.emode.utils.NvWriter.<clinit>(NvWriter.java:13)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at com.wind.emode.PhaseCheck.onCreate(PhaseCheck.java:214)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.Activity.performCreate(Activity.java:6723)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1119)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2619)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2727)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.ActivityThread.-wrap12(ActivityThread.java)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1478)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.os.Handler.dispatchMessage(Handler.java:102)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.os.Looper.loop(Looper.java:154)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at android.app.ActivityThread.main(ActivityThread.java:6121)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at java.lang.reflect.Method.invoke(Native Method)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:889)
01-03 08:34:32.728 4369 4369 E AndroidRuntime: at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:779)
01-03 08:34:32.731 1955 3172 W ActivityManager: Force finishing activity com.wind.emode/.PhaseCheck
01-03 08:34:32.737 1955 3172 D ActivityTrigger: ActivityTrigger activityPauseTrigger
01-03 08:34:32.740 1955 3172 W ActivityManager: Force finishing activity com.wind.emode/.emodeSubmenu
```
第一句说明n_readflag_NV()方法没有注册成功。检查代码发现：
```
JNINativeMethod gMethods[]{
    {"n_writeflag_NV","(IC)V",(void*)android_native_writeflag_NV},
    {"n_readflag_NV","()Ljava/lang/String",(void*)android_native_readflag_NV}
}
```
错误就在与` Ljava/lang/String `应该是` Ljava/lang/String ;`,少了分号“；”，所以参数不对，注册失败。
**另外如果java中定义的native方法没有被用到，则同样会报如上的错误**。
如：定义了native的 n_writeflag_NV和 n_readflag_NV,而在其他类中实际只使用了n_readflag_NV，那么 n_writeflag_NV没有被使用，同样会报出上面的注册失败的错误。

----

###2.java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader...couldn't find "libnvproxy_jni.so"
```
--------- beginning of crash
01-08 03:11:18.684 4297 4297 E AndroidRuntime: FATAL EXCEPTION: main
01-08 03:11:18.684 4297 4297 E AndroidRuntime: Process: com.wind.emode, PID: 4297
01-08 03:11:18.684 4297 4297 E AndroidRuntime: java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "/system/app/Emode/Emode.apk"],nativeLibraryDirectories=[/system/app/Emode/lib/arm64, /system/app/Emode/Emode.apk!/lib/arm64-v8a, /system/lib64, /vendor/lib64, /system/lib64, /vendor/lib64]]] couldn't find "libnvproxy_jni.so"
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at java.lang.Runtime.loadLibrary0(Runtime.java:984)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at java.lang.System.loadLibrary(System.java:1562)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at com.wind.emode.utils.NvProxy.<clinit>(NvProxy.java:15)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at com.wind.emode.PhaseCheck.getFlagList(PhaseCheck.java:124)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at com.wind.emode.PhaseCheck.onCreate(PhaseCheck.java:188)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.Activity.performCreate(Activity.java:6723)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1119)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2619)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2727)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.ActivityThread.-wrap12(ActivityThread.java)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1478)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.os.Handler.dispatchMessage(Handler.java:102)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.os.Looper.loop(Looper.java:154)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at android.app.ActivityThread.main(ActivityThread.java:6121)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at java.lang.reflect.Method.invoke(Native Method)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:889)
01-08 03:11:18.684 4297 4297 E AndroidRuntime: at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:779)
01-08 03:11:18.689 1901 2482 W ActivityManager: Force finishing activity com.wind.emode/.PhaseCheck
```
这里是没有在相关lib目录下找到libnvproxy_jni.so文件，本来是应该存在于system/lib64/目录下的，但是并没有，所以是在编译时就没有将这个jni库编译进去。
所以是Android.mk文件里配置的有问题。最后解决是在APK的Android.mk中添加：
![范型约束说明](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/jni_UnsatisfiedLinkError.png)

 根据以前的经验，有最后的`include $(call all-makefiles-under,$(LOCAL_PATH))`就能编译到对应jni的Android.mk，把so文件编出来的，而且mmm编译模块都可以
编出来so，但是全编就是编不到，可能是设置了`LOCAL_JNI_SHARED_LIBRARIES`和`LOCAL_REQUIRED_MODULES`有关，所以将libnvproxy_jni添加到这两个
属性中就编出来了。







