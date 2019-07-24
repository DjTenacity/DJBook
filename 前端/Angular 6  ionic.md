## Angular 6  ionic

## 一、Ionic4.x 目录结构分析

+  e2e：端对端测试文件
+  node_modules ：项目所需要的依赖包
+  resources ：android/ios 资源（更换图标和启动动画）
+  src：开发工作目录，页面、样式、脚本和图片都放在这个目录下
+  www：静态文件，ionic build --prod 生成的单页面静态资源文件
+  platforms：生成 android 或者 ios 安装包需要的资源---(cordova platform add android 后
会生成)
+  plugins：插件文件夹，里面放置各种 cordova 安装的插件
+  config.xml: 打包成 app 的配置文件
+  package.json: 配置项目的元数据和管理项目所需要的依赖
+  ionic.config.json、ionic.starter.json：ionic 配置文件
+  angular.json angular 配置文件
+  tsconfig.json: TypeScript 项目的根目录，指定用来编译这个项目的根文件和编译选项
+  tslint.json：格式化和校验 typescript



## 二、Ionic4.x src 下面文件分析

![](D:\DJGitBook\前端\img\Ionic4.x src 下面文件分析.jpg)



app：应用根目录 （组件、页面、服务、模块...）
assets：资源目录（静态文件（图片，js 框架...）
theme：主题文件，里面有一个 scss 文件，设置主题信息。
global.scss：全局 css 文件

index.html：index 入口文件
main.ts：主入口文件
karma.conf.js/test.js：测试相关的配置文件
polyfills.ts: 这个文件包含 Angular 需要的填充，并在应用程序之前加载

### app 下面文件分析：

![](D:\DJGitBook\前端\img\app 下面文件分析.jpg)



## 三、app.module.ts 分析

![](D:\DJGitBook\前端\img\app.module.ts 分析.jpg)

## 四、app-routing.module.ts 分析

![](D:\DJGitBook\前端\img\app-routing.module.ts 分析.jpg)



## Ionic4.x 创建页面以及页面跳转

```bash
//创建项目
ionic start myApp tabs
```

1、cd 到我们的项目目录
2、通过 ionic g page 页面名称 

 `ionic g page myButton`

3、创建完成组件以后会在 src 目录下面多一个 button 的目录，它既是一个页面也是一个模块。

4、如果我们想在 tab1 里面写一个按钮点击跳转到 button 页面的话我们可以通过使用Angular 的路由跳转。在 ionic4.x 中路由是完全基于 angular，用法几乎和 angular 一样。

5、ionic4.x 中跳转到其他页面不会默认加返回按钮，如果我们想给 button 页面加返回的话需要找到 button 对应的 button.page.html，然后在再头部加入 ion-back-button。

## Ionic4.x新增 tabs 页面

1、创建 tab4 模块
`ionic g page tab4`

2、修改根目录里 app-routing.module.ts 文件里面的路由配置，去掉默认增加的路由
3、tabs.router.module.ts 中新增路由
`{ path: 'tab4', loadChildren: '../tab4/tab4.module#Tab4PageModule' }`
4、**tabs.page.html 中新增底部 tab 切换按钮**

```typescript
<ion-tabs>
	<ion-tab-bar slot="bottom">
		<ion-tab-button tab="tab1">
			<ion-icon name="flash"></ion-icon>
			<ion-label>Tab One</ion-label>
		</ion-tab-button>
		
		<ion-tab-button tab="tab2">
			<ion-icon name="apps"></ion-icon>
			<ion-label>Tab Two</ion-label>
		</ion-tab-button>

		<ion-tab-button tab="tab3">
			<ion-icon name="send"></ion-icon>
			<ion-label>Tab Three</ion-label>
		</ion-tab-button>

		<ion-tab-button tab="tab4">
			<ion-icon name="settings"></ion-icon>
			<ion-label>Tab four</ion-label>
		</ion-tab-button>
	</ion-tab-bar>
</ion-tabs>
```



## Ionic4.x 自定义公共模块、数据渲染等

1、创建公共模块以及组件

```
ionic g module module/slide
ionic g component module/slide
```

2、公共模块 slide.module.ts 中暴露对应的组件

```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { SlideComponent } from './slide.component';
@NgModule({
declarations: [SlideComponent],
imports: [
CommonModule
],exports:[SlideComponent]
})
export class SlideModule { }

import { SlideModule } from '../module/slide/slide.module';
@NgModule({
imports: [
CommonModule, FormsModule, IonicModule, SlideModule, RouterModule.forChild(routes)
],declarations: [Tab4Page]
})
```

4、使用自定义模块

`<app-slide></app-slide>`

## Ionic4.x 自定义公共模块中使用 ionic 内置组件

如 app.module.ts中

```typescript
import {IonicModule, IonicRouteStrategy} from '@ionic/angular';
```

```typescript
/** 引入当前模块运行以来的其他模块 */
imports: [BrowserModule, FormsModule, IonicModule.forRoot(), AppRoutingModule],
```

## Ionic4.x 中的 UI 组件（UI Components）页面布局
#### 1、Ionic4.x 内置颜色
primary secondary tertiary success warning danger dark medium light

```typescript
<ion-button color="primary"> primary </ion-button>
<ion-button color="secondary"> secondary </ion-button>
```

#### 2、Ionic4.x 中的按钮
官方文档：https://ionicframework.com/docs/api/button

#####  1、ion-button 组件可以定义一个按钮
<ion-button >  Default</ion-button>

#####  2、color 属性定义按钮的颜色

primary secondary tertiary success warning danger dark medium light

<ion-button color="primary"> primary </ion-button>
<ion-button color="secondary"> secondary </ion-button>

....

#####  3、expand 设置按钮的宽度
此属性允许您指定按钮的宽度。默认情况下，按钮是内联块，但是设置此属性将按钮更改为全宽度块元素。

<ion-button color="primary" expand="block" > button </ion-button>

<ion-button color="primary" expand="full" > button </ion-button>

#####   4、fill 设置背景填充
此属性决定按钮的背景和边框颜色。默认情况下，按钮有一个固定的背景，除非按钮位于工
具栏内部，如果按钮在工具栏顶部默认情况下面按钮有一个透明的背景

| clear   | 具有类似于扁平按钮的透明背景的按钮。     |
| ------- | ---------------------------------------- |
| outline | 具有透明背景和可见边框的按钮。           |
| solid   | 按钮具有填充的背景。用于工具栏中的按钮。 |

```html
<ion-button expand="full" fill="outline">Outline + Full</ion-button>
<ion-button fill="clear" color="success">
	<ion-icon slot="icon-only" name="add"></ion-icon>
</ion-button>
<ion-button fill="solid" (click)="goBack()">
<ion-icon name="ios-arrow-back" slot="start"></ion-icon>
返回
</ion-button>
```

##### 5、size 设置按钮的大小

small     ---> 小按钮
default  ---> 默认按钮
large 	 ---> 大按钮

##### 6、mode 决定使用哪种平台样式

<ion-button mode='ios' color="primary">  ios 平台的按钮 </ion-button>
<ion-button mode='md'  color="secondary"> android 平台的按钮 </ion-button>

##### 7、按钮结合图标

```
<ion-button color="success">
	<ion-icon slot="start" name="star"></ion-icon>
	Left Icon
</ion-button>
<ion-button color="success">
	Right Icon
	<ion-icon slot="end" name="add"></ion-icon>
</ion-button>
```



#### 3、ionic 中的图标

  ionic 图标官网：https://ionicons.com/

name 指定图标的名称 

slot 指定图标的位置  

#### 4、ion-header 、 ion-footer、 ion-content、
ion-toolbar 、 ion-title 、 ion-buttons 、
ion-back-button
官方文档：https://ionicframework.com/docs/api/toolbar
**ion-header** 头部，**ion-content** 内容，**ion-footer** 底部
**ion-toolbar** 主要用于头部和底部，固定在页面顶部或者底部。
**ion-title** 放在 **ion-toolbar** 里面指定导航的名称
**ion-buttons** 按钮组，主要用在 **ion-toolbar** 中，工具栏中的按钮应该放在 **ion-buttons** 的内
部。
**ion-back-button** 返回按钮，放在 **ion-buttons** 里面
**ion-buttons** 里面的属性：

+ secondary    元素在 ios 模式下位于内容左侧，在 md 模式下直接位于内容右侧。
+ primary         元素在 ios 模式下位于内容右侧，在 md 模式下位于最右侧。
+ start               在 LTR 中位于内容的左侧，在 RTL 中位于内容的右侧。
+ end                 在 LTR 中位于内容的右侧，在 RTL 中位于内容的左侧

```html
<ion-toolbar>
<ion-title>Title Only</ion-title>
</ion-toolbar>
<ion-toolbar>
<ion-buttons slot="start">
<ion-back-button></ion-back-button>
</ion-buttons>
<ion-title>Back Button</ion-title>
</ion-toolbar>
<ion-toolbar>
<ion-buttons slot="secondary">
<ion-button>
<ion-icon slot="icon-only" name="contact"></ion-icon>
</ion-button>
<ion-button>
<ion-icon slot="icon-only" name="search"></ion-icon>
</ion-button>
</ion-buttons>
<ion-title>Default Buttons</ion-title>
<ion-buttons slot="primary">
<ion-button color="secondary">
<ion-icon slot="icon-only" name="more"></ion-icon>
</ion-button>
</ion-buttons>
</ion-toolbar>
```

