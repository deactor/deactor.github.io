---
title:  "Android aidl原理简要说明"
---
### 原理简述：
构建工具编译时将接口文件生成相应的Java文件，包含用于服务端接收数据的stub子类和用于客户端发送数据的proxy子类,采用的是匿名binder，在传输过程中将BBinder转化为了BpBinder(转化过程见参考，不重复写)。实际aidl的跨进程就是Binder通信，最终都是Native下的BBinder和BpBinder之间通过Binder驱动的交互过程。  

>关于Binder（吹牛逼描述，忘了从哪里抄来的了，虽然没什么用，留着看看吧）：  
>Binder是Android中的一个类，它实现了IBinder接口。  
> + 从IPC角度来说，Binder是Android的一种跨进程通信方式，Binder还可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder，这种通信方式在Linux中是没有的。  
>+ 从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager，WindowManager等）和相应ManagerService的桥梁。  
>+ 从APP的角度来说，Binder是客户端和服务端进行通信的媒介。

举个例子：
```
//IBookManager.aidl
package com.qdd.test.aidl;
interface IBookManager{
    List<Book> getBookList();
    void addBook(in Book book);
}
```
在IDE中，会自动生成一个IBookManager.java类，其中包含：
```
package com.qdd.test.aidl;
public interface IBookManager extends android.os.IInterface{

    public List<Book> getBookList() throws android.os.RemoteException;
    public void addBook(Book book) throws android.os.RemoteException;

    public static abstract class Stub extends android.os.Binder implements IBookManager{
        private static final String DESCRIPTOR = "com.qdd.test.aidl.IBookManager";
        
        public Stub(){
            this.attachInterface(this,DESCRIPTOR);
        }
        
        public static IBookManager asInterface(Ibinder obj){
            if(obj == null)
                return null;
            ...
            return new IBookManager.Stub.Proxy(obj);
        } 

        @Override
        public android.os.IBinder asBinder(){
            return this;
        }

        @Override
        public boolean onTransact(...){
            ...
        }

        public List<Book> getBookList() throws android.os.RemoteException{
                …
        }

        public void addBook(Book book) throws android.os.RemoteException{
                …
        }
        ...

        private static class Proxy implements IBookManager{
            private android.os.IBinder mRemote;
            
            Proxy(android.os.IBinder remote){
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder(){
                return mRemote;
            } 
                
            public List<Book> getBookList() throws android.os.RemoteException{
                ...
            }

            public void addBook(Book book) throws android.os.RemoteException{
                ...
            }
        }
    }
}
```

简述：
>aidl文件实际就是google为了开发者方便，提供给开发者使用，自动生成对于java接口的工具。我们完全可以不写aidl文件，直接自己实现继承IInterface接口的跨进程接口（对应java文件），只是应用开发没有必要给自己增加难度。

>aidl仅仅是个接口，通过这个接口（Binder作为载体）可以实现跨进程的通讯。具体方法的实现依然是在对应的进程当中。可以看到aidl的接口必须是继承自IInterface的，既然Binder是载体，那么Binder对象就需要实现这个接口，也就是实现子类Stub（继承自Binder），并实现定义的aidl接口。这个Stub中还有一个Proxy内部类（也实现了aidl接口）用来处理远程调用（为什么这个Proxy要在Stub内部？）。

>从AIDL实现的笔记中可以看到，服务端是new Binder.Stub的，并复写了aidl中声明的方法。而且客户端和服务端要有相同的aidl文件。  
其实整个的一个远程调用的过程就是一个序列化和反序列化的过程。客户端与服务端建立连接，进行一次远程调用，发送了一个data输入参数，一个reply参数，服务端接收data参数，在服务端执行客户端调用的方法，并将结果放入reply参数中，发回给客户端，客户端得到调用结果。  

### IInterface IBinder Binder是什么？asInterface()和asBinder()方法的区别？
+ IInterface : 是一个接口，只定义了一个方法`public IBinder asBinder();` 
+ IBinder : 也是一个接口，内部定义了binder通讯的一些协议，并不能被应用直接使用。
+ Binder : 是一个基类，它实现了IBinder接口，并且封装了底层的Binder Driver。
  
>asInterface()和asBinder()方法的区别？  
>   + asInterface():将IBinder对象转换为具体的IInterface对象。
>   + asBinder():返回IBinder对象。
>  
>这里Stub类既是Binder子类，也是IBookManager的实现类，且Binder又是IBinder的实现类，IBookManager又是IInterface的子类。所以Stub类对象(服务端持有为BBinder)即可以当作IBinder，又可以当作IInterface。在客户端拿到IBinder(此时已经是BpBinder)后要通过asInterface()方法创建Proxy对象，为IBookManager类型（可以按转型理解，类型不对自然无法调用对应类型的方法）。

>为什么服务端要实现Stub类？  
因为Stub类继承自Binder，Binder是实现了IBinder定义的通讯协议，可以真正用来通讯的。且Stub类还有一个Proxy内部类，这个Proxy实际就是给客户端用的，只有Stub中是会真正调用我们定义和实现的方法的，Proxy中我们定义的方法仅仅是在做传输，所以服务端是具体实现的地方，就要继承Stub子类并实现定义的方法。使用Binder的说法来说，就是：客户端与服务端建立连接，客户端从Binder Driver中拿到mRemote对象（IBinder实例，Proxy里面的mRemote），然后组包，调用mRemote的transact方法按序发送数据包。服务端接到请求，调用自身onTransact方法，对数据包进行解包，调用对应的方法，并将运行结果组包发回。


## 参考
[匿名binder](https://www.jianshu.com/p/0de4c4017052)  
《android开发艺术探索》