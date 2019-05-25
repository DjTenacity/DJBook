[引用](https://www.cnblogs.com/jieran/p/8296617.html)

## [Python2中的urllib、urllib2与Python3中的urllib以及第三方模块requests]
  在python2中，urllib和urllib2都是接受URL请求的相关模块，但是提供了不同的功能。两个最显著的不同如下：
#### 1、urllib2可以接受一个Request类的实例来设置URL请求的headers，例如：
```
req = urllib2.Request(  
[python] view plain copy
        url=url,  
        data=postdata,  
        headers=headers  
)  
result = urllib2.urlopen(req)
```
  我们知道，HTTP是无连接的状态协议，但是客户端和服务器端需要保持一些相互信息，比如cookie，有了cookie，服务器才能知道刚才是这个用户登录了网站，才会给予客户端访问一些页面的权限。所以我们需要保存cookie，之后附带cookie再来访问网站，才能够达到效果。
  这里就需要Python的cookielib和urllib2等的配合，将cookielib绑定到urllib2在一起，就能够在请求网页的时候附带cookie。在构造req请求之前可以获取一个保存cookies的对象，并把该对象和http处理器、http的handler资源以及urllib2的对象绑定在一起
```
cj = cookielib.LWPCookieJar()  
cookie_support = urllib2.HTTPCookieProcessor(cj)  
# 创建一个opener，将保存了cookie的http处理器，还有设置一个handler用于处理http的URL的打开  
opener = urllib2.build_opener(cookie_support, urllib2.HTTPHandler)  
# 将包含了cookie、http处理器、http的handler的资源和urllib2对象板顶在一起  
urllib2.install_opener(opener) 
```
#### 2、urllib仅可以接受URL。这意味着，你不可以伪装你的User Agent字符串等。
但是urllib提供urlencode方法用来GET查询字符串的产生，而urllib2没有。这是就是为何urllib常和urllib2一起使用的原因，如下：
`postdata = urllib.urlencode(postdata)`
（把字典形式的postdata编码一下）
Tip: if you are planning to do HTTP stuff only, check out httplib2, it is much better than httplib or urllib or urllib2.

》》》》》》》》》》》》》》》》》》》》》》》》》》》》》》》》》

下面说说Python3x中的urllib包、http包以及其他比较好使的第三方包

#### 1、Python3 urllib、http

Python3不像2x中酷虎的和服务器模块结构散乱，Python3中把这些打包成为了2个包，就是http与urllib，详解如下：

http会处理所有客户端--服务器http请求的具体细节，其中：
（1）client会处理客户端的部分
（2）server会协助你编写Python web服务器程序
（3）cookies和cookiejar会处理cookie，cookie可以在请求中存储数据
使用cookiejar示例可以在前几篇博客中基于Python3的爬虫中找到示例，如下：
```
import http.cookiejar  
import urllib.request  
import urllib.parse</span></span>  
def getOpener(head):  
    # deal with the Cookies  
    cj = http.cookiejar.CookieJar()  
    pro = urllib.request.HTTPCookieProcessor(cj)  
    opener = urllib.request.build_opener(pro)  
    header = []  
    for key, value in head.items():  
        elem = (key, value)  
        header.append(elem)  
    opener.addheaders = header  
    return opener
```
urllib是基于http的高层库，它有以下三个主要功能：
（1）request处理客户端的请求
（2）response处理服务端的响应
（3）parse会解析url
下面是使用Python3中urllib来获取资源的一些示例：
```
# 1、最简单  
import urllib.request  
response = urllib.request.urlopen('http://python.org/')  
html = response.read() 

# 2、使用 Request  
import urllib.request  
req = urllib.request.Request('http://python.org/')  
response = urllib.request.urlopen(req)  
the_page = response.read()

# 3、发送数据  
import urllib.parse  
import urllib.request  
url = '"  
values = {  
'act' : 'login',  
'login[email]' : '',  
'login[password]' : ''  
}  
data = urllib.parse.urlencode(values)  
req = urllib.request.Request(url, data)  
req.add_header('Referer', 'http://www.python.org/')  
response = urllib.request.urlopen(req)  
the_page = response.read()  
print(the_page.decode("utf8"))

# 4、发送数据和header  
import urllib.parse  
import urllib.request  
url = ''  
user_agent = 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'  
values = {  
'act' : 'login',  
'login[email]' : '',  
'login[password]' : ''  
}  
headers = { 'User-Agent' : user_agent }  
data = urllib.parse.urlencode(values)  
req = urllib.request.Request(url, data, headers)  
response = urllib.request.urlopen(req)  
the_page = response.read()  
print(the_page.decode("utf8"))

# 5、http 错误  
import urllib.request  
req = urllib.request.Request(' ')  
try:  
urllib.request.urlopen(req)  
except urllib.error.HTTPError as e:  
print(e.code)  
print(e.read().decode("utf8"))

# 6、异常处理1  
from urllib.request import Request, urlopen  
from urllib.error import URLError, HTTPError  
req = Request("http://www..net /")  
try:  
response = urlopen(req)  
except HTTPError as e:  
print('The server couldn't fulfill the request.')  
print('Error code: ', e.code)  
except URLError as e:  
print('We failed to reach a server.')  
print('Reason: ', e.reason)  
else:  
print("good!")  
print(response.read().decode("utf8"))

# 7、异常处理2  
from urllib.request import Request, urlopen  
from urllib.error import  URLError  
req = Request("http://www.Python.org/")  
try:  
response = urlopen(req)  
except URLError as e:  
if hasattr(e, 'reason'):  
print('We failed to reach a server.')  
print('Reason: ', e.reason)  
elif hasattr(e, 'code'):  
print('The server couldn't fulfill the request.')  
print('Error code: ', e.code)  
else:  
print("good!")  
print(response.read().decode("utf8"))

# 8、HTTP 认证  
import urllib.request  
# create a password manager  
password_mgr = urllib.request.HTTPPasswordMgrWithDefaultRealm()  
# Add the username and password.  
# If we knew the realm, we could use it instead of None.  
top_level_url = ""  
password_mgr.add_password(None, top_level_url, 'rekfan', 'xxxxxx')  
handler = urllib.request.HTTPBasicAuthHandler(password_mgr)  
# create "opener" (OpenerDirector instance)  
opener = urllib.request.build_opener(handler)  
# use the opener to fetch a URL  
a_url = ""  
x = opener.open(a_url)  
print(x.read())  
# Install the opener.  
# Now all calls to urllib.request.urlopen use our opener.  
urllib.request.install_opener(opener)  
a = urllib.request.urlopen(a_url).read().decode('utf8')  
print(a)

# 9、使用代理  
import urllib.request  
proxy_support = urllib.request.ProxyHandler({'sock5': 'localhost:1080'})  
opener = urllib.request.build_opener(proxy_support)  
urllib.request.install_opener(opener)  
a = urllib.request.urlopen("").read().decode("utf8")  
print(a)

# 10、超时  
import socket  
import urllib.request  
# timeout in seconds  
timeout = 2  
socket.setdefaulttimeout(timeout)  
# this call to urllib.request.urlopen now uses the default timeout  
# we have set in the socket module  
req = urllib.request.Request('')  
a = urllib.request.urlopen(req).read()  
print(a)
```
上面例子大概把常用的一些情况都罗列出来了，其中对异常的处理要严格按照：

 

try...exceptA...exceptB...except...else...finally...

的语法格式来写，详情请参考我的另一篇相关博文

》》》》》》》》》》》》》》》》》》》》》》》》

2、除了使用官方标准库的urllib，我们可以使用更好用的第三方模块，如requests

Requests 完全满足如今网络的需求，其功能有以下：

+ 国际化域名和 URLs
+ Keep-Alive & 连接池
+ 持久的 Cookie 会话
+ 类浏览器式的 SSL 加密认证
+ 基本/摘要式的身份认证
+ 优雅的键/值 Cookies
+ 自动解压
+ Unicode 编码的响应体
+ 多段文件上传
+ 连接超时
+ 支持 .netrc
+ 适用于 Python 2.6—3.4
+ 线程安全
### 附加：

+ 在python2.x中raw_input( )和input( )，两个函数都存在，其中区别为
	+ raw_input( )---将所有输入作为字符串看待，返回字符串类型
	+ input( )-----只能接收“数字”的输入，在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型（ int, float ）

+ 在python3.x中raw_input( )和input( )进行了整合，去除了raw_input( )，仅保留了input( )函数，其接收任意任性输入，将所有输入默认为字符串处理，并返回字符串类型。