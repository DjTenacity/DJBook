## 为什么现在要用Gradle？

Gradle，它是一个基于JVM的新一代构建工具，目前已经应用于多个Android开发的技术体系中，比如构建系统、插件化、热修复和组件化等等

### 1.项目自动化

Gradle是一个构建工具，那么为什么要用构建工具，这就需要先从项目自动化开始讲起。

​		在我们开发软件时，会面临相似的情况就是，我们需要去用IDE来进行编码，当完成一些功能时会进行编译、单元测试、打包等工作，这些工作都需要开发人员手动来实现。

​		而一般的软件都是迭代式开发的，一个版本接着一本版本，每个版本又可能有很多的功能，如果开发每次实现功能时都需要手动的进行编译、单元测试和打包等工作，那显然会非常耗时而且也容易出现问题，因此项目自动化应运而生，它有以下优点：

1. 它可以**尽量防止开发手动介入**从而节省了开发的时间并减少错误的发生。
2. 自动化可以**自定义有序的步骤来完成**代码的编译、测试和打包等工作，让重复的步骤变得简单。
3. IDE可能受到不同操作系统的限制，而**自动化构建是不会依赖于特定的操作系统和IDE的**，具有平台无关性。

### 2.构建工具

**构建工具用于实现项目自动化，是一种可编程的工具**，你可以用代码来控制构建流程最终生成可交付的软件。构建工具可以帮助你创建一个重复的、可靠的、无需手动介入的、不依赖于特定操作系统和IDE的构建。这么说可能有些抽象，这里拿APK的构建过程来举例

#### 2.1 APK的构建过程

APK的构建过程可以--根据官方提供的流程图如下图所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/nk8ic4xzfuQicDQBC3zDrOL4VUhM9AhzlEfzdYZ36vn122Bic6Y62yYPtSR9lWI92m5mfGMxeQ7wn0n8ic10UiciamIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个APK构建的过程主要分为以下几步：

1. 通过AAPT(Android Asset Packaging Tool)打包res资源文件，比如AndroidManifest.xml、xml布局文件等，并**将这些xml文件编译为二进制**，其中**assets和raw文件夹的文件不会被编译为二进制，最终会生成R.java和resources.arsc文件。**
2. AIDL工具会**将所有的aidl接口转化为对应的Java接口**。
3. **所有的Java代码，包括R.java和Java接口都会被Java编译器编译成.class文件**。
4. Dex工具**会将上一步生成的.class文件、第三库和其他.class文件编译成.dex文件**。
5. 上一步编译生成的.dex文件、编译过的资源、无需编译的资源（如图片等）会被**ApkBuilder工具打包成APK文件**。
6. 使用Debug Keystore或者Release Keystore对上一步生成的APK文件进行签名。
7. 如果是对APK正式签名，还需要使用zipalign工具对APK进行对齐操作，这样应用运行时会减少内存的开销。

从以上步骤可以看出，APK的构建过程是比较繁琐的，而且这个构建过程又是时常重复的，如果没有构建工具，手动去完成构建工作，无疑对于开发人员是个折磨，也会产生诸多的问题，导致项目开发周期变长。

在Gradle出现之前，有三个基于Java的构建工具：Ant、Gant和Maven，它们被应用于Java或者Android开发中，我们来看看它们都有什么特点。

#### 2.2 Apache Ant

![img](https://mmbiz.qpic.cn/mmbiz_png/nk8ic4xzfuQicDQBC3zDrOL4VUhM9AhzlEkiamtumYlxlWVePgHCxHibVQsXaNwuWvr4tUKOWGt763hMpROia184fXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Ant在这里不是蚂蚁的意思（虽然它的图标是蚂蚁），而是Another Neat Tool的意思。
它是由 James Duncan Davidson 开发的（Tomcat 最初的开发者），最初是用来构建 Tomcat。在2000年，Ant成为一个独立的项目并被发布出来。Ant 是由 Java 编写的构建工具，它的核心代码是由Java编写的，因此具有平台无关性，构建脚本是XML格式的（默认为bulid.xml），如果你熟悉XML，那么Ant 就比较容易上手。

Ant构建脚本的样式如下所示。
**bulid.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="test" default="hello">
    <echo message="running build.xml which is equivalent to build.gant"/>
    <property file="build.properties"/>    
    <target name="init"  description="init target" > 
        <echo message="Executing init target"/>
    </target> 
    <target name="hello" depends="init" description="say hello target"> 
        <echo message="${echo.msg}"/>
    </target>
</project>
```

Ant的构建脚本由三个基本元素组成：一个project（工程）、多个target（目标）和可用的task（任务）。
Apache Ant有以下缺点：

1. Ant无法获取运行时的信息。
2. XML作为构建脚本的语言，如果构建逻辑复杂，那么构建脚本就会又长又难以维护。
3. Ant需要配合Ivy(一种管理项目依赖工具)，否则Ant很难管理依赖。
4. Ant在如何组织项目结构方面没有给出任何指导，这导致Ant虽然灵活性高，但这样的灵活导致每个构建脚本都是唯一的而且很难被理解。

#### 2.3 Gant

![img](https://mmbiz.qpic.cn/mmbiz_png/nk8ic4xzfuQicDQBC3zDrOL4VUhM9AhzlETznibk9kRHr1PxYtY3m0km73PyphhtU6UmCdtwsPicbBS2zy8ib3QON0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Gant 是一个基于Ant 的构建工具，它在Ant的基础上用Groovy写的DSL（领域特定语言）。如果用Ant 实现构建，但是不喜欢用XML来编写构建脚本或者现有的XML构建脚本很难维护和管理，那么Gant 是一个不错的选择。

Gant构建文件的样式如下所示。

build.gant

```Groovy
Ant.echo(message : 'running build.gant')
Ant.property(file : 'build.properties')
def antProperty = Ant.project.properties
target(init : 'init target') {
    echo(message : 'Executing init target')
}
target(hello : 'say hello target') {
    depends(init)
    echo(message : antProperty.'echo.msg')
}
setDefaultTarget(hello)
```

这个build.gant等同于此前Ant的bulid.xml。

#### 2.4 Apache Maven

![img](https://mmbiz.qpic.cn/mmbiz_jpg/nk8ic4xzfuQicDQBC3zDrOL4VUhM9AhzlEa9upz5fIRdVSCYCFgsVwTx5Uhc7IEkDh56qrjlfTrdULcgvKzFHpFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Maven于2004年发布，它的目标是改进开发人员在使用Ant时面临的一些问题。Maven最初是为了简化 Jakarta Turbine 项目的构建，它经历了Maven到Maven3的发展，Maven作为后来者， 继承了Ant的项目构建功能， 同样采用了XML作为构建脚本的格式。Maven具有依赖管理和项目管理的功能，提供了中央仓库，能帮助我们自动下载库文件。

Maven的构建脚本的样式如下所示。

pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

Maven相比Ant的优点：

1. Ant是过程式的，开发者需要显示的指定每个目标，以及完成该目标锁需要执行的任务。每一个项目，开发着都需要重新编写这一过程，这样会产生大量的重复。Maven是声明式的，项目的构建过程和过程中的各个阶段都由插件实现，开发者只需要声明项目的基本元素就可以了，这很大程度消除了重复。
2. Ant本身是没有依赖管理，需要配合Ivy来管理依赖，而Maven本身就提供了依赖管理。
3. **Maven 使用约定而不是配置，它为工程提供了合理的默认行为**，**项目会知道去哪个目录寻找源代码以及构建运行时有那些任务去执行，**如果你的项目遵从默认值，那么只需要写几行XML配置脚本就可以了。而Ant是使用配置且没有默认行为的。

Maven的缺点：

1. Maven的提供了默认的结构和生命周期，这些可能不适合你的项目需求。
2. 为Maven写定制的扩展过于累赘。
3. Maven的中央仓库比较混乱，当无法从中央仓库中得到需要的类库时，我们可以手工下载复制到本地仓库中，也可以建立组织内部的仓库服务器。
4. 国内连接Maven的中央仓库比较慢，需要连接国内的Maven镜像仓库。
5. Maven缺乏文档，不便于使用和理解。

### 3.Gradle的特性

Gradle是一款基于JVM的专注于灵活性和性能的开源构建工具。

![img](https://mmbiz.qpic.cn/mmbiz_png/nk8ic4xzfuQicDQBC3zDrOL4VUhM9AhzlE1mBibr5iazQCdCnFQ6iaozzpqLoibrIckiazKBtKTC00jWAoLZcB7BywCaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从上图可以看出，Gradle结合Ant和Maven等构建工具的最佳特性。它有着约定优于配置的方法、强大的依赖管理，它的构建脚本使用Groovy或Kotlin DSL编写，是Android的官方构建工具。Gradle的构建脚本的样式如下所示。

build.gradle

```groovy
apply plugin:'java'
group='com.mycompany.app'
archivesBaseName='my-app'
version='1.0-SNAPSHOT'

repositories{
   mavenCentral()
}

dependencies{
   testCompile 'junit:4.11'
}
```

这个build.gradlet等同于此前Maven的pom.xml。可以看出Groovy编写构建脚本代码量更少，可读性更强。
下面列出Gradle与竞争对手不同的特性。

#### 3.1 轻松的可拓展性

Gradle 有非常良好的拓展性。如果你想要在多个构建或者项目中分享可重用代码，Gradle的插件会帮助你实现。将Gradle插件应用于你的项目中，它会在你的项目构建过程中提供很多帮助：为你的添加项目的依赖的第三方库、为你的项目添加有用的默认设置和约定（源代码位置、单元测试代码位置）。其中Android Gradle插件继承Java Gradle插件，在本系列后续的文章会介绍插件的内容。

#### 3.2 采用了Groovy

Ant和Maven的构建脚本是由XML来编写的，**如果XML逻辑复杂内容太多就不容易维护**。Gradle可以使用Groovy DSL来实现构建脚本，**Groovy 是基于Jvm一种动态语言，它的语法和Java非常相似并兼容Java**，因此你无需担心学习Groovy的成本。Groovy在Java的基础上增加了很多动态类型和灵活的特性，比起XML，Gradle更具有表达性和可读性。

#### 3.3 强大的依赖管理

Gradle提供了可配置的可靠的依赖管理方案。一旦依赖的库被下载并存储到本地缓存中，我们的项目就可以使用了。**依赖管理很好的实现了在不同的平台和机器上产生相同的构建结果**。

#### 3.4 灵活的约定

Gradle可以为构建你的项目提供引导和默认值，如果你使用这种约定，你的Gradle构建脚本不会有几行。比起Ant，Gradle不仅仅提供了约定，还可以让你轻松的打破约定。

#### 3.5 Gradle Wrapper

Gradle Wrapper是对Gradle 的包装，它的作用是**简化Gradle本身的下载、安装和构建**，比如它会在我们没有安装Gradle的情况下，去下载指定版本的Gradle并进行构建。Gradle的版本很多，所以有可能出现版本兼容的问题，这时就需要Gradle Wrapper去统一Gradle的版本，避免开发团队因为Gradle版本不一致而产生问题。

#### 3.6 可以和其他构建工具集成

Gradle可以和Ant、Maven和Ivy进行集成，比如我们可以把Ant的构建脚本导入到Gradle的构建中。

#### 3.7 底层API

Gradle显然无法满足所有企业级构建的所有要求，但是可以通过Hook Gradle的生命周期，来监控和配置构建脚本。

#### 3.8 社区的支持和推动

Gradle是一个开源的项目，它遵循了Apache License 2.0协议。Gradle的优良特性吸引了很多开发者并形成了Gradle社区，很多开源软件开发者为Gradle的核心代码做出了共享。

### 4.总结

本篇文章从项目自动化开始讲起，介绍了常用的构建工具：Ant、Gant和Maven，最后介绍了Gradle的特性，这些特性和其他竞争的构建工具相比有着很大的优势和吸引力，这也是为什么我们现在要用Gradle的原因。                     