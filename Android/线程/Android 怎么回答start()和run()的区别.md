# Android 怎么回答start()和run()的区别

​		首先你要明确这种问题是什么人问的。是个搞纯java开发的人问的还是搞android开发的人。

 **搞纯java的人心里是这样想的 **

```
run()方法：可以重复多次调用（只是thread的一个普通方法，在主线程里执行）；
需要并行处理的代码放在run方法中，start方法启动线程后自动调用run方法；

start()方法:启动一个线程。 让一个线程进入就绪队列等待分配cpu，分到cpu后才调用实现的run()方法。
```



​		线程的run()方法是由java虚拟机直接调用的，**如果我们没有启动线程（没有调用线程的start()方法）而是在应用代码中直接调用run()方法，那么这个线程的run()方法其实运行在当前线程（**即run()方法的调用方所在的线程）之中，而不是运行在其自身的线程中，从而违背了创建线程的初衷；
