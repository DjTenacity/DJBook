# [Android图片压缩实现过程及代码](http://www.codeceo.com/article/android-image-compression.html)

[Android图片压缩](http://www.codeceo.com/article/android-image-compression.html)先讲两种，一种质量压缩，一种像素压缩，前者多用于图片上传时，后者多用于本地图片展示缩略图时。

对于质量压缩，主要用到的一个方法就是：

```java
public boolean compress(CompressFormat format, int quality, OutputStream stream) {}
```

这是Bitmap类里的一个方法，第一个参数表示图片压缩的格式，android中提供了以下格式：

```java
public enum CompressFormat {
    JPEG    (0),
    PNG     (1),
    WEBP    (2);

    CompressFormat(int nativeInt) {
        this.nativeInt = nativeInt;
    }
    final int nativeInt;
}
```

第二个参数表示压缩的质量，注意这个是压缩的关键，它的取值是0到100，越小表示压缩的越厉害，第三个参数表示把压缩的数据写入了outputstream流中。

OK，来看一个例子：

```java
public byte [] compressBitmap(Bitmap bitmap,int max){
    int quality = 100;
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    bitmap.compress(Bitmap.CompressFormat.JPEG,quality,byteArrayOutputStream);
    while (byteArrayOutputStream.toByteArray().length / 1024 > max){
        byteArrayOutputStream.reset();
        quality = quality -10;
        bitmap.compress(Bitmap.CompressFormat.JPEG,quality,byteArrayOutputStream);
    }
    return byteArrayOutputStream.toByteArray();
}
```

这个方法的第一个参数不必解释，第二个参数表示你要求的压缩后图片最大可以是多少。最后可以拿到一个byte数组。我们有了这个byte数组就可以转化为file或者bitmap。

注意这种质量压缩后，像素本身没有改变。

对于像素压缩，顾名思义就是压缩像素。这里用到的一个主要的方法就是：

```java
public static Bitmap decodeFile(String pathName, Options opts) {
    Bitmap bm = null;
    InputStream stream = null;
    try {
        stream = new FileInputStream(pathName);
        bm = decodeStream(stream, null, opts);
    } catch (Exception e) {
        /*  do nothing.
            If the exception happened on open, bm will be null.
        */
        Log.e("BitmapFactory", "Unable to decode stream: " + e);
    } finally {
        if (stream != null) {
            try {
                stream.close();
            } catch (IOException e) {
                // do nothing here
            }
        }
    }
    return bm;
}
```

这是BitmapFactory中的一个静态方法，第一个参数表示file的全路径，第二个参数是关键，Options是BitmapFactory类中的一个静态内部类，它有两个非常重要的属性：

```java
/**
 * If set to true, the decoder will return null (no bitmap), but
 * the out... fields will still be set, allowing the caller to query
 * the bitmap without having to allocate the memory for its pixels.
 */
public boolean inJustDecodeBounds;

/**
 * If set to a value > 1, requests the decoder to subsample the original
 * image, returning a smaller image to save memory. The sample size is
 * the number of pixels in either dimension that correspond to a single
 * pixel in the decoded bitmap. For example, inSampleSize == 4 returns
 * an image that is 1/4 the width/height of the original, and 1/16 the
 * number of pixels. Any value <= 1 is treated the same as 1. Note: the
 * decoder uses a final value based on powers of 2, any other value will
 * be rounded down to the nearest power of 2.
 */
public int inSampleSize;
```

第一个属性inJustDecodeBounds，如果设置为ture，则返回null。

第二个属性inSampleSize表示缩放比例，大于1表示缩小了原来的多少，比如inSampleSize ＝＝ 4，就表示缩小了原来的四分之一，如果小于1则和1相同。

好了来看看这个网上遍地都是的一个像素压缩的方法：

```java
private Bitmap getimage(String srcPath) {
    BitmapFactory.Options newOpts = new BitmapFactory.Options();
    //开始读入图片，此时把options.inJustDecodeBounds 设回true了
    newOpts.inJustDecodeBounds = true;
    Bitmap bitmap = BitmapFactory.decodeFile(srcPath,newOpts);//此时返回bm为空

    newOpts.inJustDecodeBounds = false;
    int w = newOpts.outWidth;
    int h = newOpts.outHeight;
    //现在主流手机比较多是800*480分辨率，所以高和宽我们设置为
    float hh = 800f;//这里设置高度为800f
    float ww = 480f;//这里设置宽度为480f
    //缩放比。由于是固定比例缩放，只用高或者宽其中一个数据进行计算即可
    int be = 1;//be=1表示不缩放
    if (w > h && w > ww) {//如果宽度大的话根据宽度固定大小缩放
        be = (int) (newOpts.outWidth / ww);
    } else if (w < h && h > hh) {//如果高度高的话根据宽度固定大小缩放
        be = (int) (newOpts.outHeight / hh);
    }
    if (be <= 0)
        be = 1;
    newOpts.inSampleSize = be;//设置缩放比例
    //重新读入图片，注意此时已经把options.inJustDecodeBounds 设回false了
    bitmap = BitmapFactory.decodeFile(srcPath, newOpts);
    return bitmap;
}
```