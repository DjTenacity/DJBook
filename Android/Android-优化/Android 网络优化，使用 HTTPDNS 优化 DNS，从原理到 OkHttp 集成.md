# [Android 网络优化，使用 HTTPDNS 优化 DNS，从原理到 OkHttp 集成](https://juejin.im/post/5c98482c5188252d9559247e)

## 一、前言

谈到优化，首先第一步，肯定是把一个大功能，拆分成一个个细小的环节，再单个拎出来找到可以优化的点，App 的网络优化也是如此。

在 App 访问网络的时候，DNS 解析是网络请求的第一步，默认我们使用运营商的 LocalDNS 服务。有数据统计，在这一块 3G 网络下，耗时在 200~300ms，4G 网络下也需要 100ms。

解析慢，并不是 LocalDNS 最大的问题，它还存在一些更为严重的问题，例如：DNS 劫持、DNS 调度不准确（缓存、转发、NAT）导致性能退化等等，这些才是网络优化最应该解决的问题。

 想要优化 DNS，现在最简单成熟的方案，就是使用 HTTPDNS。

今天就来聊聊，DNS、HTTPDNS，以及在 Android 下，如何使用 OKHttp 来集成 HTTPDNS。

## 二、DNS 和 HTTPDNS

### 2.1 什么是 DNS

在网络的世界中，每个有效的域名背后都有为其提供服务的服务器，而我们网络通信的首要条件，就是知道服务器的 IP 地址。

但是记住域名（网址）肯定是比记住 IP 地址简单。如果有某种方法，可以通过域名，查到其提供服务的服务器 IP 地址，那就非常方便了。这里就需要用到 DNS 服务器以及 DNS 解析。

**DNS（Domain Name System），它的作用就是根据域名，查出对应的 IP 地址**，它是 HTTP 协议的前提。只有将域名正确的解析成 IP 地址后，后面的 HTTP 流程才可以继续进行下去。

DNS 服务器的要求，一定是高可用、高并发和分布式的服务器。它被分为多个层次结构。

- 根 DNS 服务器		  ：返回顶级域 DNS 服务器的 IP 地址。
- 顶级域 DNS 服务器  ：返回权威 DNS 服务器的 IP 地址。
- 权威 DNS 服务器      ：返回相应主机的 IP 地址。

这三类 DNS 服务器，类似一种树状的结构，分级存在。

![img](https://user-gold-cdn.xitu.io/2019/3/25/169b2d9e6dd583db?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当开始 DNS 解析的时候，如果 LocalDNS 没有缓存，那就会向 LocalDNS 服务器请求（通常就是运营商），如果还是没有，就会一级一级的，从根域名查对应的顶级域名，再从顶级域名查权威域名服务器，最后通过权威域名服务器，获取具体域名对应的 IP 地址。

![img](https://user-gold-cdn.xitu.io/2019/3/25/169b2d9e6e29ed1c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

DNS 在提供域名和 IP 地址映射的过程中，其实提供了很多基于域名的功能，例如服务器的负载均衡，但是它也带来了一些问题

###  2.2 DNS 的问题

DNS 的细节还有很多，本文就不展开细说了，其问题总结来说就是几点。

#### *1.* **不稳定**

DNS 劫持或者故障，导致服务不可用。

#### *2.* **不准确**

**LocalDNS 调度，并不一定是就近原则**，某些小运营商没有 DNS 服务器，直接调用其他运营商的 DNS 服务器，最终直接跨网传输。例如：用户侧是移动运营商，调度到了电信的 IP，造成访问慢，甚至访问受限等问题。

#### *3.* **不及时**

运营商可能会修改 DNS 的 TTL(Time-To-Live，DNS 缓存时间)，导致 DNS 的修改，延迟生效。

还有运营商为了保证网内用户的访问质量，同时减少跨网结算，运营商会在网内搭建内容缓存服务器，通过把域名强行指向内容缓存服务器的地址，来实现本地本网流量完全留在本地的目的。

对此不同运营商甚至实现都不一致，这对我们来说就是个黑匣子。

![img](https://user-gold-cdn.xitu.io/2019/3/25/169b2d9e6e37434e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

正是因为 DNS 存在种种问题，所以牵出了 HTTPDNS。

 

## 三、 OKHttp 接入 HTTPDNS

既然了解了 HTTPDNS 的重要性，接下来看看如何在 OkHttp 中，集成 HTTPDNS。

OkHttp 是一个处理网络请求的开源项目，是 Android 端最火热的轻量级网络框架。在 OkHttp 中，默认是使用系统的 DNS 服务 InetAddress 进行域名解析。

```java
InetAddress ip2= InetAddress.getByName("www.cxmydev.com");
System.out.println(ip2.getHostAddress());
System.out.println(ip2.getHostName());

```

而想在 OkHttp 中使用 HTTPDNS，有两种方式。

1. 通过拦截器，在发送请求之前，将域名替换为 IP 地址。
2. 通过 OkHttp 提供的 `.dns()` 接口，配置 HTTPDNS。

对这两种方法来说，当然是推荐使用标准 API 来实现了。拦截器的方式，也建议有所了解，实现很简单，但是有坑。

### 3.1 拦截器接入方式

#### *1.* **拦截器接入**

拦截器是 OkHttp 中，非常强大的一种机制，它可以在请求和响应之间，做一些我们的定制操作。

在 OkHttp 中，可以通过实现 Interceptor 接口，来定制一个拦截器。使用时，只需要在 OkHttpClient.Builder 中，调用 `addInterceptor()` 方法来注册此拦截器即可。

OkHttp 的拦截器不是本文的重点，我们还是回到拦截器去实现 HTTPDNS 的话题上，拦截器没什么好说的，直接上相关代码。

```kotlin
class HTTPDNSInterceptor : Interceptor{
    override fun intercept(chain: Interceptor.Chain): Response {
        val originRequest = chain.request()
        val httpUrl = originRequest.url()

        val url = httpUrl.toString()
        val host = httpUrl.host()

        val hostIP = HttpDNS.getIpByHost(host)
        val builder = originRequest.newBuilder()

        if(hostIP!=null){
            builder.url(HttpDNS.getIpUrl(url,host,hostIP))
            builder.header("host",hostIP)
        }
        val newRequest = builder.build()
        val newResponse = chain.proceed(newRequest)
        return newResponse
    }
}

```

在拦截器中，使用 HttpDNS 这个帮助类，通过 `getIpByHost()` 将 Host 转为对应的 IP。

如果通过抓包工具抓包，你会发现，原本的类似 `http://www.cxmydev.com/api/user` 的请求，被替换为：`http://220.181.57.xxx/api/user`。

#### *2.* **拦截器接入的坏处**

**使用拦截器，直接绕过了 DNS 的步骤，在请求发送前，将 Host 替换为对应的 IP 地址。**

这种方案，在流程上很清晰，没有任何技术性的问题。但是这种方案存在一些问题，例如：**HTTPS 下 IP 直连的证书问题、代理的问题、Cookie 的问题等等。**

其中最严重的问题是，此方案（拦截器+HTTPDNS）**遇到 https 时，如果存在一台服务器支持多个域名，可能导致证书无法匹配的问题。**

在说到这个问题之前，就要先了解一下 HTTPS 和 SNI。



**HTTPS 是为了保证安全的，在发送 HTTPS 请求之前，首先要进行 SSL/TLS  握手，握手的大致流程如下：**

1. 客户端发起握手请求，携带随机数、支持算法列表等参数。
2. 服务端根据请求，选择合适的算法，下发公钥证书和随机数。
3. 客户端对服务端证书，进行校验，并发送随机数信息，该信息使用公钥加密。
4. 服务端通过私钥获取随机数信息。
5. 双方根据以上交互的信息，生成 Session Ticket，用作该连接后续数据传输的加密密钥。

在这个流程中，客户端需要验证服务器下发的证书。首先通过本地保存的根证书解开证书链，确认证书可信任，然后**客户端还需要检查证书的 domain 域和扩展域，看看是否包含本次请求的 HOST。**

在这一步就出现了问题，当使用拦截器时，请求的 URL 中，HOST 会被替换成 HTTPDNS 解析出来的 IP。当服务器存在**多域名和证书的情况下，服务器在建立 SSL/TLS 握手时，无法区分到底应该返回那个证书**，此时的策略可能返回默认证书或者不返回，这就有可能导致客户端在证书验证 domain 时，出现不匹配的情况，最终导致 SSL/TLS  握手失败。

这就引发出来 SNI 方案，SNI（Server Name Indication）是为了解决一个服务器使用多个域名和证书的 SSL/TLS 扩展。

**SNI 的工作原理，在连接到服务器建立 SSL 连接之前，先发送要访问站点的域名（hostname），服务器根据这个域名返回正确的证书。现在，大部分操作系统和浏览器，都已经很好的支持 SNI 扩展。**

#### *3.* **拦截器 + HTTPDNS 的解决方案**

这个问题，其实也有解决方案，这里简单介绍一下。

针对 "domain 不匹配" 的问题，可以通过 hook 证书验证过程中的第二步，将 IP 直接替换成原来的域名，再执行证书验证。

而 HttpURLConnect，提供了一个 HostnameVerifier 接口，实现它即可完成替换。

```java
public interface HostnameVerifier {
    public boolean verify(String hostname, SSLSession session);
}
```

如果使用 OkHttp，可以参考 OkHostnameVerifier (source://src/main/java/okhttp3/internal/tls/OkHostnameVerifier.java) 的实现，进行替换。

本身 **OkHttp 就不建议通过拦截器去做 HTTPDNS 的支**持，所以这里就不展开讨论了，这里只提出解决的思路，有兴趣可以研究研究源码。

### 3.2 OKHttp 标准 API 接入

OkHttp 其实本身已经暴露了一个 Dns 接口，默认的实现是使用系统的 InetAddress 类，发送 UDP 请求进行 DNS 解析。

我们只需要实现 OkHttp 的 Dns 接口，即可获得 HTTPDNS 的支持。

在我们实现的 Dns 接口实现类中，解析 DNS 的方式，换成 HTTPDNS，将解析结果返回。

```kotlin
class HttpDns : Dns {
    override fun lookup(hostname: String): List<InetAddress> {
        val ip = HttpDnsHelper.getIpByHost(hostname)
        if (!TextUtils.isEmpty(ip)) {
            //返回自己解析的地址列表
            return InetAddress.getAllByName(ip).toList() 
        } else {
            // 解析失败，使用系统解析
            return Dns.SYSTEM.lookup(hostname)
        }
    }
} 
```

使用也非常的简单，在 `OkHttp.build()` 时，通过 `dns()` 方法配置。

```kotlin
mOkHttpClient = httpBuilder
        .dns(HttpDns())
        .build();
```

这样做的好处在于：

1. 还是用域名进行访问，只是底层 DNS 解析换成了 HTTPDNS，以确保解析的 IP 地址符合预期。
2. HTTPS 下的问题也得到解决，证书依然使用域名进行校验。

OkHttp 既然暴露出 dns 接口，我们就尽量使用它。

## 四、小结时刻

现在大家知道，在做 App 的网络优化的时候，第一步就是使用 HTTPDNS 优化 DNS 的步骤。

所有的优化当然是以最终效果为目的，这里提两条大厂公开的数据，对腾讯的产品，在接入 HTTPDNS 后，用户平均延迟下降超过 10%，访问失败率下降超过五分之一。而百度 App 的 Feed 业务，Android 劫持率由 0.25% 降低到 0.05%。

此种优化方案，非常依赖 HTTPDNS 服务器，所以建议使用 阿里云、腾讯云 这样相对稳定的云服务商。

references:

【1】[百度App网络深度优化系列一 DNS 优化](https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FiaPtSF-twWz-AN66UJUBDg)

【2】SIN：https://blog.csdn.net/makenothing/article/details/53292335

【3】[鹅厂网事，HTTPDNS 服务详解](https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fu6-53Kp9Jb48dKWzaJOKig)

作者：承香墨影

链接：https://juejin.im/post/5c98482c5188252d9559247e

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。