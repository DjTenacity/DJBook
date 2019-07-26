## Angular 生命周期钩子 ngOnInit() 和 ngAfterViewInit() 的区别

angular 生命周期钩子的详细介绍在 https://angular.cn/guide/lifecycle-hooks  文档中做了介绍。

ngOnInit() 在 Angular 第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件；

ngAfterViewInit() 初始化完组件视图及其子视图之后调用。

ngOnInit() 钩子应该是我们用得最频繁的一个了，在使用命令 ng g component <component-name> 生成一个组件后，就有 ngOnInit() 方法。

ngOnInit() 钩子可以作为初始化时调用一些方法。如：

![img](https://img2018.cnblogs.com/blog/1032021/201812/1032021-20181203161221733-934054313.png)

ngOnInit() 钩子也可以更改视图样式，如：

![img](https://img2018.cnblogs.com/blog/1032021/201812/1032021-20181203161431635-164067391.png)

建议在写 angular 的时候尽量少的使用 jQuery，除非一些简单好用的 jQuery 插件

而 ngAfterViewInit() 钩子是在组件视图或者子组件视图初始化完成之后调用。

 

项目中遇到了这样一个需求：

一、描述：

\1. 通过选择开始时间和结束时间，来查询列表记录：

\2. 这个功能在一个模块，4个不同的组件中用到；

\3. 初始化时开始时间是当前时间减1天，结束时间是当前时间；

\4. 开始时间和结束时间格式“2018-12-03 09:09:09”；

二、解决方法：

\1. 时间控件选择了[日期时间选择器(Bootstrap日期和时间表单组件) ](http://www.bootcss.com/p/bootstrap-datetimepicker/)；

\2. 将日期时间选择器功能进行封装成一个单独的组件 DateTimeSelectorComponent，方便4个组件中用到；

\3. 以其中一个父组件 OfflineComponent 为例，监听子组件 DateTimeSelectorComponent 的事件，子组件暴露一个 EventEmitter  属性，当事件发生时，子组件利用该属性 `emits`(向上弹射)事件。父组件绑定到这个事件属性，并在事件发生时作出回应；

\4. 待子组件返回值了，父组件才做出相应的数据请求和方法调用。这里父组件中就是用 ngAfterViewInit() 钩子。

三、最终结果：

![img](https://img2018.cnblogs.com/blog/1032021/201812/1032021-20181203163647739-193842271.png)

三、实际操作：

\1. 首先封装 DateTimeSelectorComponent 组件

- 模板：  

```html
<div class="start-end-time-search">
  <label for="startTime" class="start-time-label">起止时间</label>
  <input type="text" [(ngModel)]="startTime" placeholder="请选择开始时间" readonly id="startTime" class="start-time">
  <label for="endTime" class="end-time-label">至</label>
  <input type="text" [(ngModel)]="endTime" placeholder="请选择结束时间" readonly id="endTime" class="end-time">
</div>
```

- 组件

　　　　1.暴露一个 EventEmitter 属性，以及定义一些需要返回给父组件的属性

```typescript
  public startTime: string = "";
  public endTime: string = ""; 
  public isEnabledSearchBtn: boolean = true;  // 查询按钮是可用的
  public startTimeErrorTip: string = ""; // 开始时间选择错误提示
  public endTimeErrorTip: string = ""; // 结束时间选择错误提示
  @Output() timeInfo = new EventEmitter<object>();
```

　　　　2. 定义一个方法来格式化日期时间

```typescript
// 日期时间格式 "2018-09-20 00:00:00"
  dateTimeFormat(dateTime) {
    let dateTimeObj = {
      "year": dateTime.getFullYear(),
      "month": dateTime.getMonth() + 1,
      "day": dateTime.getDate(),
      "hours": dateTime.getHours(),
      "minutes": dateTime.getMinutes(),
      "seconds": dateTime.getSeconds()
    };
    for (let k in dateTimeObj) {
      if (dateTimeObj[k] < 10) {
        dateTimeObj[k] = "0" + dateTimeObj[k];
      }
    }
    let dateTimeString = dateTimeObj.year + "-" + dateTimeObj.month + "-" + dateTimeObj.day + " "
      + dateTimeObj.hours + ":" + dateTimeObj.minutes + ":" + dateTimeObj.seconds;
    return {"obj": dateTimeObj, "string": dateTimeString};
  }
```

返回两种类型，对象类型和字符串类型可供选择。

　　　　3. 将信息返回给父组件

```typescript
// 选择时间，选择完时间或者初始化时，调用一次，将时间信息发送给父组件
  selectTime() {
    let data = {
      isEnabledSearchBtn: this.isEnabledSearchBtn,
      startTime: this.startTime,
      endTime: this.endTime,
      startTimeErrorTip: this.startTimeErrorTip,
      endTimeErrorTip: this.endTimeErrorTip
    };
    this.timeInfo.emit(data);
  }
```

4. 组装数据(返回给父组件的信息)

```typescript
ngOnInit() {
  let initNowTime = this.dateTimeFormat(new Date());
  // 开始时间用当前时间减去一天，结束时间使用当前时间
  this.startTime = initNowTime.obj.year + "-" + initNowTime.obj.month + "-" +
    ((initNowTime.obj.day - 1) < 10 ? ("0" + (initNowTime.obj.day - 1)) : (initNowTime.obj.day - 1)) + " " +
    initNowTime.obj.hours + ":" + initNowTime.obj.minutes + ":" + initNowTime.obj.seconds;

  this.endTime = initNowTime.string;

  this.selectTime();  // 初始化的时候调用一次使得“初始化完组件视图及其子视图之后” startTime 和 endTime 有值

  /*下面处理选择的时间*/
  let jQuery: any = $;
  let that = this;
  jQuery("#startTime").datetimepicker({
    autoclose: true,
    format: "yyyy-mm-dd hh:ii:ss",
    minuteStep: 1,
    language: "zh-CN",
    todayBtn: "linked",
    todayHighlight: true,
    endDate: (new Date),
    zIndexOffset: 1000
  }).on("changeDate",function(startTime) {
    that.startTime = that.dateTimeFormat(startTime.date).string;
    if (Date.parse(that.endTime) - Date.parse(that.startTime) > 31536000000) {  // 31536000000 是一年的时间戳
      that.startTimeErrorTip = that.endTimeErrorTip == "时间跨度不能大于一年" ? "" : "时间跨度应小于一年";
      that.isEnabledSearchBtn = false;
    } else if (that.startTime > that.endTime) {
      that.startTimeErrorTip = that.endTimeErrorTip == "结束时间不能小于开始时间" ? "" : "开始时间不能大于结束时间";
      that.isEnabledSearchBtn = false;
    } else {
      that.startTimeErrorTip = "";
      that.endTimeErrorTip = "";
      that.isEnabledSearchBtn = true;
    }

    that.selectTime();
  }).data('datetimepicker');

  jQuery("#endTime").datetimepicker({
    autoclose: true,
    format: "yyyy-mm-dd hh:ii:ss",
    minuteStep: 1,
    language: "zh-CN",
    todayBtn: "linked",
    todayHighlight: true,
    endDate: (new Date),
    zIndexOffset: 1000
  }).on("changeDate", function(endTime) {
    that.endTime = that.dateTimeFormat(endTime.date).string;
    if (Date.parse(that.endTime) - Date.parse(that.startTime) > 31536000000) {  // 31536000000 是一年的时间戳
      that.endTimeErrorTip = that.startTimeErrorTip == "时间跨度应小于一年" ? "" : "时间跨度不能大于一年";
      that.isEnabledSearchBtn = false;
    } else if (that.startTime > that.endTime) {
      that.endTimeErrorTip = that.startTimeErrorTip == "开始时间不能大于结束时间" ? "" : "结束时间不能小于开始时间";
      that.isEnabledSearchBtn = false;
    } else {
      that.startTimeErrorTip = "";
      that.endTimeErrorTip = "";
      that.isEnabledSearchBtn = true;
    }

    that.selectTime();
  }).data('datetimepicker');
}
```

\2. 父组件中使用子组件

- 模板

```
<app-date-time-selector (timeInfo)="handle($event)"></app-date-time-selector>
```

父组件绑定的事件属性同子组件中 EventEmitter 属性名称一样

- 组件

```typescript
// 时间选择器操作事件
  handle(value) {
    this.startTimeErrorTip = value.startTimeErrorTip;
    this.endTimeErrorTip = value.endTimeErrorTip;
    this.startTime = value.startTime;
    this.endTime = value.endTime;
    this.isEnabledSearchBtn = value.isEnabledSearchBtn;
  }

  ngOnInit() { }

  ngAfterViewInit() {this.offlineRecordList();
  }
```

父组件接收到子组件返回的信息保存在 value 对象中，分别拆分给相应的属性。

最后，当父组件视图和子组件视图完成初始化后，再调用方法，来获取列表数据。

