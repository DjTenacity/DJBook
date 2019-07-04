##  JDK8的新特性——Lambda表达式的奇淫技巧

在JDK8之前，Java是不支持函数式编程的，所谓的函数编程，即可理解是**将一个函数（也称为“行为”）作为一个参数进行传递**。

通常我们提及得更多的是面向对象编程，**面向对象编程是对数据的抽象（各种各样的POJO类），而函数式编程则是对行为的抽象（将行为作为一个参数进行传递）**。在JavaScript中这是很常见的一个语法特性，但在Java中将一个函数作为参数传递这却行不通，好在JDK8的出现打破了Java的这一限制。

###  认识Lambda表达式 

首先来引入一个示例，不知给是否有在IDEA编写代码的经历，如果在JDK8的环境下如下所示按照Java传统的语法规则编写一个线程。

```java
new Thread(new Runnable() {
	@Override
	public void run() {
		System.out.println("sdsdsdsd");
	}
})
```

```java
new Thread(() -> System.out.println("sdsdsdsd"))
```

传统的语法规则，我们是将一个匿名内部类作为参数进行传递，我们实现了Runnable接口，并将其作为参数传递给Thread类，这实际上我们传递的是一段代码，也即我们将代码作为了数据进行传递，这就带来许多不必要的“样板代码”。

**Lambda表达式一共有三部分组成：**

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq29icp4gl5eDWnk6v5h2Q3reS0iaFVDkPoTGplW2IUJialMuHLTXUKwViaZibiaymEsDdOEcO61ibfvpM7WQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**能够接收Lambda表达式的参数类型，是一个只包含一个方法的接口。**只包含一个方法的接口称之为“**函数接口**”。

例如上面创建一个线程的示例，Runnable接口只包含一个方法，所以它被称为“函数接口”，所以它可以使用Lambad表达式来代替匿名内部类。

```
    public interface OnItemClickListener {
        void onSureItemClick();
    }

```

```
this.setListener(new OnItemClickListener() {
            @Override
            public void onSureItemClick() {
                
            }
        });
//  用Lambda表达式作为参数传递。
this.setListener(() -> {  });
```

### 有参数情况下

```java
    interface FunctionInterface {
        void test(int param);
    }

    class FunctionInterfaceTest {

        public void testLambda(){
            func(new FunctionInterface(){
                @Override
                public void test(int param) {

                }
            });
        }
        //Lambda
        public void testLambda(){
            func(param -> {

            });
        }
        private void func(FunctionInterface interface) {
            interface.test(1);
        }
    }
```

左边传递的是参数，此处并没有指明参数类型，因为它可以通过上下文进行类型推导，但在有些情况下不能推导出参数类型（在编译时不能推导通常IDE会提示），此时则需要指明参数类型。**我个人建议，任何情况下指明函数的参数类型**。

哪种情况不能推导出参数类型呢？就是函数接口是一个泛型的时候。

```java
    interface FunctionInterface<T> {
        void test(T param);
    }

    class FunctionInterfaceTest {

        public void testLambda(){
            func((Integer param) -> {

            });
        }

        private void func(FunctionInterface<Integer> functionInterface) {
            functionInterface.test(1);
        }
    }
```

###  有参数，有返回值 

```java
    interface FunctionInterface<T> {
        boolean test(T param);
    }

    class FunctionInterfaceTest {
//     	public void testLambda() {
//            func((Integer param) -> 1 == 2);
//        }
        public void testLambda() {
            func((Integer param) -> {
                return 1 == 2;
            });
        }

        private void func(FunctionInterface<Integer> functionInterface) {
            functionInterface.test(1);
        }
    }
```

###  Lambda表达式基本的语法规则： 

无参数，无返回值，() -> System.out.println(“Hello World”)；

有参数，无返回值，(x) -> System.out.println(“Hello World” + x)；

有参数，有返回值，(x, y) -> x + y。

​		这三种基本情况已经大致清楚了，特别是需要弄清，什么时候可以使用Lambda表达式代替匿名内部类，也就是Lambda表达式的应用场景是函数接口。

### 更大的好处

​		Lambda表达式这一新特性在JDK8中的引入，更大的好处则是集合API的更新，新增的Stream类库，使得我们在遍历使用集合时不再像以往那样不断地使用for循环。

```java
int count =0;
for (OrderData d : mAdapter.getList()) {
                if(d.is_click) {
                   count++;
                    return;
       }
}
//*****************JDK8使用集合****Stream类库******************
mAdapter.getList().stream().filter((data->data.is_click)).count();

```

JDK8通过流的方式则可以望文生义且代码量大大减小。

其中最为重要的是——Stream流。Stream的是通过函数式编程方式实现的在集合类上进行复杂操作的工具。若要详细讲解Stream的实现方式我相信再写一篇博客也不为过，所以此处不再考查Stream的内部实现。这里是想告诉大家，如果有幸使用JDK8的开发环境进行开发，尽量学习使用新的集合操作API。