## 热修复的几种方案

[记五月的一个Android面试经](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488166&idx=1&sn=87500da9bebf08f289e51c45b6d91569&chksm=eb477e38dc30f72e05473c9bb597e744f2787276610a6bdba89a2ef7740885c26f46528e69c8&scene=21#wechat_redirect)

[今日头条屏幕适配方案落地研究](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488123&idx=1&sn=57f61d72c2858cd2b5dc008475d89435&chksm=eb477ee5dc30f7f325c1f52dcaccf934ea6bc0a1cc9e24d31cfb8c9cc385b5e2224c89171e12&scene=21#wechat_redirect)

[Flutter  + MVP +Kotlin 实战！](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488369&idx=1&sn=8b5bd9cde2b9ae98ac6ccfdbd2d9d6de&chksm=eb477fefdc30f6f9c4fdcdeb8517f614dc291231d127dfb3f281c441403256d5665438e2b392&scene=21#wechat_redirect)

[知乎：APP的 Gradle plugin 实践](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488475&idx=1&sn=91d3f70a58249ef3e6ea47ba75fbaad7&chksm=eb477f45dc30f653ad100f7c8a6432b8c3bdbbb0d40068bf567a40552aab819cf5d73c0b5842&scene=21#wechat_redirect)

[从400多k的大小减到了2B，我的APP是怎么优化的？](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488443&idx=1&sn=41525b011e432567e6b725f0087e5bce&chksm=eb477f25dc30f6333252b37a134297156e163157e069a66827ac14a1470e2fe61e204cddf13b&scene=21#wechat_redirect)

[原文](https://www.jianshu.com/p/76d15810a8f5)

#### android[插件化之路](https://blog.csdn.net/xiangzhihong8/article/details/52876440)

热修复就是为了修复线上问题而提出的**修补方案**，程序修补过程无需重新发版。

#### 优点：

正常开发软件流程一般是： 线下开发-->上线-->发现bug-->紧急修复上线。这种方式代价太大。



![img](https:////upload-images.jianshu.io/upload_images/17289979-5f078b0e35d77910.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/966/format/webp)



而热修复的开发流程显得更加灵活，能及时修复bug，补丁包采用差量技术，生成的PATCH体积小，对应用没有侵入，几乎没有性能损耗。无需重新发布版本，实时高效热修复，无需下载新的应用。

但也有缺点：只能基于方法修复，而且兼容性不太好，不支持新增字段和修改方法，也不支持替换资源。



![img](https:////upload-images.jianshu.io/upload_images/17289979-cff32c43354878d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/928/format/webp)



## 当前热门的热修复技术

1：[qq空间超级补丁](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fasialiyazhou%2Farticle%2Fdetails%2F70495589)

2：[微信Tinker](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Flmj623565791%2Farticle%2Fdetails%2F54882693)

3：饿了么Amigo

4：美团Robust :

5：360RePlugin :

6：滴滴出行VirtualAPK 等

## 热修复实现原理

热修复是基于dex分包方案，和Android虚拟机的类加载器（ClassLoader）实现的

**dex分包原理**：在打包apk的时候，会把java文件通过类加载器编译成class文件，然后把class文件组合成class.dex文件，目的是**把不同class文件重复的东西只需保留一份,dex文件会把每一个类的id检索起来，存在一个链表里面，这个链表的长度是用short类型存储的，最大值是65536，也就是单个dex文件的方法数上限为65536**。方法数包括引用的framwork方法，library方法，还有自己写的代码方法。因为在生成.dex文件之后会生成一些多余的资源，所以系统会对dex文件进行优化，会分配一个缓冲区，缓冲区大小的限制：在android2.x的系统上缓冲区只有5MB，android4.x为8MB或者16MB，如果方法数量超过缓冲区的大小时，会造成dexopt崩溃，所以我们一般apk的方法数要控制在65536以内，如果超出，就要考虑dex分包，api14（Android5.0）之前是不支持分包的

**分包解决方案**：**将编译好的class文件，拆分成多个dex文件，将应用启动时必须用到的类和这些类的应用类放到主dex文件**，其他代码放到次dex文件，当应用启动的时候先加载主dex文件，等到应用启动后再动态的加载次dex文件。

 **类加载器**：java的类加载器是加载class文件，而Android的虚拟机无论是dvm还是art都只能识别dex文件，所以java的类加载器在Android中不适用，Android中的 Java.lang.ClassLoader类也不同与java中的Java.lang.ClassLoader类。

**Android虚拟机类加载机制**： Android虚拟机是从dex文件里面去加载一个类，虚拟机定义的关于类的加载规范必须被实现，他规定每个ClassLoader都得有一个父亲ClassLoader，以此形成一个父子的多层级关系，利用这个层级关系实现了类的双亲委派机制， 在Android虚拟机里的ClassLoader层级关系为ClassLoader—>BaseDexClassLoader —>Dex ClassLoader和PathClassLoader，其中PathClassLoader是用来加载Android系统类和应用的类DexClassLoader支持加载APK、DEX和JAR，也可以从SD卡进行加载 ClassLoader类根据一个指定的类名称，找到或者生成相应的字节码，然后从这些字节码中定义出一个java类 DexClassLoader和PathClassLoader都属于符合双亲委派模型的类加载器（因为它们没有重载loadClass方法）。也就是说，它们在加载一个类之前，回去检查自己以及自己以上的类加载器是否已经加载了这个类。如果已经加载过了，就会直接将之返回，而不会重复加载

**类加载热修复**：当分包完成之后，会形成一个dex包的有序数组，当需要加载类加载器的时候，会从数组中第一个dex包开始加载，直到找到这个类为止，当多个dex包中都有这个文件的时候，就取第一个文件，热修复是通过将已经修复了的bug包打成dex包，并将这个dex包放在有序数组的第一个，当加载类文件的时候，首先会找到修改好的dex包去替换之前存在bug的dex包，而排在后面的存在bug的dex包根据类加载器的双亲委派机制不会被加载，这就是类加载热修复方法的实现。

**热修复三方框架**：Nuwa，这个框架实现ClassLoader加载dex文件

要弄清热修复技术的原理，就要先弄清[Android的ClassLoader机制。](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fxiangzhihong8%2Farticle%2Fdetails%2F52880327)Android的ClassLoader分为PathClassLoader和DexClassLoader，它们都都继承自BaseDexClassLoader，其中PathClassLoader用来加载系统类和应用类；DexClassLoader用来加载jar、apk、dex文件。例如下面要介绍的阿里的Andfix和Sophix的原理如下：

#### AndFix：

由补丁类的classLoader加载补丁类，在native层针对不同Android架构中的不同的ArtMethod结构调用对应的replaceMethod方法按照定义好的ArtMethod结构一一替换方法的所有信息如所属类、访问权限、代码内存地址等。 稳定性较差，会受到国内ROM厂商对ArtMethod结构更改的影响,所以这正是AndFix不支持很多机型的原因。

#### Sophix:

由补丁类的classLoader加载补丁类，在native层直接memcpy(smeth,dmth,sizeof(ArtMethod))替换整个artMethod的结构。初始化类时会为这个类分配空间，AllocArtMethodArray会紧挨着的new出来放入art中的方法数组中。通过计算辅助类的前后两个方法的起始地址就可以计算出artMethod结构的大小了。 注：补丁类初始化时，也会分配自己的artMethod空间，拿这个修复过的新ArtMethod去替换旧ArtMethod的内容，不用管ArtMethod的结构。稳定性大大提高！





![img](https:////upload-images.jianshu.io/upload_images/17289979-62f64593c89b2040.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/908/format/webp)



**代码修复**有两大主要方案：一种是阿里系的底层替换方案，另一种是腾讯系的类加载方案。底层替换方案限制颇多，但时效性最好，加载轻快，立即见效。类加载方案时效性差，需要重新冷启动才能见效，但修复范围广，限制少。

[**2019最新Android面试题**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488205&idx=1&sn=e7f48a1cb80380697b0617d4e57e9cf2&chksm=eb477e53dc30f745fc4b8bd45a10da79c4a9b5342e107241a46a8b9cc958b439e642d7fd714b&scene=21#wechat_redirect)

[**知乎：APP的 Gradle plugin 实践**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488475&idx=1&sn=91d3f70a58249ef3e6ea47ba75fbaad7&chksm=eb477f45dc30f653ad100f7c8a6432b8c3bdbbb0d40068bf567a40552aab819cf5d73c0b5842&scene=21#wechat_redirect)

[**Gradle更小、更快构建APP的奇淫技巧**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247486933&idx=1&sn=bb384abc650dc5e02f195b09c891a778&chksm=eb47614bdc30e85daf52ea863e3369a940d7bb4b755e2d96a5147a7d0ce680f682349a9a988c&scene=21#wechat_redirect)

[**关于Gradle， 搞定Groovy闭包这一篇就够了**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247487745&idx=1&sn=9d1df391010bbe974cfbfb40c8f3f3bd&chksm=eb477d9fdc30f4892d5a602f6b8498293be27b1567ef55cb59a103cd42f8eaf0edc823675845&scene=21#wechat_redirect)

[**为什么我把 Run 出来的 Apk 发给老板，却装不上！**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488505&idx=2&sn=5483a8e6e993ec96f4845a4fe6313a5e&chksm=eb477f67dc30f671c448d7b770d8ea89c8cd287a4ce0ae03218c33b1199cd3825a022e1f540c&scene=21#wechat_redirect)

[**Google：就算Linux自带多进程通信，我也要采用Binder机制！**](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488497&idx=1&sn=ced096b96ef0b1a4f0cbffdd73667de4&chksm=eb477f6fdc30f67969a48eea02975f547cb04528c5fa6a315ade1564aefa4dc44f2961f07144&scene=21#wechat_redirect)

