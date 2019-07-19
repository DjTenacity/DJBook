# 子线程中使用Handler

+ 在**子线程中使用handler就意味着handler的实例是在子线程中去创建的**。

```java
Looper.prepare();
mHandler = new Handler(){
    @Override
  public void handleMessage(Message msg) {
            Log.d(TAG," mHandler is coming");
  handler_main.sendEmptyMessage(1);
  }
};
mHandler.sendEmptyMessage(1);
Looper.loop();
```

+ 如果在调用之前**必须调用Looper.prepare()方法**，这个是在当前线程中创建一个looper出来，如果是普通的应用场景可以直接使用HandlerThread，其中是带有Looper的。
+ 第二点值得注意的就是，Looper.loop()这个方法是无限循环的，所以在Looper.loop()后边的程序代码块是无法执行到的。**loop()方法的主要作用是一直不断的通过queue.next()方法来读取来自messagequeue中的msg**，这个方法是block的状态，如果queue中没有消息的话会一直阻塞在这里。
+ 关于Looper还有一个方法，当我们需要获取Looper实例时，可以直接在对应线程调用 Looper looper = Looper.myLooper(); 来获取，默认情况下，系统只会给MainThread分配一个looper。  

#### 在子线程中更新UI

在子线程中获取UI线程的looper，然后再创建handler实例，在handlerMessage()方法中去更新UI。

```java
//getMainLooper()
handler_main = new Handler(getMainLooper()){
    @Override
  public void handleMessage(Message msg) {
  helloTextView.setText("getMainLooper");
  }
};
```

