# android[插件化之路](https://blog.csdn.net/xiangzhihong8/article/details/52876440)

插件式开发通俗的讲就是把一个很大的app分成n多个比较小的app，其中有一个app是主app。基本上可以理解为让一个apk不安装也可以被运行。只不过这个运行是有很多限制的运行，所以才叫插件。

做插件化真正的目的：

+ 是为了去适应并行开发，是为了解耦各个模块，
+ 是为了避免模块之间的交叉依赖，是为了加快编译速度，从而提高并行开发效率。

## 插件化，组件化，动态加载

**Android 插件化** —— 指将一个程序划分为不同的部分，比如一般 App 的皮肤样式就可以看成一个插件

**Android 组件化** —— 这个概念实际跟上面相差不那么明显，组件和插件较大的区别就是：组件是指通用及复用性较高的构件，比如图片缓存就可以看成一个组件被多个 App 共用

**Android 动态加载** —— 这个实际是更高层次的概念，也有叫法是热加载或 Android 动态部署，指容器（App）在运⾏状态下动态加载某个模块，从而新增功能或改变某⼀部分行为





Android的插件化，目前插件包有两种格式：一种是apk,一种是dex包．

对插件的接入机制来说也有两种：一种是需要安装，一种是不需要安装．结合插件包的格式来说插件的方式只有三种：１，apk安装，２，apk不安装，３，dex包．
三种方式其实主要是解决两个方面的问题：
 １，加载插件中的类，
 ２，加载插件中的资源．第一个加载类的问题，这三个方式都可以很好的解决．

但目前三种方式都没有很完美的解决第２个问题． 

１，apk安装方式．插件apk安装后，可以在主程序中通过包名加载到插件的context,有了插件的context就可以解决加载插件资源的问题．但出现的新问题是：如果插件a,b,c间公用一个底层jar包，那么在abc间传送数据时，需要进行序列化和反序列化，因为a中jar包的data类与b中jar包的data类虽然都是同样的jar包也是同样的类，但两个类在java机制来是由不同的classloader加载的，是不同的类．那么就会出现插件间jar包冗余和数据传递的效率不好问题．总之能解决问题．

     ２，apk不安装，这个是不推荐的方式．可以通过dexclassloader加载到插件a中的类，但插件没有安装就无法获得插件apk的context，也就无法加载到资源，网络上有牛人通过将主程序的context替换关键的对象（如classloader,assertmanager等），伪造成插件的context，然后通过伪context也可以获得插件资源．目前个人验证的问题为：获取到的layout资源中的textview显示文本有问题，无法显示在layout设定的文本．同时个人猜测这种hack的方式或许有其它的未知的问题，但不排除高手已经解决了这个问题．
    
     ３，dex包，这个基本是java开发中jar包的方式．同样通过dexclassloader加载到插件中的类，但依旧没有context,也无法加载到资源．要使用布局，只能在插件中使用java代码来写布局．这种方式最简单（１，类似jar包的方式，也可以随意导出单个类的jar包然后转化为dex包，不存在打包成apk需要完整的工程和依赖关系的要求．２，底层的公用jar包所有插件可以公共一个，传递数据不用反复的序列化和反序列化．），也是最复杂的（布局文件全部代码写，难调试，难维护）．

总结：插件化是运行在【运行时】，组件化是运行在【编译时】。

用一个最简单的图表示：



插件化特点
 优点：
     1.模块解耦
     2.解除单个dex函数不能超过 65535的限制
     3.动态升级
     4.高效开发(编译速度更快)
 缺点：
     1.增加了主应用程序的逻辑难度
     2.技术有难度，目前一些成熟的框架都是闭源的

第三方的插件化方案：
[Qihoo360/DroidPlugin](Qihoo360/DroidPlugin)
CtripMobile/[DynamicAPK](https://github.com/CtripMobile/DynamicAPK)
[mmin18/AndroidDynamicLoader](https://github.com/mmin18/AndroidDynamicLoader)
[singwhatiwanna/dynamic-load-apk](singwhatiwanna/dynamic-load-apk)
[houkx/android-pluginmgr](houkx/android-pluginmgr) 
……

# Android ClassLoader详解

 不管是插件化还是组件化，都是基于系统的ClassLoader来设计的。只不过Android平台上虚拟机运行的是Dex字节码,一种对class文件优化的产物,传统Class文件是一个Java源码文件会生成一个.class文件，而Android是把所有Class文件进行合并，优化，然后生成一个最终的class.dex,目的是把不同class文件重复的东西只需保留一份,如果我们的Android应用不进行分dex处理,最后一个应用的apk只会有一个dex文件。 

Android平台的ClassLoader

Android中的ClassLoader
Android中类加载器有BootClassLoader,URLClassLoader,
PathClassLoader,DexClassLoader,BaseDexClassLoader,等都最终继承自java.lang.ClassLoader

