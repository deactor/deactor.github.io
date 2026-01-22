---
title:  "Android原理概要"
---
## Apk是怎么生成的？
apk文件主要分为代码和资源两部分，构建的主要过程如下：
1. AAPT工具（资源打包工具）将资源文件（AndroidManifest.xml、布局文件、各种 xml 资源等）打包生成R.java文件，将AndroidManifest.xml生成二进制的AndroidManifest.xml文件。
2. 使用aidl工具将所有的aidl文件生成相应的java文件。
3. 使用javac把项目中所有java文件编译成class文件。
4. 使用DX工具将所有class文件转换为Dex文件。
5. 使用apkbuidler将所有的代码及资源文件打包生成APK文件。
6. 使用apksigner或jarsigner对apk签名。
7. 使用zipalign对签名后的apk进行对齐处理。（如果apksigner签的名，这里的对齐优化要放到签名前）

间单点：
assets和res/raw资源原封不动打包进apk，其他资源文件会被编译或处理被赋予一个资源ID，并且生成resources.arsc文件（资源索引表）及R.java（资源ID）文件打包进apk。AndroidManifest.xml也会被编译生成二进制的xml文件。如果有aidl文件，aidl文件会用来生成相应的java文件，java文件先被编译成class文件再经DX工具编译成Dex文件打包进apk。之后再对apk进行签名。

----

## 资源索引表resources.arsc文件有什么作用？
Android利用这个索引表根据资源ID进行资源的查找，为不同语言、不同地区、不同设备提供相对应的最佳资源。

----

## V1和V2签名的区别？
1. V1签名是对压缩包中单个文件签名。V2签名是对整个apk签名。
2. V1使用jarsigner，也可以使用apksigner工具签名，V2只能使用apksigner进行签名。
3. V2签名比V1签名更安全，因为是对整个apk签名，修改了压缩包就无法验签通过。
4. V2签名验证时间更短（因为不需要解压就能验证），因而安装速度也加快了。

----

## apk的安装过程？
安装场景有4种：
1. 系统启动时安装
2. 应用市场安装
3. 手动点击apk安装
4. adb安装

共同的安装核心过程是：（以下均在PMS中完成）
1. 通过PackageParser类的parsePackage方法解析AndroidManifext.xml文件，将解析出的application节点下的各app组件信息，users-permission等数据存于package类中。
2. 为应用程序分配UID，并更新PackageSetting（**负责干啥的？**），如果是新应用则新生成PackageSetting,如果是旧版本更新则更新PackageSetting。
3. 将Package信息保存到PMS，以供AMS访问。
4. 更新应用程序权限信息，授权程序资源访问权。

不同之处：
1. 系统启动时安装：
读取/data/system/package.xml文件拿到上一次安装应用程序的信息（应用程序uid，权限，程序包路径等），逐步扫描程序文件夹找出各个APK并安装，安装后更新package.xml文件。
2. 手动安装：
通过系统内置的PackageInstaller调用PMS进行安装。
3. 应用市场安装：
应用市场程序通过调用PMS.installPackageAsUser()开启安装。
4. ADB安装：
PM接收命令调用PMS执行安装。

针对手动安装：
1. 点击APK安装，会启动PackageInstallerActivity，再进入InstallInstalling这两个Activity显示应用信息
2. 点击页面上的安装，将APK信息存入PackageInstaller.Session传到PMS
3. PMS会做两件事，拷贝安装包和装载代码
4. 在拷贝安装包过程中会开启Service来copyAPK、检查apk安装路径，包的状态
5. 拷贝完成以base.apk形式存在/data/app目录下
6. 装载代码过程中，会继续解析APK，把清单文件内容存放于PMS
7. 对apk进行签名校验
8. 安装成功后，更新应用设置权限，发送广播通知桌面显示APP图标，安装失败则删除安装包和各种缓存文件
9. 执行dex2oat优化

----

## android的运行时权限等级划分
+ PROTECTION_NORMAL：任何应用都可以申请，在安装应用时授权，无需用户操作
+ PROTECTION_DANGEROU：任何应用都可以申请，在安装应用时授权，需要用户操作
+ PROTECTION_SIGNATURE：只有于声明该授权的apk使用了相同的私钥签名的应用才可以申请该权限

----

## Android启动流程
Bootloader -> lk -> kernel -> init解析init.rc创建zygote(虚拟机启动) -> zygote fork自己启动SystemServer，并进入socket模式 -> SystemServer通过socket向zygote发送请求，创建系统应用进程。

----

## app的启动过程？
1. AMS启动各类组件时发现相应进程没有启动时则获取进程启动所需参数，通过Process.start()请求启动进程。
2. 该请求通过LocalSocket发送到Zygote，由Zygote进行fork()进程。
3. 进程创建后唤醒ActivityThread.main()。
4. ActivityThread.main()会创建ApplicationThread(binder线程)及ActivityThread并初始化主Handler及Looper。
5. ApplicationThread向ActivityThread发BIND_APPLICATION消息。最终通过Instrumentation类调用application的onCreate()方法。

----

## Activity的启动过程？
1. Launcher进程通过Binder向AMS发起startActivity请求。
2. AMS接收到请求后，向ActivityStack类发送启动Activity请求。
3. 判断应用进程是否已启动，如果没启动则先启动进程。(过程见上面问题)
4. ActivityStack类记录需要启动的Activity的信息并调整Activity栈将其置于栈顶并通过Binder将Activity的启动信息传给ApplicationThread。
5. ApplicationThread通过Handler将Activity的启动发送给ActivityThread。
6. ActivityThread接到请求后通过ClassLoader加载相应的Activity类，最终调用Activity的onCreate()方法。

----

## Activity的管理机制？
Activity以ActivityRecord的形式在AMS中被以任务栈的形式管理。连续启动的activity会被置于同一个任务栈中，除非activity启动模式设置了singleTask或singleInstance。Task的管理与activity所处进程无关，比如appA中的activityA以普通模式启动appB中的activityB，activityB会加入到activityA的任务栈中。

## taskAffinity属性作用？
系统使用该名称来标识应用的默认任务亲和性，即Activity倾向于属于哪个任务，默认名称是应用包名。在两种情况下发挥作用：
1. 当启动 Activity 的 intent 包含 FLAG_ACTIVITY_NEW_TASK 标记时，如果已存在与新Activity 具有相同亲和性的现有任务，则会将 Activity 启动到该任务中。如果不存在，则会启动一个新任务。
2. 当 Activity 的 allowTaskReparenting 属性设为 "true" 时，一旦和 Activity 有亲和性的任务进入前台运行，Activity 就可从其启动的任务转移到该任务。

----

## services的启动过程？
app启动service都是通过AMS去启动，AMS会先判断该service是否已启动（通过ServiceRecord判断），已启动则回调onStartCommand方法，若未启动则判断service所在进程是否启动（通过ProcessRecord判断），如果未启动则先启动进程，之后再加载Service并回调onCreate方法。bindService与startService启动方式在AMS中都是通过bringUpServiceLocked方法来启动Service，不同的是bindService不会回调onStartCommand方法。

----

## AIDL原理？
主要是通过构建工具将aidl接口文件生成相应的java文件，其中包含继承Binder的Stub子类作为服务端，以及继承IBinder的proxy子类作为客户端。在Service的onBind回调中返回的Binder对象为当前进程的本地Binder对象，对应着native层的BBinder，该Binder对象在通过AMS传递给调用方OnServiceConnected方法的过程是被转换为了BinderProxy对象，对应native层的BpBinder。调用方通过该远程Binder与服务端进程交互。

----

## AIDL的in out inout 以及oneway是什么？
实现stub的一端为服务端，另一端为客户端。
+ in:数据只能从客户端流向服务端，服务端对数据的修改不会改变客户端数据。
+ out:数据只能从服务端流向客户端，如果客户端向服务端传，则服务端接收的数据是null。
+ inout:数据双向流通。
+ oneway；客户端对服务端发起请求后，不会等待服务端响应。

----

## Binder原理？
主要就是客户端Binder与服务端Binder通过Binder驱动进行交互。

----

## Android屏幕刷新机制？
viewrootimpl.requestlayout->schreduletravel->chrographargh.postcallback->regist vsync-> onVysnc ->doframe->do callback inviewrootimpl -> onmeasure
总体而言就是，CPU提交绘制数据到GPU，GPU处理并生成最终用以显示的数据提交给屏幕进行显示。
当app调用requestLayout请求绘制时，viewrootimpl会向choreographer注册回调请求何时绘制，choreographer则向sufaceFlinger注册监听VSYNC信号，当监听到sufaceFlinger发出的VSYNC信号后则通过回调通知viewrootimpl进行绘制，会依次执行view的onMeasure，onLayout，onDraw方法得到需要绘制的数据，该数据会由渲染线程交由GPU生成Graphic buffer，其Graphic buffer有三重缓存保证渲染效率，最终的buffer数据交由surfaceFlinger通过硬件合成器合成并输出到屏幕。
1. ViewRootImpl是什么时候创建的？主要作用是什么？

2. VSYNC是什么？有什么作用？
VSYNC是用来同步帧速率（GPU合成帧的速率）与屏幕刷新率，它类似于时钟中断，当接到VSYNC信号时，app立即进行下一帧数据的绘制，sufaceFlinger合成显示已准备好的buffre数据。
3. Choreographer是什么？有什么作用？
Choreographer就是用来接收系统的VSYNC信号，统一管理应用的输入、动画和绘制等任务的执行时机。
4. 三缓冲是什么？
先讲一下双缓冲，每个surface对应的bufferqueue内部都有一两个Graphic Buffer，一个用于绘制一个用于显示。应用会先把内容绘制到离屏缓冲区，在需要显示的时候再把离屏缓冲区的内容交换到Front Graphic Buffer中去来显示。三缓冲则是相当于多加了一个离屏缓冲区。仅使用双缓冲时，当在一个VSYNC信号周期内，一个offscreen Buffer没有被释放时，下一个VSYNC开始时cpu就没法进行下一帧的绘制，导致丢帧，而三缓冲在这种情况下CPU可以用另一个offscreen Buffer进行绘制，避免丢帧。
5. 屏幕重绘的触发条件有哪些？
Activity中的布局首次绘制，以及每次调用View的invalidate()时，都会调用到ViewRootImp#requestLayout()
同一帧中出现多次requestLayout()调用，其实最终也只会绘制一次。因为方法中有一个mTraversalScheduled标志位。
>android官网：
>虽然应用可以随时提交缓冲区，但 SurfaceFlinger 仅能在屏幕处于两次刷新之间时唤醒，以接受缓冲区，这会因设备而异。这样可以最大限度地减少内存使用量，并避免屏幕上出现可见的撕裂现象（如果显示内容在刷新期间更新，则会出现此现象）。
>
>在屏幕处于两次刷新之间时，屏幕会向 SurfaceFlinger 发送 VSYNC 信号。VSYNC 信号表明可对屏幕进行刷新而不会产生撕裂。当 SurfaceFlinger 接收到 VSYNC 信号后，SurfaceFlinger 会遍历其层列表，以查找新的缓冲区。如果 SurfaceFlinger 找到新的缓冲区，SurfaceFlinger 会获取缓冲区；否则，SurfaceFlinger 会继续使用上一次获取的那个缓冲区。SurfaceFlinger 必须始终显示内容，因此它会保留一个缓冲区。如果在某个层上没有提交缓冲区，则该层会被忽略。
>
>SurfaceFlinger 在收集可见层的所有缓冲区之后，便会询问硬件混合渲染器 (HWC) 应如何进行合成。如果 HWC 将层合成类型标记为客户端合成，则 SurfaceFlinger 将合成这些层。然后，SurfaceFlinger 会将输出缓冲区传递给 HWC。

----

## Activity视图绑定过程？
待完善

----

## Android事件处理机制？
当有按键点击或触摸发生时，EventHub会从驱动中得到input事件，inputReader线程则从EventHub得到该次事件并传递给inputDispatcher线程。在分发给View之前inputDispatcher会先将事件传递给PhoneWindowManagerService做一次拦截（所以应用层是没法屏蔽home键事件，因为这里就被系统拦截处理了），之后inputDispatcher才会通过与ViewRootImpl建立的InputChannel管道将事件传递给ViewRootImpl，ViewRootImpl则按照inputStage继续分发，其中ViewPostImeInputStage就是UI层级的事件转发阶段。

**那么inputChannel是怎么建立起来的？？**
在ActivityThread创建Activity的时候会为其创建ViewRootImpl，并执行ViewRootImpl.setView()方法，这个方法中则创建了InputChannel实例，并传递给了WMS，WMS调用openInputChannelPair得到一对inputChannel相当于客户端与服务器，这个方法是native实现的，内部是socket。之后wms将客户端给到ViewRootImpl（通过InputChannel的transferTo方法将socket的客户端fd给到ViewRootImpl传递传递过来的InputChannel对象中），服务端给到inputDispatcher。通道就这么建立起来了。ViewRootImpl通过WindowInputEventReceiver接收iputDispatcher发送过来的input事件。

**UI层级的事件分发及拦截流程？**
ViewRootImpl首先将event传递给DecorView，DecorView传给Activity，Activity再传给phoneWindow,实际是直接调用了DecorView的superDispatchTouchEvent又把事件回给了DecorView,之后DecorView调用父类的dispatch方法作为ViewGroup向子View分发。父View会遍历子View的dispatchTouchEvent方法。这里注意以下几点：
1. View的dispatchTouchEvent会调用OnTouchEvent(),ViewGroup的dispatchTouchEvent方法主要是用来向子view分发事件也就是调子View的dispatchTouchEvent方法。
2. ViewGroup在向子View分发前会调用onInterceptTouchEvent判断是否拦截，如果返回true，则不再向子View分发，而是自己消费该事件，调用OnTouchEvent()方法。
3. 子View可以通过调用父View的requestDisallowInterceptTouchEvent()设置不允许父View拦截，则父View就不会调用onInterceptTouchEvent方法。
4. 事件的传递是按照事件序列来传递，ACTION_DOWN为事件的起始，如果ACTION_DOWN被ViewGroup拦截，那么后续的ACTION_MOVE，ACTION_UP都传递不到子View。子View设置不允许父View拦截将毫无意义，因为根本不会调用到子View的dispatchTouchEvent方法。因为在ACTION_DOWN被View接收后会形成一条调用链，上层的View会通过TouchTarget记录传递的子View，而不用再遍历一遍子View。所以如果ACTION_DOWN没有被父View拦截时，子View的dispatchTouchEvent才能在ACTION_DOWN传递过来时执行到而设置父View不允许拦截。如果没设置不允许拦截，而后续的ACTION_MOVE等事件被父View拦截，则会向子View发送ACTION_CANCEL事件，取消子View对该事件的处理。同时在每次ACTION_DOWN开始传递时会重置ToouchState（不允许拦截）与清除TouchTarget。

----

## ANR原理？
+ Service超时监听：
AMS启动service前会给自己的Handler发送一个service超时的信息。在成功创建service之后会取消这个超时信息。所以在指定时间内没有创建完成，就会执行超时操作，报告ANR。
+ input超时监听：
inputReader线程通过EventHub监听到底层的输入事件上报，并将其放入mInBoundQueue中，同时唤醒InputDispatcher线程。InputDispatcher从mInBoundQueue中取出事件，并重置ANR计时，并检查窗口是否就绪，如果未就绪则进入ANR检测状态，有一个全局变量记录超时时间点（事件的开始分发时间加上超时时间），如果当前时间没到超时时间点则中止本轮事件分发（注意事件并没有从mInBoundQueue中删除，同时设置了下次读取时间的唤醒时间，到唤醒时间时从mInBoundQueue取出之前没有发出去的事件继续发，所以主线程堵塞时一次按键点击就能触发ANR，在inputDispatcher这是同一事件发了两次）。如果当前时间超过了超时时间点则判定事件超时，上报ANR。注意超时时间点是前一次事件分发窗口未就绪时记录的，所以超时也是上次事件超时。


