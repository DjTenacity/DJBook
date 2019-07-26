## Http知道这些，开发Android才算合格！

### **前言**

说起HTTP大家再熟悉不过了，无论是大学的课本上还是平时的工作中，几乎每天都要和HTTP打交道。但是，就是这么熟悉的老朋友，你真的是非常了解吗？你能轻而易举就回答出我下面的几个问题吗？

① OSI模型分为哪几层？分别是什么作用？
② Http协议属于哪一层？与HTTP同一层的协议你还能列举哪些协议？
③ 说出5种以上的HTTP请求方法



是不是看起来非常简单，思考一下，能准确回答吗？
对方不想和你说话，并向你扔了一张图：
 



![img](https://mmbiz.qpic.cn/mmbiz_png/uZTP36mmecBYrLibvg16e1icMcXs5LCDXDYYRKeEibednicDCJ9Y7wSNbEcEF5vCPUl7XgibPtJrs4Q9ibwh0PEU2UHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **1.定义**

对于前面我提到的问题，我相信对于有些同学还是有点遗忘的，这很正常。就是在我们公司里面，服务端的一些很不错的同事，你突然问他OSI的分层和对应作用，他们也不能完全说的明明白白。对于上面的第一个问题其实你大可不必都要说的清清楚楚的。但是很多和我日常工作息息相关的，或者说关系比较密切的我们还是需要了解清楚的。



简单来讲，HTTP协议它就是一种让我们**可以通过本地的工具(浏览器、网络爬虫等等工具)访问远端（服务器）上的资源的一种协议**。
举一个例子：



![img](https://mmbiz.qpic.cn/mmbiz_png/uZTP36mmecBYrLibvg16e1icMcXs5LCDXD0GwgthtmvYJtYkJU9GwjvEMp8CPpXACzgELB0Mzze9N2m4jmlzTl2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP协议有哪些特点呢？

**简单快速**：HTTP报文能够被人读懂，还允许简单测试，降低了门槛，  对新人很友好



**灵活**：在 HTTP/1.0 中出现的 HTTP headers 让协议扩展变得非常容易。只要服务端和客户端就新 headers 达成语义一致，新功能就可以被轻松加入进来。



**无连接**：一个连接是由传输层来控制的，这从根本上不属于HTTP的范围。HTTP并不需要其底层的传输层协议是面向连接的，只需要它是可靠的，或不丢失消息的（至少返回错误）。**在互联网中，有两个最常用的传输层协议：TCP是可靠的，而UDP不是。因此，HTTP依赖于面向连接的TCP进行消息传递，但连接并不是必须的。**



**无状态**：在同一个连接中，两个执行成功的请求之间是没有关系的。这就带来了一个问题，用户没有办法在同一个网站中进行连续的交互，比如在一个电商网站里，用户把某个商品加入到购物车，切换一个页面后再次添加了商品，这两次添加商品的请求之间没有关联，浏览器无法知道用户最终选择了哪些商品。而使用HTTP的头部扩展，HTTP Cookies就可以解决这个问题。把Cookies添加到头部中，创建一个会话让每次请求都能共享相同的上下文信息，达成相同的状态。
注意，HTTP本质是无状态的，使用Cookies可以创建有状态的会话。



### **2.报文**

2.1 请求报文
哈哈通常来说，我们的Request是由几个部分组成：请求行、请求头部、一个空行、实体内容
看一下简单的请求的报文结构：
 



![img](https://mmbiz.qpic.cn/mmbiz_png/uZTP36mmecBYrLibvg16e1icMcXs5LCDXDRZgZRKCJxFfbJbmQZSNvjXbTyqRmmunROwzBr1vAQLJWU5Swc084Hg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



请求由以下元素组成：



- 一个HTTP的method，经常是由一个动词像`GET`, `POST` 或者一个名词像`OPTIONS`，`HEAD`来定义客户端的动作行为。通常客户端的操作都是获取资源（GET方法）或者发送HTML form表单值（POST方法），虽然在一些情况下也会有其他操作。

- 要获取的资源的路径，通常是上下文中就很明显的元素资源的URL，它没有protocol（`http://`），domain（`developer.mozilla.org`），或是TCP的port（HTTP一般在80端口）。

- HTTP协议版本号。

- 为服务端表达其他信息的可选头部headers。

- 对于一些像POST这样的方法，报文的body就包含了发送的资源，这与响应报文的body类似。

  

2.2 返回报文

![img](https://mmbiz.qpic.cn/mmbiz_png/uZTP36mmecBYrLibvg16e1icMcXs5LCDXDK1rvictU3XEmQutRQcfVIYet8CGNofrdviaIdLHAahGlRLx6iazZDKQAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



响应报文包含了下面的元素：



- HTTP协议版本号。
- 一个状态码（status code），来告知对应请求执行成功或失败，以及失败的原因。
- 一个状态信息，这个信息是非权威的状态码描述信息，可以由服务端自行设定。
- HTTP headers，与请求头部类似。
- 可选项，比起请求报文，响应报文中更常见地包含获取的资源body。

HTTP是一种简单可扩展的协议，其Client-Server的结构以及轻松扩展头部信息的能力使得HTTP可以和Web共同发展。



即使HTTP/2为了提高性能将HTTP报文嵌入到帧中这一举措增加了复杂度，但是从Web应用的角度看，报文的基本结构没有变化，从HTTP/1.0发布起就是这样的结构。会话流依旧简单，通过一个简单的 HTTP message monitor就可以查看和纠错。附带接口返回码的意义对照：
 

​       1xx：指示信息——表示请求已接收，继续处理

　　2xx：成功——表示请求已经被成功接收，理解，接受

　　3xx：重定向——要完成请求必须进行更进一步的操作

　　4xx：客户端错误——请求有语法错误或请求无法实现

　　5xx：服务器端错误——服务器未能实现合法的请求



### **3 关于Cookie**

HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

3.1 创建Cookie
Cookie就是服务器按照约定的形式将需要客户端（一般指浏览器）缓存的数据放到响应头里面。浏览器收到后便缓存起来。一般缓存的信息有Cookie的过期时间、域、路径、适用站点等等信息。一般的格式如下：

```
Set-Cookie: <cookie名>=<cookie值>
```



一个简单的例子，当服务器第一次返回时：

- 
- 
- 
- 
- 

```
HTTP/1.0 200 OKContent-type: text/htmlSet-Cookie: yummy_cookie=chocoSet-Cookie: tasty_cookie=strawberry[页面内容]
```





当浏览器第二次访问同一站点时，会将Cookie的信息带到请求头里面：

- 
- 
- 

```
GET /sample_page.html HTTP/1.1Host: www.example.orgCookie: yummy_cookie=choco; tasty_cookie=strawberry
```



3.2 两种类型的Cookie

**会话期Cookie**：如果不设置Cookie的过期时间，那么Cookie只保存在当前浏览器分配的内存中，浏览器被关闭时Cookie就失效了。但不是绝对的，有些浏览器会有会话恢复的功能，就算浏览器被关闭，下次打开时会自动全恢复，包括Cookie。



**持久性Cookie**：与会话期Cookie不同的是，持久性Cookie一般可以设置一个过期时间（Expires）或者有效时间（Max-Age），一般以文件的形式存在硬盘中。

- 

```
Set-Cookie: id=a3fWa; Expires=Wed, 12 Oct 2019 07:28:00 GMT;

```

3.3 安全性

**当机器处于不安全环境时，切记不能通过HTTP Cookie存储、传输敏感信息。**

 

标记为 `Secure` 的Cookie只应通过被HTTPS协议加密过的请求发送给服务端。但即便设置了`Secure` 标记，敏感信息也不应该通过Cookie传输，因为Cookie有其固有的不安全性，`Secure `标记也无法提供确实的安全保障。从 Chrome 52 和 Firefox 52 开始，不安全的站点（`http:`）无法使用Cookie的 `Secure` 标记。



为避免跨域脚本 (XSS) 攻击，通过JavaScript的 `Document.cookie` API无法访问带有 `HttpOnly` 标记的Cookie，它们只应该发送给服务端。如果包含服务端 Session 信息的 Cookie 不想被客户端 JavaScript 脚本调用，那么就应该为其设置 `HttpOnly` 标记。

- 

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```



关于Cookie，欧盟已经在2009/136/EC指令中提了相关要求，该指令已于2011年5月25日生效。虽然指令并不属于法律，但它要求欧盟各成员国通过制定相关的法律来满足该指令所提的要求。当然，各国实际制定法律会有所差别。



该欧盟指令的大意：在征得用户的同意之前，网站不允许通过计算机、手机或其他设备存储、检索任何信息。自从那以后，很多网站都在网站声明中添加了相关说明，告诉用户他们的Cookie将用于何处。