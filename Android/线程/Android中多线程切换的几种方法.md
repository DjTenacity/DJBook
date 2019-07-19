# [Android中多线程切换的几种方法](https://www.jianshu.com/p/31d0852c0760)

#### Thread

每个Thread对象都**可以**启动一个新的线程

```java
thread.run();//不启动新线程，在当前线程执行
thread.start();//启动新线程。
```

优先级高的Thread不是必然会先于其他Thread执行，只是系统会倾向于给它分配更多的CPU。
 默认情况下，新建的Thread和当前Thread的线程优先级一致。
 设置线程优先级有两种方式：

```java
thread.setPriority(Thread.MAX_PRIORITY);//1~10，通过线程设置
Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);//-20~19，通过进程设置
```

 这两种设置方式是相对独立的，在Android中，一般建议通过Process进程设置优先级。

#### ThreadPool

hread本身是需要占用内存的，开启/销毁过量的工作线程会造成过量的资源损耗，这种场景我们一般会通过对资源的复用来进行优化，

+ 针对IO资源我们会做IO复用（例如Http的KeepAlive），
+ 针对内存我们会做内存池复用（例如Fresco的内存池），
+ 针对CPU资源，我们一般会做线程复用，也就是线程池。

 所以，在Android开发中，一般不会直接开启大量的Thread，而是会使用ThreadPool来复用线程。

#### Runnable

Runnable主要解决如何定义每个线程的工作任务的问题。

Thread本身也是一种Runnable：

```java
public class Thread implements Runnable {
```

相比Thread而言，Runnable不关注如何调度线程，只关心如何定义要执行的工作任务，所以在实际开发中，多使用Runnable接口完成多线程开发。

#### Callable

Callable和Runnable基本类似，但是Callable可以返回执行结果。

# 线程间通信

Thread和Runnable能实现切换到另一个线程工作（Runnable需要额外指派工作线程），但它们完成任务后就会退出，并不注重如何在线程间实现通信，所以切换线程时，还需要在线程间通信，这就需要一些线程间通信机制

### Future

Future接口定义了done、canceled等回调函数，当工作线程的任务完成时，它会（在工作线程中）通过回调告知我们，我们再采用其他手段通知其他线程。

```java
        mFuture = new FutureTask<MyBizClass>(runnable) {
            @Override
            protected void done() {
                ...//还是在工作线程里
            }
        };
```

 

#### Condition

Condition其实是和Lock一起使用的，但如果把它视为一种线程间通信的工具，也说的通。
因为，Condition本身定位就是一种多线程间协调通信的工具，Condition可以在某些条件下，唤醒等待线程。

```java
Lock lock = new ReentrantLock();
 Condition notFull  = lock.newCondition(); //定义Lock的Condition
...
   while (count == items.length)
                notFull.await();//等待condition的状态
...
   notFull.signal();//达到condition的状态
```

#### Handler

​     Handler利用线程封闭的ThreadLocal维持一个消息队列，Handler的核心是通过这个消息队列来传递Message，从而实现线程间通信。

