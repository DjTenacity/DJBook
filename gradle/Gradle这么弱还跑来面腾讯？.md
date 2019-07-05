## Gradle这么弱还跑来面腾讯？

######  https://www.jianshu.com/p/1274c1f1b6a4



#### 前言

在使用Android Studio过程中没少被Gradle坑过，虽然网上有很多简单粗暴的解决方案，但极少会说清楚缘由，所以一直想看一本叫《Android Gradle权威指南》。


不过由于书中实践内容很多，更像一本工具书，而且Gradle现已发行了好几版，因此本篇仅仅是陈列出一些大的要点，尤其是那些熟悉又陌生的名词，如果想要具体了解细节和操作流程，一定要跟着书探索哟~

- Gradle入门
- Groovy基础
- Gradle构建脚本基础
- Gradle插件
- Java Gradle插件
- Android Gradle插件

#### 一. Gradle入门

**1.本书环境**

> JDK：OpenJDK 1.8.0
>
> Gradle：Gradle 2.14.1 All 版
>
> IDE：Android Studio 2.2.3
>
> Android Plugin：Android Gradle 2.2.3
>
> Android：API 23

**2.Eclipse和Android Studio**

a.开发配置区别：

- Eclipse+ADT+Ant
- Android Studio+Gradle：Gradle比Ant更灵活，有效提高开发效率

b.Eclipse迁移到AndroidStudio两种方式：

- 使用AndroidStudio直接导入Eclipse工程

​      特点：使用AS默认推荐目录结构

- 使用Eclipse导出Android Gradle配置文件，并转换成Gradle工程，再使用Android Studio把它作为Gradle工程导入

​      特点：保留原项目结构

**3.Gradle的ZIP解压后目录**

- docs：API、 DSL、指南等文档
- init.d：gradle初始化脚本目录
- lib：相关库
- media：一些icon资源
- samples：示例
- src：源文件
- getting-started.html：入门链接
- LICENSE
- NOTICE

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vL9k4qYvtFUPcThUB1AvgoXjdgXsAvQ2fZ3ibHmicwFIJKFhYfCxtILRAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**4.引例：Gradle版Hello World**

```go
//build.gradle：
task hello{//定义一个任务Task名为hello
	doLast	{//添加一个动作Action，表示在Task执行完毕后回调doLast闭包中的代码
		println'Hello World'//输出字符串，单双号均可
	}
}
//终端：
gradle hello//执行build.gradle中名为Hello的任务
//输出：
Hello World
```

**5.Gradle Wrapper**

a.含义：对Gradle一层包装，便于使用统一Gradle构建

b.目录结构：

```
|--gradle
| |--wrapper
|  |--gradle-wrapper.jar
|  |--gradle-wrapper.properties
|--gradlew
|--gradlew.bat
```

![img](https://mmbiz.qpic.cn/mmbiz_jpg/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLtGLInic94HibyFicGqD6JeemHJzlbvrDuslia7dQYpAcPrDsBoHictOicPaQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- gradle-wrapper.jar：具体业务逻辑实现的jar包
- gradle-wrapper.properties：配置文件，包含篇配置信息如下图：

 ![](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLZ6wrgXO0hvcJftpicZjE5V4km1CIVbJUtKIqONMuQhCcy8MZgWD5j4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- gradlew：Linux下可执行脚本
- gradlew.bat：Windows下可执行脚本

**c.常用命令：**

- 生成：gradle wrapper，由Wrapper Task生成
- 配置参数：

​        **-** gradle wrapper --gradle-version XXX用于指定使用的Gradle版本

​        **-** gradle wrapper --distribution-url XXX用于指定下载Gradle发行版的url地址

- 自定义：task wrapper(type:Wrapper){ //配置信息 }



**6.Gradle日志**

a.日志级别：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLcarbjwxC2LGYcD0FWXXZgZZl2MSHr8dlLel13Y9pBytiaAK4bj15v5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


b.日志输出代码：

- 使用print方法，属于quiet级别日志：println 'XX'X
- 使用内置logger：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLE2KZySdUoDlibqUBPJbXz5g7ia1zSTW2AVjbr5NA9lNVE9pyEQwqOzgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



c.日志输出控制：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vL1XKlTaibDWKdicx2NeaZExmdIhpWwFtibvTDzS7F46c7jwcWlPGCMkz2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


例如，输出QUIET级别及其之上的日志信息：gradle -q tasks



**7.Gradle命令行**

- 查看所有可执行tasks：./gradlew tasks

- 强制刷新依赖：./gradlew --refresh-dependencies assemble

- 多任务调用：./gradlew clean jar

- 查看帮助：./gradlew -?或./gradlew -h或./gradlew -help

  

#### 二.Groovy基础

> 一句话表明Groovy的地位：Groovy于Gradle，好比Java于Android



**1.特性：**

Groovy是个灵活的动态脚本语言，语法和Java很相似，又兼容Java，且在此基础上增加了很多动态类型和灵活的特性，如支持闭包和DSL



**2.语法**



- 分号不必需
- 字符串：单引号和双引号均可定义一个字符串常量，区别在于单引号不能对字符串表达式做运算，而双引号可以



```go
task printStringVar << {
def name = "张三”
println '单引号的变量计算:${name}'
println "双引号的变量计算:${name}"
}
运行./gradlew printStringVar输出结果：
单引号的变量计算:${name}
双引号的变量计算:张三
```



- 集合：以List和Map为例，介绍如何定义集合和访问集合元素



```go
//List
task printList<<{
def numList = [1,2,3,4,5,6];//定义一个List

println numList[1]//输出第二个元素
println numList[-1]//输出最后一个元素
println numList[1..3]//输出第二个到第四个元素
numList.each{
println it//输出每个元素
}
}
//Map
task printlnMap<<{
def map1 =['width':1024,'height':768]//定义一个Map

println mapl['width']//输出width的值
println mapl.height//输出height的值
map1.each{
println "Key:${it.key},Value:${it.value}"//输出所有键值对
}
}
```



- 方法：方法调用传参的括号可省略；return不必需，无return时会将最后一行代码作为其返回值；允许将代码块（闭包）作为参数传递



```go
//以集合的each方法为例，接受的参数就是一个闭包
numList.each({println it})
//省略传参的括号，并调整格式，有以下常见形式
numList.each{
println it
}
```



- 不是一定要定义成员变量才能作为类属性被访问，用get/set方法也能当作类属性



```go
task helloJavaBean<<{
Person p = new Person()
p.name = "张三"
println "名字是: ${p.name}"//输出类属性name，为张三
println "年龄是: ${p.age}"//输出类属性age，为12
}
class Person{
private String name
public int getAge(){//省略return
12
}
}
```



- 闭包：当闭包有一个参数时，默认为it，多个参数时需要一一罗列



```go
//单个参数
task helloClosure<<{
customEach{
println it
}
}
def customEach(closure){
 for(int i in 1..10){
 closure(i)
 }
}
//多个参数
task helloClosure<<{
eachMap{k,v->
 println "${k} is ${v}"
 }
}
def eachMap(closure){
def map1 = ["name":"张三","age":18]
map1.each{
 closure(it.key,it.value)
 }
}
```



#### 三.Gradle构建脚本基础

**1.Settings文件**



- 作用：初始化、设置工程树
- 文件名：settings.gradle，放在Project下
- 常用方法：include方法指定能被Gradle识别的Module



```go
//添加:app和:common这两个module参与构建
include ':app'
project(':app').projectDir = new File('存放目录')
include':common'
project(':common').projectDir = new File('存放目录')
```



**2.Build文件**



- Project的build.gradle：整个Project的共有属性，包括配置版本、插件、依赖库等信息
- Module的build.gradle：各个module私有的配置文件

![img](https://mmbiz.qpic.cn/mmbiz_jpg/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLEHicf7To0c4mC8Ruew5bP3RrEk1K5QU25ibt4sIc6vU8Krw8ibYfLK1uQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**3.Gradle任务**

**a.**含义：指原子性操作

**b.**关系：一个Gradle可包含多个Project，一个 Project可包含多个Task，即每个Project是由多个Task组成的；Task是Project的属性，属性名就是任务名

**c.**创建

- 以任务名创建：接受一个name参数



```
def task myTask = task(myTask)
myTask.doLast{
println "第一种创建Task方法，原型为Task task(String name) throws InvalidUserDataException"
}
```



以任务名+Map创建：Map参数用于对创建的task进行配置，可用配置如下图



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vL9tTxeh7HCpLd4Ziahu7UDV1dPrR6NbBJ4214Ux7SzOcaFCmUO3Rm4dA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



```go
def task myTask = task(myTask,group:BasePlugin.BUILD_GROUP)
myTask.doLast{
println "第二种创建Task方法，原型为Task task(String name,Map<String,?> args) throws InvalidUserDataException"
}
```



- 以任务名+闭包创建：常见形式



```go
task myTask{
 doLast{
   println "第三种创建Task方法，原型为Task task(String name,Closure configureClosure)"
 }
}
```



以上创建方式实际上最终都会调用TaskContainter#create()方法，使用./gradlew myTask命令执行任务



**d.**访问



- 通过任务名访问：名称.方法
- 通过TaskContainter访问：tasks['名称'].方法
- 通过路径访问：参数可以为路径或名称

   **get方式**：tasks.getByPath('路径/名称')，若不存在会抛出UnknownTaskException异常

​      **find方式**：tasks.findByPath('路径/名称')，若不存在返回null

- 通过名称访问：参数只能为名称

  **get方式**：tasks.getByName('名称')，若不存在会抛出UnknownTaskException异常

​      **find方式**：tasks.findByName('名称')，若不存在返回null



> 可见任务名称是唯一的，这是因为TaskContainer的父类 NamedDomainObjectCopllection是个具有唯一不变名字的域对象的集合



**e.**依赖：在创建任务时通过dependsOn指定其依赖的任务，可以控制任务的执行顺序



```go
task task1<<{
println 'hello'
}
task task2<<{
println 'world'
}
//依赖单个任务
task task3(dependsOn:task1){
doLast{
println 'one'
}
}
//依赖多个任务
task task4{
dependsOn task1,task2
doLast{
println 'two'
}
}
```



当执行task4时，会发现task1、task2会先执行，再执行task4



> 注：操作符<< 用在Task定义上相当于doLast



**f.**排序：除了通过强依赖来控制任务的执行顺序，还可以通过 shouldRunAfter 和 mustRunAfter 实现



```go
taskB.shouldRunAfter(taskA) //表示taskB应该在taskA执行之后执行，有可能不会按预设执行
taskB.mustRunAfter(taskA) //表示taskB必须在taskA执行之后执行
```



**g.**分组& 描述：分组是对任务的分类，便于归类整理；描述是说明任务的作用；建议两个一起配置，便于快速了解任务的分类和用途



```go
def task myTask = task(myTask)
myTask .group = BasePlugin.BUILD_GROUP
myTask .description = '这是一个构建的引导任务'
```



**h.**启用 & 禁用：enable属性可以启动和禁用任务，执行被禁用的任务输出提示该任务被跳过



```go
def task myTask = task(myTask)
myTask.enable = false //禁用任务
```



**i.**执行分析：执行Task的时候实际上是执行其拥有的actions List，它是Task对象实例的成员变量；在创建任务时Gradle会解析其中被TaskAction注解的方法作为其Task执行的action，并添加到 actions List，其中doFirst和doList会被添加到action List第一位和最后一位



**4.自定义属性**



Project、Task和SourceSet都允许用户添加额外的自定义属性、并对自定义属性进行读取和设置



- 方式：通过ext属性，添加多个通过ext代码块
- 优点：相比局部变量有广泛的作用域，可以跨Project、跨Task访问，只要能访问这些属性所属的对象即可



```go
//给Project添加自定义属性
ext.age = 18
ext{
 phone = 13888888888
 address = 'Beijing'
}
//给Task添加自定义属性
task customProperty { 
ext.inner = 'innnnnner' 

doLast{
 println project.hasProperty('customProperty') //true
 println project.hasProperty('age') //true
 println project.hasProperty('inner')//返回fasle

 println "${age}"
 println "${phone}"
 println "${inner}"
 }
}
```

#### 四.Gradle插件

**1.作用**



- 可以添加任务到项目，比如测试、编译、打包等
- 可以添加依赖配置到项目，帮助配置项目构建过程中需要的依赖，比如第三方库等
- 可以向项目中现有的对象类型添加新的扩展属性和方法等，帮助配置和优化构建
- 可以对项目进行一些约定，比如约定源代码存放位置等



> Gradle本身内置许多常用的插件，如若需要还可以扩展现有插件或者自定义插件，如Android Gradle插件就是基于内置的Java插件实现的



**2.扩展现有插件**



**a.**插件种类



- 二进制插件：实现org.gradle.api.Plugin接口的插件，可以有plugin id
- 脚本插件：严格上只是一个脚本，可以来自本地或网路



**b.**应用插件：通过Project#apply()方法，有三种用法



<一>Map参数：void apply (Map<String, ?> options)如下：

二进制插件：

- id：apply plugin:'java'
- 类型：apply plugin:org.gradle.api.plugins.JavaPlugin
- 简写：apply plugin:JavaPlugin

脚本插件：apply from:'version.gradle'

第三方发布插件：apply plugin:'com.android.application'



> 注意：应用第三方发布的作为jar的二进制插件时，必须先在buildscript{}配置其classpath才能使用，否则会提示找不到该插件



```go
//buildscript：为项目进行前提准备和初始化相关配置依赖
buildscript {
repositories {
jcenter ()
}
dependencies {
classpath 'com.android.tools.build:gradle:1.5.0"
}
}
apply plugin:'com.android.application
```



<二>闭包：void apply (Closure closure)



```go
apply {
 plugin:'java'
}
```



<三>Action：void apply (Action<? super ObjectConfigurationActicn> action)



**3.自定义插件**：实现Plugin接口、重写apply()方法

#### 五.Java Gradle插件

**1.项目结构**

使用Java插件要先应用进来：



```go
apply plugin:'java'
```



此时会添加许多默认设置和约定，比如有以下默认项目结构：



```go
|-example
| |-build.gradle
| |-src
|  |-main
|   |-java 源代码存放目录
|   |-resources 打包文件存放目录
| |-test
|  |-java 单元测试用例存放目录
|  |-resources 单元测试中使用的文件
```



**2.源集（SourceSet）**



- 作用：用于描述和管理源代码（java）及其资源（resources），如访问源代码目录、设置源集属性、更改Java原代码目录等
- 方式：通过sourceSets属性（是一个SourceSetsContainer）和sourceSets{}闭包
- 常用属性：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLsaBhWAH3x6GvSfLP1pLj3OambFDic2ibLPic3VG2C7VQ5oL2BMQRVlc3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



比如，在上述Java插件默认项目结构中的main和test就是内置的两个源集，现在更改main源集的Java源文件的存放目录到src/java下：



```go
apply plugin:'java'
sourceSets{
 main{
  java{
   srcDir 'src/java'
  }
 }
}
```



- 定义新源集：通过sourceSets{}闭包添加



```go
apply plugin:'java'
sourceSets{
 vip{
 }
}
```



此时会新建两个目录：src/vip/java和src/vip/resources



***补充***：除了SourceSet，Java插件里常用的其他属性：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLZ4ZkRNYOGKnBBW3YXFzPQHROC3Z5Z5H0CSPeDrvvrqxbcpZ1ymbMug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**3.配置第三方依赖**



**a.**依赖方式



- 外部依赖：依赖外部仓库，如Maven、Ivy等
- 项目依赖：依赖项目，依赖后可以使用该项目的Java类
- 文件依赖：如依赖Jar包，出于安全考虑不发布到Maven而是放在项目的libs文件夹下



**b.**具体方法



- 对于外部依赖，需要先在repositories{}闭包里声明依赖库的位置
- 在dependencies{}闭包添加依赖

​        \- 外部依赖：说明依赖库的group:name:version

​        \- 项目依赖：project('项目名称')

​        \- 文件依赖：files('文件名称')，多个文件逗号分开，或者fileTree(dir:'文件名称',include:'*.扩展名称')依赖指定文件夹下指定扩展名文件

- 几种依赖类型



![img](https://mmbiz.qpic.cn/mmbiz_jpg/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLKk9uiaCgswIEDu5H8Cbh4Yb3aaSaslicyszYK2uC2bickR96WVJYUJNYA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



举例：

 

```go
apply plugin:'java'
repositories {
//外部依赖 依赖Maven中心库
maveCentral() 
}
dependencies {
 //外部依赖 完整写法
 compile group:'com.squareup.okhttp3',name:'okhttp', version:'3.0.1' 
 //外部依赖 简单写法
 compile 'com.squareup.okhttp3:okhttp:3.0.1' 
 //外部依赖 指定main源集依赖
 mainCompile 'com.squareup.okhttp3:okhttp:3.0.1' 
 //项目依赖
 compile project(':example') 
 //文件依赖 依赖libs下两个Jar包
 compile files('libs/example01.jar', 'libs/example02.jar') 
 //文件依赖 指定依赖libs下所有Jar包
 compile fileTree(dir: 'libs',include: '*.jar')
}
```



**4.内置任务**



常用几种任务：

- build任务：构建项目
- clean任务：删除build目录及构建生成的文件
- assemble任务：不执行单元测试，只编译和打包
- check任务：只执行单元测试
- javadoc任务：生成Java格式的doc api文档



还有些通用任务、对源集适用的任务：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqlk6bCnhlwt8BCN1IJdt4vLJLiaCVBcZzKaWgc7bntvTqKSuwna3xY8TZwNuc3Hfgc45XKdu7ibEVtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**5.多项目构建**



- 含义：多个Gradle项目一起构建
- 方式：通过settings.gradle配置管理多项目；在每个项目都有一个build.gradle，采用项目依赖就能实现多项目协作



```go
//settings.gradle
include ':app'
project(':app').projectDir = new File('存放目录')
include ':base'
project(':base').projectDir = new File('存放目录')

//app/build.gradle
apply plugin:'java'
dependencies {
compile project(':base') 
}
```



**6.发布构件**



- 构件：Gradle构建的产物，如Jar包、Zip包等
- 意义：发布构建给其他工程使用，可以发布到本地目录、Maven、Ivy等
- 方式：明确构件类型，并通过artifacts{}闭包配置需要发布的构建，在uploadArchives{}上传发布构件



```go
//以发布jar构件为例
apply plugin:'java '
task publishJar(type:Jar)
artifacts{
 archives publishJar
}
uploadArchives{
repositories{
//发布到本地目录
flatDir{
 name 'libs'
 dirs "$projectDir/libs"
}
//发布到本地Maven库
mavenLocal()
 }
}
```



 六.[**Android Gradle插件**](https://www.jianshu.com/p/c96edcda56b4)

1.**概述**

Android Gradle插件继承于Java插件，具有Java插件的所有特性，也有自己的特性，看下官方介绍：

- 可以很容易地重用代码和资源
- 可以很容易地创建应用的衍生版本
- 可以很容易地配置、扩展以及自定义构建过程
- 和IDE无缝整合

2.**插件分类**

-  **App应用工程**：生成可运行apk应用；id: `com.android.application` 
-  **Library库工程**：生成aar包给其他的App工程公用；id: `com.android.library` 
-  **Test测试工程**：对App应用工程或Library库工程进行单元测试；id: `com.android.test` 

3.**项目结构**

```
|-example
|  |-build.gradle
|  |-example.iml
|  |-libs
|  |-proguard-rules.pro  混淆配置文件
|  |-src
|    |-androidTest
|       |-java  Android单元测试代码
|    |-main
|       |-java  App主代码
|       |-res   资源文件
|       |-AndroidManifest.xml  配置文件
|    |-test
|       |-java 普通单元测试代码
```

4.**内置任务**

- Java插件内置任务：如build、assemble、check等
- Android特有的常用任务： 
  -  `connectedCheck`任务：在所有连接的设备或者模拟器上运行check检查
  -  `deviceCheck`任务：通过API连接远程设备运行checks
  -  `lint`任务：在所有ProductFlavor上运行lint检查
  -  `install`、`uninstall`任务：在已连接的设备上安装或者卸载App
  -  `signingReport`任务：打印App签名
  -  `androidDependencies`任务：打印Android 依赖

5.[**应用实例**](https://www.jianshu.com/p/560cecfb20da)

```go
//应用插件，Android Gradle属于Android发布的第三方插件
buildscript{
    repositories{
        jcenter()
    }
    dependencies{
        classpath 'com.android.tcols.build:gradle:1.5.0'
    }
}
apply plugin:'com.android.application'
//自定义配置入口，后续详解
android{
    compileSdkVersion 23 //编译Android工程的SDK版本
    buildToolsVersion "23.0.1" //构建Android工程所用的构建工具版本

    defaultConfig{
        applicationId "org.minmin.app.example"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes{
        release{
            minifyEnabled false
            proguardFiles getDefaultPraguardFile('proguard-andrcid.txt'), 'proguard-rules.pro'
        }
    }
}
//配置第三方依赖
dependencies{
    compile fileTree(dir:'libs', include:['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcorpat-v7:23.1.1'
    compile 'com.android.support:design:23.1.1'
}
```

a.**defaultConfig**

-  **作用**：用于定义所有的默认配置，是一个ProductFlavor，若ProductFlavor没有被特殊定义，默认使用defaultConfig块指定的配置
-  **常用配置**：

| 属性名                    | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| applicationId             | 指定App包名                                                  |
| minSdkVersion             | 指定App最低支持的Android SDK                                 |
| targetSdkVersion          | 指定基于的Android SDK                                        |
| versionCode               | 配置Android App的内部版本号                                  |
| versionName               | 配置Android App的版本名称                                    |
| testApplicationId         | 配置测试App的包名，默认为applicationId + “.test”             |
| testInstrumentationRunner | 配置单元测试使用的Runner，默认为android.test.InstrumentationTestRunner |
| proguardFile              | 配置App ProGuard混淆所使用的ProGuard配置文件                 |
| proguardFiles             | 同时配置多个ProGuard配置文件                                 |
| signingConfig             | 配置默认的签名信息，也是一个ProductFlavor，可直接配置        |

b.**buildTypes**

-  **作用**：是构建类型，在Android Gradle中内置了debug和release两个构建类型，差别在于能否在设备上调试和签名不同
- 每一个BuildType都会生成一个SourceSet以及相应的`assemble<BuildTypeName>`任务
-  **常用配置**：

| 属性名              | 含义                                                  |
| ------------------- | ----------------------------------------------------- |
| applicationIdSuffix | 配置基于默认applicationId的后缀                       |
| debuggable          | 是否生成一个可供调试的Apk                             |
| jniDebuggable       | 是否生成一个可供调试JNI代码的Apk                      |
| minifyEnabled       | 是否启用Proguard混淆                                  |
| multiDexEnabled     | 是否启用自动拆分多个Dex的功能                         |
| zipAlignEnabled     | 是否开启开启zipalign优化，提高apk运行效率             |
| shrinkResources     | 是否自动清理未使用的资源，默认为false                 |
| proguardFile        | 配置Proguard混淆使用的配置文件                        |
| proguardFiles       | 同时配置多个ProGuard配置文件                          |
| signingConfig       | 配置默认的签名信息，也是一个ProductFlavor，可直接配置 |

c.**signingConfigs**

-  **作用**：配置签名设置，标记App唯一性、保护App
- 可以对不同构建类型采用不同签名方式：debug模式用于开发调试，可以直接使用Android SDK提供的默认debug签名证书；release模式用于发布，需要手动配置
-  **常用配置**：

| 属性名        | 含义                   |
| ------------- | ---------------------- |
| storeFile     | 签名证书文件           |
| storePassword | 签名证书文件的密码     |
| storeType     | 签名证书的类型         |
| keyAlias      | 签名证书中密钥别名     |
| keyPassword   | 签名证书中该密钥的密码 |

```go
android {
    signingConfigs {
        release{
            storeFile file('myFile.keystore')
            storePassword 'psw'
            keyAlias 'myKey'
            keyPassword 'psw'
        }
    }
}
```

d.**productFlavors**

-  **作用**：添加不同的渠道、并对其做不同的处理
-  **常用配置**：

| 属性名                             | 含义                                                   |
| ---------------------------------- | ------------------------------------------------------ |
| applicationId                      | 设置该渠道的包名                                       |
| consumerProguardFiles              | 对aar包进行混淆                                        |
| manifestPlaceholders               |                                                        |
| multiDexEnabled                    | 启用多个dex的配置，可突破65535方法问题                 |
| proguardFiles                      | 混淆使用的文件配置                                     |
| signingConfig                      | 签名配置                                               |
| testApplicationId                  | 适配测试包的包名                                       |
| testFunctionalTest                 | 是否是功能测试                                         |
| testHandleProfiling                | 是否启用分析功能                                       |
| testInstrumentationRunner          | 配置运行测试使用的Instrumentation Runner的类名         |
| testInstrumentationRunnerArguments | 配置Instrumentation Runner使用的参数                   |
| useJack                            | 标记是否启用Jack和Jill这个全新的、高性能的编译器       |
| dimension                          | 维度，通过flavorDimensions方法声明，声明前后代表优先级 |

```go
//定义baidu和google两个渠道，并声明两个维度，优先级为abi>version>defaultConfig
android{
    flavorDimensions "abi", "version"
    productFlavors{
        google{
            dimension "abi"
        }
       baidu{ 
           dimension "version"
       } 
}
```

e.**buildConfigFiled**

-  **作用**：在buildTypes、ProductFlavor自定义字段等配置

- 方法：
  ```go
  buildConfigField(String type,String name,String value)
  ```

  - type：字段类型
  - name：字段常量名
  - value：字段常量值

```go
android{
   buildTypes{
        debug{
            buildConfigField "boolean", "LOG_DEBUG", "true"
            buildConfigField "String", "URL", ' "http://www.ecjtu.jx.cn/" '
        }
    }
}
```

6.[**多项目构建**](https://www.jianshu.com/p/0926d25b2e3b)

和Java Grdle多项目构建一样的，通过`settings.gradle`配置管理多项目；在每个项目都有一个`build.gradle`，采用**项目依赖**就能实现多项目协作。

项目直接依赖一般适用于关联较紧密、不可复用的项目，如果想让项目被其他项目所复用，比如公共组件库、工具库等，可以单独发布出去。

7.[**多渠道构建**](https://www.jianshu.com/p/4f4d7e49716d)

a.**基本原理**

- 构建变体（Build Variant）=构建类型（Build Type）+构建渠道（Product Flavor）

> Build Type有release、debug两种构建类型
>  Product Flavor有baidu、google两种构建渠道
>  Build Variant有baiduRelease、baiduDebug、googleRelease、googleDebug四种构件产出

- 构建渠道（Product Flavor）还可以通过dimension进一步细化分组
- assemble开头的负责生成构件产物(Apk)

> assembleBaidu：运行后会生成baidu渠道的release和debug包
>  assembleRelease：运行后会生成所有渠道的release包
>  assembleBaiduRelease：运行后只会生成baidu的release包

b.**构建方式**：通过占位符`manifestPlaceholders`实现：

```go
//AndroidManifest
<meta-data 
    android: value="Channel ID" 
    android:name="UMENG_ CHANNEL"/>
//build.gradle
android{
    productFlavors{
        google{
            manifestPlaceholders.put("UMENG_ CHANNEL", "google")
        }
       baidu{
            manifestPlaceholders.put("UMENG_ CHANEL", "baidu")
       }
}
```
```go
//改进：通过productFlavors批量修改
android{
    productFlavors{
        google{
        }
       baidu{
       }
       ProductFlavors.all{ flavor->
           manifestPlaceholders.put("UMENG_ CHANEL", name) 
       }        
}
```

8.[**高级应用**](https://www.jianshu.com/p/8b2c3a3789e4)

a. **使用共享库**

- android sdk库：系统会自动链接
- 共享库：独立库，不会被系统自动链接，使用时需要在AndroidManifest通过`<uses-library>`指定

```go
//声明需要使用maps共享库，true表示如果手机系统不满足将不能安装该应用
<uses-library
    android:name="com.google.android.maps"
    android:required="true" 
/>
```

- add-ons库：存于add-ons目录下，大部分由第三方厂商或公司开发，会被自动解析添加到classpath
- optional可选库：位于platforms/android-xx/optional目录下，通常为了兼容旧版本的API，使用时需要手动添加到classpath

b. **批量修改生成的apk文件名**

- 类型： 
  -  `applicationVariants` ：仅仅适用于Android应用Gradle插件
  -  `libraryVariants` ：仅仅适用于Android库Gradle插件
  -  `testVariants` ：以上两种Gradle插件都使用
- 示例：  
  ![img](https:////upload-images.jianshu.io/upload_images/5494434-f2c007e873226106?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

> `applicationVariants`是一个DomainObjectCollection集合，通过all方法遍历每一个ApplicationVariant，这里有googleRelease和googleDebug两个变体；然后判断名字是否以.apk结尾，如果是就修改其文件名。示例中共有。

c.**动态生成版本信息**
- 原始方式：由`defaultConfig`中的`versionName`指定
- 分模块方式：把版本号等配置抽出放在单独的文件里，并用`ext{}`括起来，通过apply from将其引入到build.gradle，版本信息就被当作扩展属性直接使用了
- 从git的tag中获取
- 从属性文件中动态获取和递增

d.**隐藏签名文件信息**
-  **必要性**：为保证签名信息安全，最好直接放在项目中，而是放在服务器上
- 一种思路： 
  - 服务器：配置好环境变量，打包时直接使用
  - 本地：直接使用android提供的debug签名
  - 在signingConfigs加入以下判断

```go
signingConfigs {
    if (System.env.KEYSTORE_PATH != null) {
        //打包服务器走这个逻辑
        storeFile file(System.env.KEYSTORE_PATH)
        keyAlias System.env.ALIAS
        keyPassword System.env.KEYPASS
        storePassword System.env.STOREPASS
    } else {
        //当不能从环境变量取到签名信息时，使用本地debug签名
        storeFile file('debug.keystore')
        storePassword 'android'
        keyAlias 'androiddebugkey'
        keyPassword 'android'
    }
}
```

e.**动态添加自定义的资源**

- 针对res/values中的资源，除了使用xml定义，还可以通过Android Gradle定义

- 方法

  ：resValue(String type, String name, String value) 

  - type：资源类型，如有string、id、bool
  - name：资源名称，以便在工程中引用
  - value：资源值

```go
productFlavors{
   google{
       resValue 'string', 'channel_tips', 'google渠道欢迎你'
   }
}
```

以google为例，在debug模式下，资源文件保存目录：build/generated/res/resValues/google/debug/values/generated.xml

f.**Java编译选项**

通过`compileOptions{}`闭包进行编译配置，可配置项：

- encoding：配置源文件的编码
- sourceCompatibility：配置Java源代码的编译级别
- targetCompatibility：配置生成Java字节码的版本

```go
 android{
      compileOptions{
         encoding = 'utf-8'
         sourceCompatibility = JavaVersion.VERSI0N_ 1_ 6
         targetCompatibility = JavaVersion.VERSION_ 1_ 6
     }
}
```

g. **adb选项配置**

通过`adbOptions{}`闭包进行adb配置，可配置项：

- timeOutInMs：设置执行adb命令的超时时间，单位毫秒
- installOptions：设置adb install安装设置项 
  - -l：锁定该应用程序
  - -r：替换已存在的应用程序，即强制安装
  - -t：允许测试包
  - -s：把应用程序安装到SD卡上
  - -d：允许进行降级安装，即安装版本比手机自带的低
  - -g：为该应用授予所有运行时的权限

```go
android{
    adbOptions{
        timeOutInMs = 5*1000
        installOptions '-r', '-s'
    }
}
```

h.**DEX选项配置**

通过`dexOptions {}`闭包进行dex配置，可配置项：

- incremental：配置是否启用dx的增量模式，默认值为false
- javaMaxHeapSize：配置执行dx命令时为其分配的最大堆内存
- jumboMode：配置是否开启jumbo模式
- preDexLibraries：配置是否预dex Libraries库工程，默认值为true，开启后会提高增量构建的速度
- threadCount：配置Android Gradle运行dx命令时使用的线程数量 



---------  END  ----------

**推荐阅读**

[Android系统服务（一）解析ActivityManagerService(AMS)](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649841686&idx=1&sn=38565aab03c212dde0e8647d13197bad&chksm=83bf694db4c8e05b97dc5ca9e5eea58679c764aac8405c44f46eb4581742500101399d845290&scene=21#wechat_redirect)

[Android PMS的创建过程](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649842645&idx=1&sn=679c5c15f7a144bb168b23460bfb8003&chksm=83bf6a8eb4c8e398f9e685b9d6651bcc62bf3ac4dce4177124cfd1fdbbcd51d93f32b2529622&scene=21#wechat_redirect)

[美团跨平台框架Picasso，开启大前端的未来](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649844285&idx=1&sn=e62aff0634f09261d82ab9ef17216132&chksm=83bf6366b4c8ea70cc33a3b28c4de62ec2b05cb44a4b5a19e0cede50c06ba7e4214e4da18711&scene=21#wechat_redirect)

[彻底理解cookie、session、token](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649844338&idx=1&sn=864a2368c134e3f70766dbd99e66700c&chksm=83bf6329b4c8ea3fad2b57093a7e606b8a0efb870ff076e6583b90fc6335ea4df9f9387266cc&scene=21#wechat_redirect)