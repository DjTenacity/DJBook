## Angular 中自定义模块

#### 一、Angular 内置模块

![Angular 内置模块](C:\Users\admin\Desktop\Angular 内置模块.jpg)

####  二、Angular 自定义模块
当我们项目比较小的时候可以不用自定义模块。但是当我们项目非常庞大的时候把所有的组
件都挂载到根模块里面不是特别合适。所以这个时候我们就可以自定义模块来组织我们的项
目。并且通过 **Angular 自定义模块可以实现路由的懒加载**。

`ng g module mymodule`

![Angular 自定义模块](C:\Users\admin\Desktop\Angular 自定义模块.jpg)



创建新项目
ng new angularlazyload --skip-install

在新项目 根目录输入
cnpm install

试运行
ng serve --open

生成模块以及路由配置文件 , 如果没有--routing则只有user.module.ts
ng g module module/user --routing

创建组件：
ng g component module/user

const routes: Routes = [
  { // 配置路由模块懒加载
    path: 'user', loadChildren: './module/user/user.module#UserModule'
  }
];



## ionic

搭建ionic环境，创建ionic项目


	1、必须安装nodejs （最新稳定版本）


	2、npm i -g cordova ionic    /  cnpm i -g cordova ionic 


	3、cd到要创建ionic项目所在目录
	
	   ionic start ionicdemo  tabs

	4、npm i的时候可以按一下ctrl+c结束 ，cd到刚才的项目目录   
	
		cd ionicdemo
	
		cnpm install


	5、ionic serve 运行项目



## ionic 和cordova的区别是什么

####   **ionic是什么：**

Ionic(ionicframework)**一款开源的Html5移动App开发框架,**是AngularJS移动端解决方案 , Ionic**以流行的跨平台移动app开发框架phoengap为蓝本**，让开发者可以通过命令行工具快速生成android  ios移动app应用  

####  **phoengap是什么？**

[phonegap](http://www.phonegap100.com/)是一个用基于HTML，CSS和JavaScript的，创建移动跨平台移动应用程序的快速开发平台。它使开发者能够利用iPhone，Android，Palm，Symbian,WP7,WP8,Bada和Blackberry智能手机的核心功能——包括地理定位，加速器，联系人，声音和振动.  

通俗的讲：ionic是一款基于angularjs的html5移动app开发框架

phonegap就是一款可以打包并且可以让js调用原生的移动app框架  

**就可以把ionic当作一款html5 移动app框架，把phonegap/cordova 当作打包 并且调用原生的框架就可以了至于：为什么ionic也可以打包，上面也说了，ionic的打包插件是基于phonegap/cordova的**