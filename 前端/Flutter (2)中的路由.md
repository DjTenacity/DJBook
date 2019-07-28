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