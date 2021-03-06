###  Android-春招-[面试经历](https://www.jianshu.com/p/840688b02c7f)与劝谏-2019年

## 感叹一声

​		耗时两个月的找工作经历终于是画上句号了，几十个日日夜夜的酸甜苦辣只有裸辞的亲尝者才能体会到吧，下面想来复盘或者说总结一下这段经历。但不管怎么总结，核心还是那一句话：**一定要充分的准备！！！涉及到的知识点一项都不漏的复习一遍（至于深浅程度主要看自己平时的积累了），一则能很大程度的提高自信（不需要担心被面试官问倒）；二则面对问到的知识点时还可以扩展的说清楚该知识点在安卓或者Java体系中的关系和逻辑。**否则后续因为复习不到位而与自己理想的公司或岗位失之交臂时你会感到些懊恼。

 

## 简历准备阶段

可以说面试官了解你的唯一入口就是简历了，因此一般都是根据你写的项目经验为入口来展开来问知识点。这个时间段应该要去认真阅读自己参与过的项目代码，归纳出技术点、解决的问题以及最终达到的效果来完善项目经验。另外要注意：简历一定要写自己比较熟悉的技术点，因为简历作为一个入口，面试官很可能会层层深入的询问你，如果不是特别熟悉就可能会招架不住。
 **反例**：我在简历里写了一个`WebView`与`RecycleView`滑动冲突的项目经历，有好几个面试官犀利的看到在`fling()`滑行过程中会出现在交界处位置停顿的问题，继续问到这个情况要怎么解决，如果没有扩展去了解MD控件（`CoordinatorLayout`+ `Behavior`）滑动机制的话可能就回答不上来了。甚至还有面试官继续追问道：如果要你自己基于`onTouchEvent()`这个事件机制来解决这个问题要如何做，通过三个层次的追问就给问蒙了，囧。

## 初期准备阶段

刚开始两个礼拜还能耐得住性子在家里老老实实的复习，一个知识点一个知识点的过。第三个礼拜便开始着急了，觉得这样复习太慢有点浪费时间。于是草草把没有复习完的内容快速过了一遍，着急开始找前同事和猎头推简历，面试机会确实是来了，而且是一线互联网公司。结果可想而知都很不理想：阿里的第一轮电话面试就败下阵来、oppo勉强冲到第二轮也没能再过关。看到问题后于是停止了推简历，又老老实实的复习剩下的知识点，并做好复习笔记。虽然从失败中总结到了经验，但白白浪费了机会，得不偿失。
 该阶段复习可以参考知识点列表：<https://www.jianshu.com/p/0f82b0650909>，基本涵盖到安卓和Java的绝大部分的基础知识点了，后续阶段的复习也可以参考这里的知识点。
 另外一个总结得不错的列表可以作为补充：<https://lrh1993.gitbooks.io/android_interview_guide/content>。

## 中期阶段

过完前面的基础知识点后，这个阶段主要是去熟悉源码了。可以结合项目中用到的开源框架有针对性的阅读下源码，面试过程中一般会根据你在项目中用到的框架，询问你对这些框架的原理是否熟练掌握。通用框架一般无外乎网络库、图片库、工具类、插件化或热更新库等。这些知识点应该平时多去积累和练习为好，此时只要稍加复习即可。如果不是特别熟悉的可以去参考别人总结比较好的文章对着源码梳理，一定要在自己脑中形成知识结构，基本的实现细节要陈述出来。
 另外就是平常用到的安卓原生代码一起要去熟悉下，比如：消息机制、View的绘制流程、Binder通信、java集合、并发等。觉得这些更像是基础吧，没什么好说的。理解的越深对自己平常的运用越有帮助。

## 后期阶段

有了上面两个阶段的准备后，前两轮的基础面试基本没什么问题了。如果目标岗位是**资深开发**或者**架构师**的话，一般还会问到更底层原理和更抽象的宏观层面问题。
 底层原理方面：比如虚拟机的内存区域和gc流程、tcp的流量和拥塞控制、https建立连接的交互流程等，这里可以去找对应的技术文章熟悉了解。
 宏观层面：一般是架构模式（MVC、MVP、MVVM）、开发模式（模块化、组件化、模块组件化）以及设计模式相关问题，要能熟练掌握到灵活运用的层度，并总结出它们之间的异同特点。
 另一大块就是**算法**了，某些一线公司比较喜欢考，比如今日头条在面试邮件中就明确指明要考算法。因此要对标你的目标公司是不是要考来进行复习。具体考哪些内容，以我面试的那些历程来看，基本都没超出[《剑指offer》](https://lrh1993.gitbooks.io/android_interview_guide/content/algorithm/For-offer.html)那六十几道题的范围（可能有对应题的变形），因此花一个礼拜左右的时间把那六十几道题弄懂并自己动手实现一遍基本ok，当然一些基础算法还要自己认真去总结学习，比如排序、二分查找、链表和树的基本操作等。

 

## 面试经历

主要是根据回忆总结的（会有遗漏点）。

### 1. 腾讯（QQ音乐）

感觉不擅长互动较少的电话面，很难快速暖场，问题回答确实不好。当时还在复习的初级阶段内容都没看完，也是一部分原因，基本算是草草收场吧。问得比较多的是优化相关的问题。

### 2. 支付宝（海外版）

仍是电话面的，还是没有找到感觉，回答不在状态。最后猎头反馈的本次面评是：过往项目功能较简单、某些技术细节掌握不到位。算是浪费了机会。

### 3. 今日头条

是所有参加的面试里比较专业的面试体验吧，面试官体现了很好的技术素养。总共参加了3轮视频面试（技术面全部面完），开始还以为很有希望（感觉面得不错，基本没有阻塞的知识点），最终结果却是挂掉了，原因目前未知。当然这里也花了很长时间准备（3个礼拜左右），主要是因为要考算法，重头复习了算法，还把所有知识点重头捋了一遍。

主要考察的问题大体有如下一些：
 android:

1. webview加载h5的优化（问得很多）
2. 优化相关（包大小、启动优化、卡顿），webp的透明通道如何处理，代码压缩如何处理
3. native-jni相关；如何分析native的崩溃堆栈；
4. 进程保活
5. 插件化原理
6. 网络优化（答的不好）
7. https，fiddler抓包怎么处理的，为什么fiddler可以篡改https的数据。header中的host作用是什么
8. 懂不懂kotlin （直接说不会了）
9. activity的4种启动模式，A启动B时activity的两个acitivity的生命周期的流程是什么
10. 如何监听activity是从后台切换到了前台？不是在onResume()中处理
11. push进程的挂掉之后，再次拉起时如何恢复push进程中service的状态
12. 两个进程如何使用binder进行双向通信
13. 如何监测普通对象是否泄漏，leakcanary在dump时卡顿厉害，这里该如何优化
14. listview和recycleview的差别
15. WebSocket、socket、NIO
16. 对app架构的理解
17. activity的启动流程
18. 两个页面的消息同步怎么做？（类似以前评论sdk的在子评论页面点赞时，back回来后需要同步这个点赞状态）

java:

1. 类加载流程
2. 类的唯一标识是什么
3. gc流程
4. map都有哪些类型，特点是什么，hashmap内部结构
5. 动态代理跟静态代理区别，如何代理一个普通对象

算法：

1. 合并两个有序链表，使得最终有序。
2. Map<String, Integer> map，按value平方的升序打印key-value。
3. 1000万个0～100之间的小数，找top100。如果是保留两位小数，怎么做？

### 4. akulaku

整体的面试体验还是不错的，面试官技术素养也不错。一下午走完了所有流程（2轮技术1轮HR）。该公司应该也算是创业公司吧，有阿里的投资，内部很多产品线，主打东南亚电商和金融，目前算是个爆发增长期。

记忆比较深刻的问题有：

1. retrofit的动态代理中是如何处理接口返回类型的（因为接口申明的泛型在运行时会被擦除）
2. 在做项目架构时封装的BaseActivity/BaseFragment一般要放哪些对象
3. Binder整体的运行逻辑是怎样的（要能说出底层的大概原理）

### 5. 恒信永利

公司没什么名气，做互联网金融。但从面试过程来看技术实力还是相当不错，当时面的是架构师岗。准备不是太充分很多问题回答的确实不好（应该只复习到中期阶段），整体来讲也还可以。而且在这之后就开始慢慢找到自信了。

记忆比较深刻的问题有：

1. 架构的核心是要解决什么问题，怎样才称为好的架构
2. 桥接模式属于什么类型的设计模式（结构型模式），它是用来解决什么场景的问题
3. 画一下IM系统中用户A给用户B发送消息时数据包和信令包的交互图。
4.  `CurrentHashMap`的实现有没有看过源码，说一下它优化并发的原理。
5. 对CAS的理解，用你熟悉的并发方式实现一下生产者-消费者模式并评估它的效率。

### 6. oppo（应用商店）

公司就不说了，整体是很不错的。面试体验也还可以，参加的是他们周末的招聘会，面了2轮技术面。感觉是第二轮群面的时候表现得不好，有细节没准备充分。有点浪费机会了的感觉，导致后面也没有机会面其他部门了。

比较深刻的似乎大都是跟View相关的问题：

1. View体系中动画的绘制原理。（没有答到点上，需要看View中对动画处理的那部分源码）
2. 上面的提到的滑动冲突中`fling()`的停顿问题如何解决
3. 有哪些方式可以实现滚动一个子View
4. activity的启动流程

### 7. vivo（大数据中心-埋点sdk）

公司也不说了，整体都挺好的。总共面了3轮技术面+HR面，由于前面面经的积累，目前来看基本没再碰到应用层面的难题。

印象较深的面试题有：

1. 说一说系统ANR的实现原理，平常是如何解ANR问题的，如果在发生ANR时trace文件打印的堆栈是`MessageQueue.nativePollOnce()`处阻塞该如何定位具体问题在哪
2. 如何实现热更新的不需要重启进程就生效
3. activity的启动流程
4. 说一下乐观锁和悲观锁
5. 单例模式在实际使用中有什么缺点

### 8. 顺丰科技（快运）

公司平台还是不错的，而且是今年发展的核心业务，团队还在建设中，发展前景还是很不错的。只是工作内容会比较偏重业务，看个人对工作的兴趣吧。

由于是重业务的工作性质，面试问得最多的就是数据库相关的：

1. 如何做数据库表结构的升级。
2. 客户端数据库SQLite的极限是可以存多少数据，如果数据量特别大，如何优化数据的查询效率。
3. 画一下IM系统客户端的数据库表的结构图，实体要包含用户信息、单人会话、群聊。
4. 弱网状态下如何保证数据的最终的可用性和可靠性（http的缓存和压缩）。

### 9. 万兴科技

是一家创业版的上市公司，公司技术能力还不错，主打一款视频编辑软件，因此对标的技术点就是多媒体相关开发（c/c++、音视频、opengl）。感觉公司的商业模式比较奇怪，竟然现在还在卖软件license授权，可能是因为目标市场在国外的缘故吧。不过对多媒体开发有兴趣的还是值得试试。

## 总结

其实客户端开发的知识点就那么多，面试问来问去还是那么点东西。所以面试没有其他的诀窍，只看你对这些知识点准备的充分程度。so，出去面试时先看看自己复习到了哪个阶段就好。

作者：JarryWell

链接：https://www.jianshu.com/p/840688b02c7f

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。



Akulaku感觉面试官很刁钻，上来就拿个笔试题在问，印象比较深的Message obtain跟直接new的区别 ：本质是Message obtain是采用享元模式，所以效率会高一些
Rxjava操作符设计模式 ：看了一下源码其实就是装饰器模式
Gson泛型擦除，hashmap容量为什么是2的几次幂，归并排序，线程池如何保证核心线程不被销毁