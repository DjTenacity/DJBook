## [Android全屏 (或者说主题)](https://www.cnblogs.com/ivan240/archive/2012/11/29/2794024.html)



1 代码里面透明状态栏或者 全屏 ,要在setContentView前面:

```java
// 在代码里直接声明透明状态栏更有效 //getWindow().setFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS, WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);

//全屏
requestWindowFeature(Window.FEATURE_NO_TITLE);
  getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);

```

2 主题里面

```java
    <style name="NoTitle_FullScreen">
    	<item name="android:windowNoTitle">true</item>  
    	<item name="android:windowFullscreen">true</item>    
    </style>
```



3 checkbox 主题

```java
<style name="My_CheckBox" parent="@android:style/Widget.Material.CompoundButton.CheckBox">
    <item name="android:colorControlActivated">@color/blue_3391FF</item>
    <!--波纹的颜色-->
    <item name="android:colorControlHighlight">@color/blue_3391FF</item>
    <!--选中的颜色-->
    <!--<item name="colorAccent">@color/colorBlue</item> -->
    <!--<item name="android:colorControlNormal">@color/blue_3391FF</item>-->
</style>
```
```angular2
    <style name="MyDialogStyle" parent="Theme.AppCompat.Light.NoActionBar" >
        <!--设置dialog的背景，此处为系统给定的透明值-->
        <item name="android:windowBackground">@android:color/transparent</item>
        <!-- Dialog的windowFrame框为无-->
        <item name="android:windowFrame">@null</item>
        <!-- 是否显示标题-->
        <item name="android:windowNoTitle">true</item>　　　
        <!--是否浮现在activity之上　　-->
        <item name="android:windowIsFloating">true</item>
        <!-- 是否半透明-->
        <item name="android:windowIsTranslucent">true</item>
        <!--  是否有覆盖-->
        <item name="android:windowContentOverlay">@null</item>
        <!--设置Activity出现方式-->
        <item name="android:windowAnimationStyle">@android:style/Animation.Dialog</item>
        <!-- 背景是否模糊显示-->
        <item name="android:backgroundDimEnabled">true</item>
    </style>

```
