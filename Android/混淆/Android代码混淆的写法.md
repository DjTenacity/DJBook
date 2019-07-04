## Android代码混淆的写法

**1.使用方式，在gradle文件中设置minifyEnabled为true即可开启混淆**

```groovy
    buildTypes {
        release {
            minifyEnabled ture  //是否开启代码混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

混淆内容在proguard-android.txt文件中写。

**2.混淆设置参数**

> -optimizationpasses 4 代码混淆的压缩比例，值介于0-7
> -dontusemixedcaseclassnames 混淆后类型都为小写
> -dontskipnonpubliclibraryclasses 不去忽略非公共的库类
> -dontoptimize 不优化输入的类文件
> -dontpreverify 不做预校验的操作
> -ignorewarnings 忽略警告
> -verbose 混淆时是否记录日志
> -keepattributes Annotation 保护注解
> -printmapping proguardMapping.txt 生成原类名和混淆后的类名的映射文件
> -optimizations !code/simplification/cast,!field/,!class/merging/ 指定混淆是采用的算法

**3.保持不被混淆的设置**

保持实体类不混淆

```python
-keep class 你的实体类所在的包.** { *; }
```

保持四大组件，Application，Fragment不混淆

```python
-keep public class * extends android.app.Application
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Fragment
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.preference.Preference
```

保持 native 方法不被混淆

```python
-keepclasseswithmembernames class * {  
    native <methods>;
}
```

保持枚举enum类不被混淆

```python
-keepclassmembers enum * {  
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

保持 Parcelable 不被混淆 

```python
-keep class * implements android.os.Parcelable {   
    public static final android.os.Parcelable$Creator *;
}
```

保持第三方包不混淆，比如这里用到微信、支付宝支付第三方

```python
#支付宝混淆
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}

#微信支付混淆
-keep class com.tencent.mm.opensdk.** {*;}
-keep class com.tencent.wxop.** {*;}
-keep class com.tencent.mm.sdk.** {*;}
```

**4. 完整混淆示例：**

```python
#指定代码的压缩级别
-optimizationpasses 5

#包名不混合大小写
-dontusemixedcaseclassnames

#不去忽略非公共的库类
-dontskipnonpubliclibraryclasses

 #优化  不优化输入的类文件
-dontoptimize

 #预校验
-dontpreverify

 #混淆时是否记录日志
-verbose

#忽略警告
-ignorewarning

#保护注解
-keepattributes *Annotation*

-keep public class * extends android.app.Application
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Fragment
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.preference.Preference

-keepclasseswithmembernames class * {
    native <methods>;
}
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
-keepclassmembers class **.R$* {
    *;
}
-keep class * extends android.view.View{*;}
-keep class * extends android.app.Dialog{*;}
-keep class * implements java.io.Serializable{*;}

#butterknife
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

#volley
-dontwarn com.android.volley.**
-keep class com.android.volley.**{*;}

#fastjson
-dontwarn com.alibaba.fastjson.**
-keep class com.alibaba.fastjson.**{*;}

#happy-dns
-dontwarn com.qiniu.android.dns.**
-keep class com.qiniu.android.dns.**{*;}

#okhttp
-dontwarn com.squareup.okhttp.**
-keep class com.squareup.okhttp.**{*;}

-keep class okio.**{*;}

-keep class android.net.**{*;}
-keep class com.android.internal.http.multipart.**{*;}
-keep class org.apache.**{*;}

-keep class com.qiniu.android.**{*;}

-keep class android.support.annotation.**{*;}

-keep class com.squareup.wire.**{*;}

-keep class com.ant.liao.**{*;}

#腾讯
-keep class com.tencent.**{*;}

-keep class u.aly.**{*;}

#ImageLoader
-keep class com.nostra13.universalimageloader.**{*;}

#友盟
-dontwarn com.umeng.**
-keep class com.umeng.**{*;}

#pulltorefresh
-keep class com.handmark.pulltorefresh.** { *; }
-keep class android.support.v4.** { *;}
-keep public class * extends android.support.v4.**{
 public protected *;}
-keep class android.support.v7.** {*;}
```