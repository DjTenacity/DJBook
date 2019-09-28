# [2019 Android 高级面试题总结 从java语言到AIDL使用与原理](https://www.cnblogs.com/gooder2-android/p/10515079.html)

### 说下你所知道的设计模式与使用场景

##### a.建造者模式：

将一**个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。**

使用场景比如最常见的AlertDialog,拿我们开发过程中举例，比如Camera开发过程中，可能需要设置一个初始化的相机配置，设置摄像头方向，闪光灯开闭，成像质量等等，这种场景下就可以使用建造者模式

**装饰者模式：**动态的给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更为灵活。装饰者模式可以在不改变原有类结构的情况下曾强类的功能，比如Java中的BufferedInputStream 包装FileInputStream，举个开发中的例子，比如在我们现有网络框架上需要增加新的功能，那么再包装一层即可，装饰者模式解决了继承存在的一些问题，比如多层继承代码的臃肿，使代码逻辑更清晰

观察者模式：
代理模式：
门面模式：
单例模式：
生产者消费者模式：

### java语言的特点与OOP思想

这个通过对比来描述，比如面向对象和面向过程的对比，针对这两种思想的对比，还可以举个开发中的例子，

比如播放器的实现，面向过程的实现方式就是将播放视频的这个功能分解成多个过程，

比如，加载视频地址，获取视频信息，初始化解码器，选择合适的解码器进行解码，读取解码后的帧进行视频格式转换和音频重采样，然后读取帧进行播放，这是一个完整的过程，这个过程中不涉及类的概念，

而**面向对象最大的特点就是类，封装继承和多态是核心**，同样的以播放器为例，一面向对象的方式来实现，将会针对每一个功能封装出一个对象，吧如说Muxer，获取视频信息，Decoder,解码，格式转换器，视频播放器，音频播放器等，每一个功能对应一个对象，由这个对象来完成对应的功能，并且遵循单一职责原则，一个对象只做它相关的事情

### 说下java中的线程创建方式，线程池的工作原理。

java中有三种创建线程的方式，或者说四种

1.继承Thread类实现多线程
2.实现Runnable接口
3.实现Callable接口
4.通过线程池

线程池的工作原理：线程池可以减少创建和销毁线程的次数，从而减少系统资源的消耗，当一个任务提交到线程池时

a. 首先判断核心线程池中的线程是否已经满了，如果没满，则创建一个核心线程执行任务，否则进入下一步

b. 判断工作队列是否已满，没有满则加入工作队列，否则执行下一步

c. 判断线程数是否达到了最大值，如果不是，则创建非核心线程执行
任务，否则执行饱和策略，默认抛出异常

### handler 原理

Handler，Message，looper 和 MessageQueue 构成了安卓的消息机制，handler创建后可以通过 sendMessage 将消息加入消息队列，然后 looper不断的将消息从 MessageQueue 中取出来，回调到 Hander 的 handleMessage方法，从而实现线程的通信。

从两种情况来说，第一在UI线程创建Handler,此时我们不需要手动开启looper，因为在应用启动时，在ActivityThread的main方法中就创建了一个当前主线程的looper，并开启了消息队列，消息队列是一个无限循环，

为什么无限循环不会ANR?

**因为可以说，应用的整个生命周期就是运行在这个消息循环中的，安卓是由事件驱动的，Looper.loop不断的接收处理事件，每一个点击触摸或者Activity每一个生命周期都是在Looper.loop的控制之下的，looper.loop一旦结束，应用程序的生命周期也就结束了。**没事件也是会在空转

我们可以想想什么情况下会发生ANR，第**一，事件没有得到处理，第二，事件正在处理，但是没有及时完成，而对事件进行处理的就是looper，所以只能说事件的处理如果阻塞会导致ANR，而不能说looper的无限循环会ANR**

另一种情况就是**在子线程创建Handler,此时由于这个线程中没有默认开启的消息队列，所以我们需要手动调用looper.prepare(),并通过looper.loop开启消息**

主线程Looper从消息队列读取消息，**当读完所有消息时，主线程阻塞**。子线程往消息队列发送消息，并且往管道文件写数据，主线程即被唤醒，从管道文件读取数据，主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠。因此loop的循环并不会对CPU性能有过多的消耗。

#### 对于 Handler ，一些需要注意的地方

- 1 个线程 `Thread` 只能绑定 1个循环器 `Looper`，但可以有多个处理者 `Handler`
- 1 个循环器 `Looper` 可绑定多个处理者 `Handler`
- 1 个处理者 `Handler` 只能绑定 1 个循环器 `Looper`

`Looper` 是通过 `Looper.prepare()` 和 `Looper.prepareMainLooer()` 创建的，

### 内存泄漏的场景和解决办法

#### 1.非静态内部类的静态实例

非静态内部类会持有外部类的引用，如果**非静态内部类的实例是静态的，就会长期的维持着外部类的引用**，组织被系统回收，解决办法是使用静态内部类

#### 2.多线程相关的匿名内部类和非静态内部类

匿名内部类同样会持有外部类的引用，如果在线程中执行耗时操作就有可能发生内存泄漏，导致外部类无法被回收，直到耗时任务结束，解决办法是在页面退出时结束线程中的任务

#### 3.Handler内存泄漏

Handler导致的内存泄漏也可以被归纳为非静态内部类导致的，Handler内部message是被存储在MessageQueue中的，有些message不能马上被处理，存在的时间会很长，导致handler无法被回收，如果handler是非静态的，就会导致它的外部类无法被回收，解决办法是

**1.使用静态handler，外部类引用使用弱引用处理**

**2.在退出页面时移除消息队列中的消息**

#### 4.Context导致内存泄漏

根据场景确定使用Activity的Context还是Application的Context,因为二者生命周期不同，对于不必须使用Activity的Context的场景（Dialog）,一律采用Application的Context,单例模式是最常见的发生此泄漏的场景，比如传入一个Activity的Context被静态类引用，导致无法回收

#### 5.静态View导致泄漏

使用静态View可以避免每次启动Activity都去读取并渲染View，但是静态View会持有Activity的引用，导致无法回收，解决办法是在Activity销毁的时候将静态View设置为null（View一旦被加载到界面中将会持有一个Context对象的引用，在这个例子中，这个context对象是我们的Activity，声明一个静态变量引用这个View，也就引用了activity）

#### 6.WebView导致的内存泄漏

WebView只要使用一次，内存就不会被释放，所以WebView都存在内存泄漏的问题，通常的解决办法是为WebView单开一个进程，使用AIDL进行通信，根据业务需求在合适的时机释放掉

#### 7.资源对象未关闭导致

如Cursor，File等，内部往往都使用了缓冲，会造成内存泄漏，一定要确保关闭它并将引用置为null

#### 8.集合中的对象未清理

集合用于保存对象，如果集合越来越大，不进行合理的清理，尤其是入股集合是静态的

#### 9.Bitmap导致内存泄漏

bitmap是比较占内存的，所以一定要在不使用的时候及时进行清理，避免静态变量持有大的bitmap对象

#### 10.监听器未关闭

很多需要register和unregister的系统服务要在合适的时候进行unregister,手动添加的listener也需要及时移除

### 如何避免OOM?

#### 1.**使用更加轻量的数据结构：**

如使用ArrayMap/SparseArray替代HashMap,HashMap更耗内存，**因为它需要额外的实例对象来记录Mapping操作**，SparseArray更加高效，因为**它避免了Key Value的自动装箱，和装箱后的解箱操作**

#### 2.便面枚举的使用，可以用静态常量或者注解@IntDef替代

#### 3.**Bitmap优化**:

**a.尺寸压缩：**通过InSampleSize设置合适的缩放
**b.颜色质量：**设置合适的format，ARGB_6666/RBG_545/ARGB_4444/ALPHA_6，存在很大差异
**c.inBitmap:**使用inBitmap属性可以告知Bitmap解码器去尝试使用已经存在的内存区域，新解码的Bitmap会尝试去使用之前那张Bitmap在Heap中所占据的pixel data内存区域，而不是去问内存重新申请一块区域来存放Bitmap。

利用这种特性，即使是上千张的图片，也只会仅仅只需要占用屏幕所能够显示的图片数量的内存大小，但复用存在一些限制，具体体现在：在Android 4.4之前只能重用相同大小的Bitmap的内存，而Android 4.4及以后版本则只要后来的Bitmap比之前的小即可。使用inBitmap参数前，每创建一个Bitmap对象都会分配一块内存供其使用，而使用了inBitmap参数后，多个Bitmap可以复用一块内存，这样可以提高性能

**4.StringBuilder替代String:**

 在有些时候，代码中会需要使用到大量的字符串拼接的操作，这种时候有必要考虑使用StringBuilder来替代频繁的“+”

**5.避免在类似onDraw这样的方法中创建对象**，因为它会迅速占用大量内存，引起频繁的GC甚至内存抖动

**6.减少内存泄漏也是一种避免OOM的方法**

### 说下 Activity 的启动模式，生命周期，两个 Activity 跳转的生命周期，如果一个 Activity 跳转另一个 Activity 再按下 Home 键在回到 Activity 的生命周期是什么样的

启动模式

Standard 模式:Activity 可以有多个实例，每次启动 Activity，无论任务栈中是否已经有这个Activity的实例，系统都会创建一个新的Activity实例

SingleTop模式:当一个singleTop模式的Activity已经位于任务栈的栈顶，再去启动它时，不会再创建新的实例,如果不位于栈顶，就会创建新的实例

SingleTask模式:如果Activity已经位于栈顶，系统不会创建新的Activity实例，和singleTop模式一样。但Activity已经存在但不位于栈顶时，系统就会把该Activity移到栈顶，并把它上面的activity出栈

SingleInstance模式:singleInstance 模式也是单例的，但和singleTask不同，singleTask 只是任务栈内单例，系统里是可以有多个singleTask Activity实例的，而 singleInstance Activity 在整个系统里只有一个实例，启动一singleInstanceActivity 时，系统会创建一个新的任务栈，并且这个任务栈只有他一个Activity

### 生命周期

onCreate onStart onResume onPause onStop onDestroy
两个 Activity 跳转的生命周期

1.启动A
onCreate - onStart - onResume

2.在A中启动B
ActivityA onPause
ActivityB onCreate
ActivityB onStart
ActivityB onResume
ActivityA onStop

3.从B中返回A（按物理硬件返回键）
ActivityB onPause
ActivityA onRestart
ActivityA onStart
ActivityA onResume
ActivityB onStop
ActivityB onDestroy

4.继续返回

ActivityA onPause
ActivityA onStop
ActivityA onDestroy

### onRestart 的调用场景

（1）按下home键之后，然后切换回来，会调用onRestart()。
（2）从本Activity跳转到另一个Activity之后，按back键返回原来Activity，会调用onRestart()；
（3）从本Activity切换到其他的应用，然后再从其他应用切换回来，会调用onRestart()；

### 说下 Activity 的横竖屏的切换的生命周期，用那个方法来保存数据，两者的区别。触发在什么时候在那个方法里可以获取数据等。



### 是否了解SurfaceView，它是什么？他的继承方式是什么？他与View的区别(从源码角度，如加载，绘制等)。

SurfaceView中采用了双缓冲机制，保证了UI界面的流畅性，同时 **SurfaceView 不在主线程中绘制，而是另开辟一个线程去绘制，所以它不妨碍UI线程；**

SurfaceView 继承于View，他和View主要有以下三点区别：

（1）View底层没有双缓冲机制，SurfaceView有；

（2）view主要适用于主动更新，而SurfaceView适用与被动的更新，如频繁的刷新

（3）view会在主线程中去更新UI，而SurfaceView则在子线程中刷新；

SurfaceView的内容不在应用窗口上，所以不能使用变换（平移、缩放、旋转等）。也难以放在ListView或者ScrollView中，不能使用UI控件的一些特性比如View.setAlpha()

View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；必须在UI主线程内更新画面，速度较慢。

SurfaceView：基于view视图进行拓展的视图类，更适合2D游戏的开发；是view的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比view快，Camera预览界面使用SurfaceView。

GLSurfaceView：基于SurfaceView视图再次进行拓展的视图类，专用于3D游戏开发的视图；是SurfaceView的子类，openGL专用。



### 如何实现进程保活

a: Service 设置成 START_STICKY kill 后会被重启(等待5秒左右)，重传Intent，保持与重启前一样

b: 通过 startForeground将进程设置为前台进程， 做前台服务，优先级和前台应用一个级别，除非在系统内存非常缺，否则此进程不会被 kill

c: 双进程Service： 让2个进程互相保护对方，其中一个Service被清理后，另外没被清理的进程可以立即重启进程

d: 用C编写守护进程(即子进程) : Android系统中当前进程(Process)fork出来的子进程，被系统认为是两个不同的进程。当父进程被杀死的时候，子进程仍然可以存活，并不受影响(Android5.0以上的版本不可行）联系厂商，加入白名单

e.锁屏状态下，开启一个一像素Activity



### 冷启动与热启动是什么，区别，如何优化，使用场景等。

**app冷启动**： 当应用启动时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用， 这个启动方式就叫做冷启动（后台不存在该应用进程）。冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化Application类，再创建和初始化MainActivity类（包括一系列的测量、布局、绘制），最后显示在界面上。

**app热启动**： 当应用已经被打开，  但是被按下返回键、Home键等按键时回到桌面或者是其他程序的时候，再重新打开该app时， 这个方式叫做热启动（后台已经存在该应用进程）。热启动因为会从已有的进程中来启动，所以热启动就不会走Application这步了，而是直接走MainActivity（包括一系列的测量、布局、绘制），所以热启动的过程只需要创建和初始化一个MainActivity就行了，而不必创建和初始化Application

### 冷启动的流程

当点击app的启动图标时，安卓系统会从Zygote进程中fork创建出一个新的进程分配给该应用，之后会依次创建和初始化Application类、创建MainActivity类、加载主题样式Theme中的windowBackground等属性设置给MainActivity以及配置Activity层级上的一些属性、再inflate布局、当onCreate/onStart/onResume方法都走完了后最后才进行contentView的measure/layout/draw显示在界面上

冷启动的生命周期简要流程：

Application构造方法 –> attachBaseContext()–>onCreate –>Activity构造方法 –> onCreate() –> 配置主体中的背景等操作 –>onStart() –> onResume() –> 测量、布局、绘制显示

冷启动的优化主要是视觉上的优化，解决白屏问题，提高用户体验，所以通过上面冷启动的过程。能做的优化如下：

- 减少 onCreate()方法的工作量
- 不要让 Application 参与业务的操作
- 不要在 Application 进行耗时操作
- 不要以静态变量的方式在 Application 保存数据
- 减少布局的复杂度和层级
- 减少主线程耗时

为什么冷启动会有白屏黑屏问题？原因在于加载主题样式Theme中的windowBackground等属性设置给MainActivity发生在inflate布局当onCreate/onStart/onResume方法之前，而windowBackground背景被设置成了白色或者黑色，所以我们进入app的第一个界面的时候会造成先白屏或黑屏一下再进入界面。解决思路如下

1.给他设置 windowBackground 背景跟启动页的背景相同，如果你的启动页是张图片那么可以直接给 windowBackground 这个属性设置该图片那么就不会有一闪的效果了

```
<style name="Splash_Theme"` `parent="@android:style/Theme.NoTitleBar">`
    <item name="android:windowBackground">@drawable/splash_bg</item>`
    <item name="android:windowNoTitle">true</item>`
</style>`
```

2.采用世面的处理方法，设置背景是透明的，给人一种延迟启动的感觉。,将背景颜色设置为透明色,这样当用户点击桌面APP图片的时候，并不会"立即"进入APP，而且在桌面上停留一会，其实这时候APP已经是启动的了，只是我们心机的把Theme里的windowBackground 的颜色设置成透明的，强行把锅甩给了手机应用厂商（手机反应太慢了啦）

```xml
<style name="Splash_Theme" parent="@android:style/Theme.NoTitleBar">
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:windowNoTitle">true</item>
</style>
```

3.以上两种方法是在视觉上显得更快，但其实只是一种表象，让应用启动的更快，有一种思路，将 Application 中的不必要的初始化动作实现懒加载，比如，在SpashActivity 显示后再发送消息到 Application，去初始化，这样可以将初始化的动作放在后边，缩短应用启动到用户看到界面的时间

###  

### Android 中的线程有那些,原理与各自特点

AsyncTask,HandlerThread,IntentService

**AsyncTask原理**：内部是Handler和两个线程池实现的，Handler用于将线程切换到主线程，两个线程池一个用于任务的排队，一个用于执行任务，当AsyncTask执行execute方法时会封装出一个FutureTask对象，将这个对象加入队列中，如果此时没有正在执行的任务，就执行它，执行完成之后继续执行队列中下一个任务，执行完成通过Handler将事件发送到主线程。AsyncTask必须在主线程初始化，因为内部的Handler是一个静态对象，在AsyncTask类加载的时候他就已经被初始化了。在Android3.0开始，execute方法串行执行任务的，一个一个来，3.0之前是并行执行的。如果要在3.0上执行并行任务，可以调用executeOnExecutor方法

**HandlerThread原理**：继承自 Thread，start开启线程后，会在其run方法中会通过Looper 创建消息队列并开启消息循环，这个消息队列运行在子线程中，所以可以将HandlerThread 中的 Looper 实例传递给一个 Handler，从而保证这个 Handler 的 handleMessage 方法运行在子线程中，Android 中使用 HandlerThread的一个场景就是 IntentService
**IntentService原理**：继承自Service，它的内部封装了 HandlerThread 和Handler，可以执行耗时任务，同时因为它是一个服务，优先级比普通线程高很多，所以更适合执行一些高优先级的后台任务

**HandlerThread底层通过Looper消息队列实现的，所以它是顺序的执行每一个任务**。可以通过Intent的方式开启IntentService，IntentService通过handler将每一个intent加入HandlerThread子线程中的消息队列，通过looper按顺序一个个的取出并执行，执行完成后自动结束自己，不需要开发者手动关闭

### ANR的原因

1.耗时的网络访问
2.大量的数据读写
3.数据库操作
4.硬件操作（比如camera)
5.调用thread的join()方法、sleep()方法、wait()方法或者等待线程锁的时候
6.service binder的数量达到上限
7.system server中发生WatchDog ANR
8.service忙导致超时无响应
9.其他线程持有锁，导致主线程等待超时
10.其它线程终止或崩溃导致主线程一直等待

### 三级缓存原理

当 Android 端需要获得数据时比如获取网络中的图片，首先从内存中查找（按键查找），内存中没有的再从磁盘文件或sqlite中去查找，若磁盘中也没有才通过网络获取

LruCache 底层实现原理：

**LruCache 中 Lru 算法的实现就是通过 LinkedHashMap 来实现的。LinkedHashMap 继承于 HashMap，它使用了一个双向链表来存储 Map中的Entry顺序关系**

对于get、put、remove等操作，LinkedHashMap除了要做HashMap做的事情，还做些调整Entry顺序链表的工作。

LruCache中将LinkedHashMap的顺序设置为LRU顺序来实现LRU缓存，每次调用get(也就是从内存缓存中取图片)，则将该对象移到链表的尾端。
调用put插入新的对象**也是存储在链表尾端**，这样当内存缓存达到设定的最大值时，将**链表头部的对象**（近期最少用到的）移除。

### 说下你对 Collection 这个类的理解。

Collection是集合框架的顶层接口，是存储对象的容器,Colloction定义了接口的公用方法如add remove clear等等，它的子接口有两个，List和Set,List的特点有元素有序，元素可以重复，元素都有索引（角标），典型的有

Vector:内部是数组数据结构，是同步的（线程安全的）。增删查询都很慢。

ArrayList:内部是数组数据结构，是不同步的（线程不安全的）。替代了Vector。查询速度快，增删比较慢。

LinkedList:内部是链表数据结构，是不同步的（线程不安全的）。增删元素速度快。

而Set的是特点元素无序，元素不可以重复

HashSet：内部数据结构是哈希表，是不同步的。

Set集合中元素都必须是唯一的，HashSet作为其子类也需保证元素的唯一性。

判断元素唯一性的方式：

通过存储对象（元素）的hashCode和equals方法来完成对象唯一性的。

**如果对象的hashCode值不同，那么不用调用equals方法就会将对象直接存储到集合中；**

**如果对象的hashCode值相同，那么需调用equals方法判断返回值是否为true**，
若为false, 则视为不同元素，就会直接存储；
若为true， 则视为相同元素，不会存储。

如果要使用HashSet集合存储元素，该元素的类必须覆盖hashCode方法和equals方法。一般情况下，如果定义的类会产生很多对象，通常都需要覆盖equals，hashCode方法。建立对象判断是否相同的依据。
**TreeSet：保证元素唯一性的同时可以对内部元素进行排序，**是不同步的。

判断元素唯一性的方式：

根据比较方法的返回结果是否为0，如果为0视为相同元素，不存；如果非0视为不同元素，则存。

TreeSet对元素的排序有两种方式：

方式一：使元素（对象）对应的类实现Comparable接口，覆盖compareTo方法。这样元素自身具有比较功能。

方式二：使TreeSet集合自身具有比较功能，定义一个比较器Comparator，将该类对象作为参数传递给TreeSet集合的构造函数

### 说下AIDL的使用与原理

- 可以围绕aidl是安卓中的一种进程间通信方式

说下你对广播的理解
说下你对服务的理解，如何杀死一个服务。服务的生命周期(start与bind)。

### 其他

- 是否接触过蓝牙等开发
- 设计一个ListView左右分页排版的功能自定义View，说出主要的方法。
- 说下binder序列化与反序列化的过程，与使用过程
  是否接触过JNI/NDK，java如何调用C语言的方法
- 如何查看模拟器中的SP与SQList文件。如何可视化查看布局嵌套层数与加载时间。
- 你说用的代码管理工具什么，为什么会产生代码冲突，该如何解决
- 说下你对后台的编程有那些认识，聊些前端那些方面的知识。
- 说下你对线程池的理解，如何创建一个线程池与使用。
- 说下你用过那些注解框架，他们的原理是什么。自己实现过，或是理解他的工作过程吗？
- 说下java虚拟机的理解，回收机制，JVM是如何回收对象的，有哪些方法等





### [猎豹移动:(有笔试)](https://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247487266&idx=1&sn=708b38a9bb62b4311925146d016861da&chksm=eb4763bcdc30eaaac93ba154c9fb7d0e243cd77e19951eb0c752ebb093a6eafb62c5f7ca8497&scene=21#wechat_redirect)

- atomicinteger内存模型
- static编译时有啥不同,static 语句块,static变量,static方法,构造初始化顺序(静态绑定)
- animation和animator的用法,概述实现原理
- Handler,looper,messagequeue,thread,message,每个类功能,关系?
- **Mvc,mvp的差异**
- app闪退的原因有哪些?每种情况简述分析过程
- 如果一个app存在多进程,请列出全部的ipc方法
- 操作系统中进程和线程有什么联系和区别,系统什么时候会在用户态和内核态中切换?
- 如何加载ndk库?如何在jni中注册native函数,有几种注册方式?
- 一个app如果性能不好,怎么分析?

### 饿了么(无笔试)

- 设计的六大原则
- 如果hashmap key不一样,但是hashcode一样会怎么样?
- okhttp有什么优秀的设计模式?builder模式有什么好处?责任链模式有什么好处?
- 懒汉模式单例为什么加volaitle?
- hashmap是否线程安全?不安全会出什么问题?
- concurrenthashmap读写分别是啥情况?
- bindservice和startservice生命周期有啥不同?
- 广播有几种?广播是观察者模式?跨进城广播也是观察者模式吗?
- ams是怎么找到启动的那个activity的?
- a-b-c界面,其中b是singleinstance的,那么c界面点back返回a界面,为什么?怎么管理栈的?
- 红黑树有啥特性?
- 在oncreate里面可以得到view的宽高吗?
- view的getwidth和getmesurewidth有啥区别?
- 遍历hashmap的原理?
- 23种设计模式

### 中园博林(有笔试)

- 如何避免out of menmory和anr?
- arraymap和hashmap的区别?
- 如何实现线程同步?
- 简述android事件分发机制
- 简述view绘制流程
- 用两个栈实现一个队列

#### **口头问**

- viewpager嵌套滑动冲突怎么解决?
- svg动画
- 属性动画画一个抛物线怎么弄?

### 立思辰(无笔试)

- 为了适配多分辨率,引入什么开源框架?
- 阅读界面书架用什么控件实现?
- 布局怎么做到每行的文字左右对齐?
- 直播界面,微信对话界面实现?
- 性能优化怎么弄?

### vv音乐(有笔试)

- 笔试题很多
- sax解析xml的优点
- Contentvalue 键值类型
- androiddvm的进程与linux的进程说法正确的是?(选择题)
- Android:gravity和android:layout_gravity的区别?
- assets与res/raw的区别?
- 解释layout_weight的作用
- view如何刷新?
- animation.animationlistner干什么用的?
- android常用布局及排版效率
- collection与collections的区别
- 匿名内部类是否可以extends其他类?是否可以implement interface(接口)
- 补间动画常见的效果?有哪几个常见的插入器?
- override与overload的区别?overloaded的方法是否可以改变返回值的类型?
- sleep与wait有什么区别?
- 在android中,请简述jni的调用过程?
- 请结束android.mk的作用,并试写一个android.mk文件(包含一个.c源文件即可)
- 冒泡排序(代码实现)
- 猴子偷桃问题代码实现
- 给出两个链表的头指针比如p1,p2,判断这两个链表是否相交,写出主要思路即可

**口头问**

- 简述封装,继承,多态
- 强软弱虚引用的应用场合
- 输出一个数组,不重复?(有点忘记题目什么意思了)
- 用四个线程计算数组和(我说用join方法,或者countdownlatch,他说用线程池即可)

**什么叫安全发布对象(多线程里面)final?**

- 策略模式和命令模式是啥?
- 拓扑排序
- 数组和链表在中间位置的插入效率
- binder的原理
- art和dvm在gc上有啥不同?有啥改进?
- linux和windows下进程怎么通信的?(完全不了解)
- 性能优化做过什么工作?
- 一个类实现一个接口,接口引用指向这个类对象,可以不可以调用它的tostring方法?
- 浏览器,输入url匹配,假设有一亿条url缓存,用什么数据结构匹配?
- recycleview缓存机制相比listview缓存机制有啥改进?
- 一个长度为10的arraylist和linklist,在第五条插入,哪个更快?
- 子类复写父类的equals方法,但是子类增加了一个成员变量int,请问equals方法咋整?

### 大数医疗(有笔试)

- 手写hashmap
- 写生产者消费者模式,不可用syncronized
- treemap,hashmap应用场景

### 字节跳动(无笔试)

- dvm和art的区别
- 从framework的角度讲activity的启动流程(冷启动)
- 手写算法,二维数组,每一行,每一列都是升序,找出某数的下标,没有输出[-1,-1],最好的时间复杂度是m+n(行数+列数)
- zxing二维码开源框架流程
- contentprovider怎么升级维护?
- constaintlayout
- bitmap有几种格式,分别占多少字节

### 滴滴出行(无笔试)

- android事件分发机制,如何下发,如何上传?
- 一个界面下拉刷新要怎么实现?
- bitmap占用内存多少怎么计算?一个像素占几个字节?
- threadlocal的原理?
- framework加载activity的流程
- arraylist和linkedlist的应用场景
- 网络请求相关的框架
- 好几万条短信,滑动卡顿怎么解决?
- 有没有了解过三方开源数据库(好像是腾讯的什么数据库框架,不仅仅是懂sqlite)
- 避免内存泄漏,为什么说handler用成员内部类会内存泄漏?activity不是已经到gcroot被切断了吗?还有静态context持有activity的引用会内存泄漏,必须要持有怎么办?(及时释放)
- 计算viewgroup的层级,递归实现和非递归实现
- 自己写一个应用,包名就叫android行不行,为什么?
- 主线程looper如果没有消息,就会阻塞在那,为什么不回anr?
- 系统进程可以用webview吗?
- 原子类的了解
- 一个app多进程的好处
- 一个arraylist,里面全部是int,讲所有值是2的整数的节点删除
- arraymap了解
- binder机制
- shareprefrence原理?是否线程安全和进程安全?
- 一个app启动页另开一个进程,启动页10s后启动mainactivity,请问5s的时候有几个进程?
- java内存结构,内存模型

### 融云(有笔试)

- 冒泡排序手写
- 如何判断一个字符串是回文字符串

### 梧桐车联(电话面试没过)

- 为什么要引入activity这个组件
- shareprefrence不是进程安全,假设一个apk两个进程同时修改shareprefrence怎么办?
- contenprovider已经是进程间通信,为什么还要引入broadcastreceiver?
- a启动b,b启动c,怎样可以在c界面点back退回到a?
- startservice和bindservice生命周期有什么不同?
- 两个应用同时注册一个广播,优先级都一样,哪个会先收到广播?(有序广播?)
- 还有些其他的,忘记了

### 蚂蚁金服(电话面试没过)

- threadlocal原理
- zxing有过优化提高识别率吗?

### 京东

- arraylist里面可以不可以new一个t泛型的数组?
- 补间动画click事件还在原位怎么解决?
- 多线程并发
- 隔代数据库升级
- 性能优化