---
title:  "Android aidl线程问题"
---
#### aidl方法执行是在哪个线程中？
主要分两种情况，一种执行在binder线程池的线程中，一种执行在app执行线程中。  
三个aidl接口文件：  
```
// 该接口Binder通过service的onBinder返回，客户端通过IMyAidlInterface.Stub.asInterface()转换为IMyAidlInterface对象来使用
interface IMyAidlInterface {
    int request(String id);
    int addCallBack(ICallback callback);

    IBinder getbinder();
}
```

```
// 该接口Binder由客户端实现，通过addCallBack传递给服务端，由服务端直接使用
interface ICallback {
    int response(String id);
}
```

```
// 该接口Binder由客户端通过IMyAidlInterface对象的getbinder()方法获取IBinder对象，再通过IRealBinder.Stub.asInterface()转换为IRealBinder对象来使用
interface IRealBinder {
    int doRequst(String id);
}
```

最终实验验证结果：
1. 通过Stub.asInterface()转化后的对象，执行方法是在Binder线程中。
2. 直接使用的对象是在app线程中执行。

如：
```
//客户端app
Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                mBinder.request("123");
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    });
thread.setName("abcd");
thread.start();


private ICallback callback = new ICallback.Stub() {
    @Override
    public int response(String id) throws RemoteException {
        // 此处是执行在上面做request时所在的线程即创建的"abcd"线程中。
        Log.i(TAG, "response: thread -> " + Thread.currentThread());
        return 0;
    }
};
```

```
//服务端app
@Override
public IBinder onBind(Intent intent) {
    // 主线程中返回
    Log.i(TAG, "onBind: -> thread:" + Thread.currentThread());
    return new MyAidlInterface();
}

private class MyAidlInterface extends IMyAidlInterface.Stub{
    private ICallback mCallback;

    @Override
    public int request(String id) throws RemoteException {
        // 客户端在abcd线程中做的请求，而这里是执行在Binder线程中，即Thread name -> Binder:12430_2
        if(mCallback != null){
            mCallback.response(id + "r");
        }
        return 0;
    }

    @Override
    public int addCallBack(ICallback callback) throws RemoteException {
        mCallback = callback;
        return 0;
    }

    @Override
    public IBinder getbinder() throws RemoteException {
        return new RealBinder.Stub() {
            @Override
            public int doRequst(String id) throws RemoteException {
                // 这里同上面一样，也是执行在Binder线程中
                Log.i(TAG, "doRequst: thread -> " + Thread.currentThread());
                return 0;
            }
        };
    }
}
```

Aidl跨线程，方法并不是执行在对方线程中，而是执行在自己的线程或binder线程中。  
如果处理耗时操作，需要转到子线程中，是因为接口方法如果不是oneway类型，客户端会阻塞，等待服务端返回。  
如上面，客户端调用mBinder.request("123")方法，会发生阻塞，等到服务端的request方法执行完才会继续执行。这个原因导致服务端执行时间影响客户端线程。