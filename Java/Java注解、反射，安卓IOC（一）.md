# Java注解、反射，安卓IOC（一） 

## Java 注解 (Annotation)

Java 注解，指的是代码里边的特殊标记，可以在编译、运行时被读取，并执行相应的处理。Annotation 可用于修饰包、类、构造器、方法、变量等。

### Annotation 类型

此处来一张网上的图 (可在新标签页中放大查看)

![](https://upload-images.jianshu.io/upload_images/2630285-67130c84cb3fc769.jpg)

转自[深入理解Java：注解(Annotation)](https://link.jianshu.com?t=http://www.cnblogs.com/peida/archive/2013/04/26/3038503.html)

#### 基本 Annotation

Java中5个基本的注解分别为：

- @Override  ————  用来限定子类重写父类的方法。
- @Deprecated  ————  标记已经过时的方法。
- @SuppressWarnings  ————  抑制编译器的警告。
- @SafeVarargs  ————  Java7 抑制“堆污染”警告，可变参数与泛型类一起使用会出现类型安全警告，若开发人员确信不会出现问题，可用此注解进行声明。
- @FunctionalInterface  ————  Java8 函数式接口，检测指定某个接口中只有一个抽象方法。

#### 元 Annotation

元Annotation是用来修饰其他注解定义，即注解其他注解。
 Java中有6种元注解，其中@Native注解一般用于给IDE工具做提示用。下边具体介绍其他几种注解。

1、@Retention：指定本修饰的注解可以保留多长时间。包含了一个RetentionPolicy类的value值，所以需指定该value的值。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```

- RetentionPolicy.CLASS  ————  默认值，编译器将注解记录在字节码文件中，程序运行时，JVM不保留注解信息。
- RetentionPolicy.RUNTIME  ————  编译器将注解记录在字节码文件中，程序运行时，JVM可以获得注解信息，可通过反射获取该注解的信息。
- RetentionPolicy.SOURCE  ————  注解只保留在源代码中，编译器直接丢弃。

那怎么来选择合适的注解生命周期呢？

首先要明确**生命周期长度 SOURCE < CLASS < RUNTIME** ，所以前者能作用的地方后者一定也能作用。一般如果**需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解**；如果要在**编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解**；如果只是**做一些检查性的操作**，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。

```java
/**
 ``* 获取指定类型的注解
 ``*/
public` `<A extends Annotation> A getAnnotation(Class<A> annotationType);
 
/**
 ``* 获取所有注解，如果有的话
 ``*/
public` `Annotation[] getAnnotations();
 
/**
 ``* 获取所有注解，忽略继承的注解
 ``*/
public` `Annotation[] getDeclaredAnnotations();
 
/**
 ``* 指定注解是否存在该元素上，如果有则返回true，否则false
 ``*/
public` `boolean isAnnotationPresent(Class<? extends Annotation> annotationType);
 
/**
 ``* 获取Method中参数的所有注解
 ``*/
public` `Annotation[][] getParameterAnnotations();
```

2、@Target：指定被修饰的注解能用于哪些程序元素。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

- ElementType.ANNOTATION_TYPE: 修饰Annotation。
- ElementType.TYPE: 修饰类、接口（包括注解类型）、枚举。
- ElementType.FIELD: 修饰成员变量 , 也包括enum常量。
- ElementType.METHOD: 修饰方法定义。
- ElementType.PARAMETER: 修饰参数定义。
- ElementType.CONSTRUCTOR: 修饰构造方法。
- ElementType.LOCAL_VARIABLE: 修饰局部变量。
- ElementType.PACKAGE: 修饰包定义。

在Java8中新增了两个ElementType参数，用来限定哪些类型可以标注

- ElementType.TYPE_PARAMETER:  类型变量
- ElementType.TYPE_USE:  使用类型的任何语句

TYPE_PARAMETER举例，若要对泛型进行标注，则定义注解时需设定Target为TYPE_PARAMETER：

```java
@Target(ElementType.TYPE_PARAMETER)
public @interface Animal{}

public class Zoo<@Animal T>{
    ...
}
```

TYPE_USE可用于标注各种使用到类型的地方，举例如下(上述例子可以将TYPE_PARAMETER改为TYPE_USE)：

```java
定义：
@Target(ElementType.TYPE_USE)
public interface UseTest{}

使用：
@UseTest String content; 修饰类型，
此种写法相当于java.lang.@UseTest String content; 
若@UseTest java.lang.String content; 此为定义局部变量，写法不合法，UseTest需指定Target为LOCAL_VARIABLE。

String content = (@UseTest String) obj; //类型转换
List<@UseTest String> infos = new ArrayList<>();  //泛型
implements @UseTest XXXX;  //实现接口
throws @UseTest NullPointException;  //声明抛出异常
```

3、@Documented：被该注解修饰的注解会被javadoc工具提取成文档。如果一个注解由@Documented修饰，则使用该注解的程序api文档中会包含该注解的说明。

4、@Inherited: 此注解修饰的注解具有继承性。若@XXX被@Inherited修饰，则使用@XXX注解的类具有继承性，其子类自动被@XXX修饰。

5、@Repeatable：重复注解，Java8的新特性。

在Java8之前，重复注解的解决方案代码如下：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Student{
    String name();
}

定义一个容器注解：
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Students{
    Student[] value();
}

使用：
@Students({@Student(name = "Jack"), @Student(name = "Will")})
public class StudentTest{
    ......
}
```

在Java8中的方案则如下：

```java
//定义如上的容器注解Students，添加Repeatable注解，如下所示
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Repeatable(Students.class)
public @interface Student{
    String name();
}

使用：
@Student(name = "Jack")
@Student(name = "Will")
public class StudentTest{
    ......
}
```

## Java 反射简介

通过Java反射可以获取对象的属性、方法等。

1、获取类

```java
//第一种方式
Class stuClazz1 = Class.forName("com.lauzy.freedom.ReflectDemo.Student");
    
//第二种方式
Class stuClazz2 = Student.class;
    
//第三种方式
Student stu3 = new Student();
Class stuClazz3 = stu3.getClass();
```

2、创建对象

```java
Class stuClazz2 = Student.class;
Object stu = stuClazz2.newInstance();
    
```

3、获取属性例子

```java
Object stu = stuClazz2.newInstance();   //获取实例
Field age = stuClazz2.getDeclaredField("age");  //获取特定属性
age.setAccessible(true);    //打破封装性
age.set(stu, 25);   //设置属性
```

4、方法总结

- getDeclaredFields(): 获取所有属性。
- getDeclaredField("***"): 获取特定的属性。
- getModifiers(): 获取属性或方法的修饰符。
- getType(): 获取属性或方法的类型名。
- getDeclaredMethods()：获取所有方法。
- getReturnType()：获取方法的返回类型。
- getParameterTypes()：获取方法的参数类型。
- getDeclaredMethod("***",参数类型.class,……): 获取特定的方法。
- getDeclaredConstructors(): 获取所有的构造方法。
- getDeclaredConstructor(参数类型.class,……): 获取特定的构造方法。
- getSuperclass()：获取继承的父类。
- getInterfaces()：获取实现的所有接口。
- field.set(Object object, Object value);//设置object对象的value属性
- method.invoke(Object object, Object... values); //调用方法，values为方法的参数

5、代码实例

```java
try {
    Class stuClazz1 = Class.forName("com.lauzy.freedom.ReflectDemo.Student");
    Class stuClazz2 = Student.class;
    Student stu3 = new Student();
    Class stuClazz3 = stu3.getClass();
    for (Field field : stuClazz1.getDeclaredFields()) {
        System.out.println(Modifier.toString(field.getModifiers())  //获取属性修饰符
                + "-" + field.getType().getSimpleName()     //获取属性类型名
                + "-" + field.getName());  //获取属性名
    }
    System.out.println("--------");
    for (Method method : stuClazz2.getDeclaredMethods()) {
        System.out.println(Modifier.toString(method.getModifiers())  //获取方法修饰符
                + "-" + method.getReturnType().toString()   //方法返回类型名
                + "-" + method.getName());  //方法名
    }
    System.out.println("--------");
    System.out.println(stuClazz2.getDeclaredConstructor(String.class, String.class, int.class).toString());
    System.out.println("--------");
    System.out.println(stuClazz2.getSuperclass().getName().toString());
    System.out.println("--------");
    for (Class aClass : stuClazz2.getInterfaces()) {
        System.out.println(aClass.getName());
    }
    System.out.println("--------");
    Object stu = stuClazz2.newInstance();   //获取实例
    Field name = stuClazz2.getDeclaredField("name");  //获取特定属性
    name.setAccessible(true);    //打破封装性
    name.set(stu, "Jack");   //设置属性
    System.out.println(name.get(stu));

    Method profile = stuClazz2.getDeclaredMethod("getProfile", String.class, int.class);//特定方法
    profile.setAccessible(true);
    profile.invoke(stu, "male", 30);//调用方法
} catch (Exception e) {
    e.printStackTrace();
}
```

输出结果：

```java
private-String-name
public-String-gender
private-int-age
--------
public-class java.lang.String-getName
public-void-setName
private-void-getProfile
public-int-getAge
public-void-setAge
--------
public com.lauzy.freedom.ReflectDemo.Student(java.lang.String,java.lang.String,int)
--------
com.lauzy.freedom.AnnotationDemo.Person
--------
java.io.Serializable
--------
Jack
Name : Jack ; Gender : male ; Age : 30
```

## 自定义注解、反射获取属性

分别定义Name、Gender和SaveMoney注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface Name {
    String value() default "Will";
}
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Gender {
    String value() default "";
}
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface SaveMoney {

    int money() default 10000;

    int term() default 1;

    String platform() default "XXX";
}
```

注解的使用：

```java
public class Person {
    @Name(value = "Jack")
    @Gender(value = "man")
    public String name;

    @SaveMoney(money = 20000, term = 5, platform = "ChinaBank")
    public void saveMoney(int money) {
        System.out.println("and then he spent " + money  + " on clothes.");
    }
}
```

利用反射获取注解的属性和方法：

```java
public class AnnUtils {
    public static void test(Class<?> clazz) {

        for (Field field : clazz.getFields()) {
            if (field.isAnnotationPresent(Name.class) && field.isAnnotationPresent(Gender.class)) {
                Name name = field.getAnnotation(Name.class);
                Gender gender = field.getAnnotation(Gender.class);
                System.out.print("A " + gender.value() + " called " + name.value());
            }
        }

        try {
            Class<Person> personClass = Person.class;
            Method[] methods = personClass.getMethods();
            for (Method method : methods) {
                if (method.isAnnotationPresent(SaveMoney.class)) {
                    SaveMoney saveMoney = method.getAnnotation(SaveMoney.class);
                    System.out.print(" deposited " + saveMoney.money() + "RMB to " +
                            saveMoney.platform() + " for " + saveMoney.term() + " months, ");

                    method.invoke(personClass.newInstance(), 1000);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行

```java
AnnUtils.test(Person.class);
```

此时的输出结果为：

```
A man named Jack deposited 20000RMB to ChinaBank for 5 months, and then he spent 1000 on clothes.
```

此篇博客为基础用法及实例，下一篇[Java注解、反射，安卓IOC（二）](https://www.jianshu.com/p/428b5599c635)会介绍注解和反射在Android中的使用，运行时注解及butterknife的简单实现。

作者：PaleRider

链接：https://www.jianshu.com/p/504ea4902602

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。