## [ionic app打包和签名](https://www.cnblogs.com/nelsonlei/p/10083006.html)

1、首先在项目根目录执行  **ionic platform add android**  生成Android平台。

 

2、**配置应用签名**：在根目录下执行以下命令

```
keytool -genkey -v -keystore myApp.keystore -alias myApp -keyalg RSA -keysize 2048 -validity 20000
```

　　

　　命令说明：

```
　　 -genkey                         产生密钥 
    -alias pbnEoC.keystore          别名 demo.keystore 
    -keyalg                         RSA 使用RSA算法对签名加密 
    -validity 20000                 有效期限2000天 
    keysize:                        大小
    -keystore pbnEoC.keystore       证书的别名
```

　　

　　结果如下：会在根目录生成一个myApp.keystore的文件

![img](https://img2018.cnblogs.com/blog/1059235/201812/1059235-20181207145204861-1443600452.png)

 

 3、使用**build命令编译一个应用的发布版本**， 在platforms\android\build\outputs\apk下找到android-release-unsigned.apk文件，把它移动到根目录下（跟myApp.keystore**同目录**）。以防签名的时候找不到jar文件

```
ionic build --release android
```

 

 4、**签名应用文件**：把已经生成的  android-release-unsigned.apk  文件移到项目根目录下，不然可能会报错"无法打开 jar 文件: android-release-unsigned.apk"。在终端命令窗口进入到项目根目录。执行以下命令：

```
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore myApp.keystore android-release-unsigned.apk myApp
```

　　过程会需要一点时间，期间会提示输入keystore的密码密钥，`命令会修改apk文件并对其进行签名。`

 

　　命令说明：

```
Jarsigner       　　　　　　　 　　　　　　　　　　是工具名称
-verbose   　　　　　　　　　　 　　　　　　　　　　表示将签名过程中的详细信息打印出来，显示在控制台窗口中
-keystore myApp.keystore     　　　　　　　　　　之前生成的证书 ，表示签名所使用的数字证书所在位置/名字（同目录）
-signedjar (后面的路径是你要签名apk的路径)        表示给Apk工程目录下的 android-release-unsigned.apk 文件签名
myApp   　　　　　　　　　　　　　　　　　　　　　　 表示证书的别名，对应于生成数字证书时-alias参数后面的名称
```

 

 5、**验证apk是否签名成功**：出现一堆信息

```
jarsigner -verify -verbose -certs android-release-unsigned.apk
```

 

 6、**查看签名的信息**：

```
keytool -printcert -file META-INF/*.RSA
```

 

 7、可选择执行以下命令：**优化apk文件**-----减少在设备上占用的空间和内存。我们使用zipalign工具，它使用签名后的APK文件生成一个优化后的APK版本，用于应用上传。

 

```
添加环境变量：
    path：D:\AndroidSDK\android-sdk-windows\build-tools\23.0.3



在项目根目录下执行命令：
    jarsigner -verify -verbose -certs android-release-unsigned.apk
```



　　成功的显示：

　　![img](https://img2018.cnblogs.com/blog/1059235/201812/1059235-20181207150312281-1647126125.png)



[« ](https://www.cnblogs.com/nelsonlei/p/10071680.html)上一篇：[js时间戳与日期格式的相互转换](https://www.cnblogs.com/nelsonlei/p/10071680.html)
[» ](https://www.cnblogs.com/nelsonlei/p/10102887.html)下一篇：[js实现获取当前时间是本月第几周和年的第几周的方法](https://www.cnblogs.com/nelsonlei/p/10102887.html)

