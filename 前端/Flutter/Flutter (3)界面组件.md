# Flutter 界面组件

## Flutter AppBar  自定义顶部导航按钮图标、颜色 以及TabBar定义顶部 Tab切换 

### 一、Flutter AppBar 自定义顶部按钮图标、颜色

| 属性    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| leading | 在标题前面显示的一个控件，在首页通常显示应用 的 logo；在其他界面通常显示为返回按钮 |
| title   | title 标题，通常显示为当前界面的标题文字，可以放组件         |
|  actions       |       通常使用 IconButton 来表示，可以放按钮组                                                        |
| bottom | 通常放 tabBar，标题下面显示一个 Tab 导航栏  |
| backgroundColor  |   导航背景颜色 |
| iconTheme | 图标样式 |
| textTheme | 文字样式 |
|  centerTitle | 标题是否居中显示 |

 ```dart
import 'package:flutter/material.dart'; 
class AppBardemoPage extends StatelessWidget { 
	@override 
	Widget build(BuildContext context) { 
		return Scaffold( 
			appBar: AppBar( 
				backgroundColor:Colors.red, 
				leading:IconButton( 
					icon: Icon(Icons.menu), 
					tooltip: "Search", 
					onPressed: (){ print('menu Pressed'); } ),
				title: Text('FlutterDemo'), 
				actions: <Widget>[ 
					IconButton( 
					icon: Icon(Icons.search), 
					tooltip: "Search", 
					onPressed: (){ print('Search Pressed'); } ) ,
                IconButton( 
                	icon: Icon(Icons.more_horiz), 
                	tooltip: "more_horiz", 
                	onPressed: (){ print('more_horiz Pressed'); } ) 
                ], ),
            body: Text('这是 Appbar'),
 ); } }
 ```



 



### 二、Flutter AppBar中自定义TabBar实现顶部Tab切换 

TabBar 常见属性：

| 属性 | 描述                                                     |
| ---- | -------------------------------------------------------- |
| tabs | 显示的标签内容，一般使用 Tab 对象,也可以是其他 的 Widget |
| controller  |         TabController 对象                                                 |
| isScrollable | 是否可滚动 |
| indicatorColor | 指示器颜色 |
| indicatorWeight | 指示器高度 |
| indicatorPadding | 底部指示器的 Padding |
| indicator | 指示器 decoration，例如边框等 |
| indicatorSize | 指示器大小计算方式，TabBarIndicatorSize.label 跟文 字等宽,TabBarIndicatorSize.tab 跟每个 tab 等宽 |
| labelColor | 选中 label 颜色 |
| labelStyle | 选中 label 的 Style |
| labelPadding | 每个 label 的 padding 值 |
| unselectedLabelColor | 未选中 label 颜色 |
| unselectedLabelStyle | 未选中 label 的 Style |

 ````dart
import 'package:flutter/material.dart'; 
class AppBardemoPage extends StatelessWidget {
	@override 
	Widget build(BuildContext context) { 
		return MaterialApp( 
			home: DefaultTabController( 
				length: 2, 
				child: Scaffold( 
					appBar: AppBar( 
						title: Text('FlutterDemo'),
                		bottom: TabBar( 
                			tabs: <Widget>[ Tab(text: "热门"), Tab(text: "推荐") ] )), 
                	body: TabBarView(
                		children: <Widget>[ 
                			ListView( children: <Widget>[ 
                				ListTile(title: Text("这是第一个 tab")), 
                				ListTile(title: Text("这是第一个 tab")), 
                				ListTile(title: Text("这是第一个 tab")) ], ),
                			ListView( children: <Widget>[ 
                				ListTile(title: Text("这是第二个 tab")), 
                				ListTile(title: Text("这是第二个 tab")), 
                				ListTile(title: Text("这是第二个 tab")) ], ) ],
                                ), ), ),
	 ); } }
 ````



### 三、Flutter把TabBar放在导航最顶部

```dart
import 'package:flutter/material.dart';

class AppBardemoPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: DefaultTabController(
        length: 2,
        child: Scaffold(
          appBar: AppBar(
            // backgroundColor: Colors.red,
            leading: IconButton(
                icon: Icon(Icons.arrow_back),
                tooltip: "Search",
                onPressed: () {
                  Navigator.of(context).pop();
                }),
            title: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                Expanded(
                  flex: 1,
                  child:
                      TabBar(tabs: <Widget>[Tab(text: "热门"), Tab(text: "推荐")]),
                )  ],
            ),  ),
          body: TabBarView(
            children: <Widget>[
              ListView(
                children: <Widget>[
                  ListTile(title: Text("这是第一个 tab")),
                  ListTile(title: Text("这是第一个 tab")),
                  ListTile(title: Text("这是第一个 tab"))
                ],
              ),
              ListView(
                children: <Widget>[
                  ListTile(title: Text("这是第二个 tab")),
                  ListTile(title: Text("这是第二个 tab")),
                  ListTile(title: Text("这是第二个 tab"))
                ],  )  ],  ), 
        ),   ),  ); }
}
```







### 四、Flutter AppBar中自定义TabBar实现Tabs的另一种方法。

```dart
import 'package:flutter/material.dart'; 
class AppBardemoPage extends StatefulWidget { 
	AppBardemoPage({Key key}) : super(key: key); 
	_AppBardemoPageState createState() => _AppBardemoPageState(); 
}

class _AppBardemoPageState extends State<AppBardemoPage> with SingleTickerProviderStateMixin { 
	TabController _tabController; 
	@override 
	void dispose() { 
		_tabController.dispose(); 
		super.dispose(); 
	}
	void initState() { 
		super.initState(); 
		_tabController = new TabController(vsync: this, length: 3); 
	}
	@override 
	Widget build(BuildContext context) { 
		return new Scaffold( 
			appBar: new AppBar( 
				title: new Text('顶部 tab 切换'), 
				bottom: new TabBar( 
					tabs: <Widget>[ 
						new Tab( icon: new Icon(Icons.directions_bike), ),
						new Tab( icon: new Icon(Icons.directions_boat), ),
						new Tab( icon: new Icon(Icons.directions_bus), ),],
					controller: _tabController,
                    ), ),
                 body: new TabBarView( 
                 	controller: _tabController, 
                 	children: <Widget>[ 
                 		new Center(child: new Text('自行车')), 
                 		new Center(child: new Text('船')), 
                 		new Center(child: new Text('巴士')), ], ), ); } }
```

## Flutter Drawer 侧边栏、以及侧边栏内容布局 

###  一、 Flutter Drawer   侧边栏  

在 Scaffold 组件里面传入 drawer 参数可以定义左侧边栏，传入 endDrawer 可以定义右侧边 

栏。侧边栏默认是隐藏的，我们可以通过手指滑动显示侧边栏，也可以通过点击按钮显示侧 

边栏。 

```dart
return Scaffold(
	appBar: 
		AppBar( title: Text("Flutter App"), ),
		drawer: Drawer(child: Text('左侧边栏'),),
		endDrawer:Drawer(child: Text('右侧侧边栏'),),);
```

###  二、 Flutter DrawerHeader   

+ decoration 设置顶部背景颜色 
+ child 配置子元素 
+ padding 内边距 
+ margin 外边距

```dart
 drawer: Drawer(
          child: Column(
        children: <Widget>[
          DrawerHeader(
            decoration: BoxDecoration(
                color: Colors.yellow,
                image: DecorationImage(
                    image: NetworkImage(
                        "https://www.itying.com/images/flutter/2.png"),
                    fit: BoxFit.cover)),
            child: ListView(
              children: <Widget>[Text('我是一个头部')],
            ),
          ),
          ListTile(
            title: Text("个人中心"),
            leading: CircleAvatar(child: Icon(Icons.people)),
          ),
          Divider(),
          ListTile(
            title: Text("系统设置"),
            leading: CircleAvatar(child: Icon(Icons.settings)),
          )
        ],
      )),
```

### 三、 Flutter UserAccountsDrawerHeader 

+ decoration 设置顶部背景颜色 
+ accountName 账户名称 
+ accountEmail 账户邮箱 
+ currentAccountPicture 用户头像 
+ otherAccountsPictures 用来设置当前账户其他账户头像 
+ margin 

```dart
 drawer: Drawer(
          child: Column(
        children: <Widget>[
          UserAccountsDrawerHeader(
            accountName: Text("老师"),
            accountEmail: Text("GDJ@qq.com"),
            currentAccountPicture: CircleAvatar(
              backgroundImage:
                  NetworkImage("https://www.itying.com/images/flutter/3.png"),
            ),
            decoration: BoxDecoration(
                color: Colors.yellow,
                image: DecorationImage(
                    image: NetworkImage(
                        "https://www.itying.com/images/flutter/2.png"),
                    fit: BoxFit.cover)),
            otherAccountsPictures: <Widget>[
              Image.network("https://www.itying.com/images/flutter/4.png"),
              Image.network("https://www.itying.com/images/flutter/5.png"),
              Image.network("https://www.itying.com/images/flutter/6.png")
            ],
          ),
          ListTile(
            title: Text("个人中心"),
            leading: CircleAvatar(child: Icon(Icons.people)),
          ),
          Divider(),
          ListTile(
            title: Text("系统设置"),
            leading: CircleAvatar(child: Icon(Icons.settings)),
          )
        ],
      )),
```

### 四、Flutter 侧边栏路由跳转 

```dart
onTap: (){Navigator.of(context).pop(); Navigator.pushNamed(context, '/search'); }
```



## Flutter 中的常见的按钮组件 以及自 定义按钮组件 

Flutter 里有很多的 Button 组件很多，常见的按钮组件有：RaisedButton、FlatButton、 IconButton、OutlineButton、ButtonBar、FloatingActionButton 等。 

**RaisedButton** ：凸起的按钮，其实就是 Material Design 风格的 Button 

**FlatButton** ：扁平化的按钮 

**OutlineButton**：线框按钮 

**IconButton** ：图标按钮 

**ButtonBar**:按钮组 

**FloatingActionButton**:浮动按钮 

|   属性名称   |   值类型   |   属性值   |
| ---- | ---- | ---- |
|     textColor       |  Color       |  文本颜色   |
|     color  |   Color  |   按钮的颜色       |
|     disabledColor  |   Color   |  按钮禁用时的颜色       |
|    disabledTextColor  |  Color   | 按钮禁用时的文本颜色          |
|    splashColor   |    Color   |    点击按钮时水波纹的颜色     |
|      highlightColor | Color|  点击（长按）按钮后按钮的颜色          |
|        elevation  | double  | 阴影的范围，值越大阴影范围越大 padding 内边距      |
|      onPressed   |  VoidCallback ，一般接收一个方法   |  必填参数，按下按钮时触发的回调，接收一个 方法，传 null 表示按钮禁用，会显示禁用相关 样式      |
|      child  |     Widget  |    文本控件    |
| padding |      | 内边距 |
| shape |      | 设置按钮的形状 shape: RoundedRectangleBorder( borderRadius: BorderRadius.circular(10), ) <br>shape: CircleBorder( side: BorderSide( color: Colors.white, ) ) |



## Flutter FloatingActionButton实现类似闲鱼App底部导航凸起按钮

FloatingActionButton 简称 FAB ,可以实现浮动按钮，也可以实现类似闲鱼 app 的底部凸起导航  

+ child 子视图，一般为 Icon，不推荐使用文字 

+ tooltip FAB 被长按时显示，也是无障碍功能 

+ backgroundColor 背景颜色 

+ elevation 未点击的时候的阴影 

+ hignlightElevation 点击时阴影值，默认 12.0 

+ onPressed 点击事件回调 

+ shape 可以定义 FAB 的形状等 

+ mini 是否是 mini 类型默认 false 

```dart
return Scaffold( 
	appBar: AppBar( title: Text("Flutter App"), ),
	floatingActionButton:Container( 
		height: 80, width: 80, 
		decoration: BoxDecoration( 
			borderRadius: BorderRadius.circular(40), color: Colors.white, ),
		margin: EdgeInsets.only(top:10), 
		padding: EdgeInsets.all(8), 
		child: FloatingActionButton( 
			child: Icon(Icons.add), 
			backgroundColor: this._currentIndex==1?Colors.red:Colors.yellow, 
			onPressed: (){ // print('点击 1'); 
				setState(() { this._currentIndex=1; });},
                ), ),
     floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked, ）
```



## Flutter  中的表单 

### 一、Flutter 常用表单介绍 

Flutter 中常见的表单有 TextField 单行文本框，TextField 多行文本框、CheckBox、Radio、Switch 

CheckboxListTile、RadioListTile、SwitchListTile、Slide. 

### 二、TextField 文本框组件 

**TextField 表单常见属性：**  

maxLines 设置此参数可以把文本框改为多行文本框 

onChanged 文本框改变的时候触发的事件 

decoration hintText 类似 html 中的 placeholder 

border 配置文本框边框 OutlineInputBorder 配合使用 

labelText lable 的名称 

labelStyle 配置 lable 的样式 

obscureText 把文本框框改为密码框 

controller controller 结合 TextEditingController()可以配置表单默认显示的内容

```dart
TextField( 
	maxLines: 10, 
	// obscureText: true, 
	decoration: InputDecoration( 
		hintText: "密码框", 
		border: OutlineInputBorder() 
) )
```

```dart
String _username=TextEditingController(); 
void initState() {  
	super.initState(); 
	_username.text='这是文本框初始值'; 
}
TextField( 
	controller: _username, 
	onChanged: (value){ // print(value); 
		setState(() { this._username.text=value; }); 
	},
	decoration: InputDecoration( hintText: "请输入您的内容", ), )
```

###  三、 Checkbox 、 CheckboxListTile 多选框组件 

**Checkbox 常见属性：** 

|   属性名称   |     描述   |
| ---- | ---- |
| value |true 或者 false  |
| onChanged | 改变的时候触发的事件 |
| activeColor | 选中的颜色、背景颜色|
|checkColor | 选中的颜色、Checkbox 里面对号的颜色|

**CheckboxListTile 常见属性**：

|   属性名称   |     描述   |
| ---- | ---- |
| value |true 或者 false  |
| onChanged | 改变的时候触发的事件 |
| activeColor | 选中的颜色、背景颜色|
| title|  标题|
| subtitle | 二级标题|
| secondary | 配置图标或者图片|
|  selected | 选中的时候文字颜色是否跟着改变|



```dart
Checkbox( value: _isSelected,
        onChanged: (v) { // print(v); 
    		setState(() { this._isSelected=v; }); },
         activeColor: Colors.red, 
         checkColor:Colors.blue )
    
CheckboxListTile( 
    value: _isSelected, 
    title: Text("nodejs 视频教程"), 
    subtitle: Text("egg.js 视频教程"), 
    onChanged: (v){ 
        setState(() { this._isSelected=v; }); },
    activeColor: Colors.red, 
    secondary: Image.network("https://www.itying.com/images/flutter/1.png"), 
    selected:_isSelected )
```

### 四、Radio、RadioListTile 单选按钮组件

 **Radio 常用属性**： 

+ value 单选的值 
+ onChanged 改变时触发 
+ activeColor 选中的颜色、背景颜色 
+ groupValue 选择组的值 

**RadioListTile** 常用属性：  

+ value true 或者 false 
+ onChanged 改变的时候触发的事件 
+ activeColor 选中的颜色、背景颜色 
+ title 标题 
+ subtitle 二级标题 
+ secondary 配置图标或者图片
+  groupValue 选择组的值

```dart
int _groupValue = 1;
Radio(value: 0,
  onChanged: (v) { setState(() { this._groupValue=v; }); },
  activeColor: Colors.red, groupValue:_groupValue , ),
Radio(value: 1,
  onChanged: (v) { setState(() {this._groupValue=v; }); },
  activeColor: Colors.red, groupValue:_groupValue , )

```

```dart
int _groupValue = 1;

_handelChange(v) {
  setState(() {
    _groupValue = v;
  });
}
RadioListTile(
    value: 1,
	title: Text("nodejs 视频教程"),
	subtitle: Text("egg.js 视频教程"),
	secondary: Image.network("https://www.itying.com/images/flutter/1.png"),
	groupValue:_groupValue , onChanged: _handelChange, ),
Divider(), 

RadioListTile(
	value: 0,
	title: Container(height: 60,
	child: Text('这是文本'),
	color: Colors.red, ), 
	// subtitle: Text("egg.js 视频教程"), 
	secondary: Image.network("https://www.itying.com/images/flutter/1.png"),
	groupValue:_groupValue , 
	onChanged: _handelChange, )
```

### 五、开关 Switch

+ value 单选的值 
+ onChanged  改变时触发
+  activeColor 选中的颜色、背景颜色



## **Flutter** **官方自带日期组件 和第三方日期组件** 

### **一、 Flutter  日期和时间戳**

 **日期转化成时间戳：** 

```
var now = new DateTime.now(); 
print(now.millisecondsSinceEpoch);//单位毫秒，13 位时间戳
```

**时间戳转化成日期：** 
```
var now = new DateTime.now();
var a=now.millisecondsSinceEpoch; //时间print(DateTime.fromMillisecondsSinceEpoch(a));
```

### [二、Flutter 第三方库 date_format 的使 用](https://pub.dev/packages/date_format)

### 三、调用 flutter 自带日期组件和时间组件
 日期组件：

```dart
var _datetime=DateTime.now(); 
_showDatePicker() async{ 
	var date =await showDatePicker( 
		context: context, 
		initialDate: _datetime, 
		firstDate:DateTime(1900), 
		lastDate:DateTime(2050) );
		if(date==null) return; 
		print(date); 
		setState(() { _datetime=date; }); }
```

**时间组件：**

```dart
var _time = TimeOfDay(hour: 9, minute: 20);

_showTimePicker() async {
  var time = await showTimePicker(context: context, initialTime: _time);
  if (time == null) return;
  print(time);
  setState(() {
    this._time = time;
  });
}


Column( 
	mainAxisAlignment: MainAxisAlignment.center, 
	children: <Widget>[ 
		Row(mainAxisAlignment: MainAxisAlignment.center, 
			children: <Widget>[ 
				InkWell( child: Row( 
					children: <Widget>[ 
						Text("${formatDate(_datetime, [yyyy, '-', mm, '-', dd])}"), Icon(Icons.arrow_drop_down) ], ),
						onTap: _showDatePicker, ),
						InkWell( child: Row( 
							children: <Widget>[ 
								Text("${this._time.format(context)}"), Icon(Icons.arrow_drop_down) ], ),onTap: _showTimePicker, ), ], )], )


```

### 四、调用flutter[自带日期组件和时间组件改为中文](http://bbs.itying.com/topic/5cfb2a12f322340b2c90e764 ) 

### 五、调用flutter[第三方时间组件](https://pub.dev/packages/flutter_cupertino_date_picker)** 

DatePickerPubDemo



## **Flutter** **[轮播图组件](https://pub.dev/packages/flutter_swiper)** 