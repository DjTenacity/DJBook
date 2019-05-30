# RxJava-ZIP实现遍历List集合定时发射每条数据

场景：遍历一个List集合，每个集合元素都间隔一定时间。

RxJava实现：

```java
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        //fromIterable接收一个Iterable，每次发射一个元素（与for循环效果相同）
        Observable<String> listObservable = Observable.fromIterable(list);
        //interval定时器，间隔1秒发射一次
        Observable<Long> timerObservable = Observable.interval(1000, TimeUnit.MILLISECONDS);
        //使用zip操作符合并两个Observable
        Observable.zip(listObservable, timerObservable, new BiFunction<String, Long, String>() {
            @Override
            public String apply(String s, Long aLong) throws Exception {
                return s;
            }
        }).doOnNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                //此处接收
            }
        }).subscribe();
```



 zip操作符：
 



![img](https:////upload-images.jianshu.io/upload_images/3667617-d34a4932bf94824f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/838/format/webp)

zip操作符



也就是说，通过zip操作符将`listObservable`和`timerObservable`合并之后，实现的效果就是：
 **发射list第一个元素 → 延时1秒 → 发射list第二个元素 → 延时1秒......**

使用RxJava这种方式实现，要比for循环里使用`Thread.sleep()`更加优雅一些，还能避免`java.lang.InterruptedException`异常

> PS：delay操作符无法实现，因为delay用于事件流中，只能延迟发射事件流中的某一次事件



作者：AIllll

链接：https://www.jianshu.com/p/aa71c155e19a

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。