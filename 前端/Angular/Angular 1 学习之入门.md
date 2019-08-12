## Angular学习之入门

 ### 一、创建 angualr 组件  

```
ng g component components/header
```

使用组件 使用组件
<app-header></app-header>

### 二、Angular 绑定数据 绑定数据

声明属性的几种方式：

​      public      共有  *（默认）  可以在这个类里面使用、也可以在类外面使用

​      protected   保护类型        他只有在当前类和它的子类里面可以访问

​      private     私有                  只有在当前类才可以访问这个属性

```typescript
  //定义普通数据
  public title='我是新闻组件';

  msg='我是一个新闻组件的msg';

  private username:string='张三';

  //推荐
  public student:any='我是一个学生的属性（数据）';
  
  public userinfo:object={
      username:"张三",
      age:'20'
  }

  public message:any;
//绑定 绑定 html
  public content="<h2>我是一个html标签</h2>";

  //定义数组

  public arr=['1111','2222','33333'];

  //推荐
  public list:any[]=['我是第一个新闻',222222222222,'我是第三个新闻'];

  public items:Array<string>=['我是第一个新闻','我是第二个新闻'];
   
  public userlist:any[]=[{
    username:'张三',
    age:20
  },
  {
    username:'王五',
    age:40
  }];

  public cars:any[]=[
      {
        cate:"宝马",
        list:[
          {
            title:'宝马x1',
            price:'30万'
          }
        ]
      },
      {
        cate:"奥迪",
        list:[
          {
            title:'奥迪q1',
            price:'40万'
          }
        ]
      }
  ]
```

Angular 中使用{{}}绑定业务逻辑里面定义的数据:

```html
<h1>angualr模板里面绑定属性</h1>
<div title="我是一个div">
    鼠标瞄上去看一下
</div>
<br>
<div [title]="student">
    张三
</div>
<h1>angualr模板里面 数据循环 *ngFor</h1>
<ul>
    <li *ngFor="let item of userlist">
        {{item.username}}---{{item.age}}
    </li>
</ul> 
<ul>
    <li *ngFor="let item of cars">
        <h2>{{item.cate}}</h2>
        <ol>
            <li *ngFor="let car of item.list">
                {{car.title}}---{{car.price}}
            </li>
        </ol>
    </li>
</ul>
```

**1.** **数据文本绑定** {{}}

```html
<div> 1+1={{1+1}}  </div>
```

**2.绑定** **html** 

```html
this.h="这是一个 h2 用[innerHTML]来解析" 
<div [innerHTML]="h"> </div>
```

**二、绑定属性**

```html
<div [id]="id" [title]="msg">调试工具看看我的属性div>
```

###  三、数据循环 ngFor 

**1**、***ngFor 普通循环**

```html
<ul>
	<li *ngFor="let item of list">
		{{item}}
	</li>
</ul>
```

2**、循环的时候设置**key

```html
<ul>
    <li *ngFor="let item of userlist;let key=index">
        {{key}}---{{item.username}}---{{item.age}}
    </li>
</ul>
```

**3.template循环数据**

```html
<ul>
  <li  template="ngFor let item of items">
    {{item}}
  </li>
</ul>
```

### **四、条件判断*ngIf** 与 *ngSwitch

```html
<p *ngIf="list.length>3">这是ngIF判断是否显示</p>
<p template="ngIflist.length>3">这是ngIF判断是否显示</p>

<ul [ngSwitch]="score">
	<li *ngSwitchCase="1">已支付</li>
	<li *ngSwitchCase="2">订单已经确认</li>
	<li *ngSwitchCase="3">已发货</li>
	<li *ngSwitchDefault>无效</li>
</ul>
```

### **五、执行事件** (click)=”getData()”

```html
<button class="button" (click)="getData()">
	点击按钮触发事件
</button>
<button class="button" (click)="setData()">
	点击按钮设置数据
</button>

```

```typescript
/*自定义方法获取数据*/
getData(){ 
//获取
	alert(this.msg);
}
setData(){
	//设置值
	this.msg='这是设置的值';
}
```

### 七、表单事件 

```html
<input type="text" (keyup)="keyUpFn($event)"/>
```

```typescript
keyUpFn(e){
	console.log(e)
}
```

### **八、双向数据绑定**

**注意引入： 注意引入：FormsModule**

```
import { FormsModule } from '@angular/forms';
```

```
@NgModule({
declarations[
AppComponent,
HeaderComponent,
FooterComponent,
NewsComponent
],
imports: [
BrowserModule,
FormsModule
],
providers: [],
bootstrap: [AppComponent]
})
```

使用：

```html
<input type="text" [(ngModel)]="inputValue"/>
{{inputValue}}
```

###  九、[ngClass]、[ngStyle]

[ngClass]:

```html
<div [ngClass]="{'red': true, 'blue': false}">
	这是一个 div
</div>
public flag=false;
<div [ngClass]="{'red': flag, 'blue': !flag}">
	这是一个 div
</div>
	public arr = [1, 3, 4, 5, 6];
<ul>
	<li *ngFor="let item of arr, let i = index">
		<span [ngClass]="{'red': i==0}">{{item}}</span>
	</li>
</ul>
```

**[ngStyle]:**

```html
<div [ngStyle]="{'background-color':'green'}">你好 ngStylediv</div>
public attr='red';
<div [ngStyle]="{'background-color':attr}">你好 ngStylediv</div>
```

### **十、管道**

public today=new Date();

``` html
<p>{{today | date:'yyyy-MM-dd HH:mm:ss' }}</p>
```

**其他管道：**

http://bbs.itying.com/topic/5bf519657e9f5911d41f2a34