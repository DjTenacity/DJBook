# 为什么flutter使用dart作为编程语言

**AOT（ahead of time）静态编译**，AOT程序的典型代表是用C/C++开发的应用，它们必须在执行前编译成机器码。
**JIT(just in time)动态编译**。JIT的代表则非常多，如JavaScript、python等。



译自 [Flutter FAQ](https://flutter.io/faq/#why-did-flutter-choose-to-use-dart)

[Flutter主体上有四个评价维度](https://www.jianshu.com/p/e27d6f064416 )，主要考虑框架作者、开发者和用户的需要。我们发现有些语言能够满足一些需求，但是Dart在所有的评价维度中得分最高，能够满足我们的所有需求和标准。

Dart运行时和编译器支持两种Flutter关键特性的结合：基于JIT和hot-reloads

 能够缩减开发周期，加之Ahead-of-Time（AOT）编译器生成的高效ARM代码， 能够为产品部署提供快速启动能力和可预期的性能指标。

此外我们和Dart社区紧密合作，他们在积极的投入资源以推动Dart在Flutter中的应用。例如，Dart语言本身并没有生成本地二进制码的ahead-of-time工具链，这是获取高性能的重要因素，但是现在Dart语言支持了，因为Dart团队为Flutter打造了一套共工具链。同样地，Dart虚拟机此前在吞吐量方面被优化过，如今Dart团队在优化延迟，这对Flutter来说更加重要。

在以下几个重要指标上，Dart表现卓著：

- 面向对象
   对于Flutter，我们想要一个能够符合Flutter问题域的语言，即**创造视觉用户体验，通过面向对象语言构建用户界面框架**，业内已经有了几十年的经验。当然我们可以使用非面向对象的语言，这将意味着重复发明轮子来解决几个艰难的问题。此外，大多数的开发者已经拥有面向对象的开发经验，这使得Flutter开发更加易学。
   
+ 开发效率高
  
   Dart运行时和编译器支持Flutter的两个关键特性的组合：
   
   基于JIT的快速开发周期：Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间；
   
   基于AOT的发布包: Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能。而JavaScript则不具有这个能力。
   
+ 高性能
  
   Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。
   
   
   
   Flutter高性能主要靠两点来保证，
   
   + 首先，Flutter APP采用Dart语言开发。Dart在 JIT（即时编译）模式下，速度与 JavaScript基本持平。但是 Dart支持 AOT，当以 AOT模式运行时，JavaScript便远远追不上了。速度的提升对高帧率下的视图数据计算很有帮助。
   
   + 其次，Flutter使用自己的渲染引擎来绘制UI，布局数据等由Dart语言直接控制，所以在布局过程中不需要像RN那样要在JavaScript和Native之间通信，这在一些滑动和拖动的场景下具有明显优势，因为在滑动和拖动过程往往都会引起布局发生变化，所以JavaScript需要和Native之间不停的同步布局信息，这和在浏览器中要JavaScript频繁操作DOM所带来的问题是相同的，都会带来比较可观的性能开销。 
   
   
   
+ 快速内存分配
   Flutter框架使用**函数式流，它重度依赖底层内存分配器对小量的、短生命周期内存分配的有效处理**，在缺乏这种特性的语言中Flutter无法有效地工作。

+ 类型安全  

  由于Dart是**类型安全的语言，支持静态类型检测**，所以可以在编译前发现一些类型的错误，并排除潜在问题，这一点对于前端开发者来说可能会更具有吸引力。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。相比之下，Dart本身就支持静态类型，这是它的一个重要优势。

+ Dart团队就在你身边
  
  看似不起眼，实则举足轻重。由于有Dart团队的积极投入，Flutter团队可以获得更多、更方便的支持 

作者：_苏丽君_

链接：https://www.jianshu.com/p/139b9003817c 
