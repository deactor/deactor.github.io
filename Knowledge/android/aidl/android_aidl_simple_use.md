---
title:  "Android Aidl简单Demo"
---
服务端aidlservice：

```
package com.wind.aidltestserver;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.os.Process;

/*参考自http://www.cnblogs.com/lipeil/archive/2012/08/27/2659330.html
* 1、AIDL 适用于 进程间通信，并且与Service端多个线程并发的情况(这个多线程并发是怎么体现的？见aidl线程篇)
* 2、AIDL语法：基础数据类型都可以适用，List Map等有限适用。static field 不适用。
*/

/*
* 接口描述文件
*1、导入的包名
*2、如果有使用Object对象，需要该对象 implement Parcelable 接口，并且需要导入该接口包名+类名;如果是primitive type 不需要这步。
*3、定义方法名称。
*4、所有的.aidl文件已经需要传递的对象接口需要在Service 与Client中各一份
* */

public class aidlservice extends Service{

    @Override
    public IBinder onBind(Intent arg0) {
        // TODO Auto-generated method stub
        return mBinder;
    }

    private final Iaidlserver.Stub mBinder = new Iaidlserver.Stub() {

        @Override
        public int getPid() throws RemoteException {
            // TODO Auto-generated method stub
            return Process.myPid();
        }

        @Override
        public String getName() throws RemoteException {
            // TODO Auto-generated method stub
            return "test";
        }

        @Override
        public Data getData() throws RemoteException {
            // TODO Auto-generated method stub
            Data data = new Data();
            data.id = Process.myUid();
            data.name = "Data--";
            return data;
        }
    };
}
```
Data.java文件：

```
package com.wind.aidltestserver;

import android.os.Parcel;
import android.os.Parcelable;

/*如果aidl文件中需要传递Object对象，需要添加对应.aidl文件
* 1.该对象需实现Parcelable
* 2.需要添加对应的.aidl文件，如这里的Data.aidl
* 3.在引用Data的.aidl文件（这里是Iaidlserver.aidl）中，import该Data类。
* 4.server和client都要添加Data.aidl与Data.java文件
* */

public class Data implements Parcelable{

    public int id;
    public String name;

    public static final Parcelable.Creator<Data> CREATOR = new Creator<Data>() {

        @Override
        public Data[] newArray(int size) {
            // TODO Auto-generated method stub
            return new Data[size];
        }

        @Override
        public Data createFromParcel(Parcel in) {
            // TODO Auto-generated method stub
            return new Data(in);
        }
    };

    public Data(){

    }

    private Data(Parcel in){
        readFromParcel(in);
    }

    public void readFromParcel(Parcel in){
        id = in.readInt();
        name = in.readString();
    }

    @Override
    public int describeContents() {
        // TODO Auto-generated method stub
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        // TODO Auto-generated method stub
        dest.writeInt(id);
        dest.writeString(name);
    }

}
```
接下来是Iaidlserver.aidl与Data.aidl文件：

```
package com.wind.aidltestserver;
import com.wind.aidltestserver.Data;
interface Iaidlserver
{
    int getPid();
    String getName();
    com.wind.aidltestserver.Data getData();
}
```

```
package com.wind.aidltestserver;
parcelable Data;
```
那么看下客户端：

```
package com.wind.aidlclienttest;

import android.app.Activity;
import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.os.Process;
import com.wind.aidltestserver.Iaidlserver;//server和client的aidl包名必须相同

public class MainActivity extends Activity {

    private TextView tv;
    private Button btn;

    private Iaidlserver mIaidlserver;

    private ServiceConnection conn = new ServiceConnection() {

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            // TODO Auto-generated method stub
            mIaidlserver = null;
            btn.setEnabled(false);
        }

        @Override
        public void onServiceConnected(ComponentName arg0, IBinder arg1) {
            // TODO Auto-generated method stub
            mIaidlserver = Iaidlserver.Stub.asInterface(arg1);
            btn.setEnabled(true);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tv = (TextView) findViewById(R.id.tv);
        btn = (Button) findViewById(R.id.btn);

        bindservice();

        btn.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View arg0) {
                // TODO Auto-generated method stub
                try {
                    tv.setText("this.pid = " + Process.myPid()+ " ;remote.pid = " + mIaidlserver.getPid() + " ;name = " + mIaidlserver.getName()
                        + ";data.tid = " + mIaidlserver.getData().id + "data.name = " + mIaidlserver.getData().name);
                } catch (RemoteException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        });
        btn.setEnabled(false);
    }
    private void bindservice(){
        Intent intent = new Intent("android.intent.action.aidl").setPackage("com.wind.aidltestserver");
        bindService(intent, conn, BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        // TODO Auto-generated method stub
        super.onDestroy();
        unbindService(conn);
    }
}
```
> 很早写的demo了，不想改了，这个目录结构主要想说明aidl的接口和数据类要在客户端和服务端各放一份，且保持一致，包名啥的都要一样。

看下服务端的目录结构：
+ src
    + com.wind.aidltestserver
        + aidlservice.java
        + Data.java
        + Data.aidl
        + Iaidlserver.aidl

看下客户端的目录结构：
+ src
    + com.wind.aidlclienttest
        + MainActivity.java
    + com.wind.aidltestserver
        + Data.java
        + Data.aidl
        + Iaidlserver.aidl

#### 注意：
在android源码中编译报错，import com.wind.aidltestserver.Iaidlserver 找不到。  
在源码环境下需要将aidl文件添加到LOCAL_SRC_FILES属性下，否则编译不到，则造成找不到aidl的错误。