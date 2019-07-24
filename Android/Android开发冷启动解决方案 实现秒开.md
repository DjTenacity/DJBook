## Android开发冷启动解决方案 实现秒开(一般)

## 原理解析

###### 冷启动

###### 什么是冷启动

Android中的冷启动，使用直白的话就是：

- 当手机 重启 ，点击桌面图标启动应用的过程就是冷启动
- 未启动手机，长时 未使用，应用被 kill 后，此时点击桌面图标启动应用的过程

##### 冷启动的表现形式

未做处理的情况

- 点击桌面图标后没有反应，没有瞬间打开应用，也就是没有马上看到应用打开
- 点击桌面图标后会显示 黑屏 或者 白屏 ， 没有及时渲染出页面元素


  冷启动产生的原因

冷启动产生的主要原因要从APP的启动流程说起：

​    1.用户点击 icon
​    2.系统开始加载和启动应用
​    3.应用启动：开启空白(黑色)窗口
​    4.创建应用进程
​    5.初始化Application
​    6.启动 UI 线程
​    7.创建第一个 Activity
​    8.解析(Inflater)和加载内容视图
​    9.布局(Layout)
​    10.绘制(Draw)

下图是启动的日志信息：



![img](https://upload-images.jianshu.io/upload_images/1760510-30c5cf087b886c67.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

APP启动日志信息

从上面可以看出，从应用启动到布局和绘制，是需要时间的，这也是无法避免的，越是低端的手机上，这一过程耗费的时间。

# 解决方案

首先要明确的一点就是：冷启动无法避免，我们只能去减少冷启动的时间和适配冷启动。

## 如何减少冷启动的时间？

其实这个问题等同于如何减少应用初始化的时间，从上面的APP启动流程中，如果我们在应用初始化的操作越多，那么从初始化到绘制的时间越长，用户看到真实界面的时间也就越长，可以从如下几个方面进行：

1. 减少在 Application 中的耗时操作(懒加载)
2. 减少在 onCreate 的耗时操作

## 如何适配冷启动？

Android 为我们提供了 android:windowBackground 的解决方案，我们可以专门为 SplashActivity 设置一个背景来避免 创建空白(黑色) 窗口这一步骤的尴尬，而对于 android:windowBackground 又延伸了各种各样的方案。

 

### 1. 纯色背景 + 启动图标

这种做法在国产APP上面少见，在国外的APP常见，简单的来说就是用 layer-list 绘制一个纯色的背景加上一个启动图标，layer-list 代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:drawable="@color/colorPrimary" />

    <item>
        <bitmap
            android:gravity="center"
            android:src="@mipmap/ic_launcher" />
    </item>

</layer-list>
```

然后我们为SplashActivity创建一个主题：

```xml
<resources>
    <!-- 基本主题 -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <!--纯色加启动图标的方案-->
    <style name="SplashThemeLayer" parent="AppTheme">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowBackground">@drawable/bg_splash_layer_list</item>
    </style>
</resources>
```

最后为 SplashActivity设置主题为 SplashThemeLayer 在启动看看效果吧。



![img](https:////upload-images.jianshu.io/upload_images/1760510-6adc32c300df24e2.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/470/format/webp)

冷启动解决方案-纯色背景加启动图标

是不是实现了想要的效果？点击应用图标立即显示了我们的图标。

关于layer-list我们还可以拓展一下：例如加一个45°的线性渐变.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item>
        <shape android:shape="rectangle">
            <gradient
                android:angle="45"
                android:endColor="@color/colorPrimary"
                android:startColor="@color/colorAccent" />
        </shape>

    </item>

    <item>
        <bitmap
            android:gravity="center"
            android:src="@mipmap/ic_launcher" />
    </item>

</layer-list>
```

看看效果：



![img](https:////upload-images.jianshu.io/upload_images/1760510-05f00dd946857510.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/470/format/webp)

纯色拓展

### 2. 使用背景图片

前面的第一种方式是使用纯色背景 + 启动图标，这种方式肯定是不满足我们的产品经理的，他们要的是 个性化 的页面。
 使用背景图片也是很简单的，只需要在them将我们之前的drawable替换成我们的图片即可：

```xml
<!--使用图片的方案-->
    <style name="SplashThemeImage" parent="AppTheme">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowBackground">@mipmap/icon_splish</item>
        <!--沉浸-->
        <item name="android:windowTranslucentStatus">true</item>
    </style>
```

需要注意的是：Splash页面的背景颜色需要设置为透明 #00000000，不要设置其他背景，否则会导致图片的伸缩变形。

 从效果图可以看到，已经得到了我们平常想要的效果了，但是用这种方式又带了了另外一个问题：
 图片的内存占用和OOM，像这种启动页面的，基本上都是直接打包在APP中的，而色彩越是丰富，图片的体积就越大，大多数情况下我们是无法反驳的，我们可以通过压缩图片的方式来尽量减少图片的体积，这里推荐一个png压缩网站：[tinypng](https://links.jianshu.com/go?to=https%3A%2F%2Ftinypng.com%2F)，基本上能把我们拿到的设计图减少一半以上的体积。 

### 3. 说服产品，使用更酷炫的方式来实现 

你可以这样：



![img](https://upload-images.jianshu.io/upload_images/1760510-9db00782ddd69cd9.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/337/format/webp)

center

还可以这样：



![img](https://upload-images.jianshu.io/upload_images/1760510-6a5bf837048cf5d0.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/337/format/webp)

 

[onboarding-examples-android](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsaulmm%2Fonboarding-examples-android)

# 题外：关于热启动

## 什么是热启动

- 用户按下 Home 键返回桌面后又马上点击桌面图标启动应用（Application 仍然存活）
- 应用未完全被杀死，从 启动列表 中进入到应用（Application 仍然存活）

**大家都在看**



[通俗理解数字签名，数字证书和https](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943600&idx=1&sn=bcbf3eb80ae2be5a3a40ca53b622ea23&chksm=84fd6054b38ae9422e8a15e5f21918a533466ea58e99a7904532b721a11f2628c4f56404c8b9&scene=21#wechat_redirect)

[让你的EditText删除表情比微信更高效](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943581&idx=1&sn=37442901339654ca621836998d522075&chksm=84fd6079b38ae96f3ff450137f3ba1241702b9bcda54890ac9dc4418546c8ebfb8a393754d17&scene=21#wechat_redirect)

[Android开发小技巧之商品属性筛选与商品筛选](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943576&idx=1&sn=af3ab0c9940adc807f3589dff72bc7f9&chksm=84fd607cb38ae96a8d38ab18df61f8d4ee6bc74b9321d1f0e2e7c5671edcb91e1c2fd5dfcded&scene=21#wechat_redirect)

[ReactNative从零到完整项目-嵌入到安卓原生应用](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943592&idx=1&sn=c45b3139be7460020cd4ab67a366c61c&chksm=84fd604cb38ae95ad06e69960c98803ed8bc967fad7b39c8d81200a38d55e8422e3fe7f6dbf5&scene=21#wechat_redirect)

