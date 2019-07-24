## 通俗理解数字签名，[数字证书和](https://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943600&idx=1&sn=bcbf3eb80ae2be5a3a40ca53b622ea23&chksm=84fd6054b38ae9422e8a15e5f21918a533466ea58e99a7904532b721a11f2628c4f56404c8b9&scene=21#wechat_redirect)https

### 前言

最近在开发关于PDF合同文档电子签章的功能，大概意思就是在一份PDF合同上签名，盖章，使其具有法律效应。签章有法律效应必须满足两个条件：

- 能够证明签名，盖章者是谁，无法抵赖
- PDF合同在签章后不能被更改

在纸质合同中，由于签名字迹的不可复制性，盖章的唯一性以及纸质合同对涂改的防范措施（比如金额用大写）可以保证上述两点，从而具备法律效应，那么PDF合同如何保障呢？两个重要的概念就是数字签名和数字证书。这项技术广泛运用于文件认证，数据传输等。
为了弄懂这些，我花了2天时间从加密算法开始，到数字签名和CA证书，最后再重新认识下https的原理。



### 非对称加密

加密我了解的不多，只知道有这么两种算法：对称加密和非对称加密。

- 对称加密：加密和解密的密钥一样，比如用123加密就是用123解密，但是实际中密码都是普通数据在互联网传输的，这样一点密码被中间人截取并破解，加密直接被攻破。
- 非对称加密：把密钥分为公钥和私钥，公钥是公开的所有人都可以认领，私钥是保密的只有一个人知道。假设A要发送一封Email给B，他不想让任何其他人在传输中看到Email的内容，做法就是使用B的公钥对Email加密，只有B的私钥能够解密（B的私钥唯一性保证信件不会泄露）。
  某天出意外了，有黑客冒充A给B发送Email，并且也用B的公钥加密，导致B无法区分这封邮件是否来自A。怎么办？此时A可以用自己的私钥加密，那么B收到邮件后如果用A的公钥可以解密邮件，那么证明这封信肯定来自于A。

OK，通过这个例子我想你们基本明白非对称加密了！我总结了下面几点：

+  公钥的作用：对内容本身加密，保证不被其他人看到。
+  私钥的作用：证明内容的来源
+  公钥和私钥是配对关系，公钥加密就用私钥解密，反之亦然，用错的密钥来尝试解密会报错。

### 数字签名

接着聊上面发邮件的例子，假设A用自己的私钥对Email加密发送，这样还存在啥问题吗？当然，它至少存在下面2个问题：

- 无法验证邮件在传输中是否被篡改。是的，任何人都可以用A的公钥解密邮件并篡改数据，更要命的是B收到邮件无法验证是否被篡改！
- 对文件本身加密可能是个耗时过程，比如这封Email足够大，那么私钥加密整个文件以及拿到文件后的解密无疑是巨大的开销。

- 数字签名可以解决这两个问题，下面看看A如何利用数字签名对邮件加密：
  1.A先对这封Email执行哈希运算得到hash值简称“摘要”，取名h1
  2.然后用自己私钥对摘要加密，生成的东西叫“数字签名”
  3.把数字签名加在Email正文后面，一起发送给B
  （当然，为了防止邮件被窃听你可以用继续公钥加密，这个不属于数字签名范畴）
  4.B收到邮件后用A的公钥对数字签名解密，成功则代表Email确实来自A，失败说明有人冒充
  5.B对邮件正文执行哈希运算得到hash值，取名h2
  6.B 会对比第4步数字签名的hash值h1和自己运算得到的h2，一致则说明邮件未被篡改。

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08fictf2AmYF2BkXG61zISKGO4QiaicXgnCpOMrrpvpauAhJxAVdPzgYiauA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看完这个过程，是不是觉得数字签名不过如此。其实就是利用算法（不一定是非对称算法）对原文hash值加密，然后附着到原文的一段数据。**数字签名的作用就是验证数据来源以及数据完整性！解密过程则称为数字签名验证。**
不过先别着急，我在梳理数字签名流程时候有下面几点疑惑，不知你也是否一样？

1.如果中间人同时篡改了Email正文和数字签名，那B收到邮件无法察觉啊。
答案：**数字签名的生成需要对方私钥，所以数字签名很难被伪造**。万一私钥泄漏了呢，不好意思，你私钥都能弄丢了那这篇文章当我白写。（私钥绝对保密不参与传输）
2.公钥是公开的并且可以自行导入到电脑，如果有人比如C偷偷在B的电脑用自己公钥替换了A的公钥，然后用自己的私钥给B发送Email，这时B收到邮件其实是被C冒充的但是他无法察觉。

答案：确实存在这种情况！解决办法就是数字证书，一环套一环请接着看。

### 数字证书

上面第2点描述的安全漏洞根源在哪？就是A的公钥很容易被替换！我举个更生动的例子引出“数字证书”：

- 你是一名基层人员经常向村委会邮寄机密材料，并盖章签名，你提前给村委会一台“盖章校验器”，告诉他们用这玩意扫一下 如果校验通过就证明材料来自我。
- 某天C偷偷潜入村委会替换了你的“盖章校验器”，那么他可以光明正大地冒充你了。
- 为了防止这种情况，村委会建议你把“盖章校验器”交给派出所保管，但是你得去派出所登记一下并领取身份证书，这样一来你每次邮寄材料除了盖章签名还需要附上派出所给的证书，村委会收到后先用证书去派出所取出你的“盖章校验器”，再像往常一样来校验你的签名盖章。
- OK，“盖章校验器”就相当于你的公钥，派出所其实就是CA认证中心，这里的身份证书就是数字证书。那么数字证书是怎么生成的呢？以及如何配合数字签名工作呢？

1.首先A去找"证书中心"（certificate authority，简称CA），为公钥做认证。**证书中心用自己的私钥，对A的公钥和一些相关信息一起加密，生成"数字证书**"（Digital Certificate）：

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF089cfPbErkWbkKnNPa7GXGLmbKRRuyZKUKmy8Om0iatbAK6mQApefYMnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2.**A在邮件正文下方除了数字签名，另外加上这张数字证书**

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08aVTI0mVv39WMvibX51jy9RrAVPDicGH4KZCcLSwUYdKiaPQlw76OiaCnRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3.B收到Email后用CA的公钥解密这份数字证书，拿到A的公钥，然后验证数字签名，后面流程就和图1的流程一样了，不再赘述。
和数字签名一样我在梳理这个流程时有下面几点疑惑：

- 假设数字证书被伪造了呢？
  答案：是的，传输中数字证书有可能被篡改。因此数字证书也是经过数字签名的，是不是感觉很绕貌似陷入了“鸡生蛋蛋生鸡”，我保证这是最后一个蛋- - ！上文说道数字签名的作用就是验证数据来源以及数据完整性！B收到邮件后可以**先验证这份数字证书的可靠性，通过后再验证数字签名**。
- 要是有1万个人要给B发邮件，难道B要保存1万份不同的CA公钥吗？
  答案：不需要，**CA认证中心给可以给B一份“根证书”，里面存储CA公钥来验证所有CA分中心颁发的数字证书。CA中心是分叉树结构，类似于公安部->省公安厅->市级派出所，不管A从哪个CA分支机构申请的证书，B只要预存根证书就可以验证下级证书可靠性。**
- 如何验证根证书可靠性？
  答案：无法验证。**根证书是自验证证书，CA机构是获得社会绝对认可和有绝对权威的第三方机构，这一点保证了根证书的绝对可靠。**如果根证书都有问题那么整个加密体系毫无意义。

### 举个栗子

上面一直在说虚拟场景，下文举个实际例子看看数字签名+数字证书如何验证文件的来源，以及文件的完整性。比如下载文件：我们开发中一般是服务端给文件信息加上md5,客户端下载完成后校验md5来判断文件是否损坏，这个其实就是简单的校验机制，而很多正规企业比如google都会给官方软件签署数字签名和证书，而windows已经预置了很多CA根证书：

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08arQvHNH03eTBHzkrAJ9zhg9m2EzXLozEWpHHwrIia2Dmqw9K0aOAMicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后看下我之前从网上下载的Chrome.exe，右键属性，通过鼠标点击一步验证：

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08cjLMt4Glm7a4OwrRtPtGygbNatXeCsY32cxia6wA7dzrL4ySmUMg9Zw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Google Inc就是google从CA中心申请的数字证书。这样看来，这个软件确实来源于google官方，并且文件完整。接下来我干点坏事，用notepad打开这个exe文件并且篡改里面的内容（修改二进制数据，09 改为33），保存：

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF083RZ7yiaCKoyYQhQEFSXBKueveEicibuwBtXRCwamCWAHxRMtPeO4cTic5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再看下数字签名还正常吗？

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08NrvpAtXIURURX2gy1dUZRWqQlwcZsBwv83z2BSwTYJAW9ria1tOklkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

文件被篡改导致数字签名无效，数字证书没有问题。

### https简单介绍

数字签名和数字证书可以用于文件，当然也能用于html网页数据。本人没有https相关开发经验，故不做深入探讨只是简单介绍下。

http的安全缺陷

**1.无法验证服务端的身份**
**2.无法保证数据完整性**
**3.无法保证数据传输不被窃听**

而https就是专门解决这三个问题，**https使用数字签名+数字证书解决了前2个问题**，很多大型网站比如baidu.com都会采用https协议，网址左侧会出现绿色加锁标识：

![img](https://mmbiz.qpic.cn/mmbiz_png/jE32KtUXy6EvtNCd4vhedqn7Qk5OPF08cicWMj1SUmaxeiabmnF4GBSrjy8kmljyzibNulE81fVTX7AyygOeIiaNnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击可以查看证书，另外**浏览器都会内置CA根证书，来对这些网站的服务器证书进行校验。**
**然后，再用SSL协议对传输通道加密，保证数据传输不被窃听，这个SSL加密原理分为很多步骤不在本文讨论范围。**

 



**大家都在看**



[一文读懂苹果发布会：三款新iPhone发布](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943596&idx=1&sn=2da77a45447f1eba78f4f226b1f472b7&chksm=84fd6048b38ae95ef538a30606be3b4e2cf0803f60df27a285aa4b73bfd759d1d8ac145100f6&scene=21#wechat_redirect)

[让你的EditText删除表情比微信更高效](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943581&idx=1&sn=37442901339654ca621836998d522075&chksm=84fd6079b38ae96f3ff450137f3ba1241702b9bcda54890ac9dc4418546c8ebfb8a393754d17&scene=21#wechat_redirect)

[Android 开发最佳实践经验分享](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943551&idx=1&sn=8e5780b818373fcfbc3965b5e862d9c4&chksm=84fd609bb38ae98d00f436c0a76cc1e7da3dce4876722c9320f1da09d015c160c5f055b1eb48&scene=21#wechat_redirect)

[ReactNative从零到完整项目-嵌入到安卓原生应用](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943592&idx=1&sn=c45b3139be7460020cd4ab67a366c61c&chksm=84fd604cb38ae95ad06e69960c98803ed8bc967fad7b39c8d81200a38d55e8422e3fe7f6dbf5&scene=21#wechat_redirect)