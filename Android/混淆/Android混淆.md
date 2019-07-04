# Android[混淆](https://juejin.im/post/5d1717996fb9a07eeb13bc95#heading-8)

## 前言

目前大多数的Android 是用Java语言写的，即使现在Google非常力荐kotlin，但是还有目前很多的项目还是使用Java编写，毕竟一个语言的替换是需要时间。因此，Java代码容易被反编译也是总所周知的，因此自己的防止被反编译还是需要重视的。

主要从四个方面来介绍本文：

1.Proguard混淆

2.对抗反编译工具

3.对抗安卓模拟器

4.对抗apk重打包

 

## Proguard混淆

**代码混淆**（*Obfuscated code*）是将程序中的代码以某种规则转换为难以阅读和理解的代码的一种行为。

### **Proguard 基础**

混淆的好处就是它的目的：

- 令 APK 难以被逆向工程，即很大程度上增加反编译的成本。
- 此外，Android 当中的"混淆"还能够在打包时移除无用资源，显著减少 APK 体积。
- 最后，还能以变通方式避免 Android 中常见的 **64k** 方法数引用的限制。

混淆前：(

![混淆前] https://user-gold-cdn.xitu.io/2019/6/29/16ba23561fe3d0b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



混淆后：(

![混淆后]https://user-gold-cdn.xitu.io/2019/6/29/16ba23634d2dacb7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



从上面两张图可以看出：经过混淆处理之后，我们的 APK 中**包名、类名、成员名**等都被替换为随机、无意义的名称，增加了代码阅读和理解的困难程度，提高了反编译的成本。混淆前后 APK 的体积竟然从 2.7M 减小到了 1.4M，体积缩减了近一倍

### Android 当中的混淆

在 Android 中，我们平常所说的"混淆"其实有两层意思，一个是 Java **代码的混淆**，另外一个是**资源的压缩**。其实这两者之间并没有什么关联，只不过习惯性地放在一起来使用。

#### 启用混淆

```groovy
......
    
android {
    buildTypes {
        //一般在打 release 包时才启用混淆，因为混淆会增加额外的编译时间，所以不建议在 debug 模式下启用。
        release {
            //通过 minifyEnabled 设置为 true 来开启混淆
            minifyEnabled true
            //置 shrinkResources 为 true 来开启资源的压缩。
            shrinkResources true
            //proguard-android.txt 表示 Android 系统为我们提供的默认混淆规则文件，而 proguard-rules.pro 则是我们想要自定义的混淆规则
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

只有在启用混淆的前提下开启资源压缩才会有效！

#### 代码混淆

其实，Java 平台为我们提供了 ***Proguard*** 混淆工具来帮助我们快速地对代码进行混淆。根据 Java 官方介绍，Proguard 对应的具体中文定义如下：

- 它是一个包含代码文件**压缩**、**优化**、**混淆**和**校验**等功能的工具
- 它能够检测并删除无用的类、变量、方法和属性
- 它能够优化字节码并删除未使用的指令
- 它能够将类、变量和方法的名字重命名为无意义的名称从而达到混淆效果
- 最后，它还会校验处理后的代码，主要针对 Java 6 及以上版本和 Java ME 

#### 资源压缩

Android 中，编译器为我们提供了另外一项强大的功能：**资源的压缩**。资源压缩能够帮助我们移除项目及依赖仓库中未使用到的资源，有效地降低了apk包的大小。由于资源压缩与代码混淆是协同工作，所以，如果需要开启资源的压缩，**切记要先开启代码混淆** , 否则会报错

```
ERROR: Removing unused resources requires unused code shrinking to be turned on. See http://d.android.com/r/tools/shrink-resources.html for more information.
Affected Modules: app 
```

#### 自定义要保留的资源

当我们开启了资源压缩之后，系统会默认替我们移除所有未使用的资源，假如我们需要保留某些特定的资源，可以在我们项目中创建一个被 `<resources>` 标记的 XML 文件（如 `res/raw/keep.xml`），并在 `tools:keep` 属性中指定每个要保留的资源，在 `tools:discard` 属性中指定每个要舍弃的资源。这两个属性都接受逗号分隔的资源名称列表。同样，我们可以使用字符 ***** 作为通配符。如：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/activity_video*,@layout/dialog_update_v2"
    tools:discard="@layout/unused_layout,@drawable/unused_selector" />
 
```

#### 启用严格检查模式

正常情况下，资源压缩器可准确判定系统是否使用了资源。不过，如果您的代码（包含库）调用 `Resources.getIdentifier()`，这就表示您的代码将根据动态生成的字符串查询资源名称。这时，资源压缩器会采取防御性行为，将所有具有匹配名称格式的资源标记为可能已使用，无法移除。例如，以下代码会使所有带 `img_` 前缀的资源标记为已使用：

```java
String name = String.format("img_%1d", angle + 1);
res = getResources().getIdentifier(name, "drawable", getPackageName());
```

这时，我可以开启资源的严格审查模式，只会保留确定已使用的资源。

 

#### 移除备用资源

Gradle 资源压缩器只会移除未被应用引用的资源，这意味着它不会移除用于不同设备配置的[备用资源](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fresources%2Fproviding-resources.html%23AlternativeResources)。必要时，我们可以使用 Android Gradle 插件的 `resConfigs` 属性来移除您的应用不需要的备用资源文件（常见的有用于国际化支持的 `strings.xml`，适配用的 `layout.xml` 等）：

```groovy
 android {
    defaultConfig {
        ...
        //保留中文和英文国际化支持
        resConfigs "en", "zh"
    }
}
```

 

## 自定义混淆规则

品尝完了以上"配菜"，下面让我们来品味一下本文的"主菜"：**自定义混淆规则**。首先，我们来了解一下常见的混淆命令。

#### keep 命令

这里说的 **keep** 命令指的是一系列以 **-keep** 开头的命令，它主要用来保留 Java 中不需要进行混淆的元素。以下是常见的 -keep 命令：

- **-keep**

  作用：保留指定的类和成员，防止被混淆处理。例如：

  ```python
  # 保留包：com.moos.media.entity 下面的类以及类成员
  -keep public class com.moos.media.entity.**
  
  # 保留类：NumberProgressBar
  -keep public class com.moos.media.widget.NumberProgressBar {*;}
  ```

- **-keepclassmembers**

  作用：保留指定的类的成员（变量/方法），它们将不会被混淆。如：

  ```python
  # 保留类的成员：MediaUtils类中的特定成员方法
  -keepclassmembers class com.moos.media.MediaUtils {
      public static *** getLocalVideos(android.content.Context);
      public static *** getLocalPictures(android.content.Context);
  }
  ```

- **-keepclasseswithmembers**

  作用：保留指定的类和其成员（变量/方法），前提是它们在压缩阶段没有被删除。与`-keep` 使用方式类似：

  ```python
  # 保留类：BaseMediaEntity 的子类
  -keepclasseswithmembers public class * extends com.moos.media.entity.BaseMediaEntity{*;}
  
  # 保留类：OnProgressBarListener接口的实现类
  -keep public class * implements com.moos.media.widget.OnProgressBarListener {*;}
   
  ```

- **@Keep**

  除了以上方式，你也可以选择使用 **@Keep** 注解来保留期望代码，防止它们被混淆处理。比如，我们通过 `@Keep` 修饰一个类来保留它不被混淆：

  ```python
  @Keep
  data class CloudMusicBean(var createDate: String,
                            var id: Long,
                            var name: String,
                            var url: String,
                            val imgUrl: String)
   
  ```

  同样地，我们也可以让 `@Keep` 来修饰方法或者字段进而保留它们。

#### 其他命令

1. **dontwarn**

   *-dontwarn* 命令一般在我们引入新的 library 时会使用到，常用于处理 library 中无法解决的警告。如：

   ```python
   -keep class twitter4j.** { *; }
   
   -dontwarn twitter4j.**
    
   ```

2. 其他的命令用法可参考 Android 系统提供的默认混淆规则：

   ```python
   #混淆时不生成大小写混合的类名
   -dontusemixedcaseclassnames
   
   #不跳过非公共的库的类
   -dontskipnonpubliclibraryclasses
   
   #混淆过程中记录日志
   -verbose
   
   #关闭预校验
   -dontpreverify
   
   #关闭优化
   -dontoptimize
   
   #保留注解
   -keepattributes *Annotation*
   
   #保留所有拥有本地方法的类名及本地方法名
   -keepclasseswithmembernames class * {
       native <methods>;
   }
   
   #保留自定义View的get和set方法
   -keepclassmembers public class * extends android.view.View {
      void set*(***);
      *** get*();
   }
   
   #保留Activity中View及其子类入参的方法，如: onClick(android.view.View)
   -keepclassmembers class * extends android.app.Activity {
      public void *(android.view.View);
   }
   
   #保留枚举
   -keepclassmembers enum * {
       **[] $VALUES;
       public *;
   }
   
   #保留序列化的类
   -keepclassmembers class * implements android.os.Parcelable {
     public static final android.os.Parcelable$Creator CREATOR;
   }
   
   #保留R文件的静态成员
   -keepclassmembers class **.R$* {
       public static <fields>;
   }
   
   -dontwarn android.support.**
   
   -keep class android.support.annotation.Keep
   
   -keep @android.support.annotation.Keep class * {*;}
   
   -keepclasseswithmembers class * {
       @android.support.annotation.Keep <methods>;
   }
   
   -keepclasseswithmembers class * {
       @android.support.annotation.Keep <fields>;
   }
   
   -keepclasseswithmembers class * {
       @android.support.annotation.Keep <init>(...);
   }
    
   ```

   更多混淆命令可以参考文章：[***Proguard 最全混淆规则说明***](https://juejin.im/entry/58f6d2a10ce463006bc9e6af) ，这里就不做详细讲解了。

## ProGuard常用操作

压缩(Shrinking)：默认开启，用以减小应用体积，移除未被使用的类和成员，并且会在优化动作执行之后再次执行（因为优化后可能会再次暴露一些未被使用的类和成员）。

```python
-dontshrink #关闭压缩
```



优化（Optimization）：默认开启，在字节码级别执行优化，让应用运行的更快。

```python
-dontoptimize  #关闭优化
-optimizationpasses n  #表示proguard对代码进行迭代优化的次数，Android一般为5
```



混淆（Obfuscation）：默认开启，增大反编译难度，类和类成员会被随机命名，除非用keep保护。

```python
-dontobfuscate  #关闭混淆
```

保留泛型

```
-keepattributes Signature
```

一颗星表示只是保持该包下的类名，而子包下的类名还是会被混淆；

```python
-keep class com.thc.test.*
```



两颗星表示把本包和所含子包下的类名都保持；

```python
-keep class com.thc.test.**
```

（上面两种方式保持类后，会发现类名虽然未混淆，但里面的具体方法和变量命名还是变了）



既可以保持该包下的类名，又可以保持类里面的内容不被混淆;

```python
-keep class com.thc.test.*{*;}
```



既可以保持该包及子包下的类名，又可以保持类里面的内容不被混淆;

```python
-keep class com.thc.test.**{*;}
```



保持某个类名不被混淆（但是内部内容会被混淆）

```python
-keep class com.xlpay.sqlite.cache.BaseDaoImpl
```



保持某个类的 类名及内部的所有内容不会混淆

```python
-keep class com.xlpay.sqlite.cache.BaseDaoImpl{*;}
```



保持类中特定内容，而不是所有的内容可以使用如下：

```python
-keep class com.thc.gradlestudy.MyProguardBean{
      <init>; #匹配所有构造器
      <fields>;#匹配所有域
      <methods>;#匹配所有方法}
```



上面就保持住了MyProguardBean这个类中的所有的构造方法、变量、和方法



可以在<fields>或<methods>前面加上private 、public、native等来进一步指定不被混淆的内容

```python
-keep class com.xlpay.sqlite.cache.BaseDaoImpl{
      public <methods>;#保持该类下所有的共有方法不被混淆
      public *;#保持该类下所有的共有内容不被混淆
      private <methods>;#保持该类下所有的私有方法不被混淆
      private *;#保持该类下所有的私有内容不被混淆
      public <init>(java.lang.String);#保持该类的String类型的构造方法
  }
```



在方法后加入参数，限制特定的方法(经测试：仅限于构造方法可以混淆)

```python
-keep class com.thc.gradlestudy.MyProguardBean{
      public <init>(String);
  }
```



要保留一个类中的内部类不被混淆需要用 $ 符号

```python
#保持ProguardTest中的MyClass不被混淆
-keep class com.xlpay.sqlite.cache.ProguardTest$MyClass{*;}
```



使用Java的基本规则来保护特定类不被混淆，比如用extends，implement等这些Java规则，如下：保持Android底层组件和类不要混淆

```python
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.view.View
```



如果不需要保持类名，只需要保持该类下的特定方法保持不被混淆，需要使用keepclassmembers，而不是keep，因为keep方法会保持类名。

```python
#保持ProguardTest类下test(String)方法不被混淆
-keepclassmembernames class com.xlpay.sqlite.cache.ProguardTest{
      public void test(java.lang.String);
  }
```



如果拥有某成员，保留类和类成员

```python
-keepclasseswithmembernames class com.xlpay.sqlite.cache.ProguardTest
```



## 混淆"黑名单"

我们在了解了混淆的基本命令之后，很多人应该还是一头雾水：到底哪些内容该混淆？其实，我们在使用代码混淆时，ProGuard 对我们项目中大部分代码进行了混淆操作，为了防止编译时出错，我们应该通过 `keep` 命令保留一些元素不被混淆。所以，我们只需要知道**哪些元素不应该被混淆**：

#### 枚举

项目中难免可能会用到枚举类型，然而它不能参与到混淆当中去。原因是：枚举类内部存在 `values` 方法，混淆后该方法会被重新命名，并抛出 `NoSuchMethodException`。庆幸的是，Android 系统默认的混淆规则中已经添加了对于枚举类的处理，我们无需再去做额外工作。想了解更多枚举内部细节可以去查看源码，篇幅有限不再细说。

使用enum类型时需要注意避免以下两个方法混淆 

```python
-keepclassmembers enum * {  
      public static **[] values();  
      public static ** valueOf(java.lang.String);  
  }
```

#### 被反射的元素

被反射使用的类、变量、方法、包名等不应该被混淆处理。原因在于：代码混淆过程中，**被反射使用的元素会被重命名，然而反射依旧是按照先前的名称去寻找元素**，所以会经常发生 `NoSuchMethodException` 和 `NoSuchFiledException` 问题。

#### 实体类

实体类即我们常说的"数据类"，当然经常伴随着**序列化**与**反序列化**操作。很多人也应该都想到了，混淆是将原本有特定含义的"元素"转变为无意义的名称，所以，经过混淆的"洗礼"之后，序列化之后的 `value` 对应的 `key` 已然变为没有意义的字段，这肯定是我们不希望的。同时，反序列化的过程创建对象从根本上来说还是借助于**反射**，混淆之后 `key` 会被改变，所以也会违背我们预期的效果。

#### 四大组件

Android 中的四大组件同样不应该被混淆。原因在于：

1. 四大组件使用前都需要在 **AndroidManifest.xml** 文件中进行注册声明，然而混淆处理之后，四大组件的类名就会被篡改，实际使用的类与 `manifest` 中注册的类并不匹配，故而出错。
2. 其他应用程序访问组件时可能会用到类的包名加类名，如果经过混淆，可能会无法找到对应组件或者产生异常。

```python
# 不混淆Fragment的子类类名以及onCreate()、onCreateView()方法名
-keep public class * extends android.support.v4.app.Fragment {
    public void onCreate(android.os.Bundle);
    public android.view.View onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle);
}
# 保持Android底层组件和类不要混淆
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.view.View
```



#### JNI 调用的Java 方法

当 JNI 调用的 Java 方法被混淆后，方法名会变成无意义的名称，这就与 C++ 中原本的 Java 方法名不匹配，因而会无法找到所调用的方法。

```python
#保持native方法不被混淆
-keepclasseswithmembernames class * {    
  native <methods>; 
}
```



#### 其他不应该被混淆的

- **自定义控件不需要被混淆**
- **JavaScript 调用 Java 的方法不应混淆**  WebView`中使用了`JS`调用，需要添加如下配置：

```python
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
   public *;
}
```

- **Java 的 native 方法不应该被混淆**

- **项目中引用的第三方库也不建议混淆**

- 与服务端交互时，使用GSON、fastjson等框架解析服务端数据时，所写的JSON对象类不混淆，否则无法将JSON解析成对应的对象

- Parcelable的子类和Creator静态成员变量不混淆，否则会产生Android.os.BadParcelableException异常；

  ```python
  -keep class * implements Android.os.Parcelable { 
        # 保持Parcelable不被混淆            
        public static final Android.os.Parcelable$Creator *;
    }
  ```

#### **建议：**

发布一款应用除了设minifyEnabled为ture，你也应该设置zipAlignEnabled为true，像Google Play强制要求开发者上传的应用必须是经过zipAlign的，zipAlign可以让安装包中的资源按4字节对齐，这样可以减少应用在运行时的内存消耗。

## 混淆情况记录

例子中使用：classA和classB，在加混淆的情况下多种结果：

1. 如果classA没有被keep，则不会看到classA的class文件
2. 如果classA没有被keep，classB被保持，同时classB引用到了classA，这个时候能够看到被混淆的classA的class文件，如显示为a
3. 如果classA中通过反射，获取到classB，那么classB的类名及反射用到的方法必须keep住
4. jar包混淆，暴露出的类、方法、方法的参数需要keep住
5. 情况说明：工程Demo依赖了 小米渠道的依赖，小米依赖又依赖了Common，对Common进行混淆但是不对小米渠道混淆，那么小米的依赖中使用到的Common中的类都需要keep住



## 混淆后的堆栈跟踪

代码经过 ProGuard 混淆处理后，想要读取 **StackTrace**（堆栈追踪）信息就会变得很困难。由于方法名称和类的名称都经过混淆处理，即使程序发生崩溃问题，也很难定位问题所在。幸运的是，ProGuard 为我们提供了补救的措施，在着手进行之前，我们先来看一下 ProGuard 每次构建后生成了哪些内容。

#### 混淆输出结果

混淆构建完成之后，会在 **<module-name>/build/outputs/mapping/release/** 目录下生成以下文件：

- **dump.txt**

  说明 APK 内所有类文件的内部结构。

- **mapping.txt**

  提供混淆前后的内容对照表，内容主要包含类、方法和类的成员变量。

- **seeds.txt**

  罗列出未进行混淆处理的类和成员。

- **usage.txt**

  罗列出从 APK 中移除的代码。

#### 恢复堆栈跟踪

了解完混淆构建完毕后输出的内容之后，我们现在就来看一下之前的问题：混淆处理后，StackTrace 定位困难。如何来恢复 StackTrace 的定位能力呢？系统为我们提供了 **retrace** 工具，结合上文提到的 **mapping.txt** 文件，就可以将混淆后的**崩溃堆栈追踪信息**还原成正常情况下的 **StackTrace** 信息。主要有两种方式来恢复 StackTrace，为了方便理解，我们以下面这段崩溃信息为例，借助两种方式分别来还原：

```
 java.lang.RuntimeException: Unable to start activity 
     Caused by: kotlin.KotlinNullPointerException
        at com.moos.media.ui.ImageSelectActivity.k(ImageSelectActivity.kt:71)
        at com.moos.media.ui.ImageSelectActivity.onCreate(ImageSelectActivity.kt:58)
        at android.app.Activity.performCreate(Activity.java:6237)
        at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1107)
 
```

1. 通过 retrace 脚本工具

   首先我们要进入到 Android SDK 路径的 **/tools/proguard/bin** 目录中，这里以 Mac 系统为例，可以看到如下内容：

   

   ![retrace脚本目录](https://user-gold-cdn.xitu.io/2019/6/29/16ba23882f427945?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

   

   可以看到如上三个文件，而 `proguardgui.sh` 才是我们需要的 `retrace` 脚本（Windows系统下为 `proguardgui.bat` ）。Windows 系统中只需要双击脚本 `proguardgui.bat` 即可运行，至于 Mac 系统，如果你没有做任何配置，只需要将 `proguardgui.sh` 脚本拖动到 Mac 自带的终端中，回车键即可运行。接着，我们会看到如下界面：

   

   ![retrace脚本界面](https://user-gold-cdn.xitu.io/2019/6/29/16ba23825e40dbc7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

   

   选择 **ReTrace** 栏 ，并添加我们项目中混淆生成的 **mapping.txt** 文件所在位置，然后将我们的混淆后的崩溃信息复制到 `Obfuscated stack trace` 那一栏，点击 `ReTrace!` 按钮即可还原出我们的崩溃日志信息，结果如上图所示，我们之前的混淆日志：`at com.moos.media.ui.ImageSelectActivity.k(ImageSelectActivity.kt:71)` 被还原成了 **at com.moos.media.ui.ImageSelectActivity.initView(ImageSelectActivity.kt:71)**。`ImageSelectActivity.k` 是我们混淆后的方法名，`ImageSelectActivity.initView` 则是最初未混淆前的方法名，借助于 ReTrace 工具的帮助，我们就可以像以前一样很快定位到崩溃代码区域了。

2. 通过 retrace 命令行

   我们先要将崩溃信息复制到 `txt` 格式的文件（如：proguard_stacktrace.txt）中保存，然后执行以下命令即可（MAC系统）：

   ```
   retrace.sh -verbose mapping.txt proguard_stacktrace.txt
    
   ```

   如果你是 windows 系统，可以执行以下命令：

   ```
   retrace.bat -verbose mapping.txt proguard_stacktrace.txt
    
   ```

   最终还原的结果和之前效果一样：

   

   ![retrace命令行还原stacktrace](https://user-gold-cdn.xitu.io/2019/6/29/16ba2392da5787be?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

   

   也许你通过以上两种方式在对 stackTrace 进行恢复时，发现 **Unknown Source** 问题：

   

   ![unknown source问题](https://user-gold-cdn.xitu.io/2019/6/29/16ba23998473ad0a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

   

值得注意的是，记得在混淆规则中加上如下配置来提升我们的 StackSource 查找效率：

```
# 保留源文件名和具体代码行号
-keepattributes SourceFile,LineNumberTable
 
```

此外，我们每次使用 ProGuard 创建发布构建时都都会覆盖之前版本的 `mapping.txt` 文件，因此我们每次发布新版本时都必须小心地保存一个副本。通过为每个发布构建保留一个 `mapping.txt` 文件副本，我们就可以在用户提交的已混淆的 StackTrace 来对旧版本应用的问题进行调试和修复。

## 涨姿势的操作

经过上文的介绍，我们知道，APK 在经过代码混淆处理后，包名、类名、成员名被转化为无意义、难以理解的名称，增加反编译的成本。Android ProGuard 为我们提供了默认的"混淆字典"，即将元素名称转为英文小写字母的形式。那么，我们可以定义自己的混淆字典吗？卖个关子，我们先来看一张效果图：



![自定义混淆字典](https://user-gold-cdn.xitu.io/2019/6/29/16ba23a42a4d143f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这个波操作是不是有点"出类拔萃"了？哈哈，就不卖关子了，其实很简单，只要生成一套自己的 `txt` 格式的混淆字典，然后在混淆规则 **Proguard-rules.pro** 中应用一下即可：



![混淆字典配置](https://user-gold-cdn.xitu.io/2019/6/29/16ba23a8aad6613f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



> 本文中使用的混淆字典可以在此处查看并下载：[**proguard_tradition.txt**](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FMoosphan%2FSelfAssetRepository%2Fblob%2Fmaster%2Fgithub%2Ffiles%2Fproguard-tradition.txt)

当然，大家也可以自己去定制化自己的"混淆字典"，增加反编译的难度。

一路走下来，我们发现，从混淆技术的必要性和优点来看，它还是很值得我们去深入学习和研究的，本文带大家领略的仅仅是"冰山一角"。由于本人的技术水平有限，若大家发现有问题或者阐述不当之处，欢迎指出并修正。

## 相关参考

- [*Shrink your app*](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.android.google.cn%2Fstudio%2Fbuild%2Fshrink-code)
- [*读懂Android中的代码混淆*](https://link.juejin.im?target=https%3A%2F%2Fdroidyue.com%2Fblog%2F2016%2F07%2F10%2Funderstanding-android-obfuscated-code-by-proguard)
- [*Practical ProGuard rules example*](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2Fandroiddevelopers%2Fpractical-proguard-rules-examples-5640a3907dc9)
- [*Android ProGuard 代码混淆那些事儿*](https://link.juejin.im?target=http%3A%2F%2Fjohnnyshieh.me%2Fposts%2Fandroid-proguard)
- [*Proguard 最全混淆规则说明*](https://juejin.im/entry/58f6d2a10ce463006bc9e6af)

作者：水月沐风

链接：https://juejin.im/post/5d1717996fb9a07eeb13bc95

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 语法 

- 首先看`keep`类关键字：

| 关键字                     | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| keep                       | 保留类和类成员，防止被混淆或移除                             |
| keepnames                  | 保留类和类成员，防止被混淆，但没有被引用的类成员会被移除     |
| keepclassmembers           | 只保留类成员，防止被混淆或移除                               |
| keepclassmembernames       | 只保留类成员，防止被混淆，但没有被引用的成员会被移除         |
| keepclasseswithmembers     | 保留类和类成员，防止被混淆或移除，如果指定的类成员不存在还是会被混淆 |
| keepclasseswithmembernames | 保留类和类成员，防止被混淆，如果指定的类成员不存在还是会被混淆，没有被引用的类成员会被移除 |

- 相关通配符：

| 通配符    | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| *         | 匹配任意长度字符，但不含包名分隔符`.`。例如一个类的全包名路径是`com.othershe.test.Person`，使用`com.othershe.test.*`、`com.othershe.test.*`都是可以匹配的，但`com.othershe.*`就不能匹配 |
| **        | 匹配任意长度字符，并包含包名分隔符`.`。例如要匹配`com.othershe.test.**`包下的所有内容 |
| ***       | 匹配任意参数类型。例如`*** getName(***)`可匹配`String getName(String)` |
| ...       | 匹配任意长度的任意类型参数。例如`void setName(...)`可匹配`void setName(String firstName, String secondName)` |
| <fileds>  | 匹配类、接口中所有字段                                       |
| <methods> | 匹配类、接口中所有方法                                       |
| <init>    | 匹配类中所有构造函数                                         |

 

## 项目中使用的第三方`library`混淆规则，列举了几个常用的：

```
# okhttp
-dontwarn okhttp3.**
-dontwarn okio.**
-dontwarn javax.annotation.**
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase

# Retrofit
-dontwarn okio.**
-dontwarn javax.annotation.**
-dontnote retrofit2.Platform
-dontwarn retrofit2.Platform$Java8
-keepattributes Signature
-keepattributes Exceptions

# RxJava RxAndroid
-dontwarn sun.misc.**
-keepclassmembers class rx.internal.util.unsafe.*ArrayQueue*Field* {
    long producerIndex;
    long consumerIndex;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueProducerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode producerNode;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueConsumerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode consumerNode;
}

# Gson
-keep class com.google.gson.stream.** { *; }
-keepattributes EnclosingMethod
# xxx代表model类的全包名路径
-keep class xxx.** { *; }

# butterknie
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }
-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}
-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}

# eventbus
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```

作者：SheHuan

链接：https://www.jianshu.com/p/84114b7feb38

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。