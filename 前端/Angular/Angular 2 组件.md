## Angular中的服务以及实现toDoList数据持久化

#### 一、 Angualr 中的服务



```
 Angular是基于模块化,组件化的开发方式 ,把一些公用的方法放到服务Service里面
```

![](C:\Users\admin\Desktop\DJGitBook\前端\img\Angualr 中的服务.jpg)

#### 二、创建服务命令

```nginx
ng g service my-new-service 

//创建到指定目录下面
ng g service services/storage
```

#### 三、app.module.ts 里面引入创建的服务
1.app.module.ts 里面引入创建的服务

```typescript
import { StorageService } from './services/storage.service'
```
2.NgModule 里面的 providers 里面依赖注入服务

```typescript
@NgModule({
declarations: [
	AppComponent,
	HeaderComponent,
	FooterComponent,
	NewsComponent,
	TodolistComponent
],
imports: [
	BrowserModule,
	FormsModule
],
providers: [StorageService]bootstrap: [AppComponent]
})
export class AppModule { }
```

#### 四、使用的页面引入服务，注册服务
```typescript
import { StorageService } from '../../services/storage.service';
```

```typescript
constructor(private storage: StorageService) {
}
```


使用

```typescript
addData(){
// alert(this.username);
	this.list.push(this.username);
	this.storage.set('todolist',this.list);
}
removerData(key){
	console.log(key);
	this.list.splice(key,1);
	this.storage.set('todolist',this.list);
}
```

## Angular 中的 Dom 操作以及 @ViewChild 、 Angular 执行 css3 动画 

#### 一、Angular 中的 dom 操作（原生 js）

```javascript
ngOnInit() 方法只是组件和指令初始化完成  并不是真正的dom加载完成 ,是获取不到dom节点的

视图加载完成以后触发的方法 是 ngAfterViewInit  dom加载完成
```

```javascript
ngAfterViewInit(){
	var boxDom:any=document.getElementById('box');
	boxDom.style.color='red'
}
```



####  二、Angular 中的 dom 操作（ViewChild）


```typescript
import { Component ,ViewChild,ElementRef} from '@angular/core';
```

```typescript
@ViewChild('myattr') myattr: ElementRef;
```

```html
<div #myattr> 模板中给dom起一个名字 </div>
```

```typescript
// ngAfterViewInit生命周期函数里面获取dom
ngAfterViewInit(){
	let attrEl = this.myattr.nativeElement;
}
```

#### 三、父子组件中通过 ViewChild 调用子组件的方法 

1.调用子组件给子组件定义一个名称 

```html
<app-footer #footerChild>app-footer> 
```

2.引入 ViewChild 

```typescript
import { Component, OnInit ,ViewChild} from '@angular/core'; 
```

3.ViewChild 和刚才的子组件关联起来 

```typescript
@ViewChild('footerChild') footer; 
```

4.调用子组件 

```typescript
run(){ 
	this.footer.footerRun(); 
}
```



## Angular  父子组件以及组件之间通讯

#### 一、父组件给子组件传值-@input

父组件不仅可以给子组件传递简单的数据，还可把自己的方法以及整个父组件传给子组件

1. 父组件调用子组件的时候传入数据
`<app-header [msg]="msg">app-header>`
2. 子组件引入 Input 模块
`import { Component, OnInit ,Input } from '@angular/core'`

3. 子组件中 @Input 接收父组件传过来的数据
```typescript
export class HeaderComponent implements OnInit {
	@Input() msg:string`
	constructor() { } 
	ngOnInit() { 
	} 
}
```

4. 子组件中使用父组件的数据 

`<h2>这是头部组件--{{msg}}h2>` 

#### 二、子组件通过@Output 触发父组件的方法 

1.子组件引入 Output 和 EventEmitter
`import { Component, OnInit ,Input,Output,EventEmitter} from '@angular/core'`

2.子组件中实例化 EventEmitte

```typescript
@Output() private outer=new EventEmitter<string>();
/用 EventEmitter 和 output 装饰器配合使用 指定类型变量/
```

3. 子组件通过 EventEmitter 对象 outer 实例广播数据

```typescript
sendParent(){
	// alert('zhixing');
	this.outer.emit('msg from child')
}
```

4.父组件调用子组件的时候，定义接收事件 , outer 就是子组件的 EventEmitter 对象 outer 

`<app-header (outer)="runParent($event)">app-header> `

5.父组件接收到数据会调用自己的 runParent 方法，这个时候就能拿到子组件的数据 

```typescript
//接收子组件传递过来的数据 

runParent(msg:string){ 

alert(msg); 

} 
```

#### 三、父组件通过@ViewChild 主动获取子组件的数据和方法 

1.调用子组件给子组件定义一个名称
`<app-footer #footerChild>app-footer>`
2. 引入 ViewChild

import { Component, OnInit ,ViewChild} from '@angular/core'; 

3. ViewChild 和刚才的子组件关联起来 

@ViewChild('footerChild') footer; 

4.调用子组件 

```typescript
run(){ 
	this.footer.footerRun(); 
} 
```



#### 四、非父子组件通讯 

1、公共的服务 

2、Localstorage (推荐) 

3、Cookie