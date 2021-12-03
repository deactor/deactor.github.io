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
// 该接口Binder在服务端，由客户端通过IMyAidlInterface对象的getbinder()方法获取IBinder对象，再通过IRealBinder.Stub.asInterface()转换为IRealBinder对象来使用。
interface IRealBinder {
    int doRequst(String id);
}
```

最终实验验证结果：
1. 客户端调用服务端方法，服务端方法执行在binder线程中。
2. 客户端对服务端设置callback，客户端自身线程（如main线程）调用服务端方法A，服务端方法内调用callback的方法B。那么客户端callback的方法B相当于在方法A内执行，客户端在main线程调用的方法A,方法B则也运行于main线程中。
3. 客户端对服务端设置callback，客户端未调用服务端方法，或服务端未在客户端请求中调用callback的方法B,则客户端接到方法B的调用，方法B运行在客户端的Binder线程中。

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
        // 如果上面mBinder.request("123");是在主线程中执行的，那么这里也同样是在主线程。
        // 因为mBinder.request("123")调用后，会等服务端返回，服务端在request里是调用该response方法。
        // 即该response方法执行完，request方法才返回。也就相当于response方法是在request内执行的。
        // 如果response是服务端独立调用的，并不是在request方法内，那么客户端response执行在binder线程池中。
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
        // 客户端调用该方法传递ICallback对象过来，并不需要asInterface方法进行转换。
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

此外服务端的Binder方法是运行在Binder的线程池中的，所以Binder方法不管是否耗时都应采用同步的方式实现，以保证线程安全。

为什么说AIDL是多线程并发的？  
也就是因为服务端的Binder方法是运行在Binder的线程池中。如有两个客户端都与服务端建立了联系，他们是可以同时发起请求，因为他们在服务端的方法调用是运行在线程池中不同的线程里。
