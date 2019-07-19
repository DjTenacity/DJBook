# Android多线程：理解和简单使用总结

## 一、Android中的线程

### 1.1 定义

> 线程，可以看作是进程的实体，CPU调度资源的基本单位。本质上是一串命令（也就是程序代码），执行线程可以理解为把命令交给操作系统去执行。
>  **Java中的线程：**Java中默认一个进程只有一个线程，称之为主线程。其它线程称之为子线程也叫工作线程。
>  **Android中的线程：**Android沿用了Java线程模型，Android中主线程也叫UI线程。Android3.0以后，系统要求网络访问必须在子线程中进行。

 

### 1.2 特点

> 线程基本不拥有系统资源，只拥有在运行时必不可少的系统资源（程序计数器，一组寄存器和栈）。可以并发执行。

- ## 二、Android中线程分类及作用

  ### 2.1 按用途分类：

  -  **主线程：**又叫UI线程，由ActivityThread管理

  > 作用：运行四大组件，和用户交互以及更新UI。

  - **子线程**

  > 作用：处理耗时操作，比如网络请求，复杂计算等。

  ### 2.2 按形态分类：

  - **Thread**

  > **说明：**基本的线程，可以做一些简单的操作，经常配合Handler使用。
  >  **相关面试题：**线程的几种状态、线程安全和同步问题、如何解决线程安全问题，下文都有概述。
  >  [Android 多线程：Thread理解和使用总结](https://www.jianshu.com/p/49349eee9abc)

  - **AsyncTask**

  > **说明：**轻量级的异步操作类，方便更新UI。
  >  **相关面试题：**AsyncTask的原理、AsyncTask的优点和缺点。
  >  [Android 多线程：AsyncTask理解和使用总结](https://www.jianshu.com/p/de41c082f5c1)

  - **HandlerThread**

  > **说明：**一个使用了Looper、Handler的线程。
  >  **主要作用：**方便地实现每隔几秒更新数据的功能，如价格，图片等。比Timer使用方便并且内存占用低。
  >  [Android 多线程：HandlerThread理解和使用总结](https://www.jianshu.com/p/a03ef2a0c026)

  - **IntentService**

  > **说明：**封装了HandlerThread和一个Handler，是HandlerThread的具体使用，由于属于Service，若以比单纯的线程优先级更高。
  >  [Android 多线程：IntentService理解和使用总结](https://www.jianshu.com/p/783015252b04)
  >  [Android进程优先级](https://www.jianshu.com/p/35727ad2296a)

  - **线程池**

  > **相关面试题：**线程池的使用、线程池的种类以及区别。
  >  [Android 多线程：线程池理解和使用总结](https://www.jianshu.com/p/ebdc3b3e136e)

   

**参考资料：**

> [Android 中三种启用线程的方法](https://link.jianshu.com/?t=http://www.cnblogs.com/propheterLiu/p/6082666.html)
> [Android中AsyncTask使用详解](https://link.jianshu.com/?t=http://blog.csdn.net/iispring/article/details/50639090)
> [Android线程管理之Thread使用总结](https://link.jianshu.com/?t=http://www.cnblogs.com/whoislcj/p/5603277.html)
> [Android HandlerThread 完全解析](https://link.jianshu.com/?t=http://blog.csdn.net/lmj623565791/article/details/47079737)
> [IntentService 示例与详解](https://www.jianshu.com/p/332b6daf91f0)