[引用](https://blog.csdn.net/anyanyan07/article/details/81389873)
# Gradle配置方便快速切换生产和测试环境

实际开发项目时，肯定会遇到切换后台地址的情况。最简单的分为测试环境和生产环境，复杂一点的会有内网环境，外网环境，内网测试，外网测试，外网生产…，头都大了有木有。每次都要去手动更改url地址（手抖还有可能改错，酿成大祸），最讨厌的时，他们总会拿着手机问我他们手机上安装的是哪个环境的，因为除了地址不同，其他没有区别， 就很难受

最近发现可以利用Gradle多渠道打包配置解决上述问题。比如我的需求是有两个环境：生产环境和测试环境，可能需要每天打不同环境的包给他们测试。 

### gradle配置
在 moudle 中的 build.gradle 文件中的 android{ } 中添加 productFlavors{ }实现多渠道打包：
```
productFlavors {
        //测试版本
        app_text {
            //生成占位符，在AndroidManifest.xml文件中使用
            manifestPlaceholders = [APP_NAME   : "@string/app_name_text",
                                    APP_ICON   : "@mipmap/ic_launcher_text"]
            //gradle编译完成后，会生成static final常量，在java代码中使用
            //buildConfigField(变量类型，变量名字，变量值)
            buildConfigField("String", "SERVICE_HOST", "\"http://sss.sss.x.sss/\"")
        }
        //生产版本
        app_product {
            manifestPlaceholders = [APP_NAME   : "@string/app_name",
                                    APP_ICON   : "@mipmap/ic_launcher"]
            buildConfigField("String", "SERVICE_HOST", "\"http://xxx.xxx.x.xx:8080/\"")
        }
    }
```
注意 要加\”转义，当然上述配置中使用到的string,mipmap资源要存在，比如字符串：
```
<resources>
    <string name="app_name">某某</string>
    <string name="app_name_text">某某_测试版</string>
    ...
</resources>
```
#### 在AndroidManifest.xml文件中使用占位符
```
<application
        android:icon="${APP_ICON}"
        android:label="${APP_NAME}"
        android:theme="@style/AppTheme">
```
#### 在代码中使用静态常量
```
String url = BuildConfig.SERVICE_HOST
```
### 选择Debug的版本
在Android Studio左下角点击BuildVariants，在弹出面板中选择要构建的版本，选择后，直接debug安装到手机的版本就是你选择的版本了

![图片](https://img-blog.csdn.net/20180803160631421?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FueWFueWFuMDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 打包不同的版本
在你配置好签名等信息之后（不会的自己百度），点击右上角Gradle，在弹出的面板中可以双击不同的版本进行打包： 

![图片](https://img-blog.csdn.net/20180803161452946?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FueWFueWFuMDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

任务执行完成后在build/outputs/apk中可以看到生成的apk文件：

![图片](https://img-blog.csdn.net/20180803161705564?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FueWFueWFuMDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

ok，到此结束，再也不用到java代码中手动修改了，直接选择即可