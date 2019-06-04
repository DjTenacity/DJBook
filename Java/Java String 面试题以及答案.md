# [Java String 面试题以及答案](https://www.cnblogs.com/dudadi/p/8025038.html)

String是最常使用的Java类之一，整理的了一些重要的String知识分享给大家。

作为一个Java新手程序员，对String进行更深入的了解很有必要。如果你是有几年Java开发经验，可以根据目录选择性的阅读以下内容。

#### 1、什么是String，它是什么数据类型？

String是定义在 java.lang 包下的一个类。它不是基本数据类型。

String是不可变的，JVM使用字符串池来存储所有的字符串对象。

#### 2、创建String对象的不同方式有哪些？

- #### 和使用其他类一样通过new关键字来创建。

  使用这种方式时，JVM创建字符串对象但不存储于字符串池。我们可以调用intern()方法将该字符串对象存储在字符串池，如果字符串池已经有了同样值的字符串，则返回引用。

- #### 使用双引号直接创建。

```java
使用这种方式时，JVM去字符串池找有没有值相等字符串，如果有，则返回找到的字符串引用。否则创建一个新的字符串对象并存储在字符串池。
String str = new String("abc");
String str1 = "abc";
```

#### 3、写一个方法来判断一个String是否是回文（顺读和倒读都一样的词）？

回文就是正反都一样的词，如果需要判断是否是回文，只需要比较正反是否相等即可。String类并没有提供反转方法供我们使用，但StringBuffer和StringBuilder有reverse方法。

```java
private static boolean isPalindrome(String str) {
        if (str == null)
            return false;
        StringBuilder strBuilder = new StringBuilder(str);
        strBuilder.reverse();
        return strBuilder.toString().equals(str);
    }
```

假设面试官让你不使用任何其他类来实现的话，我们只需要首尾一一对比就知道是不是回文了。

```java
private static boolean isPalindromeString(String str) {
        if (str == null)
            return false;
        int length = str.length();
        System.out.println(length / 2);
        for (int i = 0; i < length / 2; i++) {

            if (str.charAt(i) != str.charAt(length - i - 1))
                return false;
        }
        return true;
    }
```

#### 4、如何让一个字符串变成小写或大写形式？

使用toUpperCase 和 toLowerCase 方法让一个字符串变为 大写或小写。

#### 5、如何比较两个字符串？

String内部实现了Comparable接口，有两个比较方法：compareTo(String anotherString) 和compareToIgnoreCase(String str)。

- #### compareTo(String anotherString)

  与传入的anotherString字符串进行比较，如果小于传入的字符串返回负数，如果大于则返回证书。当两个字符串值相等时，返回0.此时eqauls方法会返回true。

- #### equalsIgnoreCase(String str)

  该方法与compareTo方法类似，区别只是内部利用了Character.toUpperCase等方法进行了大小写转换后进行比较。

#### 6、如何将String转换为char,反过来呢？

这是一个误导题，String是一系列字符，所有我们没法转换成一个单一的char，但可以调用toCharArray() 方法将字符串转成字符数组。

```java
String str = "Java interview";
        
    //string to char array
    char[] chars = str.toCharArray();
    System.out.println(chars.length);
```

#### 7、如何将String转换为byte array,反过来呢？

使用String的getBytes()方法将String转成byte数组，使用String的构造方法 new String(byte[] arr) 将byte数据转为String。

```java
public class StringToByteArray {

    public static void main(String[] args) {
        String str = "PANKAJ";
        byte[] byteArr = str.getBytes();
        // print the byte[] elements
        System.out.println("String to byte array: " + Arrays.toString(byteArr));
    }
}
public class ByteArrayToString {

    public static void main(String[] args) {
        byte[] byteArray = { 'P', 'A', 'N', 'K', 'A', 'J' };
        byte[] byteArray1 = { 80, 65, 78, 75, 65, 74 };

        String str = new String(byteArray);
        String str1 = new String(byteArray1);

        System.out.println(str);
        System.out.println(str1);
    }
}
```

<div id="question8"></div>

#### 8、浅谈一下String, StringBuffer，StringBuilder的区别？

**String是不可变类，每当我们对String进行操作的时候，总是会创建新的字符串。**操作String很耗资源,所以Java提供了两个工具类来操作String - StringBuffer和StringBuilder。

StringBuffer和StringBuilder是可变类，**StringBuffer是线程安全的，StringBuilder则不是线程安全的。**所以在多线程对同一个字符串操作的时候，我们应该选择用StringBuffer。由于不需要处理多线程的情况，StringBuilder的效率比StringBuffer高。

#### 9、String是不可变的有什么好处？

String是不可变类有以下几个优点

- 由于String是不可变类，所以**在多线程中使用是安全的**，我们不需要做任何其他同步操作。
- String是不可变的，它的值也不能被改变，所以**用来存储数据密码很安全**。
- 因为java字符串是不可变的，**可以在java运行时节省大量java堆空间。因为不同的字符串变量可以引用池中的相同的字符串**。如果字符串是可变得话，任何一个变量的值改变，就会反射到其他变量，那字符串池也就没有任何意义了。

#### 10、如何分割一个String？

- #### public String[] split(String regex):

  根据传入的正则字符串进行分割，注意，如果最后一位刚好有传入的字符，返回数组最后一位不会有空字符串。

```
String s = "abcaada";
System.out.println(Arrays.toString(s.split("a")));

//以上代码输出为  [, bc, , d].
```

- #### public String[] split(String regex, int limit):

  限制分割结果数组中有几个字符串。**传入2,则结果分割后数组长度为2。**

```
String s = "Y,Kunming,Yunnan";
String[] data = s.split(",", 2);
System.out.println("Name = "+data[0]); //Y
System.out.println("Address = "+data[1]); //Kunming,Yunnan
```

实际上第一个方法调用了第二个方法，只不过不限制返回的数组长度了。

```
public String[] split(String regex) {
    return split(regex, 0);
}
```

#### 11、如何判断两个String是否相等？

有两种方式判断字符串是否相等，使用"=="或者使用equals方法。**当使用"=="操作符时，不仅比较字符串的值，还会比较引用的内存地址**。大多数情况下，我们只需要判断值是否相等，此时用equals方法比较即可。

**还有一个equalsIgnoreCase可以用来忽略大小写进行比较**。

```
String s1 = "abc";
        String s2 = "abc";
        String s3= new String("abc");
        System.out.println("s1 == s2 ? "+(s1==s2)); //true
        System.out.println("s1 == s3 ? "+(s1==s3)); //false
        System.out.println("s1 equals s3 ? "+(s1.equals(s3))); //true
```

#### 12、什么是字符串池？

顾名思义，字符串常量池就是**用来存储字符串的。它存在于Java 堆内存。**

下图解释了字符串池在java堆空间如何存在以及当我们使用不同方式创建字符串时的情况。

![img](https://img1.tuicool.com/YRzeMvv.png!web)

以下是上图的一个编程例子

```java
public class StringPool {

   
    public static void main(String[] args) {
        String s1 = "Cat";
        String s2 = "Cat";
        String s3 = new String("Cat");
        
        System.out.println("s1 == s2 :"+(s1==s2));
        System.out.println("s1 == s3 :"+(s1==s3));
    }

}
```

运行以上代码，输出如下：

```java
s1 == s2 :true
s1 == s3 :false
```

一些java题中，可能会问一段代码中有几个字符串被创建，例如：

```java
String str = new String("Cat");
```

**上面一行代码将会创建1或2个字符串。如果在字符串常量池中已经有一个字符串“Cat”，那么就只会创建一个“Cat”字符串。如果字符串常量池中没有“Cat”，那么首先会在字符串池中创建，然后才在堆内存中创建，这种情况就会创建2个对象了。**

#### 13、String的intern()方法

当intern()方法被调用，如果字符串池中含有一个字符串和当前调用方法的字符串eqauls相等，那么就会返回池中的字符串。如果池中没有的话，则首先将当前字符串加入到池中，然后返回引用。

#### 14、String是线程安全的吗？

String是不可变类，一旦创建了String对象，我们就无法改变它的值。因此，**它是线程安全的，可以安全地用于多线程环境中**。

#### 15、为什么我们在使用HashMap的时候总是用String做key？

因为字符串**是不可变的，当创建字符串时， 它的hashcode被缓存下来，不需要再次计算**。因为**HashMap内部实现是通过key的hashcode来确定value的存储位置，所以相比于其他对象更快**。这也是为什么我们平时都使用String作为HashMap对象。

#### 16、String编程题

#### 1、下面的代码输入什么

```java
String s1 = new String("abc");
String s2 = new String("abc");
System.out.println(s1 == s2);
```

输入false

#### 2、下面的代码输入什么

```java
String s1 = "abc";
StringBuffer s2 = new StringBuffer(s1);
System.out.println(s1.equals(s2));
```

输入false，因为**s2不是String类型**，String的equals方法进行了类型判断。

#### 3、下面的代码输入什么

```java
String s1 = "abc";
String s2 = new String("abc");
String s3 = "abc";
s2.intern();
System.out.println(s1 ==s2+""+s1.equals(s2));
System.out.println(s1 == s3 +""+s1.equals(s3));
```

输出false，intern()方法将返回从字符串池中的字符串对象的引用，但因为我们没有分配到S2，S2没有变化，如果该第三行代码为s2 =s2.intern()，则输出true。

equals() 方法比较的是字符串的内容~所以结果都是是 true 很好理解，至于 str1==str3  的结果也是 true ，是因为在 Java 的内存的**方法区中有一块区域叫做常量池**，str1 =“abc” 时，常量池中没有 “abc”，所以就 new 一个 “abc” 当运行 str3 = “abc” 时，常量池中存在 “abc” ，系统就会把 常量池中的 “abc” 的引用直接给 str3 所以 str1==str3 的结果为 true，因为它们的引用是一样的 

#### 4、下面的代码将创建几个字符串对象。

```java
String s1 = new String("Hello");  
String s2 = new String("Hello");
```

答案是3个对象.

第一，行1 字符串池中的“hello”对象。**常量池中的引用**

第二，行1，在堆内存中带有值“hello”的新字符串。堆中开辟一块空间，然后把引用赋值给 str1

第三，行2，在堆内存中带有“hello”的新字符串。这里“hello”字符串池中的字符串被重用。

#### 5、下面的代码输入什么

```java
String s1 = "abc";
String s2 = "a"+"b"+"c";
System.out.println(s1 ==s2+""+s1.equals(s2));
```

都是true 这个 str1==str2 为何为 true 小伙伴们知道吗？嘿嘿因为在Java中有一种叫做**常量优化的机制**，我们在赋值的时候 “a”，“b”，“c”都是常量，系统及直接把 abc 赋值给 str1 了，这时候常量池中也就存在 “abc” 了，所以str1==str2 

#### 5、**判断String类型的 s1 和 s2 是否相等**

```java
String s1 = "abc";
String s2 = "ab";
String s3 = s2+"c";
System.out.println(s3 ==s2+""+s3.equals(s2));
```

false true  『**Java 语言提供对字符串串联符号（"+"）以及将其他对象转换为字符串的特殊支持。字符串串联是通过 StringBuilder（或 StringBuffer）类及其 append 方法实现的。字符串转换是通过 toString 方法实现的...』**

也就是说当执行 str3=str1+c 的时候，首先在堆中生成一个StringBuilder（或StringBuffer）对象，然后把 ab 和 c 连接在一起 ，再利用 toString 方法生成一个 “abc”的字符串 再来进行比较..str2 的 “abc” 在常量池中，str3 在堆中所以为false~

#### 6

需求：

给三次机会，并且提示还有几次

分析：

1）需要键盘输入账户名和密码

2）需要进行循环判断

```java
public class Test1 {

 public static void main(String[] args) {
   Scanner in = new Scanner(System.in);
   for(int i=0; i<3; i++){
     System.out.println("请输入用户名");
     String userName = in.nextLine();
     System.out.println("请输入密码");
     String password = in.nextLine();
     if ("admin".equals(userName)&&"admin".equals(password)) {
       System.out.println("欢迎"+userName+"!!!");
       break;
     }else{
       System.out.println("用户名或密码错误~~");
       System.out.println("还有"+(2-i)+"次机会");
     }
   }  
 }
}  
```

 