## 浅谈final、finally、finalize有什么不同？对Java的理解

​		finally 则是 Java 保证重点代码一定要被执行的一种机制。我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。



​       finalize 是基础类 java.lang.Object 的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。它是不推荐使用的，业界实践一再证明它不是个好的办法. 因为无法保证 finalize 什么时候执行，执行的是否符合预期。使用不当会影响性能，导致程序死锁、挂起等。

​		ry-with-resources 或者 try-finally 机制，是非常好的回收资源的办法。如果确实需要额外处理，可以考虑 Java 提供的 Cleaner 机制或者其他替代方法。

# final

> **1.final的简介**

​        final可以修饰**变量，方法和类**，用于表示所修饰的内容一旦赋值之后就不会再被改变，比如String类就是一个final类型的类。即使能够知道final具体的使用方法，我想对**final在多线程中存在的重排序问题**也很容易忽略，希望能够一起做下探讨。

> **2.final的具体使用场景**        

​        final能够修饰变量，方法和类，也就是final使用范围基本涵盖了java每个地方，下面就分别以锁修饰的位置：变量，方法和类分别来说一说。

​        **2.1 变量**

​        在java中变量，可以分为成员变量以及方法局部变量。因此也是按照这种方式依次来说，以避免漏掉任何一个死角。

​            2.1.1 final成员变量

​            通常每个类中的成员变量可以分为类变量（static修饰的变量）以及实例变量。针对这两种类型的变量赋初值的时机是不同的，类变量可以在声明变量的时候直接赋初值或者在静态代码块中给类变量赋初值。而实例变量可以在声明变量的时候给实例变量赋初值，在非静态初始化块中以及构造器中赋初值。类变量有两个时机赋初值，而实例变量则可以有三个时机赋初值。当final变量未初始化时系统不会进行隐式初始化，会出现报错。这样说起来还是比较抽象，下面用具体的代码来演示。



![img](https:////upload-images.jianshu.io/upload_images/16971327-d8aa174904610495.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/336/format/webp)



​            看上面的图片已经将每种情况整理出来了，这里用截图的方式也是觉得在IDE出现红色出错的标记更能清晰的说明情况。现在我们来将这几种情况归纳整理一下：

​            **类变量**：必须要在**静态初始化块**中指定初始值或者**声明该类变量时**指定初始值，而且只能在这两个地方之一进行指定；

​            **实例变量**：必要要在**非静态初始化块**，**声明该实例变量**或者在**构造器中指定初始值**，而且只能在这三个地方进行指定。

​            2.2.2 final局部变量

​            final局部变量由程序员进行显式初始化，如果final局部变量已经进行了初始化则后面就不能再次进行更改，如果final变量未进行初始化，可以进行赋值，**当且仅有一次**赋值，一旦赋值之后再次赋值就会出错。下面用具体的代码演示final局部变量的情况。





![img](https:////upload-images.jianshu.io/upload_images/16971327-2ac30a03a73765ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/301/format/webp)



​            现在我们来换一个角度进行考虑，final修饰的是基本数据类型和引用类型有区别吗？

​            **final基本数据类型 VS final引用数据类型**

​            通过上面的例子我们已经看出来，如果final修饰的是一个基本数据类型的数据，一旦赋值后就不能再次更改，那么，如果final是引用数据类型了？这个引用的对象能够改变吗？我们同样来看一段代码。



![img](https:////upload-images.jianshu.io/upload_images/16971327-9af5327d44eb2fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/544/format/webp)



​            当我们对final修饰的引用数据类型变量person的属性改成22，是可以成功操作的。通过这个实验我们就可以看出来**当final修饰基本数据类型变量时，不能对基本数据类型变量重新赋值，因此基本数据类型变量不能被改变。而对于引用类型变量而言，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的地址不会发生改变，即一直引用这个对象，但这个对象属性是可以改变的。**

> ​            **宏变量**

​            利用final变量的不可更改性，在满足一下三个条件时，该变量就会成为一个“宏变量”，即是一个常量。

​            使用final修饰符修饰；

​            1.在定义该final变量时就指定了初始值；

​            2.该初始值在编译时就能够唯一指定。

​            3.注意：当程序中其他地方使用该宏变量的地方，编译器会直接替换成该变量的值

​    **2.2 方法**

> ​        **重写？**

​            当父类的方法被final修饰的时候，子类不能重写父类的该方法，比如在Object中，getClass()方法就是final的，我们就不能重写该方法，但是hashCode()方法就不是被final所修饰的，我们就可以重写hashCode()方法。我们还是来写一个例子来加深一下理解： 

​                先定义一个父类，里面有final修饰的方法test();



![img](https:////upload-images.jianshu.io/upload_images/16971327-781dfed704e3d166.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/222/format/webp)



​            然后FinalExample继承该父类，当重写test()方法时出现报错，如下图：



![img](https:////upload-images.jianshu.io/upload_images/16971327-98e978ecb455ff77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/301/format/webp)



​            通过这个现象我们就可以看出来**被final修饰的方法不能够被子类所重写**。

> ​        **重载！！！**



![img](https:////upload-images.jianshu.io/upload_images/16971327-606a31b54ac02847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/357/format/webp)



​            可以看出被final修饰的方法是可以重载的。经过我们的分析可以得出如下结论：

​            **1. 父类的final方法是不能够被子类重写的**

​            **2. final方法是可以被重载的**

​    **2.3 类**

​        **当一个类被final修饰时，表名该类是不能被子类继承的**。子类继承往往可以重写父类的方法和改变父类属性，会带来一定的安全隐患，因此，当一个类不希望被继承时就可以使用final修饰。还是来写一个小例子：



![img](https:////upload-images.jianshu.io/upload_images/16971327-6cf8533eefc1a257.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/441/format/webp)



> **3.final的例子**

​        final经常会被用作不变类上，利用final的不可更改性。我们先来看看什么是不变类。

> ​        **不变类**

​                **不变类的意思是创建该类的实例后，该实例的实例变量是不可改变的。满足以下条件则可以成为不可变类：**

​                1.使用private和final修饰符来修饰该类的成员变量

​                2.提供带参的构造器用于初始化类的成员变量；

​                3.仅为该类的成员变量提供getter方法，不提供setter方法，因为普通方法无法修改fina修饰的成员变量；

​                4.如果有必要就重写Object类 的hashCode()和equals()方法，应该保证用equals()判断相同的两个对象其Hashcode值也是相等的。

​            JDK中提供的八个包装类和String类都是不可变类，我们来看看String的实现。



![img](https:////upload-images.jianshu.io/upload_images/16971327-cdfd26123bc09c64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/407/format/webp)



​            可以看出String的value就是final修饰的，上述其他几条性质也是吻合的。

> \4. 多线程中你真的了解final吗？

​            上面我们聊的final使用，应该属于Java基础层面的，当理解这些后我们就真的算是掌握了final吗？有考虑过final在多线程并发的情况吗？在java内存模型中我们知道java内存模型为了能让处理器和编译器底层发挥他们的最大优势，对底层的约束就很少，也就是说针对底层来说java内存模型就是一弱内存数据模型。同时，处理器和编译为了性能优化会对指令序列有编译器和处理器重排序。那么，在多线程情况下,final会进行怎样的重排序？会导致线程安全的问题吗？下面，就来看看final的重排序。 

​        **4.1 final域重排序规则**

​            **4.1.1 final域为基本类型**

作者：hanlyn

链接：https://www.jianshu.com/p/196a4aa7f029

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。





## 对Java的理解

Java本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“一次书写，到处运行”（Write once,run anywhere），能够非常容易地获得跨平台能力；另外就是垃圾收集（GC,Garbage Collection），Java通过垃圾收集器（Garbage collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。

​    我们日常接触到JRE（Java Runtime Environment）或者JDK（Java Development Kit）。JRE：也就是Java运行时环境，包含了JVM和Java类库，以及一些模块等。而JDK可以看作是JRE的一个超集（父类），提供了更多工具，比如编译器、各种诊断工具等。

​    对于“Java是解释执行”这句话，这个说法不太正确。我们开发的Java源代码，首先通过Javac编译成字节码（bytecode/class文件），然后，在运行时，通过Java虚拟机（JVM）内嵌的解释器将字节码转换成最终的机器码。但是常见的JVM，比如我们太多数情况使用的Oracle JDK提供的Hotspot JVM，都提供了JIT（Just-In-time）编译器，也就是通常所说的动态编译器，JIT能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行。

​     "一次编译、到处运行”说的是Java语言跨平台的 特性，Java的跨平台特性与Java虚拟机的存在密 不可分，可在不同的环境中运行。比如说 Windows平台和Linux平台都有相应的JDK，安装好JDK后也就有了 Java语言的运行环境。其实 Java语言本身与其他的编程语言没有特别大的差异，并不是说Java语言可以跨平台，而是在不同 的平台都有可以让Java语言运行的环境而已，所以才有了Java—次编译，到处运行这样的效果。严格的讲，跨平台的语言不止Java—种，但 Java是较为成熟的一种。次编译，到处运 行”这种效果跟编译器有关。编程语言的处理需要 编译器和解释器。Java虚拟机和DOS类似，相当 于一个供程序运行的平台。程序从源代码到运行的三个阶段：编码一一编译一一运行一一调试。Java在编译阶段则体现了跨平台的特点。编译过程大概是这样的：首先是将 Java源代码转化成.CLASS文件字节码，这是第一次编译。.class文件就是可以到处运行的文件。然后Java字节码会被转化为目标机器代码，这是是由JVM来执行的，即Java的第二次编译。"到处运行”的关键和前提就是JVM。因为在 第二次编译中JVM起着关键作用。在可以运行 Java虚拟机的地方都内含着一个JVM操作系统。 从而使JAVA提供了各种不同平台上的虚拟机制， 因此实现了 "到处运行”的效果。需要强调的一点 是，java并不是编译机制，而是解释机制。Java 字节码的设计充分考虑了 JIT这一即时编译方式， 可以将字节码直接转化成高性能的本地机器码， 这同样是虚拟机的一个构成部分。

​    Java特性：面向对象（封装，继承，多态）平台无关性（JVM运行.class文件）语言（泛型，Lambda)类库（集合，并发，网络，IO/NIO)JRE (Java运行环境，JVM，类库）JDK (Java开发工具，包括JRE，javac，诊断工具）Java是解析运行吗？不正确！1，Java源代码经过Javac编译成.class文件2，.class文件经JVM解析或编译运行。  (1) 解析:.class文件经过JVM内嵌的解析器解析执行。  (2) 编译:存在JIT编译器（Just In TimeCompile即时编译器）把经常运行的代码作为"热点代码"编译与本地平台相关的机器码，并进行各种层次的优化。  (3) AOT编译器:Java 9提供的直接将所有代码编译成机器码执行。 