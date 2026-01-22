---
title:  "JNI详解"
---
问题写在前面：
1.如何实现程序A加载调用程序B的jnilib库？研究过发现可以实现，但是当两个程序都需要运行时会有问题。
2.JNI_OnLoad的调用过程，虽然知道java加载jni库时会回调该方法，其实现回调的过程是怎样的？
3.JNI_OnUnload的调用过程及回调时机？

jni用法：
采用动态注册的方式来写这样一个jnidemo
先定义jni的java文件，声明native函数，这里在分别在两个包中定义了两个java类，
这里只是为了测试下jni的注册，所以用了在不同包里分别定义java类：
先看下目录结构：


---
jniDemo
- src
- com.wind.jnidemo
- answerjni.java
- MainActivity.java
- com.wind.jnidemo.badanswer
- badanswerjni.java
- jni
- Android.mk
- com_wind_jnidemo_anserjni.c
- res
- Android.mk
- AndroidManifest.xml

---


answerjni.java中：

```
package com.wind.jnidemo;

import android.util.Log;

public class answerjni {
public static String answer(){
Log.d("jni","answerjni -- answer");
return Nanswer();
}

static{
Log.d("jni","System.loadLibrary");
System.loadLibrary("answerjni");
}

private static native String Nanswer();
}
```

badanswerjni.java中：

```
package com.wind.jnidemo.badanswer;

import android.util.Log;

public class badanswerjni {
/*如果之前已经加载过这个库，如answerjni中先执行过了，那么这里再执行不会有错误，但是不会执行JNI_OnLoad函数。
*所以如果确认badanswerjni在answerjni之后加载，那么这里的loadLibrary这一步是不需要的。
*但是如果badanswerjni在answerjni之前加载，这里没有loadLibrary的操作，那么就会报错。
*/
static{
Log.d("jni","badanswerjni -- System.loadLibrary");
System.loadLibrary("answerjni"); //这里是android.mk中的LOCAL_MODULE := libanswerjni；libanswerjni去掉lib前缀。
}
public static native String answer();
}
```

写对应的jni文件，一般是c或者cpp文件。名称一般用包名+类名，下划线连接的方式命名，当然这个文件名称可以是随意的。
com_wind_jnidemo_answerjni.c文件：

```
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <jni.h>
#include <assert.h>
#include <android/log.h>

/*
* 这个log是android的log，可以打印在mainlog中的。
* */
#define LOG_TAG "jni"
#undef LOG
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG,LOG_TAG,__VA_ARGS__)

/* 如果是静态注册，通过函数名称进行匹配要注意"."，自己写用动态注册比较方便，这里介绍一下是为了看代码。
* "."在native中独特意义，所以java中"."对应到natice中为"_",如果java中为"_",对应native中为"_1"。
*/

jstring native_answer(JNIEnv* env,jobject thiz){
LOGD("native_answer");
return (*env)->NewStringUTF(env,"I'm fine!And you?");
}

jstring native_badanswer(JNIEnv* env,jobject thiz){
LOGD("native_badansweranswer");
return (*env)->NewStringUTF(env,"I'm so bad!");
}
/*
* 方法表
* 将java中声明的native与这里的jni方法进行匹配映射。
* */
static JNINativeMethod gMethods[] = {
//{"java函数名","函数签名即参数和返回值","本地对应的函数指针"}
{"Nanswer","()Ljava/lang/String;",(void*)native_answer}
};

static JNINativeMethod gbMethods[] = {
{"answer","()Ljava/lang/String;",(void*)native_badanswer}
};

/*
* 为某一个类注册本地方法
* */
static int registerNativeMethods(JNIEnv* env,const char* className
,JNINativeMethod* gMethods,int numMethods){
jclass clazz;
clazz = (*env)->FindClass(env,className);
LOGD("registerNativeMethods");
if(clazz == NULL){
return JNI_FALSE;
}
if((*env)->RegisterNatives(env,clazz,gMethods,numMethods) < 0){
return JNI_FALSE;
}

return JNI_TRUE;
}

/*
* 为所有类注册本地方法
* */
static int registerNatives(JNIEnv* env){
//java中方法所在类，如果这里写错了，是注册不成功的，运行时会报错java.lang.ClassNotFoundException找不到这个类。
const char* kClassName = "com/wind/jnidemo/answerjni";
const char* kbClassName = "com/wind/jnidemo/badanswer/badanswerjni";
LOGD("registerNatives");

if(registerNativeMethods(env,kClassName,gMethods,sizeof(gMethods)/sizeof(gMethods[0])) == JNI_FALSE)
goto rfail;
if(registerNativeMethods(env,kbClassName,gbMethods,sizeof(gbMethods)/sizeof(gbMethods[0])) == JNI_FALSE)
goto rfail;
return JNI_TRUE;
rfail:
LOGD("registerNatives --- fail");
return JNI_FALSE;
}

/*
* System.loadLibrary()时调用该方法
* 如果已经加载过了，再调用System.loadLibrary()时是不会回调该方法的。
* 那么问题来了，如果一个程序A已经完成了一个jni库，我另外写了一个程序B，想使用这个jni库，怎么办？
* 就像这里的不同包的java类注册一样就可以，但有一点必须注意，那就是如果程序A已经运行加载了jni库
* 那么程序B就用不了那个jni库了，因为jni库已经加载过了，这里就不会回调JNI_OnLoad方法，你就没法得到
* JNIEnv指针。就没法去进行jni操作。
* */
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm,void* reserved){
JNIEnv* env = NULL;
jint result = -1;
LOGD("JNI_OnLoad");

if((*vm)->GetEnv(vm,(void**)&env,JNI_VERSION_1_4) != JNI_OK){//获取JNIEnv指针
LOGD("GetEnv --- fail");
return -1;
}
assert(env != NULL);

if(!registerNatives(env)){
LOGD("registerNatives --- fail");
return -1;
}
LOGD("registerNatives --- success");

result = JNI_VERSION_1_4;
return result;
}
```
看下Android.mk中如何配置：
jnidemo的Android.mk（如果是IDE工具开发第三方应用应该是没有这个Android.mk的）：

```
LOCAL_PATH:= $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := $(call all-subdir-java-files)
LOCAL_PACKAGE_NAME := jnidemo
include $(BUILD_PACKAGE)

include $(call all-makefiles-under,$(LOCAL_PATH))
```

jni文件下的Android.mk：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := libanswerjni #lib+库名

LOCAL_PRELINK_MODULE := false
LOCAL_LDLIBS := -llog

LOCAL_SRC_FILES := \
com_wind_jnidemo_answerjni.c #这里要用文件全名
include $(BUILD_SHARED_LIBRARY)
```

MainActivity测试代码如下：

```
package com.wind.jnidemo;

import com.wind.jnidemo.R;
import com.wind.jnidemo.badanswer.badanswerjni;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity {

private TextView tv ;
private Button btn;
private Button btn2;
private int clicktime = 0;

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

tv = (TextView) findViewById(R.id.tv);
btn = (Button) findViewById(R.id.btn);
btn2 = (Button) findViewById(R.id.btn2);

btn.setOnClickListener(new View.OnClickListener() {

@Override
public void onClick(View arg0) {
// TODO Auto-generated method stub
Log.d("jni","btn----onclick");
clicktime++;
tv.setText(answerjni.answer() + clicktime);
}
});
btn2.setOnClickListener(new View.OnClickListener() {

@Override
public void onClick(View arg0) {
// TODO Auto-generated method stub
Log.d("jni","btn2----onclick");
clicktime++;
tv.setText(badanswerjni.answer() + clicktime);
}
});
}
}
```
点击Button ，TextView就会显示对应从jni中返回的内容。

有几个关于jni的问题:

---

1.JNI_OnLoad中传入的JavaVM是什么？
虚拟机在JNI中的代表，每个java进程只有一个。

---
2.JNIEnv类主要作用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
这是jni的环境，提供jni下的各种操作。所有的操作都可以在libnativehelper/include/nativehelper/jni.h中找到。

---
3.java和native对应的函数。native多两个参数JNIEnv*和jobject有何用？
第一个参数JNIEnv：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
JNI的上下文，类似于context，里面保存了JNI的函数指针及一些信息。通过JNIEnv可以做很多事情， 如调用
java层的函数，操作jobject对象。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
JNIEnv是线程相关的。一个线程就有一个JNIEnv，就类似于一个Activity就会有一个Context。JNIEnv是由虚拟机创建，
即JavaVM，它是进程独一份的。可以通过JavaVM的AttachCurrentThread函数得到JNIEnv结构体。另外在后台线程退出前，
需要调用JavaVM的DetachCurrentThread函数来释放对应的资源。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
后台进程是啥意思？退出时释放是怎么操作的？
第二个参数jobject：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
表明是调用的哪个对象的native方法。如果在java中定义的是static的native方法，那么这个jobject就是jclass，如果java中
static中的native方法没有参数可省略不写。

---
4.jni中的垃圾回收：
jni中三种类型的引用：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
LocalReference：本地引用，所有非全局变量，包括传参，函数结束则变量回收,以及定义在函数外部的也是，在jni层
回到java时就会释放。最好使用完后就释放JNIEnv的DeleteLocalRef(),不要等到函数结束自动释放。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
Global Reference：全局引用（其实这个词不准确，容易引起误导，以为定义的全局变量就是全局引用），不主动释放，
永远不会被回收。使用JNIEnv的NewGlobalRef创建和DeleteGlobalRef()释放；
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
Weak Global Reference：弱全局引用，运行过程中可能被回收，所以在程序中使用它之前要调用JNIEnv的IsSameObject
判断它是不是被回收了。

---

jni访问java
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
相当于java中的反射机制。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
通过JNIEnv去操作jobject/jclass。操作jobject/jclass的属性和方法。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
jfieldID->java类的成员变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
jmethodID->java类的成员方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

获取：
jfieldID GetFieldID/GetStaticFieldID(jclass clazz,const char* name,const char* sig);
jmethodID GetMethodID/GetStaticMethodID(jclass clazz,const char* name,const char* sig);
clazz为java类（FindClass返回的jclass对象），name为属性/方法名,sig为属性/方法签名。
使用：
1.可以使用JNIEnv下的Call<type>Method(JNIEnv *env,jobject obj,jmethodID methodID,...)调用。
...表示java中方法的对应参数，给java方法传参。
如：CallIntMethod,CallVoidMethod。
如果是java中的static的函数，可以使用CallStatic<Type>Method;
2.可以使用JNIEnv下的Get<type>Field(JNIEnv *env,jobject obj,jfieldID fieldID)及
Set<type>Field(JNIEnv *env,jobject obj,jfieldID fieldID,NativeType value)
对jobject的属性进行读写。如果是static的，则用GetStatic<type>Field、SetStatic<type>Field
具体有如下type：

get | set
---|---
GetObjectField() | SetObjectField()
GetBooleanField() | SetBooleanField()
GetByteField() | SetByteField()
GetCharField() | SetCharField()
GetShortField() | SetShortField()
GetIntField() | SetIntField()
GetLongField() | SetLongField()
GetFloatField() | SetFloatField()
GetDoubleField() | SetDoubleField()

jstring的使用：
jstring是java在jni中的代表，在native层中，c是没有string类型的。
jchar* -> jstring:
jstring NewString(JNIEnv *Env,const jchar* unicodeChars,jsize len);
char* -> jstring:
jstring NewStringUTF(JNIEnv *Env,const char* UTFchars)

jstring -> jchar*
const jchar* GetStringChars(JNIEnv *Env,jstring string,jboolean* isCopy)
jstring -> char*
const char* GetStringUTFChars(JNIEnv *Env,jstring string,jboolean* isCopy)
这两个方法在字符串用完后需要手动ReleaseStringChars()及ReleaseStringUTFChars()释放本地字符串资源。
void ReleaseStringChars(JNIEnv *Env,jstring string,const jchar* chars)
void ReleaseStringUTFChars(JNIEnv *Env,jstring string,const char* utf)
一般较多的使用是带UTF的方法，将java的jstring和native的char*进行转换使用。
demo实验结果：
1.GetMethodID及GetStaticMethodID没有成功，总是报错no this method。
这个问题在于编译时，编译器代码执行了混淆操作，导致找不到。下面有解决方法。
2.对于string类型的field没有使用getObjectField()成功获取其值。同理对于引用类型的field要怎么反射调用？
以及返回值为string类型的method要怎么反射调用？
这个使用类型转换就可以了，下面也可以看到实现方式，另外对于jni中所使用的引用除非是NewGlobalRef
主动创建的全局引用，不会被自动回收。仅仅声明在函数体外的native的变量依然是本地引用，在native回到java
或函数体内执行完就会被回收。之前因为在JNI_OnLoad中保存fieldID，在另一个函数A中使用这个ID，虽然fieldID
声明在函数体外，但是依然在执行函数A时发生引用的ID已被销毁的问题。

看下jni引用java的实现：
还是上面的demo：
修改answerjni.java

```
package com.wind.jnidemo;

import android.util.Log;

public class answerjni {
public static String answer(){
Log.d("jni","answerjni -- answer");
return Nanswer();
}

/*这个属性将在jni中获取和修改
* */
public static String test = "hello";

/*这个方法将被jni调用
* */
public static int answertest(int i){
Log.d("jni","answerjni -- answertest i = " + i);
return i;
}

static{
Log.d("jni","System.loadLibrary ---- ");
System.loadLibrary("answerjni");
}

private static native String Nanswer();
}
```

修改相应的com_wind_jnidemo_answerjni.c的native_answer函数：

```
jstring native_answer(JNIEnv* env,jobject thiz){
LOGD("native_answer");
testclzz = (*env)->FindClass(env,"com/wind/jnidemo/answerjni"); //拿到jclass
testID = (*env)->GetStaticFieldID(env,testclzz,"test","Ljava/lang/String;"); //获取属性ID
jstring test = (jstring)(*env)->GetStaticObjectField(env,testclzz,testID); //得到对应属性值
char* ntest = (*env)->GetStringUTFChars(env,test,NULL); //将的到jstring转换为char*在native中使用
LOGD("native_answer --- test = %s",ntest);//打印输出
(*env)->ReleaseStringUTFChars(env,test,ntest);//用完释放

answertestID = (*env)->GetStaticMethodID(env,testclzz,"answertest","(I)I"); //获取方法ID
jint result = (*env)->CallStaticIntMethod(env,testclzz,answertestID,2); //执行方法，并传参“2”
LOGD("native_answer --- result = %d",result); //打印执行结果
return (*env)->NewStringUTF(env,"I'm fine!And you?");
}
```
另外需要配置项目的Android.mk取消代码混淆，否则可能出现找不到方法。
Android.mk:

```
LOCAL_PATH:= $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := $(call all-subdir-java-files)
LOCAL_PACKAGE_NAME := jnidemo
LOCAL_PROGUARD_ENABLED := disabled #取消代码混淆，否则jni中调用java方法，会找不到方法，报no this method错误
include $(BUILD_PACKAGE)
include $(call all-makefiles-under,$(LOCAL_PATH))
```

java与jni的类型转换关系：
表1 基本数据类型转换
Java|Native类型|符号属性|字长
---|---|---|---
boolean|jboolean|无符号|8位
byte|jbyte|无符号|8位
char|jchar|无符号|16位
short|jshort|有符号|16位
int|jint|有符号|32位
long|jlong|有符号|64位
float|jfloat|有符号|32位
double|jdouble|有符号|64位


表2 引用数据类型转换
Java引用类型|Native类型|Java引用类型|Native类型
---|---|---|---
All objects|jobject|char[]|jcharArray
java.lang.Class|jclass|short[]|jshortArray
java.lang.String|jstring|int[]|jintArray
Object[]|jobjectArray|long[]|jlongArray
boolean[]|jbooleanArray|float[]|jfloatArray
byte[]|jbyteArray|double[]|jdoubleArray
java.lang.Throwable|jthrowable||


表3 JNI类型签名标示
引用类型用“L+包名+类名+;”标示，数组用“[”前缀标示。
类型标示|java类型|类型标示|java类型
---|---|---|---
Z|boolean|F|float
B|byte|D|double
C|char|L/java/lang/String;|String
S|short|[I|int[]
I|int|[L/java/lang/object;|Object[]
J|long||
