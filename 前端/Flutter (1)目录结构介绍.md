#  Flutter   目录结构介绍及入门

[部分参考](https://mp.weixin.qq.com/s?__biz=MzIxMTg5NjQyMA==&mid=2247485542&idx=1&sn=50eb6af8ab777f921671d61348eefe70&chksm=974f196da038907bd855b22386c169c7b5fbf2a91bfacdab2e8d07a4a4ec9fffe175d36fcb55&mpshare=1&scene=1&srcid=&key=273f0e6221a1571ce05b57ee7de8878a1333859e12aca83730b21513b49ad37cb80342071fb828566b686e56514b67375ac5c5b0ce31fe00c046f7d1fe2acd1df3d4fd47484216612656313948f8b939&ascene=1&uin=OTQyMTg1MzE1&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=vjtAv3D03llfrVwfXQch6Pn1v%2BKlTaVQqhjBsfiTF%2B7JhkMoFtIeFVW4CwfJjUrM)

Flutter的嵌套真的很难受 , 如果不对UI布局进行模块拆分，那绝对是噩梦般的体验。而且不像web/rn开发样式可以单独抽离，Flutter这种将样式当做属性的处理方式，一眼看去真的很难理清dom结构，对于新接手代码的开发人员而言，需要费点时间理解。 

###  一、Flutter目录结构介绍

| 文件夹 | 作用 |
| ------ | ---- |
|    android |  android 平台相关代码         |
|       ios |    ios 平台相关代码     	|
|     lib   |   flutter 相关代码，我们主要编写的代码就在这个文件夹   |
|   test     |   用于存放测试代码    |
|   pubspec.yaml    |    配置文件，一般存放一些第三方库的依赖。    |

### **二、Flutter 入口文件、入口方法**

每一个 flutter 项目的 lib 目录里面都有一个 main.dart 这个文件就是 flutter 的入口文件 

main.dart 里面的 
```dart
void main() { 

	runApp(MyApp()); 

} 

也可以简写 
void main()=>runApp(MyApp()); 
```

其中的 main 方法是 dart 的入口方法。runApp 方法是 flutter 的入口方法。 

MyApp 是自定义的一个组件 

### 三、Flutter第一个Demo Center组件的使用

```dart
import 'package:flutter/material.dart';
void main() {
	runApp(Center(
		child: Text( "我是一个文本内容", 
                    textDirection: TextDirection.ltr, 
                   ),
    ));
}
```

# 

### 四、 Flutter   把内容单独抽离成一个组件 

在 Flutter 中自定义组件其实就是一个类，这个类需要继承 StatelessWidget/StatefulWidget 

前期我们都继承 StatelessWidget。后期给大家讲 StatefulWidget 的使用。 

**StatelessWidget** 是无状态组件，状态不可变的 widget 

**StatefulWidget** 是有状态组件，持有的状态可能在 widget 生命周期改变 



```dart
import 'package:flutter/material.dart';
void main(){
	runApp(MyApp());
}
class MyApp extends StatelessWidget{
	@override
	Widget build(BuildContext context) {
	// TODO: implement build
	return Center(
		child: Text( "我是一个文本内容", 
                    textDirection: TextDirection.ltr, ), );
	}
}
```

### 五、给 Text 组件增加一些装饰 

 ```dart
import 'package:flutter/material.dart';
void main(){
	runApp(MyApp());
}
class MyApp extends StatelessWidget{
	@override
	Widget build(BuildContext context) {
		// TODO: implement build
		return Center(
			child: Text( "我是 Dart 一个文本内容", 
			textDirection: TextDirection.ltr, 
			style:TextStyle(
			fontSize: 40.0, 
			fontWeight: FontWeight.bold, 
			// color: Colors.yellow
			color: Color.fromRGBO(255, 222, 222, 0.5)
			), ), );
	}
}
 ```

### 六、件用MaterialApp和Scaffold两个组件装饰App

 #### 1 、MaterialApp 
**MaterialApp** 是一个方便的 Widget，它封装了应用程序实现 Material Design 所需要的
一些 Widget。一般作为顶层 widget 使用。

​	**常用的属性：** 

​	home（主页） 

​	title（标题） 

​	color（颜色） 

​	theme（主题） 

​	routes（路由） 

​	... 

#### **2、Scaffold**

**Scaffold** 是 Material Design 布局结构的基本实现。此类提供了用于显示 drawer、
snackbar 和底部 sheet 的 API。
**Scaffold 有下面几个主要属性：**
appBar - 显示在界面顶部的一个 AppBar。
body - 当前界面所显示的主要内容 Widget。
drawer - 抽屉菜单控件。
...

```dart
import 'package:flutter/material.dart';
void main(){
	runApp(MyApp());
}
class MyApp extends StatelessWidget{
	@override
	Widget build(BuildContext context) {
		// TODO: implement build
		return MaterialApp(
			title:"我是一个标题", 
			home:Scaffold(
				appBar: AppBar(
					title:Text('IT  '),
                    elevation: 30.0, //设置标题阴影 不需要的话值设置成 0.0
				),
				body: MyHome(), 
				),
				theme: ThemeData( //设置主题颜色
					primarySwatch: Colors.yellow
				), );
		}
}
class MyHome extends StatelessWidget{
	@override
	Widget build(BuildContext context) {
		// TODO: implement build
		return Center(
			child: Text( 
				"我是 Dart 一个文本内容", 
				textDirection: TextDirection.ltr, 
				style: TextStyle(
					fontSize: 40.0, 
					fontWeight: FontWeight.bold, 
					color: Colors.black38
					// color: Color.fromRGBO(255, 222, 222, 0.5)
			), ), 
		);;
	}
}
```

## Container 组件、 Text 组件  

### Text组件

```dart
const Text(
  this.data, //显示的文本信息
{ Key key,
  this.style,
  this.textAlign,
  this.softWrap,
  this.overflow,
  this.maxLines,
})
```



|        名称         | 功能                                                         |
| :-----------------: | :----------------------------------------------------------- |
|    **textAlign**    | 文本对齐方式（center 居中，left 左 对齐，right 右对齐，justfy 两端对齐） |
|    textDirection    | 文本方向（ltr 从左至右，rtl 从右至 左）                      |
|    **overflow**     | 文字超出屏幕之后的处理方式（默认直接截断, clip 裁剪，fade 渐隐，ellipsis 省略号） |
| **textScaleFactor** | 字体显示倍率                                                 |
|    **maxLines**     | 文字显示最大行数 , 会根据`overflow`属性决定如何截断处理      |
|      **style**      | 文本样式 , Flutter提供了一个`TextStyle`类，最常用的`fontSize`，`fontWeight`，`color`，`backgroundColor`和`shadows`等属性都是通过它设置的； |
|      softWrap       | 文字是否换行                                                 |

**下面是 TextStyle 的参数 ：**

| 名称                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| **decoration**      | 文字装饰线（none 没有线，lineThrough 删除线，overline 上划线，underline 下划线） |
| **decorationColor** | double 两根线，solid 一根实线，wavy 波浪                     |
| **decorationStyle** | 文字装饰线风格（[dashed,dotted]虚线，double 两根线，solid 一根实线，wavy 波浪 |
| **wordSpacing**     | 单词间隙（如果是负值，会让单词变得更紧凑                     |
|        letterSpacing             | 字母间隙（如果是负值，会让字母变得更紧 凑）                  |
|         fontStyle            | 文字样式（italic 斜体，normal 正常体）                       |
|         fontSize            | 文字大小                                                     |
|        color             | 文字颜色                                                     |
| **fontWeight**      | 字体粗细（bold 粗体，normal 正常体）                         |
 有时还会遇到富文本的需求 , 我们可以用Flutter提供的`Text.rich`构造函数来创建相应的文本组件

```dart
Text.rich(TextSpan(
  children: [
    TextSpan(
      '￥',
      style: TextStyle(
        fontSize: 12,
        color: Color(0xFFFF7528),
      ),
    ),
    TextSpan(
      '258',
      style: TextStyle(
        fontSize: 15,
        color: Color(0xFFFF7528),
      ),
    ),
  ]
))
```



[更多参数](https://docs.flutter.io/flutter/painting/TextStyle-class.html)

### Container 组件

![img](https://mmbiz.qpic.cn/mmbiz_png/QFjUqsncFKkDicp9pSD5drz7xmFHs5A6d5xtNbJLxOIJTVwlER4cdia4DibS3HAU5ibrv8hicLYjSzLsLvId3CeibfQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

| 名称          | 功能 |
| ------------- | ---- |
| **alignment** | topCenter：顶部居中对齐<br/>topLeft：顶部左对齐<br/>topRight：顶部右对齐<br/>center：水平垂直居中对齐<br/>centerLeft：垂直居中水平居左对齐<br/>centerRight：垂直居中水平居右对齐<br/>bottomCenter 底部居中对齐<br/>bottomLeft：底部居左对齐<br/>bottomRight：底部居右对齐 |
| **decoration** | decoration: BoxDecoration( color: Colors.blue,<br/>border: Border.all(<br/>color: Colors.red,<br/>width: 2.0,<br/>),<br/>borderRadius:<br/>BorderRadius.all(<br/>Radius.circular(8.0)<br/>)<br/>) |
| **margin** | margin 属性是表示 Container 与外部其他<br/>组件的距离。<br/>EdgeInsets.all(20.0), |
| **padding** | padding 就 是 Container 的 内 边 距 ， 指<br/>Container 边缘与 Child 之间的距离<br/>padding: EdgeInsets.all(10.0) |
| **transform** | 让 Container 容易进行一些旋转之类的transform: Matrix4.rotationZ(0.2) |
| **height** |   容器高度   |
| **width** |   容器宽度    |
| **child** |   容器子元素   |



`Container`组件是最常用的布局组件之一，可以认为它是web开发中的`div`，rn开发中的`View`。其往往可以用来控制大小、背景颜色、边框、阴影、内外边距和内容排列方式等。我们先来看下其构造函数：

```dart
Container({
  Key key,
  double width,
  double height,
  this.margin,
  this.padding,
  //背景颜色，等同于web/rn中的backgroundColor.. 用Color(0xFFFF0000)或Colors.red
  Color color,
   //用来决定Container组件的子组件将以何种方式进行排列
  this.alignment,
  //盒约束 , 通过minWidth/maxWidth/minHeight/maxHeight等属性来限制容器的宽高
  BoxConstraints constraints,
  Decoration decoration,
  this.foregroundDecoration,
  this.transform,
  this.child,
})
```

#### transform

`transform`属性和我们在web/rn中经常用到的基本也没有差别，主要包括：平移，缩放、旋转和倾斜。在Flutter中，封装了矩阵变换类Matrix4帮助我们进行变换：

- `translationValues(x, y, z)`: 平移x, y, z；
- `rotationX(radians)`: x轴旋转radians弧度；
- `rotationY(radians)`: y轴旋转radians弧度；
- `rotationZ(radians)`: z轴旋转radians弧度；
- `skew(alpha, beta)`: x轴倾斜alpha度，y轴倾斜beta度；
- `skewX(alpha)`: x轴倾斜alpha度；
- `skewY(beta)`: y轴倾斜beta度；



#### decoration

该属性非常强大，字面意思是装饰，因为通过它你可以设置边框，阴影，渐变，圆角等常用属性。`BoxDecoration`继承自`Decoration`类，因此我们通常会生成一个`BoxDecoration`实例来设置这些属性。

##### 1) 边框

可以用`Border.all`构造函数直接生成4条边框，也可以用Border构造函数单独设置不同方向上的边框。不过令人惊讶的是官方提供的边框竟然不支持`虚线`（issue在这里）。

```
// 同时设置4条边框：1px粗细的黑色实线边框
BoxDecoration(
  border: Border.all(color: Colors.black, width: 1, style: BorderStyle.solid)
)

// 设置单边框：上边框为1px粗细的黑色实线边框，右边框为1px粗细的红色实线边框
BoxDecoration(
  border: Border(
    top: BorderSide(color: Colors.black, width: 1, style: BorderStyle.solid),
    right: BorderSide(color: Colors.red, width: 1, style: BorderStyle.solid),
  ),
)
```

##### 2) 阴影

阴影属性和web中的`boxShadow`几乎没有区别，可以指定`x`，`y`，`blur`，`spread`，`color`等属性。

```
BoxDecoration(
  boxShadow: [
    BoxShadow(
      offset: Offset(0, 0),
      blurRadius: 6,
      spreadRadius: 10,
      color: Color.fromARGB(20, 0, 0, 0),
    ),
  ],
)
```

##### 3) 渐变

如果你不想容器的背景颜色是单调的，可以尝试用`gradient`属性。Flutter同时支持线性渐变和径向渐变：

```
// 从左到右，红色到蓝色的线性渐变
BoxDecoration(
  gradient: LinearGradient(
    begin: Alignment.centerLeft,
    end: Alignment.centerRight,
    colors: [Colors.red, Colors.blue],
  ),
)

// 从中心向四周扩散，红色到蓝色的径向渐变
BoxDecoration(
  gradient: RadialGradient(
    center: Alignment.center,
    colors: [Colors.red, Colors.blue],
  ),
)
```

##### 4) 圆角

通常情况下，你可能会用到`BorderRadius.circular`构造函数来同时设置4个角的圆角，或是`BorderRadius.only`构造函数来单独设置某几个角的圆角：

```dart
// 同时设置4个角的圆角为5
BoxDecoration(
  borderRadius: BorderRadius.circular(5),
)

// 设置单圆角：左上角的圆角为5，右上角的圆角为10
BoxDecoration(
  borderRadius: BorderRadius.only(
    topLeft: Radius.circular(5),
    topRight: Radius.circular(10),
  ),
)
```

`Container`组件的属性很丰富，虽然有些用法上和web/rn有些许差异，但基本上大同小异，所以过渡起来也不会有什么障碍。另外，由于`Container`组件是单子节点组件，也就是只允许子节点有一个。所以在布局上，很多时候我们会用`Row`和`Column`组件进行`行/列`布局。



# Flutter图片组件

### 一、Flutter  图片组件 

```dart
Image({
  Key key,
    /***最常用到主要有两种（AssetImage和NetworkImage）。使用AssetImage之前，需要在pubspec.yaml文件中声明好图片资源，然后才能使用；而NextworkImage指定图片的网络地址即可，主要是在加载一些网络图片时会用到**/
  @required this.image,
  this.width,
  this.height,
  // 图片的背景颜色，当网络图片未加载完毕之前，会显示该背景颜色；
  this.color,
  this.fit,
  this.repeat = ImageRepeat.noRepeat,
})
//mage.network和Image.asset构造函数，其实是语法糖。比如下方的两段代码结果是完全一样的：
Image(
  image: NetworkImage('https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1402367109,4157195964&fm=27&gp=0.jpg'),
  width: 100,
  height: 100,
)

Image.network(
  'https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1402367109,4157195964&fm=27&gp=0.jpg',
  width: 100,
  height: 100,
)
```

 

图片组件是显示图像的组件，Image 组件有很多构造函数，这里我们只给大家讲两个
Image.asset， 本地图片
Image.network 远程图片

 **Image**  组件的常用属性 : 

|   名称     |  类型    |   说明   |
| ---- | ---- | ---- |
|   alignment   |  Alignment    |   图片的对齐方式   |
|   color 和 colorBlendMode   |      | 设置图片的背景颜色，通常和 colorBlendMode 配合使用，这样可以是图片颜色和背景色混合。上面的图是进行了颜色的混合，绿色背景和图片红色的混合 |
| fit | BoxFit | fit 属性用来控制图片的拉伸和挤压，这都是根据父容器来的。<br/>BoxFit.fill:全图显示，图片会被拉伸，并充满父容器。<br/>BoxFit.contain:全图显示，显示原比例，可能会有空隙。<br/>BoxFit.cover：显示可能拉伸，可能裁切，充满（图片要充满整个容器，还不变形）。<br/>BoxFit.fitWidth：宽度充满（横向充满），显示可能拉伸，可能裁切。<br/>BoxFit.fitHeight ：高度充满（竖向充满）,显示可能拉伸，可能裁切。<br/>BoxFit.scaleDown：效果和 contain 差不多，但是此属性不允许显示超过源图片大小，可小不可大。 |
|   repeat   |   平铺   | 决定当图片实际大小不足指定大小时是否使用重复效果<br>ImageRepeat.repeat : 横向和纵向都进行重复，直到铺满整个画布<br/>ImageRepeat.repeatX: 横向重复，纵向不重复。<br/>ImageRepeat.repeatY：纵向重复，横向不重复。 |
| width | 宽度 | 一般结合 ClipOval 才能看到效果 |
|  height    |   高度   |  一般结合 ClipOval 才能看到效果    |

   

更多属性参考：https://api.flutter.dev/flutter/widgets/Image-class.html

```dart
return Center(
	child: Container(
		child: Image.network( 
			"http://pic.baike.soso.com/p/20130828/20130828161137-1346445960.jpg", 						alignment: Alignment.topLeft, 
				color: Colors.red, 
				colorBlendMode: BlendMode.colorDodge, 
				// repeat: ImageRepeat.repeatX, 
				fit: BoxFit.cover, 
			), 
			width: 300.0, 
			height: 400.0, 
			decoration: BoxDecoration(
				color: Colors.yellow
			), 
		), 
);
```

### 二、Flutter **引入本地图片**

![](C:\Users\admin\Desktop\DJGitBook\前端\img\1562147840.jpg)

然后，打开 pubspec.yaml 声明一下添加的图片文件，**注意要配置对**

```dart
flutter:
  uses-material-design: true
  assets:
  	-imgages/a.jpeg
  	-imgages/2.0x/a.jpeg
  	-imgages/3.0x/a.jpeg
```

在代码中就可以用了

```dart
child: Container(
	child: Image.asset("images/a.jpeg", 
	fit:BoxFit.cover
	),
	width: 300.0, 
	height: 300.0, 
	decoration: BoxDecoration(
		color: Colors.yellow
),),
```

###  三、 Flutte 实现圆角以及实现圆形图片 

```dart
return Center(
	child: Container(
		child:ClipOval(
		child:Image.network("https://www.itying.com/images/201905/
thumb_img/1101_thumb_G_1557845381862.jpg", 
		width: 150.0, 
		height: 150.0, 
		) , )
), );
```

### 四、Icon(图标组件)

![img](https://mmbiz.qpic.cn/mmbiz_png/QFjUqsncFKkDicp9pSD5drz7xmFHs5A6dOfSbWharfPom1Cot4RDdNtXYfkDLibiadOKV9JuloLlCEdHqHrt2bc7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`Icon`图标组件相比于图片有着放大不会失真的优势，在日常开发中也是经常会被用到。Flutter更是直接内置了一套`Material`风格的图标（你可以在这里预览所有的图标类型）。看下构造函数：

```dart
const Icon(
  this.icon, {
  Key key,
  this.size,
  this.color,
})
```

- `icon`: 图标类型；
- `size`: 图标大小；
- `color`: 图标颜色。

# ListView 基础列表组件、水平列表组件、图标组件

### 一、Flutter 列表组件概述
列表布局是我们项目开发中最常用的一种布局方式。Flutter 中我们可以通过 ListView 来定义
列表项，支持垂直和水平方向展示。通过一个属性就可以控制列表的显示方向。列表有一下
分类：
1、垂直列表
2、垂直图文列表
3、水平列表
4、动态列表
5、矩阵式列表

### 二、Flutter 列表参数

| 名称 | 类型 | 说明 |
| ---- | ---- | ---- |
|   scrollDirection   |   Axis   |   Axis.horizontal 水平列表<br>Axis.vertical 垂直列表   |
|  padding    |    EdgeInsetsGeometry  |   内边距   |
|  resolve    |  bool    |   组件反向排序   |
|  组件反向排序    |   组件反向排序   |   列表元素   |

  

  ### 三、Flutter 基本列表

```dart
class HomeContent extends StatelessWidget {
	@override
	Widget build(BuildContext context) {
	// TODO: implement build
		return Center(
			child: ListView(
				children: <Widget>[
					ListTile(
						leading: Icon(Icons.phone), 
						title: Text("this is list",style: TextStyle(fontSize: 28.0),), 
						subtitle: Text('this is list this is list'), ),
					ListTile(
						title: Text("this is list"), 
						subtitle: Text('this is list this is list'), 
						trailing: Icon(Icons.phone), ),
					ListTile(
						title: Text("this is list"), 
						subtitle: Text('this is list this is list'), ),
					ListTile(
							title: Text("this is list"), 
							subtitle: Text('this is list this is list'), ),
					ListTile(
							title: Text("this is list"), 
							subtitle: Text('this is list this is list'), )],
                ));
	}
}
```

### 四、Flutter 水平列表

```dart
class HomeContent extends StatelessWidget {
	@override
	Widget build(BuildContext context) {
		// TODO: implement build
		return Container(
			height:200.0, 
			margin: EdgeInsets.all(5), 
			child:ListView(
				scrollDirection: Axis.horizontal, 
				children: <Widget>[
					Container(
						width:180.0, 
						color: Colors.lightBlue, 
						),
					Container(
						width:180.0, 
						color: Colors.amber, 
						child: ListView(
							children: <Widget>[
								Image.network(
'https://resources.ninghao.org/images/childhood-in-a-picture.jpg' ),
								SizedBox(height: 16.0), 
								Text('这是一个文本信息', 
									textAlign: TextAlign.center, 															
									style: TextStyle(fontSize: 16.0
								), )
							], ),),
					Container(
						width:180.0, 
						color: Colors.deepOrange, ),
					Container(
						width:180.0, 
						color: Colors.deepPurpleAccent, ),
                ], ));
          }
}
```

### 五、Flutter 列表组件概述
Flutter 中我们可以通过 ListView 来定义
列表项，支持垂直和水平方向展示。通过一个属性就可以控制列表的显示方向。列表有一下
分类：
1、垂直列表
2、垂直图文列表
3、水平列表
4、动态列表
5、矩阵式列表（网格布局）

### 六、Flutter GridView 组件的常用参数
当数据量很大的时候用矩阵方式排列比较清晰。此时我们可以用网格列表组件 GridView 实
现布局。
GridView 创建网格列表有多种方式，下面我们主要介绍两种。
    1、可以通过 GridView.count 实现网格布局
    2、通过 GridView.builder 实现网格布局
**常用属性：**

| 名称             | 类型                                                         | 说明                                  |
| ---------------- | ------------------------------------------------------------ | ------------------------------------- |
| scrollDirection  | Axis                                                         | 滚动方法                              |
| padding          | EdgeInsetsGeometry                                           | 内边距                                |
| resolve          | bool                                                         | 组件反向排序                          |
| crossAxisSpacing | double                                                       | 水平子 Widget 之间间距                |
| mainAxisSpacing  | double                                                       | 垂直子 Widget 之间间距                |
| crossAxisCount   | int                                                          | 一行的 Widget 数量                    |
| childAspectRatio | double                                                       | 子 Widget 宽高比例                    |
| children         |                                                              | <Widget>[ ]                           |
| gridDelegate     | SliverGridDelegateWithFix<br>edCrossAxisCount（常用）<br>SliverGridDelegateWithMax<br>CrossAxisExtent<br> | 控制布局主要用在GridView.builder 里面 |

## Flutter 页面布局 

 Paddiing Row Column Expanded 组件详解 

#### **一、Flutter Paddiing 组件** 

在 html 中常见的布局标签都有 padding 属性，但是 Flutter 中很多 Widget 是没有 padding 属 性。这个时候我们可以用 Padding 组件处理容器与子元素直接的间距。 

`Row`和`Column`组件其实和web/rn中的Flex布局（弹性盒子）特别相似 , `Row`组件的主轴就是横向，`Column`组件的主轴就是纵向。且它们的构造函数十分相似

+ padding  --> padding值 ,Edgelnsetss设置填充的值
+ child	--> 子组件



#### 二、Flutter Row 水平布局组件

+ mainAxisAlignment 	主轴的排序方式 , 默认都是从start开始  从左到右

+ crossAxisAlignment 	次轴的排序方式   默认都是居中

+ children 			   组件子元素

+ ##### mainAxisSize   是在主轴上的尺寸 ,是包裹其内容，还是撑满其父容器。它的可选值有`MainAxisSize.min`和`MainAxisSize.max`。由于其默认值都是`MainAxisSize.max`，所以主轴方向上默认大小都是尽可能撑满父容器的



#### 三、Flutter Column 垂直布局组件

由于`Column`组件次轴方向上（即水平）默认是居中对齐，所以水平方向上不会撑满其父容器，此时需要指定`CrossAxisAlignment.stretch`才可以

+ mainAxisAlignment 	主轴的排序方式  , 默认都是从start开始  从上到下

+ crossAxisAlignment 	次轴的排序方式  默认都是居中

+ children 			  组件子元素  

+ ##### mainAxisSize    是在主轴上的尺寸 ,是包裹其内容，还是撑满其父容器。它的可选值有`MainAxisSize.min`和`MainAxisSize.max`。由于其默认值都是`MainAxisSize.max`，所以主轴方向上默认大小都是尽可能撑满父容器的

#### 四、Flutter Expanded 类似 Web 中的 Flex布局

Expanded 可以用在 Row 和 Column 布局中

+ flex 	  元素站整个父 Row /Column 的比例
+ child 	子元素



#### 五、Flutter Stack 组件

Stack 表示堆的意思，我们可以用 Stack 或者 Stack 结合 Align 或者 Stack 结合 Positiond 来实现页面的定位布局  绝对定位布局在web/rn开发中也是使用频率较高的一种布局方式，Flutter也提供了相应的组件实现，需要将`Stack`和`Positioned`组件搭配在一起使用 . Stack组件就是绝对定位的容器

属性 说明

+ alignment 配置所有子元素的显示位置
+ children 子组件

##### Stack Align

Stack 组件中结合 Align 组件可以控制每个子元素的显示位置
属性 
+ alignment 配置所有子元素的显示位置
+ child 子组件
  

##### Stack Positioned

Stack 组件中结合 Positioned 组件也可以控制每个子元素的显示位置

Stack组件就是绝对定位的容器，`Positioned`组件通过`left`，`top`，`right`，`bottom`四个方向上的属性值来决定其在父容器中的位置。

+ top 子元素距离顶部的距离
+ bottom 子元素距离底部的距离
+ left  子元素距离左侧距离
+ right 子元素距离右侧距离
+ child 子组件

```dart
Container(
  height: 100,
  color: Colors.yellow,
  child: Stack(
    children: <Widget>[
      Positioned(
        left: 10,
        top: 10,
        child: Container(width: 10, height: 10, color: Colors.red),
      ),
      Positioned(
        right: 10,
        top: 10,
        child: Container(width: 10, height: 10, color: Colors.red),
      ),
      Positioned(
        left: 10,
        bottom: 10,
        child: Container(width: 10, height: 10, color: Colors.red),
      ),
      Positioned(
        right: 10,
        bottom: 10,
        child: Container(width: 10, height: 10, color: Colors.red),
      ),
    ],
  ),
)
```



#### 六、Flutter AspectRatio 组件
​	AspectRatio 的作用是根据设置调整子元素 child 的宽高比。

​	AspectRatio 首先会在布局限制条件允许的范围内尽可能的扩展，widget 的高度是由宽度和比率决定的，类似于 BoxFit 中的 contain，按照固定比率去尽量占满区域。

​	如果在满足所有限制条件过后无法找到一个可行的尺寸，AspectRatio 最终将会去优先适应布局限制条件，而忽略所设置的比率。

+ aspectRatio 宽高比，最终可能不会根据这个值去布局，
  具体则要看综合因素，外层是否允许按照这
  种比率进行布局，这只是一个参考值
+ child 子组件



#### 七、Flutter Card 组件

Card 是卡片组件块，内容可以由大多数类型的 Widget 构成，Card 具有圆角和阴影，这让它看起来有立体感。

+ margin 外边距
+ child 子组件
+ Shape Card 的阴影效果，默认的阴影效果为圆角的长方形边。

#### 八、Flutter RaisedButton 定义按钮
Flutter 中通过 RaisedButton 定义一个按钮。RaisedButton 里面有很多的参数，这一讲我们只
是简单的进行使用。
```dart
return RaisedButton(
child: Text('女装'), textColor: Theme.of(context).accentColor, onPressed: (){
}, );
```

#### 九、可以实现流布局的Wrap 组件
Wrap 可以实现流布局，单行的 Wrap 跟 Row 表现几乎一致，单列的 Wrap 则跟 Row 表
现几乎一致。但 Row 与 Column 都是单行单列的，Wrap 则突破了这个限制，mainAxis 上空
间不足时，则向 crossAxis 上去扩展显示。

+ direction 主轴的方向，默认水平
+ alignment 主轴的对其方式
+ spacing 主轴方向上的间距
+ textDirection 文本方向
+ verticalDirection 定义了 children 摆放顺序，默认是 down，见Flex 相关属性介绍。
+ runAlignment run 的对齐方式。run 可以理解为新的行或者列，如果是水平方向布局的话，run 可以理解为新的一行 
+ runSpacing run 的间距

## Flutter 中自定义有状态组件
在 Flutter 中自定义组件其实就是一个类，这个类需要继承 StatelessWidget/StatefulWidget。

**StatelessWidget** 是无状态组件，状态不可变的 widget
**StatefulWidget** 是有状态组件，持有的状态可能在 widget 生命周期改变。通俗的讲：如果我们想改变页面中的数据的话这个时候就需要用到 **StatefulWidget**



##  BottomNavigationBar 自定义底部导航条、以及实现页面切换

BottomNavigationBar 是底部导航条，可以让我们定义底部 Tab 切换，bottomNavigationBar
是 Scaffold 组件的参数。

+ items List<BottomNavigationBarItem> 底部导航条按钮集合
+ iconSize icon
+ currentIndex 默认选中第几个
+ onTap 选中变化回调函数
+ fixedColor 选中的颜色
+ type 
  	+ BottomNavigationBarType.fixed
	+ BottomNavigationBarType.shifting