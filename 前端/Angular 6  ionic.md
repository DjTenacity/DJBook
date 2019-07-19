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

