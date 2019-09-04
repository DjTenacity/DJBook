# [性能优化——Android图片压缩与优化的几种方式](https://blog.csdn.net/u013928412/article/details/80358597)

图片优化压缩方式大概可以分为以下几类：更换图片格式，质量压缩，采样率压缩，缩放压缩，调用jpeg压缩等

### 1、设置图片格式

Android目前常用的图片格式有png，jpeg和webp，

+ png：无损压缩图片格式，支持Alpha通道，Android切图素材多采用此格式
+ jpeg：有损压缩图片格式，不支持背景透明，适用于照片等色彩丰富的大图压缩，不适合logo
+ webp：是一种同时提供了有损压缩和无损压缩的图片格式，派生自视频编码格式VP8，

从谷歌官网来看，无损webp平均比png小26%，有损的webp平均比jpeg小25%~34%，无损webp支持Alpha通道，有损webp在一定的条件下同样支持，有损webp在Android4.0（API 14）之后支持，无损和透明在Android4.3（API18）之后支持

采用webp能够在保持图片清晰度的情况下，可以有效减小图片所占有的磁盘空间大小

### 2、质量压缩

**质量压缩并不会改变图片在内存中的大小，仅仅会减小图片所占用的磁盘空间的大小，因为质量压缩不会改变图片的分辨率**，而图片在内存中的大小是根据width*height*一个像素的所占用的字节数计算的，宽高没变，在内存中占用的大小自然不会变，质量压缩的原理是**通过改变图片的位深和透明度来减小图片占用的磁盘空间大小，所以不适合作为缩略图**，可以用于想保持图片质量的同时减小图片所占用的磁盘空间大小。另外，**由于png是无损压缩，所以设置quality无效**，以下是实现方式：

```java
/**

- 质量压缩
  *
- @param format  图片格式 jpeg,png,webp
- @param quality 图片的质量,0-100,数值越小质量越差
  */
  public static void compress(Bitmap.CompressFormat format, int quality) {
  File sdFile = Environment.getExternalStorageDirectory();
  File originFile = new File(sdFile, "originImg.jpg");
  Bitmap originBitmap = BitmapFactory.decodeFile(originFile.getAbsolutePath());
  ByteArrayOutputStream bos = new ByteArrayOutputStream();
  originBitmap.compress(format, quality, bos);
  try {
      FileOutputStream fos = new FileOutputStream(new File(sdFile, "resultImg.jpg"));
      fos.write(bos.toByteArray());
      fos.flush();
      fos.close();
  } catch (FileNotFoundException e) {
      e.printStackTrace();
  } catch (IOException e) {
      e.printStackTrace();
  }
  }
```



### 3、采样率压缩

采样率压缩是通过设置BitmapFactory.Options.inSampleSize，**来减小图片的分辨率，进而减小图片所占用的磁盘空间和内存大小。**
设置的inSampleSize会导致压缩的图片的宽高都为1/inSampleSize，整体大小变为原始图片的inSampleSize平方分之一，当然，这些有些注意点：

+ 1、inSampleSize小于等于1会按照1处理
+ 2、inSampleSize只能设置为2的平方，不是2的平方则最终会减小到最近的2的平方数，如设置7会按4进行压缩，设置15会按8进行压缩。 
  具体的代码实现方式如下：

 ```java
/**
 * @param inSampleSize  可以根据需求计算出合理的inSampleSize
*/
public static void compress(int inSampleSize) {
File sdFile = Environment.getExternalStorageDirectory();
File originFile = new File(sdFile, "originImg.jpg");
BitmapFactory.Options options = new BitmapFactory.Options();
//设置此参数是仅仅读取图片的宽高到options中，不会将整张图片读到内存中，防止oom
options.inJustDecodeBounds = true;
Bitmap emptyBitmap = BitmapFactory.decodeFile(originFile.getAbsolutePath(), options);

options.inJustDecodeBounds = false;
options.inSampleSize = inSampleSize;
Bitmap resultBitmap = BitmapFactory.decodeFile(originFile.getAbsolutePath(), options);
ByteArrayOutputStream bos = new ByteArrayOutputStream();
resultBitmap.compress(Bitmap.CompressFormat.JPEG, 100, bos);
try {
    FileOutputStream fos = new FileOutputStream(new File(sdFile, "resultImg.jpg"));
    fos.write(bos.toByteArray());
    fos.flush();
    fos.close();
} catch (FileNotFoundException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
}
 ```



### 4、缩放压缩

通过减少图片的像素来降低图片的磁盘空间大小和内存大小，可以用于缓存缩略图
实现方式如下：

```java
public void compress(View v) {
    File sdFile = Environment.getExternalStorageDirectory();
    File originFile = new File(sdFile, "originImg.jpg");
    Bitmap bitmap = BitmapFactory.decodeFile(originFile.getAbsolutePath());
    //设置缩放比
    int radio = 8;
    Bitmap result = Bitmap.createBitmap(bitmap.getWidth() / radio, bitmap.getHeight() / radio, Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(result);
    RectF rectF = new RectF(0, 0, bitmap.getWidth() / radio, bitmap.getHeight() / radio);
    //将原图画在缩放之后的矩形上
    canvas.drawBitmap(bitmap, null, rectF, null);
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    result.compress(Bitmap.CompressFormat.JPEG, 100, bos);
    try {
        FileOutputStream fos = new FileOutputStream(new File(sdFile, "sizeCompress.jpg"));
        fos.write(bos.toByteArray());
        fos.flush();
        fos.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```



### 5、JNI调用JPEG库

Android的图片引擎使用的是阉割版的skia引擎，去掉了图片压缩中的哈夫曼算法——一种耗CPU但是能在保持图片清晰度的情况下很大程度降低图片的磁盘空间大小的算法，这就是为什么ios拍出的1M的照片比Android5M的还要清晰的原因。
由于笔者能力有限，暂时没有对NDK这块做深入研究，后期将会将此方法补全

### 6、其他

还有一些技巧，比如在缩放压缩的时候，Bitmap.createBitmap(bitmap.getWidth() / radio, bitmap.getHeight() / radio, Bitmap.Config.ARGB_8888)，如果不需要图片的透明度，可以将ARGB_8888改成RGB_565，这样之前每个像素占用4个字节，现在只需要2个字节，节省了一半的大小。

### 7、总结

+ 1、使用webp格式的图片可以在保持清晰度的情况下减小图片的磁盘大小，是一种比较优秀的，google推荐的图片格式
+ 2、**质量压缩可以减小图片占用的磁盘空间，不会减小在内存中的大小**
+ 3、**采样率压缩**可以通过**改变分辨率**来减小图片所占用的磁盘空间和内存空间大小，但是**采样率只能设置2的n次方**，可能图片的最优比例在中间
+ 4、**尺寸压缩同样也是通过改变分辨率来减小图片所占用的磁盘空间和内存空间大小，缩放的尺寸没有什么限制**
+ 5、jni调用jpeg库来弥补安卓系统skia框架的不足，也是比较优秀的解决方式（方法会在后期补充）
  既然尺寸压缩和采样率压缩都是通过改变图片的分辨率来降低大小，有什么区别吗？
  其中一个原因已经说明了，**采样率的压缩比例会受到限制，尺寸压缩不会，但是采样率压缩的清晰度会比尺寸压缩的清晰度要好一些，** 

 在缩小同样的倍数的情况下，采样率压缩看起来会比较柔和一些，尺寸压缩则锯齿比较多
————————————————
版权声明：本文为CSDN博主「孟凡勇」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u013928412/article/details/80358597