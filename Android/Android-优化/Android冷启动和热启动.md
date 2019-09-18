# [Android](http://www.codeceo.com/article/android-start-fast.html)冷启动和热启动

## 应用的启动

### 启动方式

通常来说，在安卓中应用的启动方式分为两种：冷启动和热启动。

1、冷启动：当启动应用时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用，这个启动方式就是冷启动。

2、热启动：当启动应用时，后台已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程是依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程中来启动应用，这个方式叫热启动。

### 特点

1、冷启动：冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化Application类，再创建和初始化MainActivity类（包括一系列的测量、布局、绘制），最后显示在界面上。

2、热启动：热启动因为会从已有的进程中来启动，所以热启动就不会走Application这步了，而是直接走MainActivity（包括一系列的测量、布局、绘制），所以热启动的过程只需要创建和初始化一个MainActivity就行了，而不必创建和初始化Application，因为一个应用从新进程的创建到进程的销毁，Application只会初始化一次。

上面说的启动是点击app的启动图标来启动的，而另外一种方式是进入最近使用的列表界面来启动应用，这种不应该叫启动，应该叫恢复。

## 应用启动的流程

在安卓系统上，应用在没有进程的情况下，应用的启动都是这样一个流程：当点击app的启动图标时，安卓系统会从Zygote进程中fork创建出一个新的进程分配给该应用，之后会依次创建和初始化Application类、创建MainActivity类、加载主题样式Theme中的windowBackground等属性设置给MainActivity以及配置Activity层级上的一些属性、再inflate布局、当onCreate/onStart/onResume方法都走完了后最后才进行contentView的measure/layout/draw显示在界面上，所以直到这里，应用的第一次启动才算完成，这时候我们看到的界面也就是所说的第一帧。

所以，总结一下，应用的启动流程如下：

Application的构造器方法——>attachBaseContext()——>onCreate()——>Activity的构造方法——>onCreate()——>配置主题中背景等属性——>onStart()——>onResume()——>测量布局绘制显示在界面上。

## 测量应用启动的时间

在上面这个启动流程中，任何一个地方有耗时操作都会拖慢我们应用的启动速度，而应用启动时间是用毫秒度量的，对于毫秒级别的快慢度量我们还是需要去精确的测量到到底应用启动花了多少时间，而根据这个时间来做衡量。

### 什么才是应用的启动时间

从点击应用的启动图标开始创建出一个新的进程直到我们看到了界面的第一帧，这段时间就是应用的启动时间。

我们要测量的也就是这段时间，测量这段时间可以通过adb shell命令的方式进行测量，这种方法测量的最为精确，命令为：

```
adb shell am start -W [packageName]/[packageName.MainActivity]
```

执行成功后将返回三个测量到的时间：

1、ThisTime:一般和TotalTime时间一样，除非在应用启动时开了一个透明的Activity预先处理一些事再显示出主Activity，这样将比TotalTime小。

2、TotalTime:应用的启动时间，包括创建进程+Application初始化+Activity初始化到界面显示。

3、WaitTime:一般比TotalTime大点，包括系统影响的耗时。

下面是测量一个应用冷启动和热启动的时间：

冷启动：

![Android性能优化之加快应用启动速度](http://static.codeceo.com/images/2016/01/4f6e4815f633874bbb27e8159a48ff5a.png)

热启动：

![Android性能优化之加快应用启动速度](http://static.codeceo.com/images/2016/01/f04ce614d4554672884ae2b64022043d.png)

可以看到在进程已经存在的情况下，只需要重新初始化MainActivity，这样的启动比较快，不过大多数情况下应用的启动都是冷启动，因为用户都会在任务列表中手动关闭遗留的应用进程。

## 减少应用启动时的耗时

针对冷启动时候的一些耗时，如上测得这个应用算是中型的app，在冷启动的时候耗时已经快700ms了，如果项目再大点在Application中配置了更多的初始化操作，这样将可能达到1s，这样每次启动都明显感觉延迟，所以在进行应用初始化的时候采取以下策略：

1、在Application的构造器方法、attachBaseContext()、onCreate()方法中不要进行耗时操作的初始化，一些数据预取放在异步线程中，可以采取Callable实现。

2、对于sp的初始化，因为sp的特性在初始化时候会对数据全部读出来存在内存中，所以这个初始化放在主线程中不合适，反而会延迟应用的启动速度，对于这个还是需要放在异步线程中处理。

3、对于MainActivity，由于在获取到第一帧前，需要对contentView进行测量布局绘制操作，尽量减少布局的层次，考虑StubView的延迟加载策略，当然在onCreate、onStart、onResume方法中避免做耗时操作。

遵循上面三种策略可明显提高app启动速度。

## 优化应用启动时的体验

对于应用的启动时间，只能是尽量的避免一些耗时的、非必要的操作在主线程中，这样相对可以缩减一部分启动的耗时，另外一方面在等待第一帧显示的时间里，可以加入一些配置以增加体验，比如加入Activity的background，这个背景会在显示第一帧前提前显示在界面上。1、先为主界面单独写一个主题style，设置一张待显示的图片，这里我设置了一个颜色，然后在manifest中设置给MainActivity：

```
<style name="AppTheme.Launcher"> <item name="android:windowBackground">@drawable/bule</item> </style>
//...
        <activity  android:name=".MainActivity" android:label="@string/app_name" android:theme="@style/AppTheme.Launcher">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

2、然后在MainActivity中加载布局前把AppTheme重新设置给MainActivity：

```
@Override
    protected void onCreate(Bundle savedInstanceState) {

        setTheme(R.style.AppTheme);
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
}
```

这样在启动时会先显示background，然后待界面绘制完成再显示主界面