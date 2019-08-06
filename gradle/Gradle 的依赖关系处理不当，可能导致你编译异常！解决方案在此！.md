## [Gradle 的依赖关系处理不当，可能导致你编译异常！解决方案在此！](https://juejin.im/post/5ac2f5946fb9a028e33ba087)

当我们作为一个初学者接触 Gradle 的时候，大部分时间在使用它来添加依赖库。而你所依赖的依赖库，可能又依赖其他的库，这非常的常见，这样的情况被称为依赖传递。

这样错综复杂的依赖关系，如果处理不好，可能达不到你预期的效果，而你有深究过吗？

我们带着问题来看看如何解决 Gradle 依赖关系导致的问题。



在 Android Studio 中，Gradle 构建过程对于开发者来说，很大程度上是抽象的。作为一个新的 Android 开发者，我们第一次遇到 Gradle 通常是在 *build.gradle* 文件中添加一个远程依赖项。

让我们看看如何阅读 Gradle 依赖关系树，并解决与依赖关系有关的问题。

 

要查看 Gradle 依赖关系树（Gradle 解析依赖关系的方式），我们可以到位于 Android Studio 底部的 Terminal 选项卡并输入以下命令：

```
gradlew app:dependencies
```

这将显示所有 [构建变体](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.android.com%2Fstudio%2Fbuild%2Fbuild-variants.html) 的依赖关系解析树。我们可以在上面的命令中添加一个标识来查看特定构建变体的配置。例如 `--configuration releaseCompileClasspath`将向我们展示 `release` 变体的依赖树。

> 关于构建变体，建议阅读官方文档：
>
> https://developer.android.com/studio/build/build-variants.html

以上是上述命令的输出：

```xml
releaseCompileClasspath - Resolved configuration for compilation for variant: release
+--- com.android.databinding:library:1.3.1
|    +--- com.android.support:support-v4:21.0.3
|    |    \--- com.android.support:support-annotations:21.0.3 -> 27.0.2
|    \--- com.android.databinding:baseLibrary:2.3.0-dev -> 3.0.1
+--- com.android.databinding:baseLibrary:3.0.1
+--- com.android.databinding:adapters:1.3.1
|    +--- com.android.databinding:library:1.3 -> 1.3.1 (*)
|    \--- com.android.databinding:baseLibrary:2.3.0-dev -> 3.0.1
+--- com.android.support.constraint:constraint-layout:1.0.2
|    \--- com.android.support.constraint:constraint-layout-solver:1.0.2
\--- com.android.support:appcompat-v7:27.0.2
     +--- com.android.support:support-annotations:27.0.2
     +--- com.android.support:support-core-utils:27.0.2
     |    +--- com.android.support:support-annotations:27.0.2
     |    \--- com.android.support:support-compat:27.0.2
     |         +--- com.android.support:support-annotations:27.0.2
     |         \--- android.arch.lifecycle:runtime:1.0.3
     |              +--- android.arch.lifecycle:common:1.0.3
     |              \--- android.arch.core:common:1.0.0
     +--- com.android.support:support-fragment:27.0.2
     |    +--- com.android.support:support-compat:27.0.2 (*)
     |    +--- com.android.support:support-core-ui:27.0.2
     |    |    +--- com.android.support:support-annotations:27.0.2
     |    |    \--- com.android.support:support-compat:27.0.2 (*)
     |    +--- com.android.support:support-core-utils:27.0.2 (*)
     |    \--- com.android.support:support-annotations:27.0.2
     +--- com.android.support:support-vector-drawable:27.0.2
     |    +--- com.android.support:support-annotations:27.0.2
     |    \--- com.android.support:support-compat:27.0.2 (*)
     \--- com.android.support:animated-vector-drawable:27.0.2
          +--- com.android.support:support-vector-drawable:27.0.2 (*)
          \--- com.android.support:support-core-ui:27.0.2 (*)

```

在查找目的之前，理解 Gradle 依赖关系树的格式很重要。

先来谈谈以下三个符号，它们的目的仅用于格式化：

- `+- - -` 是依赖分支库的开始。
- `|`  标识还是在之前的依赖库中的依赖，显示它依赖的库。
- `\- - -` 是依赖库的末尾。

星号（*） 在依赖库的末尾，意味着该库的进一步依赖关系不会显示，因为它们已经列在其他某个子依赖树中。

最重要的标识是 `->` 。

如果 Gradle 发现多个依赖库都依赖到同一个库但是不同版本，那么它必须做出选择。毕竟包含同一个库的不同版本是没有意义的。在这种情况下，Gradle 默认选择该库的最新版本。

例如：

```
| + — — com.android.support:support-v4:21.0.3
| | \ — — com.android.support:support-annotations:21.0.3 -> 27.0.2

```

在上面，Gradle 告诉说，在 `support-v4:21.0.3` 依赖关系树中， `support-annotations:21.0.3` 依赖于更新的版本 `support-annotations:27.0.2` ，所以 `27.0.2` 将被使用。



![img](https://user-gold-cdn.xitu.io/2018/4/3/162898f440f1894a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



现在我们知道如何阅读 Gradle 依赖关系解析树，我们回到本文的核心问题：**所有 com.android.support 库必须使用完全相同的版本**。

所有支持库都属于以下组 `com.android.support`。正如我们在 Gradle 的依赖关系树中看到的那样，`com.android.support:support-v4:21.0.3` 是唯一具有版本 `21.0.3` 并且未解析到最新版本的支持库 `27.0.2`，这就是造成冲突的原因。

如何解决这个问题？有几种方式，可以做到这一点：

**1、通过 ResolutionStrategy **

通过 ResolutionStrategy 强制指定 Gradle 的版本。

```groovy
android {
        configurations.all {
        // To resolve the conflict for com.android.databinding:library:1.3.1
        // dependency on support-v4:21.0.3        
        resolutionStrategy.force 'com.android.support:support-v4:27.0.2'
    }
}

```

> 关于 ResolutionStrategy 的具体细节，官方文档中有详细描述：
>
> https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html

**2、在 build.gradle 中指明版本**

要在 *build.gradle*  中，明确指定添加的是 `com.android.support:support-v4:27.0.2`。这将让Gradle 覆盖旧版本。

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    // To resolve the conflict for com.android.databinding:library:1.3.1
    // dependency on support-v4:21.0.3
    implementation 'com.android.support:support-v4:27.0.2'
    implementation 'com.android.support:appcompat-v7:27.0.2'
}

```

对我来说，在 *build.gradle* 中显式添加依赖关系似乎更加的自然，并且可以留下注释。当我们再次更新库来检查它是否仍然需要显式添加时，这个注释将提醒我关注它。

一旦添加并进行项目同步之后，错误文本将消失。现在，如果我们再次运行依赖关系命令，我们将看到`support-v4:21.0.3`解决`-> 27.0.2`。



![img](https://user-gold-cdn.xitu.io/2018/4/3/162898f440eb76d9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**3 、除了删除冲突包外，我们还可以用Gradle的 exclude group 将指定的包名排除到编译范围外：**

```
//bmob-sdk：Bmob的android sdk包，包含了Bmob的数据存储、文件等服务，以下是最新的bmob-sdk:
   implementation ('cn.bmob.android:bmob-sdk:3.5.5'){ // gson-2.6.2
        exclude group: 'com.squareup.okhttp3'
        exclude group: 'com.squareup.okio'
        exclude group: 'com.google.code.gson'
//        exclude(module: 'gson')     // 防止版本冲突
    } 
```



大部分时候 Gradle 会正确解决依赖关系。而了解了 Gradle 的依赖关系，我想遇到这样的问题我们应该更清楚如何去解决它。

我希望这篇文章能让我们更进一步了解 Gradle 依赖关系树和解决方案。

推荐阅读：

- [漫画：程序员，你能“管理”好你的产品经理吗？](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485186%26idx%3D1%26sn%3Daf106a580af73b2202d73a1301e991cb%26chksm%3D97851e23a0f297351f51de17f3d273c9fdcf67fee57f8dd4723092819aa5a565f45a2b98bb6b%23rd)
- [官方新出的 Kotlin 扩展库 KTX](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485193%26idx%3D1%26sn%3D34ab2875c304ddcea82262075bb3e720%26chksm%3D97851e28a0f2973edef8ad45671a5b10a4522a52681de7db6991809c6595f40d5a78c3af4fe3%23rd)
- [不懂批判性思维，可能正在限制你的程序员生涯](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485204%26idx%3D1%26sn%3D4f67041011bbbfdc6213003168785b94%26chksm%3D97851e35a0f297230941b4e2b1b78536f30f1f117fae96bdb04de8f7fb3604e2d30b682ec2ca%23rd)
- [Android 开发，遇上 Emoji 头疼吗？](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485072%26idx%3D1%26sn%3De1acf7aad9cc66fddadec62cb3eafb62%26chksm%3D97851fb1a0f296a75bcaa87442f254304a49239dd64bc93fcdfe6974f1cf11d15c403aa5ed26%23rd)
- [Andorid 签名和多渠道打包方案 | VasDolly](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485235%26idx%3D1%26sn%3D21e3c291f9ee4f8b8a3f7579ca8a9888%26chksm%3D97851e12a0f2970489b61a6cc1d9ff3b1d4162cbce1860bf49f3731342040a34d01bbe23f284%23rd)

 