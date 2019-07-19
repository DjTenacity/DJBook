# [java中线程创建的四种方式](https://blog.csdn.net/m0_37840000/article/details/79756932)

java中创建线程的四种方法以及区别
Java使用Thread类代表线程，所有的线程对象都必须是Thread类或其子类的实例。Java可以用四种方式来创建线程，如下所示：

1）继承Thread类创建线程

2）实现Runnable接口创建线程

3）使用Callable和Future创建线程

4）使用线程池例如用Executor框架

#####  
###  继承Thread类创建线程 

通过继承Thread类来创建并启动多线程的一般步骤如下

1】定义Thread类的子类，并重写该类的run()方法，该方法的方法体就是线程需要完成的任务，run()方法也称为线程执行体。

2】创建Thread子类的实例，也就是创建了线程对象

3】启动线程，即调用线程的start()方法  

```java
public class MyThread extends Thread{//继承Thread类 
　　public void run(){ 
　　//重写run方法 
　　} 
} 
public class Main { 
　　public static void main(String[] args){ 
　　　　new MyThread().start();//创建并启动线程 
　　} 
} 
```

###  实现Runnable接口创建线程 

通过实现Runnable接口创建并启动线程一般步骤如下：

1】定义Runnable接口的实现类，一样要重写run()方法，这个run（）方法和Thread中的run()方法一样是线程的执行体

2】创建Runnable实现类的实例，并用这个实例作为Thread的target来创建Thread对象，这个Thread对象才是真正的线程对象

3】第三部依然是通过调用线程对象的start()方法来启动线程 

```java
public class MyThread2 implements Runnable {//实现Runnable接口
　　public void run(){
　　//重写run方法
　　}
}

public class Main {
　　public static void main(String[] args){
　　　　//创建并启动线程
　　　　MyThread2 myThread=new MyThread2();
　　　　Thread thread=new Thread(myThread);
　　　　thread().start();
　　　　//或者    new Thread(new MyThread2()).start();
　　}
}
```

### 使用Callable和Future创建线程



和Runnable接口不一样，Callable接口提供了一个call（）方法作为线程执行体，call()方法比run()方法功能要强大。
+ call()方法可以有返回值
+ call()方法可以声明抛出异常

Java5提供了Future接口来代表Callable接口里call()方法的返回值，并且为Future接口提供了一个实现类FutureTask，这个实现类既实现了Future接口，还实现了Runnable接口，因此可以作为Thread类的target。在Future接口里定义了几个公共方法来控制它关联的Callable任务。

>boolean cancel(boolean mayInterruptIfRunning)：视图取消该Future里面关联的Callable任务

>V get()：返回Callable里call（）方法的返回值，调用这个方法会导致程序阻塞，必须等到子线程结束后才会得到返回值

>V get(long timeout,TimeUnit unit)：返回Callable里call（）方法的返回值，最多阻塞timeout时间，经过指定时间没有返回抛出TimeoutException

>boolean isDone()：若Callable任务完成，返回True

>boolean isCancelled()：如果在Callable任务正常完成前被取消，返回True

介绍了相关的概念之后，创建并启动有返回值的线程的步骤如下：

1】创建Callable接口的实现类，并实现call()方法，然后创建该实现类的实例（从java8开始可以直接使用Lambda表达式创建Callable对象）。
2】使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值 
3】使用FutureTask对象作为Thread对象的target创建并启动线程（因为FutureTask实现了Runnable接口） 
4】调用FutureTask对象的get()方法来获得子线程执行结束后的返回值  

```java
public class Main { 
　　public static void main(String[] args){ 
　　　MyThread3 th=new MyThread3(); 
　　　 //使用Lambda表达式创建Callable对象 
　　   //使用FutureTask类来包装Callable对象 
　　　FutureTask<Integer> future=new FutureTask<Integer>( 
　　　　(Callable<Integer>)()->{ 
　　　　　　return 5; 
　　　　} 
　　  );
	//实质上还是以Callable对象来创建并启动线程
　　　new Thread(task,"有返回值的线程").start();
　　  try{ 
　　　　System.out.println("子线程的返回值："+future.get());//get()方法会阻塞，直到子线程执行结束才返回 
 　　 }catch(Exception e){ 
　　　　ex.printStackTrace(); 
　　　} 
　　} 
} 
```

### 使用线程池例如用Executor框架

**引入的Executor框架的最大优点是把任务的提交和执行解耦。**

要执行任务的人只需把Task描述清楚，然后提交即可。这个Task是怎么被执行的，被谁执行的，什么时候执行的，提交的人就不用关心了。具体点讲，提交一个Callable对象给ExecutorService（如最常用的线程池ThreadPoolExecutor），将得到一个Future对象，调用Future对象的get方法等待执行结果就好了。

Executor框架的内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。

​     Executor框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。  

+ Executor接口中之定义了一个方法execute（Runnable command），该方法接收一个Runable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类。

+ ExecutorService接口继承自Executor接口，它提供了更丰富的实现多线程的方法，比如，ExecutorService提供了关闭自己的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 

​     可以调用ExecutorService的shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致ExecutorService停止接受任何新的任务且等待已经提交的任务执行完成(已经提交的任务会分两类：

​      一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭ExecutorService。因此我们一般用该接口来实现和管理多线程。

​      ExecutorService的生命周期包括三种状态：运行、关闭、终止。

​      创建后便进入运行状态，当调用了shutdown（）方法时，便进入关闭状态，此时意味着ExecutorService不再接受新的任务，但它还在执行已经提交了的任务，当素有已经提交了的任务执行完后，便到达终止状态。如果不调用shutdown（）方法，ExecutorService会一直处在运行状态，不断接收新的任务，执行新的任务，服务器端一般不需要关闭它，保持一直运行即可。

​      Executors提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口。

##  **剩下的看链接**














runOnUiThread(new Runnable() {
​			@Override
​			public void run() {
​				get();
​			}
​		});