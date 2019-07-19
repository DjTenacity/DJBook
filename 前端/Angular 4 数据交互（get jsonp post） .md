## Angular 中的数据交互（get jsonp post）

#### 一、Angular get 请求数据
Angular5.x 以后 get、post 和和服务器交互使用的是 HttpClientModule 模块。
1、在 app.module.ts 中引入 HttpClientModule 并注入

`import {HttpClientModule} from '@angular/common/http';`

```
imports: [
	BrowserModule,
	HttpClientModule
]
```

2、在用到的地方引入 HttpClient 并在构造函数声明 

`import {HttpClient} from "@angular/common/http";` 

`constructor(public http:HttpClient) { }` 

3、get 请求数据

```typescript
var api = "http://a.itying.com/api/productlist";
this.http.get(api).subscribe(response => {
	console.log(response);
});
```

#### 二、Angular post 提交数据
Angular5.x 以后 get、post 和和服务器交互使用的是 HttpClientModule 模块。
1、在 app.module.ts 中引入 HttpClientModule 并注入
`import {HttpClientModule} from '@angular/common/http';`

```
imports: [
	BrowserModule,
	HttpClientModule
]
```

2、在用到的地方引入 HttpClient、HttpHeaders 并在构造函数声明 HttpClient
`import {HttpClient,HttpHeaders} from "@angular/common/http";`
`constructor(public http:HttpClient) { }`
3、post 提交数据

```typescript
const httpOptions = {
	headers: new HttpHeaders({ 'Content-Type': 'application/json' })
};
var api = "http://127.0.0.1:3000/doLogin";
this.http.post(api,{username:'张三',age:'20'},httpOptions).subscribe(response => {
	console.log(response);
});
```

#### 三、 Angular Jsonp 请求数据  

1、在 app.module.ts 中引入 HttpClientModule、HttpClientJsonpModule 并注入 

`import {HttpClientModule,HttpClientJsonpModule} from '@angular/common/http';` 

```typescript
imports: [ 
 BrowserModule, 
 HttpClientModule, 
 HttpClientJsonpModule 
] 
```



2、在用到的地方引入 HttpClient 并在构造函数声明 

`import {HttpClient} from "@angular/common/http";` 

`constructor(public http:HttpClient) { }` 

3、jsonp 请求数据 

```typescript
var api = "http://a.itying.com/api/productlist"; 

this.http.jsonp(api,'callback').subscribe(response => { 
	console.log(response); 
});
```

#### 四、Angular 中使用第三方模块 axios 请求数据
1、安装 axios
`cnpm install axios --save`
2、用到的地方引入 axios
`import axios from 'axios';`
3、看文档使用

```typescript
axios.get('/user?ID=12345')
	.then(function (response) {
		// handle success
		console.log(response);
	})
	.catch(function (error) {
		// handle error
		console.log(error);
	})
	.then(function () {
		// always executed
	});
```





## Angular 中的路由

路由就是根据不同的url地址 , 动态的让根组件挂载其他组件 来实现一个单页面应用

#### 一、Angular 创建一个默认带路由的项目

1. 命令创建项目
`ng new angualrdemo08 --skip-install`

![](C:\Users\admin\Desktop\angular路由.jpg)

2. 创建需要的组件

```typescript
ng g component home
ng g component news
ng g component newscontent
```

3. 找到 app-routing.module.ts 配置路由 

引入组件 

```typescript
import { HomeComponent } from './home/home.component'; 

import { NewsComponent } from './news/news.component'; 

import { NewscontentComponent } from './newscontent/newscontent.component'; 
```

配置路由 

```typescript
const routes: Routes = [ 

{	path: 'home', component: HomeComponent}, 

{	path: 'news', component: NewsComponent}, 

{	path: 'newscontent/:id', component: NewscontentComponent}, 

	{ 
		path: '', 
		redirectTo: '/home', 
		pathMatch: 'full' 
	} 

]; 
```



4. 找到 app.component.html 根组件模板，配置 router-outlet 显示动态加载的路由 

```html
<h1>
<a routerLink="/home">首页</a>
<a routerLink="/news">新闻</a>
</h1>
<router-outlet></router-outlet>
```

#### 二、Angular routerLink 跳转页面 默认路由

```html
<a routerLink="/home">首页</a>
<a routerLink="/news">新闻</a>
```

```typescript
//匹配不到路由的时候加载的组件 或者跳转的路由
{
	path: '', /任意的路由/
	// component:HomeComponent
	redirectTo:'home'
}
```

#### 三、Angular routerLinkActive 设置, routerLink 默认选中路由

```html
<h1>
<a routerLink="/home" routerLinkActive="active">首页</a>
<a routerLink="/news" routerLinkActive="active">新闻</a>
</h1>
```

```html
<h1>
<a [routerLink]="[ '/home' ]" routerLinkActive="active">首页</a>
<a [routerLink]="[ '/news' ]" routerLinkActive="active">新闻</a>
</h1>
```

```css
.active{
	color:red;
}
```

#### 四、动态路由

1.配置动态路由

```typescript
const routes: Routes = [
{path: 'home', component: HomeComponent},
{path: 'news', component: NewsComponent},
{path: 'newscontent/:id', component: NewscontentComponent},
{
	path: '',
	redirectTo: '/home',
	pathMatch: 'full'
	}
];
```



2.跳转传值
`<a [routerLink]="[ '/newscontent/',aid]">跳转到详情</a>`
`<a routerLink="/newscontent/{{aid}}">跳转到详情</a>`
3.获取动态路由的值
`import { ActivatedRoute} from '@angular/router';`

```typescript
constructor( private route: ActivatedRoute) {
}
ngOnInit() {
console.log(this.route.params);
	this.route.params.subscribe(data=>this.id=data.id);
}
```

#### 五、动态路由的 js 跳转

1. 引入
    `import { Router } from '@angular/router';`

2. 初始化

  ```typescript
  export class HomeComponent implements OnInit {
  constructor(private router: Router) {
  }
  ngOnInit() {
  }
  goNews(){
  // this.router.navigate(['/news', hero.id]);
  this.router.navigate(['/news']);
  }
  }
  ```

  3.路由跳转
  `this.router.navigate(['/news', hero.id]);`

#### 六、路由 get 传值 js 跳转

1. 引入 NavigationExtras

   `import { Router ,NavigationExtras} from '@angular/router';`

2. 定义一个 goNewsContent 方法执行跳转，用 NavigationExtras 配置传参。 

   ```typescript
   goNewsContent(){ 
   
   let navigationExtras: NavigationExtras = { 
   	queryParams: { 'session_id': '123' }, 
   	fragment: 'anchor' 
   }; 
   
   	this.router.navigate(['/news'],navigationExtras); 
   } 
   ```

3. .获取 get 传值

```typescript
constructor(private route: ActivatedRoute) {
	console.log(this.route.queryParams);
}
```



#### 七、父子路由

1. 创建组件引入组件

  ```typescript
  import { NewsaddComponent } from './components/newsadd/newsadd.component';
  import { NewslistComponent } from './components/newslist/newslist.component';
  
  ```

2. 配置路由

   ```typescript
{ 
   	path: 'news', 
	component:NewsComponent, 
   	children: [ 
	{ 
   		path:'newslist', 
		component:NewslistComponent 
   	}, 
	{ 
   		path:'newsadd', 
		component:NewsaddComponent 
   	} ] 
} 
   ```

   3. 父组件中定义 router-outlet 

   `<router-outlet></router-outlet>`

