# 学Android 这么久，intent传递数据最大多少呢？

当我们用Intent传输大数据时，有可能会出现错误：

```kotlin
val intent = Intent(this@MainActivity, Main2Activity::class.java)
val data = ByteArray(1024 * 1024)
intent.putExtra("111", data)
startActivity(intent)
```

如上我们传递了1M大小的数据时，结果程序就一直反复报如下TransactionTooLargeException错误：



![img](https:////upload-images.jianshu.io/upload_images/15233518-62ffe11752bb9242?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



但我们平时传递少量数据的时候是没问题的。由此得知，通过intent在页面间传递数据是有大小限制的。本文我们就来分析下为什么页面数据传输会有这个量的限制以及这个限制的大小具体是多少。

###    ![img](https:////upload-images.jianshu.io/upload_images/15233518-77cb6a59fa7ec372.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1/format/webp)   

##### startActivity流程探究

首先我们知道Context和Activity都含有startActivity,但两者最终都调用了Activity中的startActivity:

```java
@Override
public void startActivity(Intent intent, @Nullable Bundle options) {
    if (options != null) {
        startActivityForResult(intent, -1, options);
    } else {
        // Note we want to go through this call for compatibility with
        // applications that may have overridden the method.
        startActivityForResult(intent, -1);
    }
}
```

 

而startActivity最终会调用自身的startActivityForResult，省略了嵌套activity的代码：

```java
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
        @Nullable Bundle options) {
  options = transferSpringboardActivityOptions(options);
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, this,
                intent, requestCode, options);
        if (ar != null) {
            mMainThread.sendActivityResult(
                mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                ar.getResultData());
        }
        if (requestCode >= 0) {
            // If this start is requesting a result, we can avoid making
            // the activity visible until the result is received.  Setting
            // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
            // activity hidden during this time, to avoid flickering.
            // This can only be done when a result is requested because
            // that guarantees we will get information back when the
            // activity is finished, no matter what happens to it.
            mStartedActivity = true;
        }

        cancelInputsAndStartExitTransition(options);
}
```

 

```java
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, Bundle options) {
    IApplicationThread whoThread = (IApplicationThread) contextThread;
   ...
    try {
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess(who);
        int result = ActivityManager.getService()
            .startActivity(whoThread, who.getBasePackageName(), intent,
                    intent.resolveTypeIfNeeded(who.getContentResolver()),
                    token, target != null ? target.mEmbeddedID : null,
                    requestCode, 0, null, options);
        checkStartActivityResult(result, intent);
    } catch (RemoteException e) {
        throw new RuntimeException("Failure from system", e);
    }
    return null;
}
```

 。

接着调用了ActivityManger.getService().startActivity ,getService返回的是系统进程中的AMS在app进程中的binder代理：

```java
/**
 * @hide
 */
public static IActivityManager getService() {
    return IActivityManagerSingleton.get();
}

private static final Singleton<IActivityManager> IActivityManagerSingleton =
    new Singleton<IActivityManager>() {
        @Override
        protected IActivityManager create() {
            final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
            final IActivityManager am = IActivityManager.Stub.asInterface(b);
            return am;
        }
};
```

接下来就是App进程调用AMS进程中的方法了。简单来说，系统进程中的AMS集中负责管理所有进程中的Activity。app进程与系统进程需要进行双向通信。

比如打开一个新的Activity，则需要调用系统进程AMS中的方法进行实现，AMS等实现完毕需要回调app进程中的相关方法进行具体activity生命周期的回调。

所以我们在intent中携带的数据也要从APP进程传输到AMS进程，再由AMS进程传输到目标Activity所在进程。有同学可能由疑问了，目标Acitivity所在进程不就是APP进程吗？

其实不是的，我们可以在Manifest.xml中设置android:process属性来为Activity, Service等指定单独的进程，所以Activity的startActivity方法是原生支持跨进程通信的。

##### 接下来简单分析下binder机制。

**binder介绍**



![img](https:////upload-images.jianshu.io/upload_images/15233518-f7a413484b0edc83?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)



普通的由Zygote孵化而来的用户进程，所映射的Binder内存大小是不到1M的，准确说是 110241024) - (4096 *2) ：这个限制定义在

frameworks/native/libs/binder/processState.cpp类中，如果传输说句超过这个大小，系统就会报错，因为Binder本身就是为了进程间频繁而灵活的通信所设计的，并不是为了拷贝大数据而使用的：

```c++
#define BINDER_VM_SIZE ((1*1024*1024) - (4096 *2))
```

并可以通过cat proc/[pid]/maps命令查看到。

而在内核中，其实也有个限制，是4M，不过由于APP中已经限制了不到1M，这里的限制似乎也没多大用途：

```
static int binder_mmap(struct file *filp, struct vm_area_struct *vma)
{
    int ret;
    struct vm_struct *area;
    struct binder_proc *proc = filp->private_data;
    const char *failure_string;
    struct binder_buffer *buffer;
    //限制不能超过4M
    if ((vma->vm_end - vma->vm_start) > SZ_4M)
        vma->vm_end = vma->vm_start + SZ_4M;
    。。。
    }
```

其实在TransactionTooLargeException中也提到了这个：

> The Binder transaction buffer has a limited fixed size, currently 1Mb, which
>
> is shared by all transactions in progress for the process.  Consequently this
>
> exception can be thrown when there are many transactions in progress even when
>
> most of the individual transactions are of moderate size.



![img](https:////upload-images.jianshu.io/upload_images/15233518-67e37acab1b4978d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



只不过不是正好1MB，而是比1MB略小的值。

##### 小结

至此我们来解答开头提出的问题，startActivity携带的数据会经过BInder内核再传递到目标Activity中去，因为binder映射内存的限制，所以startActivity也就会这个限制了。

##### 替代方案

一、写入临时文件或者数据库，通过FileProvider将该文件或者数据库通过Uri发送至目标。一般适用于不同进程，比如分离进程的UI和后台服务，或不同的App之间。之所以采用FileProvider是因为7.0以后，对分享本App文件存在着严格的权限检查。

二、通过设置静态类中的静态变量进行数据交换。一般适用于同一进程内，这样本质上数据在内存中只存在一份，通过静态类进行传递。需要注意的是进行数据校对，以防多线程Data Racer出现导致的数据显示混乱。

##### 参考资料

听说你Binder机制学的不错，来面试下这几个问题（一）

<https://www.jianshu.com/p/adaa1a39a274>

[源码分析:startActivity流程](https://www.jianshu.com/p/dc6b0ead30aa) 

插入个内容，文中提到一个

> 这个限制定义在frameworks/native/libs/binder/processState.cpp类中

很多同学可能不知道去哪里看这个类，这里跟大家分享一个非常快速便捷的查看方式，比较适合偶尔查找一两个类：



![img](https:////upload-images.jianshu.io/upload_images/15233518-e5c494bf5ccef691?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



直接进入搜索就好了，包含各个版本的源码：



![img](https:////upload-images.jianshu.io/upload_images/15233518-6dc4622dcc3be509?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



如果你指定了某个具体的版本，还有非常便利的提示：



![img](https:////upload-images.jianshu.io/upload_images/15233518-f0c850cc0a27247e?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)



###  

作者：Android进阶开发

链接：https://www.jianshu.com/p/deb696063c7d

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。