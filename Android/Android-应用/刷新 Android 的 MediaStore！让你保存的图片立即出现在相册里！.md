# 刷新 Android 的 MediaStore！让你保存的图片立即出现在相册里！

## 一、序

App 内，创建一个文件并保存文件到本地的需求，是很常见的 I/O 操作。而如果这个文件变成了一张图片，那你涉及到的就不仅仅是一个 I/O 操作了，还需要考虑如何更新 MediaStore，这样才可以在系统相册中，看到它。

这里说的 **MediaStore，本质上是 Android 维护的一个文件系统的数据库**，它记录了当前磁盘上所有的文件索引，我们可以通过它，快速的查找当前系统的文件。

**MediaStore 刷新的时机是不一定的**，也就是说，保存的一张图片文件，MediaStore 并不会立即刷新文件系统，将此文件索引记录下来。而系统本身是存在一些自动刷新 MediaStore 的时机，例如：重启手机。表现就是，当你保存了一张图片到本地文件夹中之后，通过文件管理器类的 App，可以在目录下找到这涨照片，但是在系统相册中，是无法立即看到它的，同时你想用诸如 微信、QQ 去分享这张图片的时候，也是找不到的。所以在我们保存图片文件之后，去触发系统刷新 MediaStore 就尤为重要了。

本文就来讲讲，如何在保存图片之后，刷新系统 MediaStore 那些事。

刷新系统 Media 通常有如下几种方式：

- 通过操作 MediaStore 类。
- 发送广播更新 MediaStore。
- 通过操作 MediaScannerConnection 类。

这三种方式，各有优缺点，我们慢慢分析。

## 二、操作 MediaStore

这里说的操作 MediaStore，实际是操作它的一个内部类 `MediaStore.Images.Media`，它提供了几个 `inserImage ()` 方法，供我们向 MediaStore 中插入图片数据，并产生一个缩略图。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488a203a8e3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这个方法传递进去的是一个 Bitmap 对象，其余的 `title` 和 `description` 分别是图片文件的名称和一段描述。

举个 Kotlin 的例子：

 ```
MediaStore.Images.Media.insertImage(
        contentResolver,
        mShareBitmap!!,
        "image_file",
        "file")

 ```



使用 `inserImage()` 方法，不需要我们指定路径，会自动将图片保存至 `Picture` 目录下。它也不支持我们指定路径。如果我们对图片保存的路径没有要求，并且保存的是一个 Bitmap 对象，此方法是非常的方便的。

细心的朋友可能已经发现了 `inserImage()` 还有一个其他的重载方法，支持我们传递进去一个图片文件路径，不过我并不推荐使用这个方法，因为它会将原本的图片，再 Copy 一份，到 `Picture` 目录下，也就是说你最终在磁盘上会得到两张相同的图片。 

![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488a1f75cdd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这一点，看源码是最清晰的。它首先使用 `BitmapFactory.decodeFile()` 方法，得到一个 Bitmap，然后再去调用保存 Bitmap 对象的 `inserImage()` 方法，所以我们最终在磁盘上会有两张一模一样的图片。

## 三、发送广播

### 3.1 那些广播可以更新 MediaStore

说到广播，在 Android 4.4 之前，是可以通过 `ACTION_MEDIA_MOUNTED` 广播，来通知系统刷新 MediaStore 的，不过假如你现在还在依赖这条广播，你会得到一个错误信息。

 ```
E/AndroidRuntime(23718): java.lang.SecurityException: Permission Denial: not allowed to send broadcast android.intent.action.MEDIA_MOUNTED from pid=23718, uid=10097

 ```

在 Android 4.4 之后，这个广播只能由系统进行广播，App 只能对该广播进行监听，在当前的系统分布环境下，这条路已经走不通了。

这样设计也很好理解，毕竟扫描全盘是非常的耗资源，所以系统肯定要把全盘扫描的权限拿在自己手里不开放出来，避免被第三方 App 滥用。

不过 Android 依然给我们提供了替代方案，那就是用 `MediaScannerConnection` 或者发送 `ACTION_MEDIA_SCANNER_SCAN_FILE` 广播。

接下来就来说说 `ACTION_MEDIA_SCANNER_SCAN_FILE` 这个广播。

### 3.2 使用广播刷新

通过广播刷新 MediaStore 的方式非常的简单，只需要指定文件路径和 Action 就好了。

```
val saveAs = "Your_Created_Image_File_Path"
val contentUri = Uri.fromFile(File(saveAs))
val mediaScanIntent = Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE,contentUri)
sendBroadcast(mediaScanIntent)

```

正常情况下，它是没有问题的，不过假如你发现它不生效，就需要检查一下你文件的路径是否传递正确。

通过查看 MediaScannerReceiver 的源码，可以发现 `onReceive()` 方法中，针对 `ACTION_MEDIA_SCANNER_SCAN_FILE` 还有一个限制条件，那就是传递进去的文件绝对路径，必须是以 `Environment.getExternalStorageDirectory()` 方法的返回值开头。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488a180fb17?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



有兴趣可以仔细阅读源码，这里是 Android  6.0 的源码：

> http://androidxref.com/6.0.0_r1/xref/packages/providers/MediaProvider/src/com/android/providers/media/MediaScannerReceiver.java

本质上，还是 `/mnt/sdcard/` 路径就认，而 `/sdcard/` 就无法使用，所以只要我们不硬编码文件路径，这个问题基本上也就不存在。

这里也提醒我们，一定不要在代码里，硬编码文件路径，算是一个编码规范了。

### 3.3 删除文件后刷新MediaStore

本文一直都在说添加新文件的时候，如何刷新 MediaStore 的问题。但是其实还涉及到另外一个问题，我们删除了一个已经被收录在 MediaStore 中的文件，怎么办？在本文里也顺便讲一下。

既然放在这一小节讲，首先想到的是，直接再发一个广播出去，刷新这个路径，但是查阅最终执行扫描前的 MediaScanner 的 `scanSingleFile()` 方法，你就会知道这样的方式是行不通的。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488a1a78a00?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在这里可以看到，当你传递进去的文件路径，指向的文件不存在的时候，会直接 `return` 出去了，就执行不到刷新的逻辑里。

所幸的是，我在 DownloadManager 类中，找到了刷新删除文件的解决办法，依然是通过 ContentResolver 来解决。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488a205e975?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这里通过 ContentResolver 来向 MediaStore 中发起一个删除文件的操作，只需要传递进去一个文件的绝对路径即可。

## 四、操作 MediaScannerConnection 类

### 4.1 使用 MediaScannerConnection

刷新 MediaStore 还有一个最通用也是我推荐的一个方法，那就是使用 MediaScannerConnection 进行操作。

不同于 `MediaStore.Image.Media` 和广播的方式，使用 MediaScannerConnection 不仅可以保存文件，还可以指定文件路径，最好的就是，它还支持刷新完成的回调。

**如果我们对时序有要求，并且需要制定文件保存路径的话，最好的方式就是直接使用 MediaScannerConnection 类进行操作，并且这也应该是兼容最好的方式。**

这里我们主要是利用 MediaScannerConnection 类的 `scanFile()` 方法进行触发扫描。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488c7d54182?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



通过 `scanFile()` 方法，我们只需要制定一个待刷新的文件路径和对应的 MimeType 即可，它支持传递多个路径，也可就是支持批量扫描。

注意这里的 MimeType 是一定要填写的，并且不能写通配符 `*/*` 或 `null`，否则会导致刷新失败，通常我们保存的是一个图片的话，只需要传递 `image/jpeg` 即可。

最后一个参数， onScanCompletedListener 中可以监听我们扫描的结果，需要注意的是，假如这里扫描的是多个文件路径，它也会被回调多次。所以如果有什么在刷新之后的后续操作，就需要特殊处理一下（原因后面是说）。

```
MediaScannerConnection.scanFile(this
        , arrayOf(picFile.absolutePath)
        , arrayOf("image/jpeg"), { path, uri ->

    Log.i("cxmyDev", "onScanCompleted : " + path)

})

```

`scanFile()` 方法的使用还是很简单的，没什么需要额外交代的了。

### 4.2 MediaScannerConnection 原理

依然是从源码中找答案，我们先来看看 `scanFile()` 方法的实现。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488cc93cf08?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在 `scanFile()` 里，创建了一个 MediaScannerConnection 并调用了 `connect()` 方法。接下来我们继续看 `connect()` 方法。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488d188bd58?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在 `connect()` 方法中，可以看到，它实际上是 `bindServer()` 了 `MediaScannerService` 这个系统服务，所有的操作都在 MediaScannerService 中。

MediaScannerService 的源码，有兴趣可以去这里查看：

> http://androidxref.com/6.0.0_r1/xref/packages/providers/MediaProvider/src/com/android/providers/media/MediaScannerService.java

这是一个系统服务，我到这里就不继续跟下去了，回过头来继续看源码。

不过看到 `connect()` 方法的时候，那对应的，一定有 `disconnect()` 方法存在了，前面 `bindService()` 了一个系统服务，我们一定要有一个时机去调用 `unbindService()`，否则就会造成泄露。

MediaScannerConnection 确实提供了 `disconnect()` 方法，但是我们通过 `scanFile()` 方法拿不到这个对象。这里处理的非常的巧妙，不需要我们手动去触发 `disconnect()`，它是自维护的。

继续看 `scanFile()` 里被我们忽略的 `ClientProxy` 类，逻辑都在这里面。



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fc488e4076b2e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在 `scanNextPath()` 中，会去判断传递进去的文件路径是否都扫描过，如果已经没有更多需要扫描的路径了，就自己去调用 `disconnect()` 方法，回收资源。

到这里，也就解答了我们刚才的疑问，MediaScannerConnection 已经帮我们考虑了很多事情，我们只需要调用它的标准 API 就好了。

## 五、查缺补漏

### 5.1 扫描其他类型的媒体文件

在 Android 下，不仅仅只有图片，对于其他媒体文件，使用本文介绍的方法，也是适用的。

### 5.2 避免某个目录被 MediaStore 扫描

看完到这里应该会知道，哪怕我们什么都不做，在手机下次重启的时候，系统依然会去全盘扫描文件系统，更新 MediaStore。

但是有时候，我们有一些目录下的媒体文件，并不想让 MediaStore 扫描到，例如在 SDCard 上缓存的图片、图标等，这些我们都不想出现在系统相册内。

解决办法其实在官方文档中已经写了。

> https://developer.android.com/guide/topics/data/data-storage.html

这里简单说一下，当不需要被 MediaStore 扫描的目录下，创建一个名为 `.nomedia` 的空文件，它将阻止媒体扫描程序读取这个目录下的媒体文件。也就无法通过 MediaStore 分享给其他程序。

当然，一些重要的文件，依然建议放在自己的私有目录下。

## 六、小结

关于在 MediaStore 刷新图片，本文基本上就算是讲清楚了。我推荐的方法，是使用 MediaScannerConnection 来实现。

推荐阅读：

- [漫画：程序员，你能“管理”好你的产品经理吗？](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485186%26idx%3D1%26sn%3Daf106a580af73b2202d73a1301e991cb%26chksm%3D97851e23a0f297351f51de17f3d273c9fdcf67fee57f8dd4723092819aa5a565f45a2b98bb6b%23rd)
- [App 多语言翻译，机器翻译也能快如闪电！](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485349%26idx%3D1%26sn%3D9a3cdcc74e2836ef69fa90e43ac37ee2%26chksm%3D97851e84a0f29792b4b72e4dfe6f82fca81b748210fa63f4da9ed57c012e74d4b429aafe032a%23rd)
- [2017 最权威区块链报告(内含下载)](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485328%26idx%3D1%26sn%3Df663eace998cf4742b86dc3fccc7d642%26chksm%3D97851eb1a0f297a732ac78305f84f901b64baa499a8df29c91b7f69498e35912d7cc57b78588%23rd)
- [Google 的 Flutter 学习资料！](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485243%26idx%3D1%26sn%3D58ed14bb169514278a0a0804be108988%26chksm%3D97851e1aa0f2970c35aa77d9f7ff63671688e6b97a8f5e5050cf98a566e80cd9aa9409420178%23rd)
- [远程控制智能电视，方案已开源！](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485256%26idx%3D1%26sn%3D85f937d1169159e4437bad3f8e375f4e%26chksm%3D97851e69a0f2977f47d411e24b328f8cc48e01872014b06a0c55ad43f56adf9e51500af7ef0a%23rd)

作者：承香墨影

链接：https://juejin.im/post/5ae0541df265da0b9d77e45a

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。