```java
有float类型的

向上取整:Math.ceil()   //只要有小数都+1
向下取整:Math.floor()   //不取小数
四舍五入:Math.round()  //四舍五入

```

[android6.0中floatmath用什么替代](https://zhidao.baidu.com/question/938772510489341692.html) 
解决方案是使用数学类:(float) Math,例如(float)Math.sqrt(),我也是刚遇到这...Android6.0使用 Math.floor 代替 FloatMath.floor 即可



float a =1.0f - 0.9f ;

float b = 0.9f - 0.8f;



a==b ?

![](D:\DJGitBook\img\float.png)

java里面。浮点计算都会用BigDecimal。 或者一般折扣系数，金额。后台设计里面都是用整数。

转换的时候容易精度丢失

价格用分做单位。

![](D:\DJGitBook\img\float2.jpg)

float 和 double  需要转成了二进制计算的  

包装类也不相等   精度在小数点后18位

精度丢失mongodb里面也会出现。95折在mongodb里面成了94折



**Java中对象的生命周期**

1）创建阶段（Created）：为对象分配存储空间，开始构造对象，从超类到子类对static成员初始化；超类成员变量按顺序初始化，递归调用超类的构造方法，子类成员变量按顺序初始化，子类构造方法调用。

分配存储空间--->超类到子类对static成员初始化--->超类成员变量-->递归调用超类的构造方法--->子类成员变量--->

2）应用阶段（In Use）：对象至少被一个强引用持有着。

3）不可见阶段（Invisible）：程序运行已超出对象作用域

4）不可达阶段（Unreachable）：该对象不再被强引用所持有

5）收集阶段（Collected）：假设该对象重写了finalize()方法且未执行过，会去执行该方法。

6）终结阶段（Finalized）：对象运行完finalize()方法仍处于不可达状态，等待垃圾回收器对该对象空间进行回收。

7）对象空间重新分配阶段（De-allocated）：垃圾回收器对该对象所占用的内存空间进行回收或再分配，该对象彻底消失。







**原子性：提供互斥访问，同一时刻只能有一个线程对数据进行操作**。

1）volatile：解决变量在多个线程间的可见性，但不能保证原子性，只能用于修饰变量，不会发生阻塞。volatile能屏蔽编译指令重排，不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面。多用于并行计算的单例模式。volatile规定CPU每次都必须从内存读取数据，不能从CPU缓存中读取，保证了多线程在多CPU计算中永远拿到的都是最新的值。

 

2）synchronized：互斥锁，操作互斥，并发线程过来，串行获得锁，串行执行代码。解决的是多个线程间访问共享资源的同步性，可保证原子性，也可间接保证可见性，因为它会将私有内存和公有内存中的数据做同步。可用来修饰方法、代码块。会出现阻塞。synchronized发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生。非公平锁，每次都是相互争抢资源。

 

3）lock是一个接口，而synchronized是java中的关键字，synchronized是内置语言的实现。lock可以让等待锁的线程响应中断。在发生异常时，如果没有主动通过unLock()去释放锁，则可能造成死锁现象，因此使用Lock时需要在finally块中释放锁。



**Android内存泄露研究**

Android内存泄漏指的是进程中某些对象（垃圾对象）已经没有使用价值了，但是它们却可以直接或间接地引用到gc roots导致无法被GC回收。无用的对象占据着内存空间，使得实际可使用内存变小，形象地说法就是内存泄漏了。

场景

+ 类的静态变量持有大数据对象

+ 静态变量长期维持到大数据对象的引用，阻止垃圾回收。

+ 非静态内部类的静态实例
  + 非静态内部类会维持一个到外部类实例的引用，如果非静态内部类的实例是静态的，就会间接长期维持着外部类的引用，阻止被回收掉。

+ 资源对象未关闭
  + 资源性对象如Cursor、File、Socket，应该在使用后及时关闭。未在finally中关闭，会导致异常情况下资源对象未被释放的隐患。

+ 注册对象未反注册
  + 未反注册会导致观察者列表里维持着对象的引用，阻止垃圾回收。

+ Handler临时性内存泄露

 

Handler通过发送Message与主线程交互，Message发出之后是存储在MessageQueue中的，有些Message也不是马上就被处理的。在Message中存在一个 target，是Handler的一个引用，如果Message在Queue中存在的时间越长，就会导致Handler无法被回收。如果Handler是非静态的，则会导致Activity或者Service不会被回收。 

由于AsyncTask内部也是Handler机制，同样存在内存泄漏的风险。 

此种内存泄露，一般是临时性的。

**在** **Java** **中，为什么不允许从静态方法中访问非静态变量？** 

Java 中不能从静态上下文访问非静态数据只是因为非静态变量是跟具体的对象实例关联 

的，而静态的却没有和任何实例关联。 



**ANR是什么？怎样避免和解决ANR?**

 

ANR:Application Not Responding，即应用无响应

ANR一般有三种类型：

1：KeyDispatchTimeout(5 seconds) –主要类型

按键或触摸事件在特定时间内无响应

2：BroadcastTimeout(10 seconds)

BroadcastReceiver在特定时间内无法处理完成

3：ServiceTimeot(20 seconds) –小概率类型

Service在特定的时间内无法处理完成

 

超时的原因一般有两种：

 (1)当前的事件没有机会得到处理（UI线程正在处理前一个事件没有及时完成或者looper被某种原因阻塞住）

 (2)当前的事件正在处理，但没有及时完成

 UI线程尽量只做跟UI相关的工作，耗时的工作（数据库操作，I/O，连接网络或者其他可能阻碍UI线程的操作）放入单独的线程处理，尽量用Handler来处理UI thread和thread之间的交互。

 

UI线程主要包括如下：

Activity:onCreate(), onResume(), onDestroy(), onKeyDown(), onClick()

AsyncTask: onPreExecute(), onProgressUpdate(), onPostExecute(), onCancel()

Mainthread handler: handleMessage(), post(runnable r)

other

![]()





Android Profiler 提供有 CPU、Memory 和 Network 三大调试分析利器，实时跟踪 Apk 的运行状态，可以帮助我们可视化地做一些性能调优工作。
这三个工具在开发阶段非常实用，比如 CPU Profiler 能够分析应用中的线程使用情况，Memory Profiler 能够检测出内存泄漏，Network Profiler 能够拦截网络请求实现抓包功能等。