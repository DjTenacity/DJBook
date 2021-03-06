# Android[强制横屏或竖屏注意事项](https://blog.csdn.net/chj00128/article/details/50681092)

一个Activity如果在onReusume里没有特别声明，或没在AndroidManifest.xml配置成横屏或竖屏，在旋转时其声明周期为：

onCreate——onStart—onResume—屏幕旋转—-onPause（失去焦点）—-onStop（彻底看不见）—onDestory，

然后重新onCreate—onStart—-onResume，即又走了一遍。

02-17 15:47:17.933 E/MainActivity( 7461): start onSaveInstanceState~~~ 
02-17 15:47:17.933 E/MainActivity( 7461): start onPause~~~ 
02-17 15:47:17.933 E/MainActivity( 7461): start onStop~~~ 
02-17 15:47:17.933 E/MainActivity( 7461): start onDestroy~~~ 
02-17 15:47:17.974 E/MainActivity( 7461): start onCreate~~~ 
02-17 15:47:17.994 E/MainActivity( 7461): start onStart~~~ 
02-17 15:47:17.994 E/MainActivity( 7461): start onRestoreInstanceState~~~ 
02-17 15:47:17.994 E/MainActivity( 7461): start onResume~~~

为此需要强制设为横屏或竖屏，方法大概三种:

**1、AndroidManifest.xml 里面设置**

```
<activity
    android:name=".MainActivity"
    android:label="@string/app_name" 
    android:screenOrientation ="landscape">
</activity>
```

```java
@Override  
protected void onResume() {
	if(getRequestedOrientation() !=ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE){
 		 setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
	}
	super.onResume();
}
```

这种方法强烈不建议，谁用谁知道，在大的app不同方向启动时会慢的要死！如果在AndroidManifest.xml中设置了android:configChanges=”orientation”,竖屏状态打开也会调用onConfigurationChanged接口.

```
02-17 15:51:05.696 E/MainActivity( 7666): start onCreate~~~ 
02-17 15:51:05.756 E/MainActivity( 7666): start onStart~~~ 
02-17 15:51:05.856 E/MainActivity( 7666): start onResume~~~ 
02-17 15:51:05.896 E/MainActivity( 7666): start onConfigurationChanged~~~ 
```

3、在onConfigurationChanged里判断，为了onConfigurationChanged在监听屏幕方向变化有效需要以下条件:

​	a、AndroidManifest.xml增加权限(经过测试5.1不需要,其它自己测试看吧)

​	b、AndroidManifest.xml里设置的MiniSdkVersion和TargetSdkVersion属性大于等于13(经过测试5.1已经是22,自然不需要)

​	c、在AndroidManifest.xml的Activity里增加(经过测试5.1只需要orientation,其它自己测试,可能还需要screenSize和layoutDirection):

android:configChanges=”orientation”

经过上面就可以在onConfigurationChanged()检测屏幕方向变化事件，如果一旦在AndroidManifest.xml里设死了方向，这块就失效了。newConfig.orientation = 1 时表示竖屏，为**2则横屏**。通过判断这，然后setRequestedOrientation()来设置横屏或竖屏。

```
在activity中 
 // 设置为横屏模式  
    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
  // 设置为竖屏模式  
    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
  // 设置为跟随系统sensor的状态 
    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_SENSOR);
 
```



这个onConfigurationChanged()相关的配置，最大的好处让Activity方向变化时不进行onPause/onStop/onDestory等操作，但如果想通过setRequestOrientation的方式设置横屏或竖屏，带来的负面效果跟第2种方式一样，慢的要死。

总之，方向还是在配置里写死吧！
--------------------- 

在manifast.xml 中 给当前activity配置 防止activity在屏幕切换时 activity 重建问题

```java
android:configChanges="orientation|keyboardHidden|screenSize"
```

获取屏幕方向

```
int orientation = getResources().getConfiguration().orientation;
        if (orientation == Configuration.ORIENTATION_LANDSCAPE) {
            setContentView(R.layout.horizontallayout);
        } else if (orientation == Configuration.ORIENTATION_PORTRAIT) {
            setContentView(R.layout.verticallayout);}
```

二 禁止屏幕旋转 
第一 要先设置屏幕方向

manifast.xml 设置给activity屏幕方向

```java
android:screenOrientation="portrait"// 设置竖屏
android:configChanges="orientation|keyboardHidden|screenSize" // 防止activity重建
```

```java
//在java 代码中设置
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
//二者选一即可

//第二   为了防止用户开启了屏幕切换权限 进行屏幕监听
 @Override
    public void onConfigurationChanged(Configuration newConfig) {
              // 进行数据处理
        super.onConfigurationChanged(newConfig);
    }
//第三  保存数据 防止activity 被销毁重建 进行数据处理
 @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
    // 保存数据
    }
//或者在onpush 方法中保存

//在oncreate() 方法中获取数据 进行设置即可
```

注意 此处会有一个问题 就是 当客户开启了屏幕旋转的权限时 横屏启动时可能会出现闪退 情况

先说一下 我们的app 启动 app 启动默认的竖屏启动的 但是如果在启动过程中 app 感应到重力发生变化 activity 就会发生改变 无论你有没有设置方向 而此时就有可能删除 这是数据问题 异常自己捕获就好了
--------------------- 
