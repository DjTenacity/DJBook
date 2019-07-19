# [AtomicReference 和 volatile 的区别](https://www.cnblogs.com/lpthread/p/3909231.html)

首先volatile是java中关键字用于修饰变量，AtomicReference是并发包java.util.concurrent.atomic下的类。
首先volatile作用，当一个变量被定义为volatile之后，看做“程度较轻的 synchronized”，具备两个特性：
1.保证此变量对所有线程的可见性(当一条线程修改这个变量值时，新值其他线程立即得知)
2.禁止指令重新排序
注意volatile修饰变量不能保证在并发条件下是线程安全的，因为java里面的**运算**并非原子操作。
java.util.concurrent.atomic工具包，支持在单个变量上解除锁的线程安全编程。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。

volatile是不能保证原子性的, 写了一点junit. 这里使用了包装类Integer, 来验证 对引用操作 的原子性. 可以看到使用了AtomicReference可以保证结果是正确的.

```java
private static volatile Integer num1 = 0;
    private static AtomicReference<Integer> ar=new AtomicReference<Integer>(num1);

    @Test
    public void dfasd111() throws InterruptedException{
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable(){
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++)
                        while(true){
                            Integer temp=ar.get();
                            if(ar.compareAndSet(temp, temp+1))break;
                        }
                }       
            }).start();
        }
        Thread.sleep(10000);
        System.out.println(ar.get()); //10000000
    }
        
    @Test
    public void dfasd1112() throws InterruptedException{
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable(){
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        num1=num1++;
                    }
                }       
            }).start();
        }
        Thread.sleep(10000);
        System.out.println(num1); //something like 238981
    }
```

如果想让运算具有原子性， 请使用：

AtomicInteger

AtomicLong

类似i++这样的"读-改-写"复合操作(在一个操作序列中, 后一个操作依赖前一次操作的结果), 在多线程并发处理的时候会出现问题, 因为可能一个线程修改了变量, 而另一个线程没有察觉到这样变化, 当使用原子变量之后, 则将一系列的复合操作合并为一个原子操作,从而避免这种问题, i++=>i.incrementAndGet() 
原子变量只能保证对一个变量的操作是原子的, 如果有多个原子变量之间存在依赖的复合操作, 也不可能是安全的, 另外一种情况是要将更多的复合操作作为一个原子操作, 则需要使用synchronized将要作为原子操作的语句包围起来. 因为涉及到可变的共享变量(类实例成员变量)才会涉及到同步, 否则不必使用synchronized