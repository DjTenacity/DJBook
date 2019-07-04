# [你发布的Android 应用安全吗？](https://juejin.im/post/5b07866e51882538be0d23ac)

## 前言

目前大多数的Android 是用Java语言写的，即使现在Google非常力荐kotlin，但是还有目前很多的项目还是使用Java编写，毕竟一个语言的替换是需要时间。因此，Java代码容易被反编译也是总所周知的，因此自己的防止被反编译还是需要重视的。

主要从四个方面来介绍本文：

1.Proguard混淆

2.对抗反编译工具

3.对抗安卓模拟器

4.对抗apk重打包

 

## Proguard混淆

**代码混淆**（*Obfuscated code*）是将程序中的代码以某种规则转换为难以阅读和理解的代码的一种行为。

### **Proguard好处**

混淆的好处就是它的目的：

- 令 APK 难以被逆向工程，即很大程度上增加反编译的成本。
- 此外，Android 当中的"混淆"还能够在打包时移除无用资源，显著减少 APK 体积。
- 最后，还能以变通方式避免 Android 中常见的 **64k** 方法数引用的限制。

### **基本语法：**

```java
-include {filename}    从给定的文件中读取配置参数   
-basedirectory {directoryname}    指定基础目录为以后相对的档案名称   
-injars {class_path}    指定要处理的应用程序jar,war,ear和目录   
-outjars {class_path}    指定处理完后要输出的jar,war,ear和目录的名称   
-libraryjars {classpath}    指定要处理的应用程序jar,war,ear和目录所需要的程序库文件   
-dontskipnonpubliclibraryclasses    指定不去忽略非公共的库类。   
-dontskipnonpubliclibraryclassmembers    指定不去忽略包可见的库类的成员。  


保留选项   
-keep {Modifier} {class_specification}    保护指定的类文件和类的成员   
-keepclassmembers {modifier} {class_specification}    保护指定类的成员，如果此类受到保护他们会保护的更好  
-keepclasseswithmembers {class_specification}    保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。   
-keepnames {class_specification}    保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除）   
-keepclassmembernames {class_specification}    保护指定的类的成员的名称（如果他们不会压缩步骤中删除）   
-keepclasseswithmembernames {class_specification}    保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后）   
-printseeds {filename}    列出类和类的成员-keep选项的清单，标准输出到给定的文件   

压缩   
-dontshrink    不压缩输入的类文件   
-printusage {filename}   
-whyareyoukeeping {class_specification}       

优化   
-dontoptimize    不优化输入的类文件   
-assumenosideeffects {class_specification}    优化时假设指定的方法，没有任何副作用   
-allowaccessmodification    优化时允许访问并修改有修饰符的类和类的成员   

混淆   
-dontobfuscate    不混淆输入的类文件   
-printmapping {filename}   
-applymapping {filename}    重用映射增加混淆   
-obfuscationdictionary {filename}    使用给定文件中的关键字作为要混淆方法的名称   
-overloadaggressively    混淆时应用侵入式重载   
-useuniqueclassmembernames    确定统一的混淆类的成员名称来增加混淆   
-flattenpackagehierarchy {package_name}    重新包装所有重命名的包并放在给定的单一包中   
-repackageclass {package_name}    重新包装所有重命名的类文件中放在给定的单一包中   
-dontusemixedcaseclassnames    混淆时不会产生形形色色的类名   
-keepattributes {attribute_name,...}    保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and   

InnerClasses.   
-renamesourcefileattribute {string}    设置源文件中给定的字符串常量

```

**实际代码：**

```java
-ignorewarnings                     # 忽略警告，避免打包时某些警告出现  
-optimizationpasses 5               # 指定代码的压缩级别  
-dontusemixedcaseclassnames         # 是否使用大小写混合  
-dontskipnonpubliclibraryclasses    # 是否混淆第三方jar  
-dontpreverify                      # 混淆时是否做预校验  
-verbose                            # 混淆时是否记录日志  
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*        
# 混淆时所采用的算法  

-libraryjars   libs/treecore.jar  

-dontwarn android.support.v4.**     #缺省proguard 会检查每一个引用是否正确，但是第三方库里面往往有些不会用到的类，没有正确引用。如果不配置的话，系统就会报错。  
-dontwarn android.os.**  
-keep class android.support.v4.** { *; }        # 保持哪些类不被混淆  
-keep class com.baidu.** { *; }    
-keep class vi.com.gdi.bgl.android.**{*;}  
-keep class android.os.**{*;}  

-keep interface android.support.v4.app.** { *; }    
-keep public class * extends android.support.v4.**    
-keep public class * extends android.app.Fragment  

-keep public class * extends android.app.Activity  
-keep public class * extends android.app.Application  
-keep public class * extends android.app.Service  
-keep public class * extends android.content.BroadcastReceiver  
-keep public class * extends android.content.ContentProvider  
-keep public class * extends android.support.v4.widget  
-keep public class * extends com.sqlcrypt.database  
-keep public class * extends com.sqlcrypt.database.sqlite  
-keep public class * extends com.treecore.**  
-keep public class * extends de.greenrobot.dao.**  


-keepclasseswithmembernames class * {       # 保持 native 方法不被混淆  
   native <methods>;  
}  

-keepclasseswithmembers class * {            # 保持自定义控件类不被混淆  
   public <init>(android.content.Context, android.util.AttributeSet);  
}  

-keepclasseswithmembers class * {            # 保持自定义控件类不被混淆  
   public <init>(android.content.Context, android.util.AttributeSet, int);  
}  

-keepclassmembers class * extends android.app.Activity { //保持类成员  
  public void *(android.view.View);  
}  

-keepclassmembers enum * {                  # 保持枚举 enum 类不被混淆  
   public static **[] values();  
   public static ** valueOf(java.lang.String);  
}  

-keep class * implements android.os.Parcelable {    # 保持 Parcelable 不被混淆  
 public static final android.os.Parcelable$Creator *;  
}  

-keep class MyClass;                              # 保持自己定义的类不被混淆 
```

## 防止反编译

**防止反编译方法**

1 . Proguard混淆不仅仅可以混淆代码，还能反编译工具失效或者奔溃

2 . 使用现在国内的APK加固方法，就我所知目前你想要在360市场和腾讯的市场上发布应用，都要必须要使用他们对应的市场的加固方式：360加固和乐固加固

我们常用的反编译工具有哪些呢?

- apkTool
- baksmali
- dex2.jar
- JEB

我们通过上面的两种方法可以让这些反编译工具反编译出来不易阅读甚至让反编译工具失效或者。

由于前面已经提到过混淆的方法就不在赘述，使用国内的APK加固方法就更加的简单了，到指定的官网下载工具加固即可，文档描述很详细。

## 防止模拟器

**防止模拟器逆向分析**

原因：一般被处于逆向分析状态是在Android模拟器中运行的，所以我们只需要在代码中判断我们当前的APK是否运行在模拟器中即可。

检测是否是模拟器 一般有一下几种方式：

- 检测模拟器上的几个特殊文件
- 检测模拟器上的特殊号码
- 检测设备IDS是不是“000000000000000”
- 检测是否还有传感器、蓝牙
- 检测手机上才有的硬件信息
- 检测手机的运营商

下面我用代码来演示一下：

检查IDS

```java
/**
    * 检查IDS
    *
    * @param context
    * @return
    */
   public static boolean chechDeviceIDS(Context context) {
       @SuppressLint("ServiceCast")
       TelephonyManager telecomManager = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
       @SuppressLint("MissingPermission")
       String deviceId = telecomManager.getDeviceId();
       if (deviceId.equalsIgnoreCase(DEVICE_ID)) {
           Log.e(TAG, "chechDeviceIDS==" + DEVICE_ID);
           return true;
       }
       return false;
   } 
```

**检查模拟器特有文件**

```java
/**
    * 检查模拟器特有的文件
    *
    * @param context
    * @return
    */
   public static boolean chechDeviceFile(Context context) {
       for (int i = 0; i < DEVICE_FILE.length; i++) {
           String file_name = DEVICE_FILE[i];
           File qemu_file = new File(file_name);
           if (qemu_file.exists()) {
               Log.e(TAG, "chechDeviceFile==" + true);
               return true;
           }
       }
       return false;
   } 
```

**检查模拟器特有号码**

```java
/**
    * 检查特有电话号码
    *
    * @param context
    * @return
    */
   public static boolean chechDevicePhone(Context context) {
       @SuppressLint("ServiceCast")
       TelephonyManager telecomManager = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
       @SuppressLint("MissingPermission")
       String phoneNumber = telecomManager.getLine1Number();
       for (String phone : DEVICE_PHONE) {
           if (phone.equalsIgnoreCase(phoneNumber)) {
               Log.e(TAG, "chechDevicePhone==" + phoneNumber);
               return true;
           }
       }
       return false;
   }
复制代码
```

**检查模拟器是否含有这些设备**

```java
/**
    * 检查特是否含有设备
    *
    * @param context
    * @return
    */
   public static boolean chechDeviceBuild(Context context) {
       String board = Build.BOARD;
       String bootloader = Build.BOOTLOADER;
       String brand = Build.BRAND;
       String device = Build.DEVICE;
       String hardware = Build.HARDWARE;
       String model = Build.MODEL;
       String product = Build.PRODUCT;

       if (board.equalsIgnoreCase("unknown") || bootloader.equalsIgnoreCase("unknown")
               || brand.equalsIgnoreCase("generic") || model.equalsIgnoreCase("sdk")
               || product.equalsIgnoreCase("goldfish")) {
           Log.e(TAG, "chechDeviceBuild==" + "find emulatorBuild");
           return true;
       }
       return false;
   } 
```

运行结果：



![img](https://user-gold-cdn.xitu.io/2018/5/25/163956bcecc922d8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



我们可以看到打印结果是都显示是模拟器，提醒一点获取这些信息的时候别忘记添加权限：

通过上面我们可以判断是否是模拟器，那么这时我们就可以通过判断来杀死APP或者杀死APP所在的进程。

## 二次打包

**防止二次打包**

- 什么是二次打包？
- 通过反编译工具得到smali代码，然后再由smali代码，重打包形成APK，最后重新签名才能运行。

如果程序处于破解状态，那么我们的APK肯定是要运行在真机或者模拟器上，必须要重新签名，那么此时的签名和原来的签名必然不一致，我们就可以在程序的入口判断我们的签名，来以此判断二次打包。

**代码如下：**

```java
public class MainActivity extends AppCompatActivity {
   private static final String TAG = "MainActivity";

   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       Log.e(TAG, "getSignature=="+getSignature("demo.lt.com.repacking"));
   }

   private int getSignature(String packageName) {
       PackageManager packageManager = this.getPackageManager();
       PackageInfo info = null;
       int sig = 0;
       try {
           info = packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNATURES);
           Signature[] signatures = info.signatures;
           sig = signatures[0].hashCode();
       } catch (Exception e) {
           sig = 0;
           e.printStackTrace();
       }

       return sig;
   }
} 
```

获取到签名hashCode是唯一值：



![img](https://user-gold-cdn.xitu.io/2018/5/25/163956c6f1d08249?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



因此我们只需要再增加代码即可判断二次打包：

```java
public class MainActivity extends AppCompatActivity {
   private static final String TAG = "MainActivity";

   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       Log.e(TAG, "getSignature==" + getSignature("demo.lt.com.repacking"));
       if (getSignature("demo.lt.com.repacking") != -567503403) {
           Log.e(TAG, "被重新打包了");
       }
   }

   private int getSignature(String packageName) {
       PackageManager packageManager = this.getPackageManager();
       PackageInfo info = null;
       int sig = 0;
       try {
           info = packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNATURES);
           Signature[] signatures = info.signatures;
           sig = signatures[0].hashCode();
       } catch (Exception e) {
           sig = 0;
           e.printStackTrace();
       }

       return sig;
   }
} 
```

这里只是打印了个Log，实际代码中可以结束进程或者杀死APP。

作者：一个不羁的码农

链接：https://juejin.im/post/5b07866e51882538be0d23ac

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

 