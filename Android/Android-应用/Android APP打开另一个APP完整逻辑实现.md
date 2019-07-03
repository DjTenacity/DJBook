# Android APP打开另一个APP完整逻辑实现

# 需求分析

1.A点击拉起B；

2.如果B没安装，下载安装；

3.如果B已安转，未在后台运行点击打开B，传值账号密码，做跨登录；

4.如果B已安装，且正在后台运行，A打开B直接显示在后台运行的页面；

简版流程图：

![img](https://img-blog.csdn.net/20180509170238919)

原理与实现
1.先说A拉起B可实现的几种方法
（1）包名，特定Activity名拉起

```java
Intent intent = new Intent(Intent.ACTION_MAIN);
/**知道要跳转应用的包命与目标Activity*/
ComponentName componentName = new ComponentName("com.kkkk.xxxx", "com.kkkk.xxxx.xxx.login.WelcomeActivity");
intent.setComponent(componentName);
intent.putExtra("", "");//这里Intent传值
startActivity(intent);
```

B应用需要在manifest文件对应Activity添加

android:exported="true"
（2）包名拉起（这里就是进去启动页）

```java
Intent intent = getPackageManager().getLaunchIntentForPackage("kuyu.com.xxxx");
if (intent != null) {
    intent.putExtra("type", "110");
    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    startActivity(intent);
}
```

（3）url拉起
--------------------- 
```java
 Intent intent = new Intent();
intent.setData(Uri.parse("csd://pull.csd.demo/cyn?type=110"));
intent.putExtra("", "");//这里Intent当然也可传递参数,但是一般情况下都会放到上面的URL中进行传递
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

B应用manifest文件需配置（注意：在原有intent-filter下方另外添加，不是在原先里面，两个同时存在）

```xml
<intent-filter>
    <data
        android:host="pull.csd.demo"
        android:path="/cyn"
        android:scheme="csd" />
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
</intent-filter>
```

优点：不暴露包命   缺点：host path schemeA应用和B应用得规定死
2.判断B应用是否安装

```java
/** 检查包是否存在  */
  private boolean checkPackInfo(String packname) {
  PackageInfo packageInfo = null;
  try {
      packageInfo = getPackageManager().getPackageInfo(packname, 0);
  } catch (PackageManager.NameNotFoundException e) {
      e.printStackTrace();
  }
  return packageInfo != null;
  }
```

3.判断B应用是否在后台运行并直接打开

 * ```java
 public static Intent getAppOpenIntentByPackageName(Context context,String packageName){
  //Activity完整名
      String mainAct = null;
      //根据包名寻找
      PackageManager pkgMag = context.getPackageManager();
      Intent intent = new Intent(Intent.ACTION_MAIN);
      intent.addCategory(Intent.CATEGORY_LAUNCHER);
      intent.setFlags(Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED|Intent.FLAG_ACTIVITY_NEW_TASK);
    
  List<ResolveInfo> list = pkgMag.queryIntentActivities(intent,
          PackageManager.GET_ACTIVITIES);
  for (int i = 0; i < list.size(); i++) {
          ResolveInfo info = list.get(i);
          if (info.activityInfo.packageName.equals(packageName)) {
              mainAct = info.activityInfo.name;
              break;
          }
      }
      if (TextUtils.isEmpty(mainAct)) {
      return null;
      }
      intent.setComponent(new ComponentName(packageName, mainAct));
      return intent;
      }
    
    public static Context getPackageContext(Context context, String packageName) {
        Context pkgContext = null;
        if (context.getPackageName().equals(packageName)) {
            pkgContext = context;
        } else {
            // 创建第三方应用的上下文环境
            try {
                pkgContext = context.createPackageContext(packageName,
                        Context.CONTEXT_IGNORE_SECURITY
                            | Context.CONTEXT_INCLUDE_CODE);
         } catch (PackageManager.NameNotFoundException e) {
             e.printStackTrace();
         }
     }
     return pkgContext;
 }
 
 public static boolean openPackage(Context context, String packageName) {
     Context pkgContext = getPackageContext(context, packageName);
     Intent intent = getAppOpenIntentByPackageName(context, packageName);
     if (pkgContext != null && intent != null) {
         pkgContext.startActivity(intent);
         return true;
     }
     return false;
 }
 if (checkPackInfo("kuyu.com.xxxxx")) {
 openPackage(this,"kuyu.com.xxxxx");
 } else {
     Toast.makeText(this, "没有安装" + "",Toast.LENGTH_LONG).show();
     //TODO  下载操作
 }
 ```
 
 
 
 这里运用的是模拟点击图标启动，不会出现程序多开，和栈顶Activity重复或者顺序错乱的问题。
 当然Activity的LaunchMode最好设为“singletop”

4. B应用接受传值跨登录操作
   一般启动页有几种操作

（1）定时直接跳转登录页面

这个就简单了，直接在handle发送跳转做判断接收intent操作就可以了

```java
if(getIntent().hasExtra("xxxx")){
    otherOpen();
}else {
    mHandler.removeMessages(0);
    mHandler.sendEmptyMessageDelayed(0, 3000);
}
```



（2）做了用户信息保存，跳过登录的，这个时候就通过handle的消息判断，做出相应操作

```java
/**  跳去首页／登录页面 */
  Handler handler = new Handler() {
  @Override
  public void handleMessage(Message msg) {
      switch (msg.arg1) {
          case 1009:
              goToActivity(MainActivity.class);
              WelcomeActivity.this.finish();
              break;
          case 1010:
              gotoLogin(handler1, runnable);
              break;
      }
  }
};
```

五丶参考文章
[Android从一个APP跳转到另一个APP的主界面或某页面，并传递数据](https://blog.csdn.net/hust_twj/article/details/73477454)
Android-跳转第三方app[重复启动问题](https://www.jianshu.com/p/c6eedc8fd7a2)
通过 PackageManager [获得你想要的](https://juejin.im/post/59e87adf51882578c52683ba) App 信息 