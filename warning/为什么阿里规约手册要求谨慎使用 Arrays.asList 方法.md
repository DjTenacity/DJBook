## [为什么阿里规约手册要求谨慎使用 Arrays.asList 方法](https://mp.weixin.qq.com/s?__biz=MzI2OTQ4OTQ1NQ==&mid=2247486586&idx=1&sn=00ec99aacd96149fcc9bc7f7f7f8c36d&chksm=eadec83adda9412cafddf2d29ae747cd9b199fe852857af73df510fdac6664274366b5115403&scene=0&xtrack=1&key=f2cbae3bb612f42ed4a6df5cdb7e840e1cda99605cc9f5b607855e54c8940e690b60e6370179796647f7c888173df31f5c5e4fe99ab4d0f87855cf6f5daa796913b46c184af6f452e583531ec1de374e&ascene=1&uin=OTQyMTg1MzE1&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=LquZw%2F7ZlEREZk5Jg%2FChEs14H4F3tgPgTk7SWbbtjj7%2BjivbtY4rRd0IuoPEHqil)

[链接](https:*//www.jianshu.com/p/2a62bd40677c)：*

# **前言**



在开发中，有时候会碰到把多个参数，或者说把数组转成List的需求，通常我们会使用 Arrays.asList()方法。但是该方法在使用的过程中，稍有不慎就会出现严重的异常。有如下代码：



```java
@Test
public void test(){    
	List<String> list = Arrays.asList("a", "a", "2");    
	System.out.println(list.size());
    list.add("blog.happyjava.cn");
    System.out.println(list.size());
 }

```

运行之后，出现了异常：

![img](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjERibqkiaYTy2nlugTgZjepfKHBE3tFkfHWvKVIkxlMmiboS2Q4kficdibJ6O14GRESBaKVUDEcqmWJSEA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在阿里Java规约中有强制性的要求：使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常。

![img](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjERibqkiaYTy2nlugTgZjepfKibbFVCpsjX74qQGvZKus9CNLproOicicWq0bHo4yVe4MX2oXgkL4gWYOQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

阿里规约里已经提示了asList返回的对象是Arrays的一个内部类。那么这个内部类，跟我们一般用到的List（如ArrayList）有什么不一样之处呢，下面我们就来分析下。



# **Arrays.asList()源码分析**



通过IDEA查看该方法源码，如下：

![img](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjERibqkiaYTy2nlugTgZjepfKbDgueN8QFPX83kzhWrBv92LkrzuytRm6cUXWu2mVd6xmYqTHMAGq2w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里返回了一个ArrayList，看起来似乎没什么问题，但是这个ArrayList跟我们常用的java.util.ArrayList不一样。通过IDEA点击跳转，可以看到该ArrayList是Arrays的一个内部类。

![img](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjERibqkiaYTy2nlugTgZjepfKQJw0uoev1NCibItKkLNibjJ3mgnryEfsRJDeKe9YdrzzXa2G9vQco8cg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

该内部类的源码其实不多，通过IDEA的structure，我们可以看到它实现的方法如下：

![img](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjERibqkiaYTy2nlugTgZjepfKjYYIlHyGXKbNHwbyZt7RgEgLERparKT1UFib9lxH3Sn2Fw3GIDzic3pg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到，这里是没有实现我们最常用的add方法的。

那么，调用add等方法的时候，UnsupportedOperationException 异常是哪里抛出来的呢？我们看它继承的java.util.AbstractList类，该类的add方法如下：

```java
public boolean add(E e) {
	add(size(), e);
    return true;
}

```

这里有一个重载的add方法，再点进去查看：



```java
public void add(int index, E element) {    
    throw new UnsupportedOperationException();
}

```

可以看到，是这里抛出了UnsupportedOperationException。

#  

# **总结**



Arrays.asList()是开发中非常常用的方法，所以我们一定要了解其存在的坑点。如果把其返回的ArrayList当做了我们常用的java.util.ArrayList，那么是很容易埋下生产隐患的。





**推荐阅读：**

[你一定得知道的 Google Java 编程风格指南](http://mp.weixin.qq.com/s?__biz=MzI2OTQ4OTQ1NQ==&mid=2247486560&idx=1&sn=db6804fd8c5cfa736bb8cd7113f90682&chksm=eadec820dda94136b3d9a0ef285d361bcedf1e159a479ae9e5c20881cd30ed230385204c8b16&token=1003531771&lang=zh_CN&scene=21#wechat_redirect)！

[Spring AOP 中 JDK 和 CGLib 动态代理哪个更快？](http://mp.weixin.qq.com/s?__biz=MzI2OTQ4OTQ1NQ==&mid=2247486554&idx=1&sn=727f1255b5c78c6bf819c893ded57fce&chksm=eadec81adda9410cab7284f8248e7d960c15d6bcd409d175ee4361a2db34af4062a4d2ba91b7&token=1003531771&lang=zh_CN&scene=21#wechat_redirect)

[SpringBoot+JWT+Shiro+MybatisPlus实现Restful快速开发后端脚手架](http://mp.weixin.qq.com/s?__biz=MzI2OTQ4OTQ1NQ==&mid=2247486552&idx=1&sn=628ea0603ed3ca9f9c74c161e4815d85&chksm=eadec818dda9410e263c3b0783997b2989c634014677cf10b99e0917d76e18b00a57db6bf661&token=1003531771&lang=zh_CN&scene=21#wechat_redirect)