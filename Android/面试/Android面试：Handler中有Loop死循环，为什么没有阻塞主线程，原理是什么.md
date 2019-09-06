## [腾讯Android面试：Handler中有Loop死循环，为什么没有阻塞主线程，原理是什么](https://www.jianshu.com/p/ea7beaeeee16)

[Android面试总结](https://www.jianshu.com/p/af9410234de9)

[LeetCode刷题原因及刷题目录](https://www.jianshu.com/p/822aff36f4e0)

[史上最全的Java面试题总汇，不再惧怕面试官，成功坐等offer](https://www.jianshu.com/p/85150b9586f8)

[Android最简单的热修复原理解析](https://www.jianshu.com/p/6f0522f7ede4)





##### 前言

Android的消息机制主要是指Handler的运行机制，对于大家来说Handler已经是轻车熟路了，可是真的掌握了Handler？本文主要通过几个问题围绕着Handler展开深入并拓展的了解。

 

#### Questions

1. Looper 死循环为什么不会导致应用卡死，会消耗大量资源吗？
2. 主线程的消息循环机制是什么（死循环如何处理其它事务）？
3. ActivityThread 的动力是什么？（ActivityThread执行Looper的线程是什么）
4. Handler 是如何能够线程切换，发送Message的？（线程间通讯）
5. 子线程有哪些更新UI的方法。
6. 子线程中Toast，showDialog，的方法。（和子线程不能更新UI有关吗）
7. 如何处理Handler 使用不当导致的内存泄露？

**回答一： Looper 死循环为什么不会导致应用卡死？**

> 线程默认没有Looper的，如果需要使用Handler就必须为线程创建Looper。我们经常提到的主线程，也叫UI线程，它就是ActivityThread，ActivityThread被创建时就会初始化Looper，这也是在主线程中默认可以使用Handler的原因。

```java
 new Thread(new Runnable() {
        @Override
        public void run() {
            Log.e("qdx", "step 0 ");
            Looper.prepare();

            Toast.makeText(MainActivity.this, "run on Thread", Toast.LENGTH_SHORT).show();

            Log.e("qdx", "step 1 ");
            Looper.loop(); 
            Log.e("qdx", "step 2 "); 
        }
    }).start();
```

Looper.loop() ; 里面维护了一个死循环方法，所以按照理论，上述代码执行的应该是 step 0 –>step 1 也就是说循环在Looper.prepare();与Looper.loop(); 之间。

> 在子线程中，如果手动为其创建了Looper，那么在所有的事情完成以后应该调用quit方法来终止消息循环，否则这个子线程就会一直处于等待（阻塞）状态，而如果退出Looper以后，这个线程就会立刻（执行所有方法并）终止，因此建议不需要的时候终止Looper。

