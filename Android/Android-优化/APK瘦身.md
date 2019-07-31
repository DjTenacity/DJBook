## [APK瘦身](https://www.jianshu.com/p/147b54f53e10)

### 一、icon 图标使用 svg

icon 尽量使用svg 文件,而不要使用png文件

> 首先 svg 文件是以xml文件的方式存在的,占用空间小,而且能够根据设备屏幕自动伸缩不会失真.

Android 本身是不支持直接导入svg文件的,所以我们需要将svg 文件进行转换一下.如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBHmvHib6zBModQkUhMyicqIxW2QiaOMz6cjtCNfia9ft81krgYQN1Q5g8XQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBzic9rP5ibxbytic9NtXgfgLSqcWuboP2MFam7ajA1QNPfXcOB90VugwXg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用如下:

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYB51fL2F2Pfw4qGCNGcibcFk6hF4sluibsYcoDOl4aIXYbtJLwvHoKYPsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 二、icon状态区分使用 Tint 着色器

Tint着色器能够实现图片变色 ,利用Tint显示不同颜色的图片 ,在原本需要多张相同图片不同颜色的情况,能够减少apk的体积

UI效果如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBlBAPN5VPiaglUu8fqo21AdibS99ia27NEzhADPg5ia4Tfsx7zCmVupjsjg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注意了,这是同一张图片的不同效果

使用如下:

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYB1yFzzib5cgctwvFVm1VdEQ9EVfs0X73QaE80IjWtVOzGx0SqPxrNjUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 三、需要多套不同尺寸的icon时,使用 svg

Android studio 自带功能,可以自行配置需要的icon尺寸,打包时会自动生成对应尺寸的png 图片.

使用如下:
在app的build.graldle中的defaultConfig 标签下:

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBnRcgj3638udibEQGl3FGEZkPa8VlUicx1pE1ENgUBibUmEvA7Dwv2cgrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时,drawable文件如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBUmZ7PspPvEH7kL2IC6oXH9xaHITGN032nEFNR0cYajeQaXf2LvDACQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

打包后如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYByIZOoaxrhlAHjjgwxYMxKhq4zhgWl4D048GNOcaOb63eibKoQvGWuXA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBCs3slzGKPKZyt8GRdtyFJPMCu5qEqvlkR1SdBaUwzzR1fyhmSOeAcg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

以后APP内就只需要一套图就可解决多套图造成apk体积增大的问题了

### 四、App内大图压缩,使用webp格式图片

WebP格式，谷歌开发的一种旨在加快图片加载速度的图片格式。图片压缩体积大约只有JPEG的2/3，并能节省大量的服务器宽带资源和数据空间。

使用如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBT4MZpye5lp9W2bZTkNsWDRqUvwBhBP1EyARVPCGtmImCNkOIa3f0yg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

转化前后对比

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYB46MhtnBe4DbUtuElNrVweaZWsyqlFIhJhTob5hVBfWptaEBic92fhsg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 五、 移除无用资源

- 一键移除 (不推荐)

> 一键移除未用到的资源,如果出现使用动态id加载资源会出现问题,而且这是物理删除,一旦删除将找不回了,所以能不用尽量别用,非要用请事先备份res文件.

使用如下

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBPtQeJ2SJc2f94D9mx2UfMTF8MqUdxdDRJUB5HIMvlCApGOrEicU6TWw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBNzw3IN3BlQLGaL1iapAkaw9OKZXO99yCFI80PRxvcSEk343LxluIibXw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBzx31eicjPsic37hYO6OnAACcbRcVPC8hpibO3bNbncicg6qUHRgFEA3WhQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBC4hEqYWGLDdRtL5QQ0G5Vyx6qmI3VBcpGOl87boziboxdL6kV9SATVw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 使用 shrinkResources 进行移除,配合 //Zipalign优化

> 使用 shrinkResources 必须先开启代码混淆 minifyEnabled

使用如下:

```
buildTypes {
     release {
       //开启代码混淆
       minifyEnabled true
       //Zipalign优化
       zipAlignEnabled true
       //移除无用的resource文件
       shrinkResources true
       proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
     }
}
```

打包后效果如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBqYovXYAbP6fj9DPyJC4FWAxMlXWf0AnyrVLTeP1Kog4G5MzuV7fUHA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

虽然图片还存在. 但400多k的大小变成了2B

### 六、资源打包设置

由于第三方库的引入,如appcompat-v7的引入库中包含了大量的国际化资源,可根据自身业务进行相应保留和删除

原始包如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBg1llSI0B9cJMNndY1SW99eaibLmKXTH37kxO7OvyXNH5iaEYcmmxlMGw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

原始包中存在各国的语言,所以我们一般只需要保留中文即可,配置如下:

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYB4nq8uaw9Lnod6ht4B9j3keTg4kNexoArqaWdeHbHLBicJCbkCqzOL0w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置后如下:

![img](https://mmbiz.qpic.cn/mmbiz/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYB9yTffXicmE5wA5L176LcQibRiaic1fr3Ig8WuibvDd5yictIen4eyX8xM5Xw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 七、动态库打包配置

如果项目中包含第三方SDK或者直接使用了NDK,如果不进行配置会自动打包全cpu架构的动态库进入apk,而对于真机,只需要保留一个armeabi或者armeabi-v7a就可以了,所以可以进行一下配置

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBXmCPibbkMsWfhD1r31D74ic021FFImjRW1UiajFfeJG9WPxpGzzv8t8WQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 八、开启代码混淆压缩

![img](https://mmbiz.qpic.cn/mmbiz_png/CvQa8Yf8vq3eR1icvCxVrpoC3YwyIrMYBibwn6lm166iaPkNWpPKOp8owrOEYnYSYTgRtkh91LoQ1WuBQibLc4vmXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

关于代码混淆配置,这里就不再多说,不了解的可以自行去网上了解一下

至此,apk 极致优化八道步骤就结束了,如果你的apk没有进行过任何优化,那么
这八道工序下来,目测你的apk体积至少缩减到一半,赶快 去试试这神奇的优化吧



--------  END  ----------

**推荐阅读**

[我身边那些起点低的年轻人最后都怎样了？](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649844679&idx=1&sn=2a9a8b45836e1859686c68e6095eea78&chksm=83bf629cb4c8eb8ac75d78a1eb0ebce71a99a06a16bb7f77bf09e1cc33467edafb3c0e409fd2&scene=21#wechat_redirect)

[热修复原理之热修复框架对比和代码修复](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649842083&idx=1&sn=e2553d245ba7601f790fe9f0271815f3&chksm=83bf68f8b4c8e1eebc95c26b4677edf2e8f2fa082dc4c6b5ab2dafb37afce4f3d1dabb0feb91&scene=21#wechat_redirect)

[不了解这12个语法糖，别说你会Java！](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649844656&idx=1&sn=25c788f6323a988b7ee62a7351bbaed3&chksm=83bf62ebb4c8ebfd338ee13b94de84c3353b389cf5daf1f284a00cf7742839dbc9bd78dd8cc6&scene=21#wechat_redirect)


  