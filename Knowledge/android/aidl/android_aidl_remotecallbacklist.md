---
title:  "Android RemoteCallBackList"
---
## 应用场景
用于对多个客户端的callback进行管理。其内部有一个ArrayMap用于存放客户端的callback(IBinder对象)，并实现了死亡监听处理，在客户端死亡时将对应的callback从ArrayMap中移除。
> 优点：
> 1. 自动管理客户端的死亡监听，当客户端死亡时从list中移除以及在解注册时断开死亡监听等不再需要手动去写。
> 2. Broadcast遍历线程安全，内部的读写操作都加了synchronized同步。


## 使用方式
### 遍历
```java
int n = callbacks.beginBroadcast(); // 返回callback数量。同时将callback放到mActiveBroadcast数组中。
while (n > 0) {
    i--;
    try {
        // getBroadcastItem(i)即从mActiveBroadcast中获取callback对象。
        callbacks.getBroadcastItem(i).somethingHappened();
    } catch (RemoteException e) {
        // The RemoteCallbackList will take care of removing
        // the dead object for us.
    }
}
callbacks.finishBroadcast(); // 遍历完毕，将mActiveBroadcast数组清空。
```
> **疑问**：ArrayMap就能取出callback，为什么要先从ArrayMap中取出放到mActiveBroadcast数组中再取？  
> RemoteCallbackList有提供直接从ArrayMap取值的操做，但是非线程安全，因为取操作都是要先获取size和index，通过index来取。
> 比如通过getRegisteredCallbackCount获取size之后发生了register，就导致size没包含刚register进来callback，index也不再准确。  
> 而先存放到mActiveBroadcast数组中，虽然也没有包含刚register的callback，但至少不会影响index，不会造成数据混乱。

### 死亡处理
自己写个类，继承RemoteCallbackList，重写onCallbackDied方法。
> RemoteCallbackList有一个public的onCallbackDied方法，该方法体是空的。在callback死亡监听中会调用该方法。

### 针对特定客户端处理
使用带cookie的register方法进行注册。利用cookie对client进行标识。
> RemoteCallbackList内部有一个CallBack内部类，该类一个成员是client的IBinder对象，另一个是类型为Object的cookie。默认的register是将cookie设为null的，我们可以利用cookie做一些额外的事情，比如身份标识。

如：
```java
// 注册callback，同时设置cookie为字符串"radio"。
register(callback,"radio")

// 找到这个叫radio的callback进行处理
int n = callbacks.beginBroadcast();
while (n > 0) {
    i--;
    try {
        CallBack cb = callbacks.getBroadcastItem(i);
        String cookie = (String)mCbs.getBroadcastCookie(i);
        if(TextUtils.equals(cookie,"radio")){
            cb.somethingHappened();
        }
    } catch (RemoteException e) {
        // The RemoteCallbackList will take care of removing
        // the dead object for us.
    }
}
callbacks.finishBroadcast(); // 遍历完毕，将mActiveBroadcast数组清空。
```

### kill方法
RemoteCallbackList的kill方法，即标记killed状态同时清空ArrayMap，killed状态下register方法会返回false。
> 官方给的应用场景是：This should be used when a Service is stopping, to
> prevent clients from registering callbacks after it is
> stopped.我还没真正用到该方法。

### broadcast方法
没具体使用，看起来就是实现了begin到finish的过程，通过回调的方式(action接口闭包)处理逻辑。
```java
public void broadcast(Consumer<E> action) {
    int itemCount = this.beginBroadcast();

    try {
        for(int i = 0; i < itemCount; ++i) {
            action.accept(this.getBroadcastItem(i));
        }
    } finally {
        this.finishBroadcast();
    }

}
```
