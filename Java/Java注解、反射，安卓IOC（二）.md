# Java注解、反射，安卓IOC（二）

## 运行时注解

首先我们自己简单实现类似 xUtils 这种运行时注解框架。

### 绑定 View 控件

创建注解：

```java
@Retention(RetentionPolicy.RUNTIME)//运行时注解
@Target(ElementType.FIELD)//Target为属性
public @interface FindView {
    int value() default -1;
}
```

View解析代码：

```java
public class ViewInject {

    public static void bind(Activity activity) {
        inject(new ViewFinder(activity), activity);
    }

    public static void bind(View view) {
        inject(new ViewFinder(view), view);
    }

    public static void bind(View view, Object obj) {
        inject(new ViewFinder(view), obj);
    }

    private static void inject(ViewFinder finder, Object obj) {
        injectFields(finder, obj);
        injectMethods(finder, obj);
    }

    private static void injectFields(ViewFinder finder, Object obj) {
        Class<?> clazz = obj.getClass();
        Field[] fields = clazz.getDeclaredFields();//获取所有变量
        for (Field field : fields) {
            if (field.isAnnotationPresent(FindView.class)) {
                FindView findView = field.getAnnotation(FindView.class);//获取注解
                if (findView.value() < 0) {
                    throw new IllegalArgumentException("The id can't be -1.");
                } else {
                    View view = finder.findViewById(findView.value());
                    try {
                        field.setAccessible(true);//破坏封装
                        field.set(obj, view); //设置属性
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

主要流程为通过反射获取并遍历所有变量，若变量被注解修饰，则将注解的 ID 赋值给指定方法并调用。

### 绑定 OnClick 事件

创建注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD) //Target为方法
public @interface OnClick {
    int[] id();
}
```

OnClick 事件注入：

```java
private static void injectMethods(ViewFinder finder, final Object obj) {
    Method[] methods = obj.getClass().getDeclaredMethods();
    for (final Method method : methods) {
        if (method.isAnnotationPresent(OnClick.class)) {
            OnClick onClick = method.getAnnotation(OnClick.class);
            if (onClick.id().length != 0) {
                for (int i : onClick.id()) {
                    View view = finder.findViewById(i);
                    method.setAccessible(true);
                    view.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            try {
                                method.invoke(obj, v);
                            } catch (IllegalAccessException e) {
                                e.printStackTrace();
                            } catch (InvocationTargetException e) {
                                e.printStackTrace();
                            }
                        }
                    });
                }
            }
        }
    }
}
```

主要流程为通过反射获取并遍历所有方法，若方法被注解修饰，遍历所有的 ID ，将注解的 ID 赋值给 findViewById 方法，然后在 setOnClickListener 调用 method 方法。

在 Activity 中的使用：

```java
public class IocActivity extends AppCompatActivity {

    @FindView(R.id.txt_ioc_test)
    private TextView mTxtTest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_ioc);
        ViewInject.bind(this);
        mTxtTest.setText("测试");
    }

    @OnClick(id = {R.id.btn_ioc_test, R.id.btn_ioc_test2})
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btn_ioc_test:
                Toast.makeText(this, "Click1", Toast.LENGTH_SHORT).show();
                break;
            case R.id.btn_ioc_test2:
                Toast.makeText(this, "Click2", Toast.LENGTH_SHORT).show();
                break;
        }
    }
}
```

以上为运行时注解的简单实现，即 xUtils 使用的注解方法。但是这种方法因为通过一系列反射获取属性、方法等，对性能会有所影响，所以不建议在实际项目中使用，下边介绍下轻量级的编译时注解。

## 编译时注解

ButterKnife 源码解析网上已经有很多不错的文章了，例如这篇 [ButterKnife源码分析](https://www.jianshu.com/p/0f3f4f7ca505) 讲的就很好。这里主要介绍下自己的大致实现以及编译时注解在 Android Studio 中的使用。

首先介绍下大概的项目结构，如下图所示：



![img](https:////upload-images.jianshu.io/upload_images/2630285-dcd0823419ae1eac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/157/format/webp)

APT结构.png

- annotation module: Java library  - 定义一系列注解。
- injetc module: Android library  - 定义注解的接口及调用方法。
- compiler module: Java library  - 自定义编译时注解 AbstractProcessor 在编译期间生成 java 代码。
- app: 使用方法。

本篇文章主要为介绍及学习，所以此处仅实现 setContentView 的编译时注解。

1、声明注解

```java
@Retention(RetentionPolicy.CLASS) //编译时注解
@Target(ElementType.TYPE) //修饰类
public @interface ContentView {
    int value();
}
```

2、声明外界接口及方法

声明接口：

```java
public interface ContentInjector<T> {
    void injectContent(T obj, Activity activity); //此处仅用 Activity 参数即可实现文章的 demo
}
```

提供方法：

```java
public class ContentViewInject {
    public static void bind(Activity activity) {//绑定
        injectContentView(activity);
    }
    private static void injectContentView(Activity activity) {
        Class<? extends Activity> clazz = activity.getClass();
        try {
            ContentInjector injector = (ContentInjector) Class.forName(clazz.getName()
                    + "$$ViewBinder").newInstance();
            injector.injectContent(activity, activity);
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

以上代码为简单使用，butterknife 中的 UnBinder 解绑，使用 Map 缓存等暂不考虑。
 此处的 Class.forName("") 以及 class.newInstance() 会对性能略有影响，butterknife 在此进行了 map 缓存优化。

3、自定义 AbstractProcessor，此处需将 module 设置为 Java library 才可继承 AbstractProcessor。

```java
@AutoService(Processor.class)
@SupportedSourceVersion(value = SourceVersion.RELEASE_7)
public class ContentViewInjectProcessor extends AbstractProcessor {


    //可用 @SupportedAnnotationTypes("com.lauzy.ContentView") 注解 ContentViewInjectProcessor 代替
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> supportTypes = new LinkedHashSet<>();
        supportTypes.add(ContentView.class.getCanonicalName());
        return supportTypes;
    }

    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        String packageName;
        String className;
        //遍历每个被 ContentView 修饰的 class 文件
        for (Element element : roundEnvironment.getElementsAnnotatedWith(ContentView.class)) {
            if (element.getKind() == ElementKind.CLASS) {
                TypeElement typeElement = (TypeElement) element;
                PackageElement packageEle = (PackageElement) element.getEnclosingElement();
                packageName = packageEle.getQualifiedName().toString();//获取包名
                //因为是 ElementKind.CLASS 类型，所以可以直接强制转换，获取类名
                className = typeElement.getSimpleName().toString();

                int layoutId = typeElement.getAnnotation(ContentView.class).value();//获取注解的 id
                
                //拼接 Java 类的字符串
                StringBuilder builder = new StringBuilder();
                builder.append("package ").append(packageName).append(";\n");
                builder.append("import android.view.View;\n");
                builder.append("import android.app.Activity;\n");
                builder.append("import com.freedom.lauzy.inject.ContentInjector;\n");
                builder.append('\n');

                builder.append("public class ").append(className + "$$ViewBinder");
                builder.append("<T extends ").append(className).append(">");
                builder.append(" implements ContentInjector<T>");
                builder.append(" {\n");
                builder.append("@Override\n")
                        .append("public void injectContent(final T source, Activity activity) {\n");
                builder.append("    ((Activity) source).setContentView(" + layoutId);
                builder.append(");\n");
                builder.append("}\n\n}\n");


                //写入 Java 文件
                try {
                    JavaFileObject fileObject = processingEnv.getFiler().createSourceFile(
                            packageName + "." + className + "$$ViewBinder",
                            typeElement);
                    Writer writer = fileObject.openWriter();
                    writer.write(builder.toString());
                    writer.flush();
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                    System.out.println("error");
                }
            }
        }
        return true;
    }
}
```

butterknife 中使用了 [javapoet](https://link.jianshu.com?t=https://github.com/square/javapoet) 生成 Java 代码文件

此 module 的 gradle 配置如下：

```java
apply plugin: 'java'

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.google.auto.service:auto-service:1.0-rc2' // google 的生成源代码库
    compile project(':annotation')
}

sourceCompatibility = "1.7"
targetCompatibility = "1.7"
```

4、app 使用

app 的 gradle 配置如下：

```java
dependencies {
    ...
    annotationProcessor project(':compiler')
    compile project(':annotation')
    compile project(':inject')
}
```

activity 中使用：

```java
@ContentView(R.layout.activity_ioc)
public class IocActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ContentViewInject.bind(this);
    }
}
```

这样，整个流程就结束了。此时 build 整个项目则会在 app/build/generated/source/apt/debug/com.lauzy.freedom.lauzycode/IOC 文件夹下生成一个IocActivity$$ViewBinder 的 Java 文件，代码如下：

```java
package com.lauzy.freedom.lauzycode.IOC;
import android.app.Activity;
import com.freedom.lauzy.inject.ContentInjector;

public class IocActivity$$ViewBinder<T extends IocActivity> implements ContentInjector<T> {
@Override
public void injectContent(final T source, Activity activity) {
    ((Activity) source).setContentView(2130968606);
}
}
```

可以看到，其实是在编译时生成了一个 Java 文件，并在 activity 的 onCreate 方法中调用了 setContentView 方法。

注意事项：

1、若不使用 com.google.auto.service:auto-service:1.0-rc2 这个 google 的生成源代码库，则需要手动创建一个 META_INF 文件来指定注解。
 需要在 compiler 中创建 compiler/src/main/resources/META-INF/services 目录(注意 META-INF 中间不是下划线，减号即可)，并新建 txt 文件 javax.annotation.processing.Processor，
 文件中写入自定义 AbstractProcessor 的全名称，如： com.lauzy.ContentViewInjectProcessor 。多个的话换行写入。

如下图所示：



![img](https:////upload-images.jianshu.io/upload_images/2630285-03ecce9a6a33cf1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/459/format/webp)

META_INF

2、本文使用 annotationProcessor 的注解处理器代替 [android-apt](https://link.jianshu.com?t=https://bitbucket.org/hvisser/android-apt) ，Google 内置的注解处理器，建议使用。

### 分析

编译时注解的优点 ：在于对性能影响很小的情况下，大量简化程序员的代码，像 butterknife 在首次查找类的时候对性能稍有影响，其他情况下影响微乎其微。
 编译时注解的缺点 ：build 过程生成更多的代码，增加了类和方法的数量；对性能影响很小，但是多少会有的。

本人认为编译时注解在优化代码，提高效率方面是有很大优势的，远远大于其缺点。

作者：PaleRider

链接：https://www.jianshu.com/p/428b5599c635

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。