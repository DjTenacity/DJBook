### 打包

#### 1 Android Studio 打release包报错 Expected a name but was STRING at line 1 column 99 path $[0].apkInfo.vers

 + Android Studio 打release包报错

 这个问题看log是json问题但不是代码中的问题，而是apk保存地址中的output.json文件的问题，删除掉就好了

#### 2 FileProvider的使用及应用更新时提示：解析包出错、[失败等问题](<https://blog.csdn.net/XST891205/article/details/79169552>)



#### 3  [解决android textview自动换行问题](http://aichixihongshi.iteye.com/blog/1407853)

半角字符与全角字符混乱所致！一般情况下，我们输入的数字、字母以及英文标点都是半角，所以占位无法确定。它们与汉字的占位大大的不同，由于这个原因，导致很多文字的排版都是参差不齐的。对此找到了两种办法可以解决这个问题： 

1. 将textview中的字符全角化。即将所有的数字、字母及标点全部转为全角字符，使它们与汉字同占两个字节，这样就可以避免由于占位导致的排版混乱问题了。 半角转为全角的代码如下，只需调用即可。  

```
/*** 半角转换为全角  */      
public static String ToDBC(String input) {        
​         char[] c = input.toCharArray();
​         for (int i = 0; i < c.length; i++) {     
​         if (c[i] == 12288) {    
​         c[i] = (char) 32;      
​         continue; }
​          if (c[i] > 65280 && c[i] < 65375)
​             c[i] = (char) (c[i] - 65248);
​         }
​         return new String(c);
​     }  
```





2. 去除特殊字符或将所有中文标号替换为英文标号。利用正则表达式将所有特殊字符过滤，或利用replaceAll（）将中文标号替换为英文标号。则转化之后，则可解决排版混乱问题。 

```
/** * 去除特殊字符或将所有中文标号替换为英文标号  */
 public static String stringFilter(String str) {     
	// 替换中文标号
	str = str.replaceAll("【", "[").replaceAll("】", "]").replaceAll("！", "!").replaceAll("：", ":");
	String regEx = "[『』]"; // 清除掉特殊字符 
	Pattern p = Pattern.compile(regEx);
    Matcher m = p.matcher(str);
    return m.replaceAll("").trim();  }  
```

#### 4 ScheduledExecutorService取代 Timer

多线程并行处理定时任务时，Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用ScheduledExecutorService则没有这个问题。 
            

    //org.apache.commons.lang3.concurrent.BasicThreadFactory
    ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
    executorService.scheduleAtFixedRate(new Runnable() {
        @Override
        public void run() {
            //do something
        }
    },initialDelay,period, TimeUnit.HOURS);
#### 5   **依赖冲突 **    基本常量类型，都是值传递

gradlew app:dependencies查看所有的依赖

```groovy
    configurations.all {
        //        resolutionStrategy.force 'com.android.support:support-v4:27.1.1'
        resolutionStrategy.eachDependency {            
            DependencyResolveDetails details ->                
            	def requested = details.requested                
            		if (requested.group == 'com.android.support') {                    
                        if (!requested.name.startsWith("multidex")) {                     
                            details.useVersion '27.1.1'                    
                        }  }        }    }
```




 compile ('第三方库' ,{
        exclude group: 'com.google.gson'
    })

或者

android {
        configurations.all {
        // To resolve the conflict for com.android.databinding:library:1.3.1
        // dependency on support-v4:21.0.3        
        resolutionStrategy.force 'com.android.support:support-v4:27.0.2'
    }
}





#### 6 Manifest[问题总结](https://www.jianshu.com/p/ea687b0fb955)：Error:Execution failed for task ':app:processDebugManifest'. > Manifest merger faile...

##### 1:see logs

Manifest的错误，后来才发现Manifest的错误是需要通过打开`AndroidManifest.xml`文件自己寻找问题的……

##### 2:多个FileProvider冲突的问题

原链接：https://www.jianshu.com/p/199872d3204f
 当应用中存在多个FileProvider的时候（比如在引入了一个第三方开源，例takePhoto，开源框架为了适配android7.0文件的访问也使用了FileProvider），在编译时便会报错：

在报了错误信息之后androidStudio也给出了一个解决方法，增加 tools:replace="android:authorities"属性，这么一来编译时通过了，但是在使用takePhoto的时候却出现了致命错误UndeclaredThrowableException；

此时我们就会想到使用自定义的FileProvider来避免冲突，既自己写一个FileProvider继承自android.support.v4.content.FileProvider，然后在清单文件里完成配置；特别要注意一点（被这个细节小坑了一下），在配置自定义的FileProvider的时候，resource指向的xml一定要保证唯一，比如takePhoto使用了最原始的命名file_paths.xml，那么自定义的FileProvider需要用另一个xml配置（比如file_paths1.xml），避免takePhoto使用的异常（压缩图片失败等问题）



#### 7  卸载那些你想卸载又不能卸载的系统预置的 APP。

```
adb shell pm uninstall [-k] [--user USER_ID] 包名
```

参数说明：

- -k    卸载应用且保留数据与缓存，如果不加 -k 则全部删除。
- --user 指定用户 id，Android 系统支持多个用户，默认用户只有一个，id=0。

可以用这个命令，user 和 debug 版本都可以用，所有应用都能卸载掉，是不是有点狠，不过，我喜欢。

比如这里卸载 360 浏览器：

```
adb shell pm uninstall -k --user 0 com.qihoo.browser
```

看到 Success 字样，代表卸载成功。

##### **1、我想卸载某个应用，但不知道这个应用包名？**

也分享下，打开应用，执行如下命令：

```
adb shell dumpsys window | grep mCurrentFocus
```

返回：

```
mCurrentFocus=Window{38a8f240 u0 com.qihoo.browser/com.qihoo.browser.BrowserActivity}
```

能看到包名和当前页面类名，完美。

##### **2、我把一些系统应用卸载了，怎么恢复？**

黑科技虽好，但也不要把那些必要的应用给卸载了，如电话，如果真的冲动卸载了，可以通过恢复出厂设置方式恢复，问题不大，不要慌。



#### 8 startForeground(1, new Notification())  自动切回桌面

是因为targetSdkVersion : 28的原因, 安卓8及以上对Notification的显示做了限制更改 , 要求上传name和id



#### [9  Android8.0 通知栏不显示、无横幅通知问题解决](https://blog.csdn.net/Mr_Leixiansheng/article/details/84942589)

#### 10 如何使混淆过后的第三方界面与自己的app风格统一

application里面 registerActivityLifecycleCallbacks 拿到特定的Activity 然后再修改



#### [11 Android 混淆问题排查](https://blog.csdn.net/github_33304260/article/details/85269844) 

###### [Android之如何看混淆后的错误日志代码](https://blog.csdn.net/u011068702/article/details/52873454)

###### Android 代码proguard混淆之后的错误log

正式上线的项目都会做代码混淆防止反编译，目前本人使用的是progurad。

项目中加入了UncatchExceptionHandler存储错误日志到sd卡，以便发现Realease版本的bug。但是最近发现一个问题，代码混淆的同时，错误日志也被混淆了。

得到的错误诸如  com.xxx.a.b.c(unkonw source); 无法定位具体的类名和方法。

于是需要将错误日志还原。需要以下工具。

+ 1.项目目录的progurad下有一个mapping.txt文件 这是混淆后的名字 和原名字的映射关系。

+ 2.在ADT安装目录的sdk\tools\proguard\bin下有三个工具，其中proguardgui.bat就是用来还原的图形化工具。

+ 3打开proguardgui，选择ReTrace，之后的界面就不用介绍了，各位聪明的程序猿们，开心的还原你的错误日志然后苦逼的去改bug吧。

+ 补充：如果想要在log中保留错误的代码行号，需要在混淆配置中添加下面这行

```
#保留crash日志的行号
-keepattributes SourceFile,LineNumberTable 
```

