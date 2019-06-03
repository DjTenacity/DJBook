## Android 通过 APT 解耦模块依赖

APT 是什么？Annotation Process Tool，注解处理工具。  
这本是 Java 的一个工具，但 Android 也可以使用，他可以用来处理编译过程时的某些操作，比如 Java 文件的生成，注解的获取等。

在 Android 上，我们使用 APT 通常是为了生成某些处理标注有指定注解的方法、类或变量，比如 EventBus3.0开始，就是使用 APT 去处理onEvent 注解的；dagger2、butterknife 等著名的开源库也都是使用 APT 去实现的。再举一个大家非常熟悉的实际使用场景：在 Android 模块化重构的过程中，就会需要大量用到 APT 去生成作为跨模块转发层的中间类，在我之前讲《饿了么模块化平台设计》中的铁金库 `IronBank`就大量使用了 APT 与 AOP 技术去实现跨模块的处理工作。

### 实现 APT

当然，本文要讲的是 APT 的新玩法，讲 APT demo 的文章有太多了，大家随便网上搜一下就一大把，如果会了的同学，可以跳过本节。  
要实现一个简单的 APT demo 是很容易的。首先在 idea 中创建一个 Java 工程(由于 Android Studio 不能直接创建 Java 工程，我们选用 idea 更简单)

1、首先创建一个我们需要处理的注解声明：

```java
@Retention(RetentionPolicy.SOURCE)
@Target({ElementType.METHOD})
public @interface Produce {

    Class<?> returnType() default Produce.class;

    Class<?>[] params() default {};
}
```

关于注解类的创建以及上面各个给注解类加注解的含义，在我很早之前的一篇博客《Android注解式绑定控件，没你想象的那么难》中已经有很详细的介绍了，不知道的同学可以再去看一看。

2、第二步，我们为了之后处理方便，创建一个 `JavaBean` 用来封装需要的数据。

```java
class ItemData {
    Element element;
    String className = "";
    String returnType = "";
    String methodName = "";
    String[] params = {};
}
```

3、最后就是最重要的一个类了：注解是处理方式

```
public class MyAnnotationProcessor extends AbstractProcessor {
}
```

所有的注解处理类必须继承自系统的`AbstractProcessor`，如果想要让这个注解处理类生效，还要在我们的工程中创建一个 meta 文件，meta 文件中写好要提供注解处理功能的那个类的包名+类名。比如我的是这样写的：  

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jE32KtUXy6FYK5MVOFtOWXDz9zCe4srchNX8wREg0CLW3HSBPt0t75pHzHogPCJBibAicpWYAxVWFYvGzmpPbZtg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3.1、重写两个方法

```java
public class MyAnnotationProcessor extends AbstractProcessor {

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> supportTypes = new HashSet<>();
        supportTypes.add(Produce.class.getCanonicalName());
        return supportTypes;
    }

    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        boolean isProcess = false;
        try {
            isProcess = true;
            List<ItemData> creatorList = parseProduce(roundEnvironment);
            genJavaFile(creatorList);
        } catch (Exception e) {
            isProcess = false;
        }
        return isProcess;
    }
}
```

`getSupportedAnnotationTypes`是用来告诉 APT，我要关注的注解类型是哪些类型。这里只有一个注解`@Produce`所以我们的 set 就只添加了一个类型。   
`process()`就是真正用于处理注解的函数，这里我是通过`parseProduce()`返回了所有被`@Produce`修饰的方法的信息，就是我们前面封装的 JavaBean，包含了方法所在类名、方法返回值、方法名、方法参数等信息。   
然后再通过`genJavaFile()`去生成方法对应的跨模块的中间类。

### 生成类文件

在 APT 中，要生成一个类办法有很多，比如读取某个 Java 文件模板，将文件内的类模板转换成目标代码；可以使用`square`公司开源的`javapoet`库，通过传参直接输出目标类文件；也可以最简单的直接通过输出流将一个 Java 代码字符串输出到文件中。

比如，写 demo 我就直接用输出 Java 字符串的办法了。（代码节选，删掉多余类声明、try…catch）

```java
private void genJavaFile(List<Item> pageList) {
    JavaFileObject jfo = processingEnv.getFiler().createSourceFile(PACKAGE + POINT + className);
    PrintStream ps = new PrintStream(jfo.openOutputStream());
    ps.println(String.format("public class %s implements com.kymjs.Interceptor {", className));
    ps.println("\tpublic <T> T interception(Class<T> clazz, Object... params) {");

    for (Item item : pageList) {
        ps.print(String.format("if (%s.class.equals(clazz)", item.returnType));
        // 省略多参数判断逻辑
        for (int count = 0; count < item.params.length; count++) {

        }
        ps.println(") {");
        ps.print(String.format("\t\t\tobj = (T) %s.%s(", item.className, item.methodName));
        // 参数类型判断逻辑
        for (int count = 0; count < item.params.length; count++) {

        }
        ps.println(");} else ");
    }
    ps.println("{\n}return obj;}}");
    ps.flush();
}
```

最终，就会在工程目录下生成类似这样的一个文件：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jE32KtUXy6FYK5MVOFtOWXDz9zCe4srcLJ1TqCiaaBqAODK8WhCoR90f0Rr29vtovU1bag4icjSRHvZiaEaSjVcCg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 运行时加载类

本节介绍的内容，相关详细内容建议优先阅读：《优雅移除模块间耦合》这篇我在 droidcon 大会上分享的文字稿。   
新类生成好了以后，自然需要让生成的类生效，通常我们之间使用 ClassLoader 加载我们生成好的类。而在生效之前的编译阶段，会碰上一个很大的问题：普通的单 module 的 Android 工程使用 APT 不会有任何问题，但是多 module 使用的时候就会发生每个 module 都有一个包名类名完全相同的生成类，这就会发生类冲突了。

最简单的解决类冲突的办法就是让每次生成的类，类名都不一样。  
比如你可以讲类的文件加一个 `hashcode`或者随机数后缀，这样就基本能避免类冲突问题了(只能说基本，毕竟hashcode、random也有重复的几率)。

但是如果类名不一样的话，如何在运行时通过 ClassLoader 加载一个不知道类名的类呢？有两种办法，一种是通过接口遍历，给每个 APT 生成的类一个空接口父类，在运行时遍历所有类的父接口，是否是这个接口的，如果是就用ClassLoader加载他；另一种办法是通过类前缀，比如让所有类都有一个特殊的前缀，在运行时就能知道所有 APT 生成类了。   
这种方法对应的代码我可以给大家看一下(节选，删掉某些不重要的代码)：

```java
private void getAllDI(Context context) {
    mInterceptors.writeLock().lock();
    try {
        ApplicationInfo info = context.getPackageManager().getApplicationInfo(context.getPackageName(), 0);
        String path = info.sourceDir;
        DexFile dexfile = new DexFile(path);
        Enumeration entries = dexfile.entries();
        byte isLock = NONE;

        while (entries.hasMoreElements()) {
            String name = (String) entries.nextElement();
            if (name.startsWith(PACKAGE + "." + SUFFIX)) {
                threadIsRunned = true;
                if (isLock <= 0) {
                    mInterceptors.writeLock().lock();
                    isLock = LOCK;
                }
                Class clazz = Class.forName(name);
                if (Interceptor.class.isAssignableFrom(clazz) && !Interceptor.class.equals(clazz)) {
                    mInterceptors.add((Interceptor) clazz.newInstance());
                }
            } else {
                if (isLock > 0) {
                    mInterceptors.writeLock().unlock();
                    isLock = UNLOCK;
                }
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        mInterceptors.writeLock().unlock();
    }
}
```

由于遍历所有类是一个耗时操作，所以通常我们将其放在线程中，因此还需要保证多个线程的线程安全问题，防止类还没有被 ClassLoader 加载，就已经去访问这个类的情况。

另一种实现方式就是通过额外的 gradle 插件，在编译期讲所有 APT 生成类找到，记录到某个类中，这样就可以在加载的时候避免遍历所有类这步耗时操作。或者，如果实际需求中 APT 生成类中的内容是允许乱序的，比如本例中将所有类中加了@Produce 注解的方法记录下来这样的操作，也可以在编译期，将所有 APT 生成的类的内容集中到一个统一的类中，在运行时加载这个固定类（事实上我们就是这么做的），这样就能大大提高初始化时的速度了。    