## android壁纸---仿抖音实现一个动态壁纸

#### 一、概述：

​		壁纸运行在一个Android服务之中，这个服务的名字叫做WallpaperService。当用户选择了一个壁纸之后，此壁纸所对应的WallpaperService便会启动并开始进行壁纸的绘制工作。
　　

　　Engine是WallpaperService中的一个内部类，实现了壁纸窗口的创建以及Surface的维护工作。Engine内部实现了SurfaceView，我们只需要在其内部利用MediaPlayer + SurfaceView就可以播放动态壁纸了。

#### 二、实现：

WallpaperService需要一个xml去配置，然后在AndroidManifest.xml中声明

```java
<wallpaper xmlns:android="http://schemas.android.com/apk/res/android"
    android:thumbnail="@mipmap/icon_lacation_black___cm">
</wallpaper>
```

继承WallpaperService实现我们自己的壁纸服务VideoLiveWallpaper

```java
public class VideoLiveWallpaper extends WallpaperService {
    @Override
    public Engine onCreateEngine() {
        return new VideoEngine();
    }

    class VideoEngine extends Engine {
        private MediaPlayer mMediaPlayer;

        @Override
        public void onCreate(SurfaceHolder surfaceHolder) {
            super.onCreate(surfaceHolder);
        }

        @Override
        public void onDestroy() {
            super.onDestroy();
        }

        @Override
        public void onSurfaceCreated(SurfaceHolder holder) {
            super.onSurfaceCreated(holder);
            mMediaPlayer = new MediaPlayer();
            mMediaPlayer.setSurface(holder.getSurface());
            try {
                mMediaPlayer.setDataSource(new File(FileUtil.getDCIMCameraDir(), "hlj_wallpaper").getAbsolutePath());
                mMediaPlayer.setLooping(true);
                mMediaPlayer.setVolume(0, 0);
                mMediaPlayer.prepare();
                mMediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                    @Override
                    public void onPrepared(MediaPlayer mp) {
                        mMediaPlayer.start();
                    }
                });

            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onSurfaceDestroyed(SurfaceHolder holder) {
            super.onSurfaceDestroyed(holder);
            mMediaPlayer.release();
            mMediaPlayer = null;
        }

        @Override
        public void onVisibilityChanged(boolean visible) {
            if (visible) {
                mMediaPlayer.start();
            } else {
                mMediaPlayer.pause();
            }
        }
    }
}
```

接着声明这个服务同时声明我们上面写的xml配置

```java
 <service
            android:name=".VideoLiveWallpaper"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_WALLPAPER"
            android:process=":wallpaper">
            <!-- 配置intent-filter -->
            <intent-filter>
                <action android:name="android.service.wallpaper.WallpaperService" />
            </intent-filter>
            <!-- 配置meta-data -->
            <meta-data
                android:name="android.service.wallpaper"
                android:resource="@xml/wallpaper" />
        </service>
```

重点在onSurfaceCreated方法中，这里为了可以动态切换不同的壁纸，我是指定去加载一个固定目录下的视频文件，然后不断的复制新文件到这个目录，因为一旦开启切换壁纸这个方法就会调用，所以当调用后再动态通知去更改路径不起作用。

所以我在更换壁纸前先清空

```java
 try {
     WallpaperManager.getInstance(getContext())
     .clear();
     } catch (IOException e) {
     e.printStackTrace();
     }
```

再去复制需要替换的壁纸到指定目录

```java
 copyFile(file.getAbsolutePath(),new File(FileUtil.getDCIMCameraDir(),
   "hlj_wallpaper").getAbsolutePath());                               
```

copyFile方法如下所示：

```java
  /**
     * 复制单个文件
     *
     * @param oldPath String 原文件路径 如：c:/fqf.txt
     * @param newPath String 复制后路径 如：f:/fqf.txt
     * @return boolean
     */
    public void copyFile(final String oldPath, final String newPath) {
        progressBar.setVisibility(View.VISIBLE);
        Observable.create(new Observable.OnSubscribe<Boolean>() {
            @Override
            public void call(Subscriber<? super Boolean> subscriber) {
                try {
                    int byteSum = 0;
                    int byteRead ;
                    File oldFile = new File(oldPath);
                    if (oldFile.exists()) { //文件存在时
                        InputStream inStream = new FileInputStream(oldPath); //读入原文件
                        FileOutputStream fs = new FileOutputStream(newPath);
                        byte[] buffer = new byte[1444];
                        while ((byteRead = inStream.read(buffer)) != -1) {
                            byteSum += byteRead; //字节数 文件大小
                            System.out.println(byteSum);
                            fs.write(buffer, 0, byteRead);
                        }
                        inStream.close();
                        subscriber.onNext(true);
                        subscriber.onCompleted();
                    }
                } catch (Exception e) {
                    System.out.println("复制单个文件操作出错");
                    e.printStackTrace();
                    subscriber.onCompleted();
                }
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Boolean>() {
                    @Override
                    public void onCompleted() {
                        progressBar.setVisibility(View.GONE);
                    }

                    @Override
                    public void onError(Throwable e) {
                        progressBar.setVisibility(View.GONE);
                    }
                    @Override
                    public void onNext(Boolean aBoolean) {
                        progressBar.setVisibility(View.GONE);
                        setToWallPaper(getContext());
                    }
                });
    }
```

setToWallPaper方法就是真正的开启设置壁纸操作了

```java
  public static void setToWallPaper(Context context) {
        final Intent intent = new Intent(WallpaperManager.ACTION_CHANGE_LIVE_WALLPAPER);
        intent.putExtra(WallpaperManager.EXTRA_LIVE_WALLPAPER_COMPONENT,
                new ComponentName(context, VideoLiveWallpaper.class));
        context.startActivity(intent);
    }
```

至此，一个简单的动态壁纸就搞定了。