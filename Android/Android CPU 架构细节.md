## Android CPU 架构细节

##  一. 前言

早在今年一月份，Google 就发布通知，在今年 8 月 1 日开始，上架的 App，除了提供 32 位的版本之外，还需要提供 64 位的版本。

这眼看着离强制升级窗口，只剩下最后两个月的时间，很多第三放来源的 so 支持库，如果没有提供 64 位的版本，还需要同步催促合作方更新。



## 二. Android CPU 架构细节

### 2.1 这是强制规范

早在 2015 年 Google 发布 Android 5.0 版本时，就加入了 64 位处理器的支持，当时就提出了以 19 年 8 月为最后的更新支持期限，并在今年又重申了这个强制要求。

只要你的 App 存在国际版，需要上架 Google Play，这个规定都必须准守。

### 2.2 那些 APK 需要支持 64 位？

那假如你有一个国际化的 App 需要维护，在今年 8 月 1 日之后，更新 Google Play 时，就必须提供 64 位的版本。

**那这里说的 64 位版本支持，到底是什么？**

如果你的应用，完全是使用 Java 或者 Kotlin 编写代码，不包含任何原生（Native）的支持，那么就表示这个应用已经支持 64 位。

但是应用内使用了任何原生（Native）的支持（so 库），就需要针对这些 so 文件，针对不同的 CPU 架构提供不同的版本的 so 支持。

需要注意的是，有些时候，在我们自身的代码中，确实没有用到原生的支持，但是在 App 中使用的一些第三方库中却包含了。

此时最稳妥的方式，就是针对最终打包生成的 APK 文件进行分析，来判断是否需要提供 64 位架构的支持。

**那 CPU 架构是什么？什么又是 ABIs？**

在 Android 中，虽然 ARM 的 CPU 架构是主流，但是目前至少支持几类 CPU 架构，ARM 下的 ARMv5/ARMv7/ARMv8，x86 下的 x86/x86_64，以及很不常见的 MIPS 类架构。这里的每一种 CPU 类型对应了一种 API（Application Binary Interface），例如 armeabi-v7a 中的 "armeabi"  指的就是 ARM 这种类型的 ABI，后面的 “v7a” 指的是 ARMv7。

通常我们可以简单的理解：

![img](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSweJUWeTqAWhEQPNLCcta4ZZ47Vyt5S4xnJRqiaeQsQbPrWCqP7bH98icGYDH9gPgNEWxh0TpbOdM9A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这三个概念是相通的，通常在技术讨论中，说的是一个东西。

### 2.3 为什么是强制的？

谷歌之所以会有强制更新的要求，很大一方面原因是因为作为开发者，更新补全 ABIs 的动力并不足。

主要原因来自以下几个方面：

*1.* **APK 体积增大**

针对不同 CPU 架构提供对应的 so 库，当然是效率最高的做法。但是这种做法，最直接的影响，就是 APK 文件的增大，有些时候补全这些 so 支持，会导致整个 APK 体积有几 MB 到几十 MB 的增幅。

APK 体积优化，很多公司都将其算做是一个 API 指标，加入一个新特性，导致 APK 体积的增大，在很多时候都是不允许的，为此换技术方案都是常有的事。

从增长的角度来看，越小的 APK，用户下载的意愿就更大，转化率就越高。

但是随着现在流量越来越便宜，近期 iOS 已经将 蜂窝数据下载限制从 150MB 放宽至 200MB，针对安装包的体积优化标准，也可以适当的放宽了。

*2.* **本身有一定的兼容性**

应用市场中，很多 APP 其实都只有 armeabi 或者 armeabi-v7a 的支持，而市面上的设备，支持的并不是只有这两种 CPU 架构。

但是这并没有影响在这些设备上运行这些 App，这就是 CPU 架构的兼容性。

不同架构，并不意味着之间一定是不兼容的，在不同版本下，其实提供了两种 ABI 支持，分别是

- **主要 ABI**：与系统本身使用的原生代码一样，最优方案。
- **辅助 ABI**：支持的另一个 ABI 方案，兼容方案。

这种兼容策略就不在这里展开说了，最简单的就是 64 位的 arm64-v8a 在支持本身的 CPU 架构之外，还兼容支持 armeabi-v7a、armeabi；x86_64 同时也兼容支持 X86 和 armeabi。

你看，虽然添加 64 位的支持，可以有效的使用硬件的优势，提升性能，但大部分时候，采用兼容方案，是一种更简单的方式。

*3.* **没有对应架构的 so 文件**

这个原因就比较尴尬了，我们 App 中使用到的原生代码，其实有两种。

一种是我们自己编写的，源码在手，想提供对应的支持，修改配置重新编译一下就解决了。

另一种来自第三方提供的，这种时候我们没有源码，无法做到重新编译，只能和第三方沟通，看能不能提供一个对应 CPU 架构的 so 库。这种情况就非常的不可控了。

例如比较常见的一个 WebView 的替换方案，腾讯 X5 内核，本身就不提供 X86 的库。

![img](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSweJUWeTqAWhEQPNLCcta4ZeficuELm1R8se3oyPICcLf0z7ZdKlqAftYOjJfadjUSZJsITYiafebkQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

官方给的建议是使用 armeabi 或者 armeabi-v7a。

在前文有提到，ABIs 本身是有一些兼容规则的，但是这种兼容规则，是有条件的。

举个例子：64 位的 arm64-v8a 是可以向下兼容的，但是这有个前提，那就是如果你的项目中，有 armeabi-v7a 和 arm64-v8a 两个目录，就需要保证这两个目录下支持的 so 库文件保持一致。

![img](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSweJUWeTqAWhEQPNLCcta4Z2wu13gcHHaw3wUU6SfBSqTicdOd0auvBjDxjJG7L2n21EbcML4HcClQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在左边的情况下，如果 arm64-v8a 的手机用到 b.so 时，就会去 arm64-v8a 目录下找，当然是找不到 b.so 文件的，就会直接抛异常，而不会再去 armeabi-v7a 目录下继续寻找。

**如果需要提供多套 ABIs 的支持，就需要保证所有 ABI 目录下，对应的 so 文件保持一致。**

而在一些特殊的情况下，我们无法提供对应平台的 so 库，例如腾讯 X5 内核这种情况，就需要做个取舍了。

在没有 Google Play 的强制策略下，同时又因为各方考虑，大多数时候我们可能会舍弃其他 ABIs 的支持。但是现在既然强制执行了，腾讯 X5 内核就可能升级以提供 64 位的 so 库，毕竟一边是无法上架，另外一遍是一个 WebView 的内核，谁都知道怎么取舍。



## 三、支持 64 位架构

### 3.1 是否包含 64 位库？

介绍了 Android 下 CPU 架构的一些细节，接下来就要开始正题了，如何升级并支持 64 位架构。

从前文中应该了解到，支持对应的 ABIs，反映在项目中，就是存在对应 ABIs 架构的目录，并且目录中有完备的 so 库支持。

Google 并不要求我们支持所有的 64 位架构，但是对于已经支持的每种原生 32 位架构，就必须包含对应的 64 位架构。

例如：

- **对于 ARM 架构**，有 armeabi-v7a（32位） 就必须 arm64-v8a（64位）。
- **对于 x86 架构**，有 x86（32位） 就必须有 x86_64（64位）

这就要求我们有对应的目录，并且目录中包含对应的 so 文件。APK 中提供了完备的 ABIs 支持，运行的之后，会选取对应的最优支持进行加载和使用。

需要注意的是，有时候我们将 32 位的 so 复制到 64 位中，运行不会出现异常，但是这依然存在隐患。最好的办法是根据不同的架构，编译对应的 so 文件，原则上，我们的目标是确保应用可以在仅支持 64  位架构的环境中正常运行。

### 3.2 判断是否支持 64 位架构

前面也提到，我们的项目中，可能会引入一些第三方库，导致在不明确的情况下，引入了一些预期之外的 ABIs 库。

通常我们的做法是在 Gradle 中增加 abiFilters 过滤，来确保不会在打包输出的 APK 中存在预期之外的 ABIs 目录和 so 库。

```
ndk {
    //设置支持的SO库架构
    abiFilters 'armeabi-v7a' 
}
```

那么我们拿最终打包输出的 APK 文件去分析，是最稳妥的办法。

分析的方法有两种：

*1.* **AS 的 APK 分析器**

在 Android Studio 中，从菜单依次选择  Build → Analyze APK…

![img](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSweJUWeTqAWhEQPNLCcta4ZegjoeGPPShDuwNwcrKVaSpYdzbLy2zDwITkqibZRMacI6NiaX4luN5Ug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

选择需要分析的 APK 文件，查看其 lib 目录，是否存在预期的 ABIs 目录以及完备的 so 文件。

![img](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSweJUWeTqAWhEQPNLCcta4ZhNEtHwcb21I0x9TdvmkahjqfGqmicXM3AukHLvNxJqZiaVkAMrsTyiaPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

*2.* **使用 zipinfo 命令进行分析**

得到待分析的 APK 文件，就可以通过 zipinfo + grep 命令，输出其内包含的 so 文件。

```
> zipinfo -1 YOUR_APK_FILE.apk | grep \.so$
    lib/armeabi-v7a/libmain.so
    lib/armeabi-v7a/libmono.so
    lib/armeabi-v7a/libunity.so
    lib/arm64-v8a/libmain.so
    lib/arm64-v8a/libmono.so
    lib/arm64-v8a/libunity.so
```

依然是去看对应目录和 so 文件是否完备。

### 3.3 在 64 位设备上测试应用

支持 64 位架构是为了让我们利用 CPU 的特性，以提升性能，但是稳定依然是我们首先要保证的，所以在升级之后，就需要进行测试。

要测试 App，最简单的方式是使用 adb 命令安装该应用，可以配合 `--abi` 参数，用以指示要将那些 so 库，安装到设备上，这样我们在这个设备上安装的 App，就会仅包含我们制定的库。

```
# 成功安装 APK :
> adb install --abi armeabi-v7a YOUR_APK_FILE.apk
Success

# 如果 APK 中不包含 64 位 so 文件:
> adb install --abi arm64-v8a YOUR_APK_FILE.apk
adb: failed to install YOUR_APK_FILE.apk: Failure [INSTALL_FAILED_NO_MATCHING_ABIS: Failed to extract native libraries, res=-113]

# 如果你的设备（手机）不支持 64 位架构
> adb install --abi arm64-v8a YOUR_APK_FILE.apk
ABI arm64-v8a not supported on this device
```

去年上市的手机，大部分都是 64 位架构的，找一款来测试即可。

### 3.4 分包处理

如果我们的应用只需要在国内分发，当前的策略对我们并不影响，保持原样就好了。但是如果存在国际版，需要上架 Google Play 就一定要重视此次升级。

在 Google Play 上传 APK，是可以根据 CPU 架构上传不同的 APK 的，也就是我们可以针对 32 位上传一个 APK，再上传一个 64 位的 APK。

此时就需要用到 Gradle 的打包技巧了，分别输出几个仅包含对应平台的 APK，以此完成 Google Play 的要求，分别上传 32 位的支持 APK 和 64 位的支持 APK，这样能够 APK 文件不至于增大很多。

```groovy
android {
    ... 
    splits {
        abi {
            enable true
            reset()
            include 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a' //select ABIs to build APKs for
            universalApk true //generate an additional APK that contains all the ABIs
        }
    }
    // map for the version code
    project.ext.versionCodes = ['armeabi': 1, 'armeabi-v7a': 2, 'arm64-v8a': 3, 'mips': 5, 'mips64': 6, 'x86': 8, 'x86_64': 9]

    android.applicationVariants.all { variant ->
        // assign different version code for each output
        variant.outputs.each { output ->
            output.versionCodeOverride =
                    project.ext.versionCodes.get(output.getFilter(com.android.build.OutputFile.ABI), 0) * 1000000 + android.defaultConfig.versionCode
        }
    }
 }
```

这里利用 Gradle 的 splite 配置，有兴趣可以直接查阅文档，就不展开讲了。

## 四. 小结时刻

在本文中，我们借此次 Google Play 的强制支持 64 位架构的事情，讲解了 Android 下 so 库的一些兼容问题。

如果你在 Google Play 上有应用需要更新，别忘了提前准备需要的 so 库，大多数原生支持的第三方库，在此之前其实都已经提供了对应的 64 位架构。我们只需要在最终日期之前，仔细的进行增加 so 文件，以达到适配的效果。

更新完成之后，别忘了测试

[Android系统服务（一）解析ActivityManagerService(AMS)](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649841686&idx=1&sn=38565aab03c212dde0e8647d13197bad&chksm=83bf694db4c8e05b97dc5ca9e5eea58679c764aac8405c44f46eb4581742500101399d845290&scene=21#wechat_redirect)

[牛逼了，实现滴滴出行App功能！](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649844815&idx=1&sn=7bcdf1333bf26ec2656efd0b3c4d877a&chksm=83bf6514b4c8ec02ff9b554f4d9c5d6231dd6d3b20416adb9979d0061fdc5775261be010b1eb&scene=21#wechat_redirect)