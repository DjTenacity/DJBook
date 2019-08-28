##  [Android客户端路由总线设计](https://iluhcm.com/2017/07/12/design-of-router-using-in-android/)

当一个产品的业务规模上升到一定程度，或者是跨团队开发时，团队/模块间的合作问题就会暴露出来。

+ 如何保持团队间业务的往来？

+ 如何互不影响或干涉对方的开发进度？

+ 如何调用业务方的功能？

组件化给上述问题提供了一个答案。
组件化所要解决的核心问题是**解耦**，路由正是为了解决模块间的解耦而出现的。

### 传统的页面跳转

页面跳转主要分为三种，App页面间跳转、H5跳转回App页面以及App跳转至H5

#### **App页面间跳转**

这种方式的不足之处是当包含多个模块，但模块间没有相互依赖时，这时候的跳转会变得相当困难。

如果已知其他模块的类名以及对应的路径，可以通过Intent.setComponent(Component)方法启动其他模块的页面，但往往模块的类名是有可能变化的，一旦业务方把模块换个名字，这种隐藏的Bug对于开发的内心来说是崩溃的。另一方面，这种重复的模板代码，每次至少写两行才能实现页面跳转，代码存在冗余。



#### **H5-App页面跳转**

对于考拉这种**电商应用，活动页面具有时效性和即时性**，这两种特性在任何时候都需要得到保障。运营随时有可能更改活动页面，也有可能要求点击某个链接就能跳转到一个App页面。

传统的做法是对WebViewClient.shouldOverrideUrlLoading(WebView, String)进行拦截，判断url是否有对应的App页面可以跳转，然后取出url中的params封装成一个Intent传递并启动App页面。

```java
public static Intent startActivityByUrl(Context context, String url, boolean fromWeb, boolean outer) {
    if (StringUtils.isNotBlank(url) && url.startsWith(StringConstants.REDIRECT_URL)) {  
        try {
            String realUrl = Uri.parse(url).getQueryParameter("target");
            if (StringUtils.isNotBlank(realUrl)) {
                url = URLDecoder.decode(realUrl, "UTF-8");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    Intent intent = null;
    try {
        Uri uri = Uri.parse(url);
        String host = uri.getHost();
        List<String> pathSegments = uri.getPathSegments();
        String path = uri.getPath();
        int segmentsLength = (pathSegments == null ? 0 : pathSegments.size());
        if (!host.contains(StringConstants.KAO_LA)) {
            return null;
        }
        if((StringUtils.isBlank(path))){
            do something...
            return intent;
        }
        if (segmentsLength == 2 && path.startsWith(StringConstants.JUMP_TO_GOODS_DETAIL)) {
            do something...
        } else if  {
            //...............海了去了的else if
            do something...
        } 
    } catch (Exception e) {
        e.printStackTrace();
    }
    if (intent != null && !(context instanceof Activity)) {
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    }
    return intent;
}
```

这种做法的弊端在于：

- 判断不合理。 
- 耦合性太强。已知拦截的所有页面的引用都必须能够拿到，否则无法跳转；
- 代码混乱。PATH非常多，从众多的PATH中匹配多个已知的App页面，想必要判断匹配规则就要写很多函数解决；
- 拦截过程不透明。开发者很难在URL拦截的过程中加入自己的业务逻辑，如打点、启动Activity前添加特定的Flag等；
- 没有优先级概念，也无法降级处理。同一个URL，只要第一个匹配到App页面，就只能打开这个页面，无法通过调整优先级跳转到别的页面或者使用H5打开。

#### **App页面-H5跳转**

 启动一个WebViewActivity即可。



### 页面路由的意义

路由最先被应用于网络中，路由的定义是**通过互联的网络把信息从源地址传输到目的地址的活动**。

页面跳转也是相当于从源页面跳转到目标页面的过程，每个页面可以定义为一个统一资源标识符（URI），在网络当中能够被别人访问，也可以访问已经被定义了的页面。

路由常见的使用场景有以下几种：

1. App接收到一个通知，点击通知打开App的某个页面（OuterStartActivity）

2. 浏览器App中点击某个链接打开App的某个页面（OuterStartActivity）

3. App的H5活动页面打开一个链接，可能是H5跳转，也可能是跳转到某一个native页面（WebViewActivity）

4. 打开页面需要某些条件，先验证完条件，再去打开那个页面（需要登录）

5. App内的跳转，可以减少手动构建Intent的成本，同时可以统一携带部分参数到下一个页面（打点）

除此之外，使用路由可以避免上述弊端，能够降低开发者页面跳转的成本。

## 路由总线

#### 路由框架

![](http://nos.netease.com/knowledge/0d94b9af-bb78-4a7c-9322-e23bcfec551c)

考拉路由框架主要分为三个模块：**路由收集**、**路由初始化**以及**页面路由**。

+ 路由收集阶段，定义了基于Activity类的注解，通过`Android Processing Tool`(以下简称“APT”)收集路由信息并生成路由表类；
+ 路由初始化阶段，根据生成的路由表信息注入路由字典；
+ 页面路由阶段，则通过路由字典查找路由信息，并根据查找结果定制不同的路由策略。

### 路由设计思路

总的来说，考拉路由设计追求的是功能模块的解耦，能够实现基本路由功能，以及开发者使用上足够简单。考拉路由的前两个阶段对于路由使用者几乎是无成本的，只需要在使用路由的页面定义一个类注解`@Router`即可，页面路由的使用也相当简单，后面会详细介绍。

#### 功能设计

路由在一定程度上和网络请求是类似的，可以分为请求、处理以及响应三个阶段。这三个阶段对使用者来说既可以是透明的，也可以在路由过程中进行拦截处理。考拉路由框架目前支持的功能有：

1. 支持基本Activity的启动，以及startActivityForResult回调；✅
2. 支持不同协议执行不同跳转；（kaola://*、http(s)://*、native://*等）✅
3. 支持多个SCHEME/HOST/PATH跳转至同一个页面；（(pre.)*.kaola.com(.hk)）✅
4. 支持路由的正则匹配；✅
5. 支持Activity启动使用不同的Flag；✅
6. 支持路由的优先级配置；✅
7. 支持对路由的动态拦截、监听以及降级；✅

以上功能保证了考拉业务模块间的解耦，也能够满足目前产品和运营的需求。



#### 接口设计

![](http://nos.netease.com/knowledge/130829f3-7355-44ee-af48-b2a7e8ffbbae)

一个好的模块或框架，需要事先设计好接口，预留足够的权限供调用者支配，才能满足各种各样的需求。



**针对接口编程，而不是针对实现编程**

这条规则排在最前面的原因是，针对接口编程，不管是对开发者还是对使用者，真的是**百利而无一害**。在路由版本迭代的过程中，底层对接口无论实现怎样的修改，也不会影响到上层调用。对于业务来说，路由的使用是无感知的。

**不随意暴露不必要的API**

“要区别设计良好的模块与设计不好的模块，最重要的因素在于，这个模块对于外部的其他模块而⾔言，是否隐藏其内部数据和其他实现细节。设计良好的模块会隐藏所有的实现细节，把它的API与它的实现清晰地隔离开来。然后，模块之间只通过它们的API进行通信，一个模块不需要知道其他模块的内部工作情况。这被称为封装(encapsulation)。”（摘自Effective Java, P58）

**单一职责**

无论是类还是方法，均需要遵循单一职责原则。一个类实现一个功能，一个方法做一件事

**提供默认实现**

针对接口编程的好处是随时可以替换实现，考拉路由框架在路由过程中的所有监听、拦截以及路由过程都提供了默认的实现。使用者即可以不关心底层的实现逻辑，也可以根据需要替换相关的实现。

### 考拉路由实现原理

考拉路由框架基于注解收集路由信息，通过APT实现路由表的动态生成，类似于ButterKnife的做法，在运行时导入路由表信息，并通过正则表达式查找路由，根据路由结果实现最终的页面跳转。

https://iluhcm.com/2017/07/12/design-of-router-using-in-android/