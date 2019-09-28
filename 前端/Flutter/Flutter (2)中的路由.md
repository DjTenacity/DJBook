## **Flutter中的普通路由、普通路由传值、命名路由、命名路由传值**

### 一 、Flutter 中的路由

Flutter 中的路由通俗的讲就是页面跳转。在 Flutter 中通过 Navigator 组件管理路由导航。并提供了管理堆栈的方法。如：Navigator.push 和 Navigator.pop

**Flutter 中给我们提供了两种配置路由跳转的方式：1、基本路由 2、命名路由**



### 二、Flutter 中的基本路由使用
> 比如我们现在想从 HomePage 组件跳转到 SearchPage 组件。
> 1、需要在 HomPage 中引入 SearchPage.dart
> import '../SearchPage.dart';
>
> 2、在 HomePage 中通过下面方法跳转

```dart
RaisedButton(
	child: Text("跳转到搜索页面"), 
    onPressed: (){
		Navigator.of(context).push(
			MaterialPageRoute(
				builder: (BuildContext context){
					return SearchPage();
				}
			)
		);
	},color: Theme.of(context).accentColor, textTheme: 		ButtonTextTheme.primary
)
```

### 三、Flutter 中的基本路由跳转传值
比如我们现在想从 HomePage 组件跳转到 SearchPage 组件传值。
1、需要在 HomPage 中引入 SearchPage.dart
import '../SearchPage.dart';
2、在 HomePage 中通过下面方法跳转

```dart
RaisedButton(
	child: Text("跳转到搜索页面"), 
	onPressed: (){
		Navigator.of(context).push(
			MaterialPageRoute(
				builder: (BuildContext context){
					return SearchPage(title:"表单"); //传值
				}
			)
		);
	},color: Theme.of(context).accentColor,
	textTheme: ButtonTextTheme.primary
)
```

### 四、Flutter 中的命名路由

1、配置路由

```dart
import 'package:flutter/material.dart';
import 'pages/Tabs.dart';
import 'pages/Search.dart';
import 'pages/Form.dart';
void main() => runApp(MyApp());
class MyApp extends StatelessWidget {
@override
Widget build(BuildContext context) {
return MaterialApp(
	// home:Tabs(),
	initialRoute: '/', 
	routes: {
		'/':(contxt)=>Tabs(),
		'/search':(contxt) =>SearchPage(),
		'/form': (context) => FormPage(), 
		}, );
	}
}

```

2、路由跳转

```dart
RaisedButton(
	child: Text("跳转到搜索页面"), 
	onPressed: (){
		Navigator.pushNamed(context, '/search');
	},
	color: Theme.of(context).accentColor, 
	textTheme:ButtonTextTheme.primary
)
```

### 五、Flutter 中的命名路由跳转传值

[官方文档](https://flutter.dev/docs/cookbook/navigation/navigate-with-arguments)

1、配置路由：

```dart

class MyApp extends StatelessWidget {
final routes = {
'/': (contxt) => Tabs(),
'/search': (contxt) => SearchPage(),
'/form': (context, {arguments}) => FormPage(arguments: arguments), };
@override
Widget build(BuildContext context) {
return MaterialApp(
home: Tabs(), onGenerateRoute: (RouteSettings settings) {
// 统一处理
final String name = settings.name;
final Function pageContentBuilder = this.routes[name];
if (pageContentBuilder != null) {
if (settings.arguments != null) {
final Route route = MaterialPageRoute(
builder: (context) => pageContentBuilder(context, arguments: settings.arguments));
return route;} else {
final Route route = MaterialPageRoute(
builder: (context) => pageContentBuilder(context));
return route;
}
}
});
}
}

```

2、跳转传值

```dart
RaisedButton(
child: Text("跳转到表单演示页面"), onPressed: (){
Navigator.pushNamed(context, '/form',arguments: { "id":20
});
},color: Theme.of(context).accentColor, textTheme: ButtonTextTheme.primary
)

```

3、接收参数

```
import 'package:flutter/material.dart';
class FormPage extends StatelessWidget {
final Map arguments;
FormPage({this.arguments});
@override
Widget build(BuildContext context) {
return Scaffold(
appBar: AppBar(
title: Text("搜索"), ),body:Text("我是一个表单页面 ${arguments != null ? arguments['id'] : '0'}")
);}
}

```

### 六、Flutter 中的命名路单独抽离到一个文件

```dart
import 'package:flutter/material.dart';
import '../pages/Tabs.dart';
import '../pages/Search.dart';
import '../pages/Form.dart';

final Map<String, Function> routes = {
	'/':(contxt,{arguments})=>Tabs(),
	'/search':(contxt,{arguments}) =>SearchPage(arguments: arguments),
	'/form': (context,{arguments}) =>FormPage(arguments: arguments), 
};

var onGenerateRoute=(RouteSettings settings) {
	// 统一处理
	final String name = settings.name;
	final Function pageContentBuilder = routes[name];
	if (pageContentBuilder != null) {
	final Route route = MaterialPageRoute(
		builder: (context) =>
		pageContentBuilder(context, arguments: settings.arguments));
		return route;
	}
};
```

```dart
import 'package:flutter/material.dart';
import 'routes/Routes.dart';
void main() => runApp(MyApp());
class MyApp extends StatelessWidget {
	@override
	Widget build(BuildContext context) {
		return MaterialApp(
			// home:Tabs(),
			initialRoute: '/', onGenerateRoute: onGenerateRoute
		);
	}
}
```

## Flutter 中的路由 路由替换 返回到根路由

### 一、Flutter中返回到上一级页面

`Navigator.of(context).pop();` 

### 二、Flutter中替换路由

比如我们从用户中心页面跳转到了 registerFirst 页面，然后从 registerFirst 页面通过 pushReplacementNamed 跳转到了 registerSecond 页面。这个时候当我们点击 registerSecond 的返回按钮的时候它会直接返回到用户中心。  registerSecond 替代 registerFirst 

`Navigator.of(context).pushReplacementNamed('/registerSecond');` 

### 三、Flutter 返回到根路由 

比如我们从用户中心跳转到 registerFirst 页面，然后从 registerFirst 页面跳转到 registerSecond 页面，然后从 registerSecond 跳转到了 registerThird 页面。这个时候我们想的是 registerThird 注册成功后返回到用户中心。 这个时候就用到了返回到根路由的方法。

```dart
Navigator.of(context).pushAndRemoveUntil( 
    new MaterialPageRoute(builder: (context) => new Tabs(index:1)), (route) => route == null 
);
```

