# **Flutter** **目录结构介绍**

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

# Container 组件、 Text 组件详解 

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

### 七、**Text** **组件**

|        名称         | 功能                                                         |
| :-----------------: | :----------------------------------------------------------- |
|    **textAlign**    | 文本对齐方式（center 居中，left 左 对齐，right 右对齐，justfy 两端对齐） |
|    textDirection    | 文本方向（ltr 从左至右，rtl 从右至 左）                      |
|    **overflow**     | 文字超出屏幕之后的处理方式（clip 裁剪，fade 渐隐，ellipsis 省略号） |
| **textScaleFactor** | 字体显示倍率                                                 |
|    **maxLines**     | 文字显示最大行数                                             |
|      **style**      | 字体的样式设置                                               |

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
     更多参数：https://docs.flutter.io/flutter/painting/TextStyle-class.html


### 八、**Container** **组件**

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





# Flutter图片组件

### 一、Flutter  图片组件 

图片组件是显示图像的组件，Image 组件有很多构造函数，这里我们只给大家讲两个
Image.asset， 本地图片
Image.network 远程图片

 **Image**  组件的常用属性 : 

|   名称     |  类型    |   说明   |
| ---- | ---- | ---- |
|   alignment   |  Alignment    |   图片的对齐方式   |
|   color 和 colorBlendMode   |      | 设置图片的背景颜色，通常和 colorBlendMode 配合使用，这样可以是图片颜色和背景色混合。上面的图是进行了颜色的混合，绿色背景和图片红色的混合 |
| fit | BoxFit | fit 属性用来控制图片的拉伸和挤压，这都是根据父容器来的。<br/>BoxFit.fill:全图显示，图片会被拉伸，并充满父容器。<br/>BoxFit.contain:全图显示，显示原比例，可能会有空隙。<br/>BoxFit.cover：显示可能拉伸，可能裁切，充满（图片要充满整个容器，还不变形）。<br/>BoxFit.fitWidth：宽度充满（横向充满），显示可能拉伸，<br/>可能裁切。<br/>BoxFit.fitHeight ：高度充满（竖向充满）,显示可能拉<br/>伸，可能裁切。<br/>BoxFit.scaleDown：效果和 contain 差不多，但是此属性不允许显示超过源图片大小，可小不可大。 |
|   repeat   |   平铺   | ImageRepeat.repeat : 横向和纵向都进行重复，直到铺满整个画布<br/>ImageRepeat.repeatX: 横向重复，纵向不重复。<br/>ImageRepeat.repeatY：纵向重复，横向不重复。 |
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
									textAlign: TextAlign.center, 															style: TextStyle(fontSize: 16.0
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

