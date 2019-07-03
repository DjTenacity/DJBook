# [安卓的数据共享——从一个APP中调用另一个APP数据的方法](https://blog.csdn.net/Spring_East/article/details/80915437 )

在Android中如何在一个APP中调用另一个APP中的数据呢？大致有以下五种方法可以实现

1、首选项信息-Shared Preferences

2、文件

3、SQLite

4、Content Provider

5、广播

6 .  用同一个UserId来共享数据（只有签名作者相同的才能共享UserId）

下来对上述的五种方法进行详细的解析：

### Shared Preferences

1、首先对于Shared Preferences系统提供了三种权限的管理模式，分别是

            MODE_PRIVATE: 私有模式
    
            MODE_WORLD_READABLE: 全局可读模式
    
            MODE_WORLD_WRITEABLE: 全局可写模式
    
    将访问权限模式改为MODE_WORLD_READABLE，然后运行填入一组数据，则可以对外部应用公开，创建的数据文件可以被其它应用程序读取。
    
    通过Share Perferences类存储的首选项信息数据只能是以键值对的形式处理。

2、对于文件而言，可以按照自定义的格式来保存和读取少量数据，通常使用数据文件的格式。

    Android的数据文件的读写操作采用了Java API中的FileInputStream类和FileOutputStream类，以及相关的一系列方法。数据可以保存在机身内存中或者SD卡中，为了共享数据的方便，一般将数据文件保存在SD卡中。
    
    将数据文件放置在SD卡是，要在AndroidManifest.xml文件中申明对SD卡的使用权限和在程序中编写检查SD卡是否可用的程序，因为在程序的运行过程中有可能出现SD卡暂时不能用的情况。

3、SQLite数据库

    SQLite是一种轻量级的基于文件的数据库管理系统，具有小巧、高效的特点，特别适合用于手机等嵌入式设备中来进行大量数据的存储和各种操作。Android平台提供了对SQLite的良好支持，尤其是提供了各种数据库操作的API，方便开发者编写应用程序。

4、Content Provider

    我们可以通过ContentResolver来访问ContentProvider中提供的数据，ContentResolver是一个抽象类，我们可以通过Context的getContentResolver来获取，实际上获取的是ApplicationContentResolver，ApplicationContentResolver继承自ContentResolver，当ContentProvider所在的进程没有启动的时候，第一次访问时候，会触发ContentProvider的创建和其所在进程的启动，通过增删查改四个方法都可以触发。

5、广播

    本应用通过广播将数据发送给所有的应用以实现数据共享。广播只适合少量数据的共享。
---------------------
 

6 用同一个UserId来共享数据（只有签名作者相同的才能共享UserId）

在<manifest>文件中android:sharedUserId属性  来使两个应用在同一进程中运行

用同一个UserId来共享数据（只有签名作者相同的才能共享UserId）

第一个app（产生）

```java
//把数据以文件形式保存在手机内存中

String data="aaaa";

FileOutputStream fos = openFileOutput("a.txt",Mode_Append(追加))；

捕捉异常  Exception

fos.write(data.getBytes());

fos.flush();//刷新输出流

finally{关闭}
```

最后Manifext中

```
android：shareduserid“”自己起id保证唯一性“”
```

在另外一个app中读取刚才app产生的数据文件

1 Manifext 

```java
btnRead.setOnclickListener{//监听器

//拿到第一个app上下文对象

Context context=createPackageContext（"com.ni....."（packName）,Context.context_ignore_security（忽略安全性））

//通过上下文对象找到资源

File file=new File（context.getFileDir（）+"/"+"a.txt"）

if（file,exists（））{FileInputStream fis=new FileInputStream（file）

把数据读到字节数组中

byte【】data=new byte【fis.available（）】；

fis.read（data）;

把数据转换正字符串

String dataStr=new String（data）；

//将字符串显示到textview上

rvshow。setText（dataStr）;

fis.close（）；最后关闭流

}

}
```





作者：十二分热爱 
来源：CSDN 
原文：https://blog.csdn.net/weixin_41988545/article/details/80082064 
版权声明：本文为博主原创文章，转载请附上博文链接！