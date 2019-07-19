# Android 多线程：HandlerThread理解和使用总结

多线程的应用在`Android`开发中是非常常见的，常用方法主要有：

1. 继承Thread类
2. 实现Runnable接口
3. Handler
4. AsyncTask
5. HandlerThread   :  已封装好的轻量级异步类

### 作用

1. 实现多线程
   在工作线程中执行任务，如 耗时任务
2. 异步通信、消息传递
   **实现工作线程 & 主线程（UI线程）之间的通信**，即：将工作线程的执行结果传递给主线程，从而在主线程中执行相关的`UI`操作

### 优点:

+ HandlerThread内部维护了一个消息队列，避免多次创建和销毁子线程来进行操作。

+ 方便实现异步通信，即不需使用 “任务线程（如继承`Thread`类） + `Handler`”的复杂组合

### 工作原理

内部原理 = `Thread`类 + `Handler`类机制，即：

- 通过继承`Thread`类，，在`run()`方法中通过`Looper.prepare()`来创建消息队列，`Looper.loop()`来循环处理消息。**快速地创建1个带有Looper对象的新工作线程**
- 通过封装`Handler`类，**快速创建Handler & 与其他线程进行通信**



### 使用

- `HandlerThread`的本质：继承`Thread`类 & 封装`Handler`类
- `HandlerThread`的使用步骤分为5步

```java
// 步骤1：创建HandlerThread实例对象  传入参数 = 线程名字，作用 = 标记该线程
   HandlerThread mHandlerThread = new HandlerThread("handlerThread");

// 步骤2：启动线程
   mHandlerThread.start();

// 步骤3：创建工作线程Handler & 复写handleMessage（）
// 作用：关联HandlerThread的Looper对象、实现消息处理操作 & 与其他线程进行通信
// 注：消息处理操作（HandlerMessage（））的执行线程 = mHandlerThread所创建的工作线程中执行
  Handler workHandler = new Handler( handlerThread.getLooper() ) {
            @Override
            public boolean handleMessage(Message msg) {
                ...//消息处理
                return true;
            }
        });

// 步骤4：使用工作线程Handler向工作线程的消息队列发送消息
// 在工作线程中，当消息循环时取出对应消息 & 在工作线程执行相关操作
  // a. 定义要发送的消息
  Message msg = Message.obtain();
  msg.what = 2; //消息的标识
  msg.obj = "B"; // 消息的存放
  // b. 通过Handler发送消息到其绑定的消息队列
  workHandler.sendMessage(msg);

// 步骤5：结束线程，即停止线程的消息循环
  mHandlerThread.quit();

```

要记得在`onPause()`和`onDestroy()`中暂停更新和停止mHandlerThread以释放内存。

```
 @Override
    protected void onPause() {
        super.onPause();
        isUpdate = false;
        workHandler.removeMessages(MSG_UPDATE_INFO);
    }

    @Override
    protected void onDestroy(){
        super.onDestroy();
        //释放资源
        mHandlerThread.quit();
    }
```

