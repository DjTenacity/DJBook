[AIDL ](https://www.jianshu.com/u/fb093dd92ed8)

AIDL 是 Android 特有的 IPC 进程间通讯方式

AIDL 的写法其实和绑定服务的代码差不多，IBander 也是 android 默认提供的一个 AIDL 接口

需要注意的是 5.0 之后，**不能隐式启动 service**，不能想以前一样定义 action 来启动服务了，尤其是不是跨应用启动服务，这也算是一种安全上的考虑

若是想非常详细的了解 AIDL ，请看慕课网的科普视频

- AIDL-小白成长记

### AIDL 写法

1 使用 Android 提供的方式生命一个 AIDL 接口

![](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUss7KGd4a31v5fia1iaiatP8XQXu22eYMAibUdVuzcSBwPiarATibzalwtQVVw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​	然后在这个AIDL 接口中声明业务需要的方法

```java
// IBanZheng.aidl
package com.bloodcrown.bcremoteservice.aldl;
// Declare any non-default types here with import statements
interface IBanZheng {

    void banZheng();

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```

这就是系统帮我们创建的 AIDL 接口，在里面我写了一个 banZheng() 的方法，剩下的都是系统的事了，Android 系统会根据我们声明的这个 AIDL 接口创建一个相关的 IPC 通讯类出来：

位置在：

![](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsJXPGDBI7aBLYmVQB6agpwib69n3ddiaJ30tqSxCuxluncEEfH5XV57AA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

系统会在帮我们创建出一个单独的 aidl 包出来，里面放我们声明的 AIDL 类，注意不是实现类

![](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUszWzDYiaLmxJAAmh36xXAY3WDuv5jbS3Q7aABVibwQWfAHAfy2rqJTUMg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

详细的代码，有点长，系统帮我们添加的代码都是进行 ipc通讯的

```java
package com.bloodcrown.bcremoteservice.aldl;
// Declare any non-default types here with import statements

public interface IBanZheng extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.bloodcrown.bcremoteservice.aldl.IBanZheng {
        private static final java.lang.String DESCRIPTOR = "com.bloodcrown.bcremoteservice.aldl.IBanZheng";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.bloodcrown.bcremoteservice.aldl.IBanZheng interface,
         * generating a proxy if needed.
         */
        public static com.bloodcrown.bcremoteservice.aldl.IBanZheng asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.bloodcrown.bcremoteservice.aldl.IBanZheng))) {
                return ((com.bloodcrown.bcremoteservice.aldl.IBanZheng) iin);
            }
            return new com.bloodcrown.bcremoteservice.aldl.IBanZheng.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_banZheng: {
                    data.enforceInterface(DESCRIPTOR);
                    this.banZheng();
                    reply.writeNoException();
                    return true;
                }
                case TRANSACTION_basicTypes: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    long _arg1;
                    _arg1 = data.readLong();
                    boolean _arg2;
                    _arg2 = (0 != data.readInt());
                    float _arg3;
                    _arg3 = data.readFloat();
                    double _arg4;
                    _arg4 = data.readDouble();
                    java.lang.String _arg5;
                    _arg5 = data.readString();
                    this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.bloodcrown.bcremoteservice.aldl.IBanZheng {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public void banZheng() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_banZheng, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }

            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(anInt);
                    _data.writeLong(aLong);
                    _data.writeInt(((aBoolean) ? (1) : (0)));
                    _data.writeFloat(aFloat);
                    _data.writeDouble(aDouble);
                    _data.writeString(aString);
                    mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_banZheng = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    public void banZheng() throws android.os.RemoteException;

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
```

其实我们主要看这个类：

```java
public static abstract class Stub extends android.os.Binder implements com.bloodcrown.bcremoteservice.aldl.IBanZheng
```

和我们在绑定服务时，在服务中写的用户返回的内部类一样不一样，都是继承 Binder 类，实现我们自己声明的接口，然后我们使用也是使用这个stub 类，我们的目的就是让系统帮我们创建出这个 stub 类 

1. 在 service 中使用这个 stub 类

现在我们写的内部类直接继承这个 stub 即可

```java
 class MediaIBander extends IBanZheng.Stub {

        @Override
        public void banZheng() throws RemoteException {
            Log.d(TAG, "办证啦...");
        }
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }
    }
```

然后在绑定生命周期函数内返回这个内部类

```java
@Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "remoteService - onBind...");
        return new MediaIBander();
    }
```

在 activity 中接受数据，转换对象类型

```java
public class MyRemoteServiceConnection implements ServiceConnection {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iBanZheng = IBanZheng.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    }
```

### AIDL 深入理解

AIDL 是 android 系统 IPC 进程间通讯协议，但是记住仅仅只是 android ，换个平台就不是 AIDL 了。

android 中每个进程都有自己独立的虚拟机 ， 每个虚拟机的内存是时独立的，所以进程间通信依靠传递对象引用是不行的，因为内存是不连续的， A进程 的内存中有的对象，你把对象引用给到 B进程内存里面可是没这个对象的，所以 google 就提供了 AIDL

AIDL  
![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsF2KQGTpKiapN6cudgJXQL1yxjHluLNGSNIsYYPRl9odmHUETuW0MpQw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进程1 的请求会通过 AIDL 发送给系统，系统根据请求标识找到进程2，把进程1 的请求交给进程2去处理，同理进程2处理完后把结果通过 AIDL 再发送给进程1

AIDL 只支持基本数据类型，集合，Parcelable 序列化类型数据的传输

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsp9s9QwTSic69THyh9RjLHb2gc5ibmI0QmMH18mp41b7pPd29CJX0Ms3g/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

AIDL 方法种若要传递对象类型，对象类型需要实现 Parcelable 序列化接口。Parcelable 的原理就是把大的数据类型对象，打散成一个个系统能支持的基本数据类型数据，然后集中打个包传递过去，到目标进程再组合成对象类型。这个过程也叫打包，拆包

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUs8OsQK1b6WRYxyHcpYcwjaLfjQj14RlGm7e74FCj9VXOu9IpUibUoe0w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

系统帮我们生成的 AIDL 对象，里面一个 Stub 类型的内部类，Stub 里面又有一个 Proxy 类型的内部类

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsia41wNWictsQsNGIsrjgNGlxnnOHiamwCoo9k6sf8ctX4icVbgaQ1BKmWw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

IPC 通信的核心方法就在于其中的 transact 和 onTransact 方法

- transact 会调用系统底层 IPC 去传递数据
- onTransact 会接受系统底层 IPC 传递过来的数据

我们跟着代码来看一下：

1. 声明一个 AIDL 类，内部有一个叫 banZheng 的方法

```java
   interface IBanZheng {
   
       void banZheng();
   }
```

2. 获取远程进程代理

   ```java
    @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               iBanZheng = IBanZheng.Stub.asInterface(service);
           }
   ```

   上面这是客服端获取远程服务的 binder ，我们跟进去看看

```java
 public static com.bloodcrown.bcremoteservice.aldl.IBanZheng asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.bloodcrown.bcremoteservice.aldl.IBanZheng))) {
                // 是同一个进程，返回 Stub 对象本身
                return ((com.bloodcrown.bcremoteservice.aldl.IBanZheng) iin);
            }
            // 不是同一个进程，返回远程进程的代理
            return new com.bloodcrown.bcremoteservice.aldl.IBanZheng.Stub.Proxy(obj);
        }
```

这个方法是 Stub 的方法，我们可以看到，当服务端和客户端不再同一个进程时，其实我们拿到的只是远程进程的代理类，这个代理类会帮我们进程 IPC 底层的通讯

1. Proxy 调用底层通讯方法

```java
   @Override
               public void banZheng() throws android.os.RemoteException {
                   android.os.Parcel _data = android.os.Parcel.obtain();
                   android.os.Parcel _reply = android.os.Parcel.obtain();
                   try {
                       // 先把数据打包成可以通过 IPC 通讯传递
                       _data.writeInterfaceToken(DESCRIPTOR);
                       // 调用底层 IPC 方法传递数据(此时线程会挂起，也就是卡线程了)
                       mRemote.transact(Stub.TRANSACTION_banZheng, _data, _reply, 0);
                       // 等待远程返回结果(此时线程会激活，重新执行下面任务)
                       _reply.readException();
                   } finally {
                       _reply.recycle();
                       _data.recycle();
                   }
               }
```

Proxy 实现了我们声明的 AIDL 接口，所以我们掉远程 binder 的方法时就是调用的 Proxy 的相关方法，我们看上面的方法，先把数据序列化打包，再调用底层的 transact 方法进行 IPC 通讯，然后等待远程返回结果，这里会卡住线程，所以客户端的 IPC 请求最好在非 UI 线程执行

1. Stub 的 onTransact 方法是服务端核心，服务端在 onTransact 方法种接受到客服端传过来的参数，然后计算，再把结果写会去，客户端才能收到结果，注意服务端的 binder 是在系统的 binder 线程池中执行的，不要我们自己再起线程了。

------

### AIDL 远程是异步方法

不知道大家有没有想过，远端若是耗时方法的话，会不会卡客户端线程，我们来测试一下，在两端分别打印日志，标记关键节点时间，远端延迟3秒返回数据，大家看一下结果就能清楚了

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUscToHr87xD2og0qaHUybRxTobniagXuJZsib27icf7ia2EFBZdtfhGOw5FQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

客户端

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsD2emgHovlDyHDwmGoKo06sABe6JaUic04icicRe6bZqu7RbbzYSww9b4A/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

服务端

看日志很清楚了吧，客户端调 AIDL 方法可是会卡主线程的， 所以我们需要注意啊，远端若是耗时的话，我们需要在客户端点开线程，再进行 AIDL 远程通信

我们现在知道了客户端的  AIDL 方法是异步任务会卡主线程，那么大家就不想知道服务端的方法是怎么运行的吗

系统有句话这么说：

> 服务端的 binder 方法运行在系统的 AIDL 线程池里

换句话说，AIDL 在服务端已经跑在单独的线程里了，不用我们自己开线程了，这里我测试了下，打印服务端启动时所在线程和 binder 执行任务时所在线程

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsmUSSsSGlxDnaaDUHyBwDiaUvHll0j5bwIrkVpJrOH7tt3Bno1Pa9A8w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)img

看日志就清楚了把，系统对于 AIDL 在服务端是有优化的，自动开线程池

------

### AIDL 双向通讯

AIDL 是单项通讯的，假设我们实现了 AIDL  A -> B 进程的通信，那么在 A binder 联通的时刻把 A 实现的 AIDL.Stub 传递给 B ，B 就能通过这个传过来的  AIDL.Stub 实现 B -> A 的通讯了，AIDL.Stub 对象可以直接 IPC 传递的

参考例子可以看：

- Android AIDL SERVICE 双向通信 详解

------

### binder 如何理解

先看图：

![img](https://mmbiz.qpic.cn/mmbiz/Jkr8IKxMfUcj6SE3B6PHGa5gQHialNeUsBzVvaCWZ2GIfs1jMS07f281BMiciaPyU6Nlulz8OMyHwnj5Pnv1kXO9w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)img

binder 和 AIDL 一样都是 android 的 IPC 通信机制，不同于 unix 其他的 IPC ，binder 对每个进程的 UID 支持非常好，安全性高

至于我们在使用时感觉 binder 不是夸进程的，那是错觉。android 4大组件都是有声明周期的，什么时候 应该怎么运行都是由 ActivityManageService 控制的，看上面的图应该知道 binder 是组件与 ActivityManageService 通信的，若 Activity 与 Service 在同一个进程，那么内存共享，传递的数据可以给过去，这就是我们常见的与 Service 的通信。

若 Activity 与 Service 不在同一个进程，内存不能共享，那么数据就得通过 android 系统特有的 AIDL IPC 通道先序列化然后过去再反序列化，AIDL 的意思就在于跨进程传递数据了

详细的大家请看：

- Android Binder机制浅析

------

### AIDL 解读补充

AIDL 跨进程通信的核心 Proxy ，大家看构造方法，注意 remote 就是那个远程 Service，remote 恰恰就是 IBinder 对象，所以 IBinder 才是 android 中 IPC 通讯的基石

```java
private static class Proxy implements com.lypeer.ipcclient.BookManager {
    private android.os.IBinder mRemote;

    Proxy(android.os.IBinder remote) {
        //此处的 remote 正是前面我们提到的 IBinder service
        mRemote = remote;
    }

    @Override
    public java.util.List<com.lypeer.ipcclient.Book> getBooks() throws android.os.RemoteException {
        //省略
    }

    @Override
    public void addBook(com.lypeer.ipcclient.Book book) throws android.os.RemoteException {
        //省略
    }
    //省略部分方法
}
```

我们在客户端抓换 binder 为指定接口时系统的操作，判断 binder 在本进程有没有实例，就是客户端和服务端是不是在一个进程，是的话就返回对象，queryLocalInterface 方法就是干的这事，不在一个进程的话，就需要 proxy 代理了，proxy 代理了 AIDL 实现的接口方法里的数据序列化，反序列化，至于 IPC 通信靠的还是 binder 去实现

```java
public static com.lypeer.ipcclient.BookManager asInterface(android.os.IBinder obj) {
    //验空
    if ((obj == null)) {
        return null;
    }
    //DESCRIPTOR = "com.lypeer.ipcclient.BookManager"，搜索本地是否已經
    //有可用的对象了，如果有就将其返回
    android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
    if (((iin != null) && (iin instanceof com.lypeer.ipcclient.BookManager))) {
        return ((com.lypeer.ipcclient.BookManager) iin);
    }
    //如果本地没有的话就新建一个返回
    return new com.lypeer.ipcclient.BookManager.Stub.Proxy(obj);
}
```

我们在客户端调用 AIDL 的方法，过程是下面这样走的，核心就是调起 IBinder 类型的 mRemote 对象的 transact 方法，transact 方法就会进行跨进程通信了

```java
@Override
public java.util.List<com.lypeer.ipcclient.Book> getBooks() throws android.os.RemoteException {
    //很容易可以分析出来，_data用来存储流向服务端的数据流，
    //_reply用来存储服务端流回客户端的数据流
    android.os.Parcel _data = android.os.Parcel.obtain();
    android.os.Parcel _reply = android.os.Parcel.obtain();
    java.util.List<com.lypeer.ipcclient.Book> _result;
    try {
        _data.writeInterfaceToken(DESCRIPTOR);
        //调用 transact() 方法将方法id和两个 Parcel 容器传过去
        mRemote.transact(Stub.TRANSACTION_getBooks, _data, _reply, 0);
        _reply.readException();
        //从_reply中取出服务端执行方法的结果
        _result = _reply.createTypedArrayList(com.lypeer.ipcclient.Book.CREATOR);
    } finally {
        _reply.recycle();
        _data.recycle();
    }
    //将结果返回
    return _result;
}
```

然后在服务端的 onTransact 方法接受远程数据，反序列化出来，执行操作过程，然后通过 reply.writeString 把结果写回去

```
@Override
public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
    switch (code) {
        case INTERFACE_TRANSACTION: {
            reply.writeString(DESCRIPTOR);
            return true;
        }
        case TRANSACTION_getBooks: {
            //省略
            return true;
        }
        case TRANSACTION_addBook: {
            //省略
            return true;
        }
    }
    return super.onTransact(code, data, reply, flags);
}
```

参考自：

- Android：学习AIDL，这一篇文章就够了(下)

------

### 使用 Messenger，handle 实现 IPC

Messenger 默认实现了 ibinder ，是对 AIDL 的封装，大体的使用过程如下，我就不贴代码了，看代码的请看：

- Android中的Service：Binder，Messenger，AIDL（2）

1. 服务端实现一个Handler，在 onBind 时 return mMessenger.getBinder() 返回给自客户端用来通信
2. 客户端接受 messager 对象  Messenger mService = new Messenger(service);
3. 通过 Messenger 发送消息 mService.send(msg);
4. 客户端也给服务端提供一个 Messenger 就能实现双向通信了

------

### AIDL中的 in，out，inout

什么是 in，out，inout ，是 AIDL 声明参数在进程2端作用域的标记，看下面这个 AIDL 接口

```
// BookManager.aidl
package com.lypeer.ipcclient;
import com.lypeer.ipcclient.Book;

interface BookManager {    
    //保证客户端与服务端是连接上的且数据传输正常
    List<Book> getBooks();

    //通过三种定位tag做对比试验，观察输出的结果
    Book addBookIn(in Book book);
    Book addBookOut(out Book book);
    Book addBookInout(inout Book book);
}
```

AIDL 的参数默认是 in 的，一般我们也不写

- in ： 参数在服务端的任何变化不会映射到客户端
- out：服务端接受的是一个 空内容的变量对象，在 服务端 对这个变量的任何修改都会映射到客户端
- inout： 服务端接受的是客户端传过来的参数有内容，在 服务端 对这个变量的任何修改都会映射到客户端，而客户端的修改不会映射到服务端

详细可以看：

- 你真的理解AIDL中的in，out，inout么？

最后我说一下，用 AIDL 双向通讯的话不用 in，out 来实现，连官方也不是用 in，out 来实现的，其次我没发现  in，out 的应用场景，暂时作为知识点了解

------

### 参考资料

- Android Studio 创建AIDL Demo
- [Android跨进程通信IPC之11——AIDL
  这个是一个 IPC 通信的系列，](https://www.jianshu.com/p/232fcbee0488)

# 

## 

## 

## 推荐阅读

[Android 进程间通信](http://mp.weixin.qq.com/s?__biz=MzI4MTQyNDg3Mg==&mid=2247484445&idx=1&sn=da59e39c6710e92cd7a31fec543ec0ff&chksm=eba8229adcdfab8c3f11ff24ec5432eb619ef018711a25c7cc5cf621d54a35e16c6ae078703c&scene=21#wechat_redirect)

[Android 系统如何预置 APP？](http://mp.weixin.qq.com/s?__biz=MzI4MTQyNDg3Mg==&mid=2247485643&idx=1&sn=28d7a95b2d1bb92c0400523e59f8ff7b&chksm=eba82e4cdcdfa75ae9fcf16acff5cace11fbf0cc95bc4612fb21e1d78fc0e87d01b3ddcd297c&scene=21#wechat_redirect)