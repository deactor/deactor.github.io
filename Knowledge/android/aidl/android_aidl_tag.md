---
title:  "Android aidl定向tag"
---
>AIDL中的定向 tag 表示了在跨进程通信中数据的流向:  
>+ in 表示数据只能由客户端流向服务端
>+ out 表示数据只能由服务端流向客户端  
>+ inout 则表示数据可在服务端与客户端之间双向流通
>
>其中，数据流向是针对在客户端中的那个传入方法的对象而言的。  
>+ in :表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；
>+ out :表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；
>+ inout :表现为服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。

>**注：**  
这里的客户端和服务端指的是实现aidl的stub的一方。不是说实现service的一方。比如回调接口callback.aidl，是由客户端实现stub，然后传递给service，供service给client回调数据。那么callback.aild中的tag应当是in。虽然数据流向是从service到client。但是callback的stub是在client中实现的，tag是针对stub的，此时的数据是流向实现stub方的，所以此时tag是in。

### 参考
https://www.open-open.com/lib/view/open1469494342021.html