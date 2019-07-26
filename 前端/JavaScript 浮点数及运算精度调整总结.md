# [JavaScript 浮点数及运算精度调整总结](http://www.codeceo.com/article/javascript-float-number.html)

JavaScript 只有一种数字类型 Number，而且在Javascript中所有的数字都是以IEEE-754标准格式表示的。[浮点数](http://www.codeceo.com/article/float-number.html)的精度问题不是JavaScript特有的，因为有些小数以**二进制表示位数是无穷的**。

十进制       二进制
0.1              0.0001 1001 1001 1001 …
0.2              0.0011 0011 0011 0011 …
0.3              0.0100 1100 1100 1100 …
0.4              0.0110 0110 0110 0110 …
0.5              0.1
0.6              0.1001 1001 1001 1001 …

所以比如 1.1，其程序实际上无法真正的表示 ‘1.1′，而只能做到一定程度上的准确，这是无法避免的精度丢失：1.09999999999999999

在JavaScript中问题还要复杂些，这里只给一些在Chrome中测试数据：

```javascript
console.log(1.0-0.9 == 0.1)    //false
console.log(1.0-0.8 == 0.2)    //false
console.log(1.0-0.7 == 0.3)    //false
console.log(1.0-0.6 == 0.4)    //true
console.log(1.0-0.5 == 0.5)    //true
console.log(1.0-0.4 == 0.6)    //true
console.log(1.0-0.3 == 0.7)    //true
console.log(1.0-0.2 == 0.8)    //true
console.log(1.0-0.1 == 0.9)    //true
```

那如何来避免这类 1.0-0.9 != 0.1 的非bug型问题发生呢？下面给出一种目前用的比较多的解决方案, 在判断浮点运算结果前对计算结果进行精度缩小，因为在精度缩小的过程总会自动四舍五入:

```javascript
(1.0-0.9).toFixed(digits)  // toFixed() 精度参数digits须在0与20之间
console.log(parseFloat((1.0-0.9).toFixed(10)) === 0.1)   //true
console.log(parseFloat((1.0-0.8).toFixed(10)) === 0.2)    //true
console.log(parseFloat((1.0-0.7).toFixed(10)) === 0.3)    //true
console.log(parseFloat((11.0-11.8).toFixed(10)) === -0.8)   //true
```

写成一个方法：

```javascript
//通过isEqual工具方法判断数值是否相等
function isEqual(number1, number2, digits){
  digits = digits == undefined? 10: digits; // 默认精度为10
  return number1.toFixed(digits) === number2.toFixed(digits);
}
console.log(isEqual(1.0-0.7, 0.3));  //true
//原型扩展方式，更喜欢面向对象的风格
Number.prototype.isEqual = function(number, digits){
  digits = digits == undefined? 10: digits; // 默认精度为10
  return this.toFixed(digits) === number.toFixed(digits);
}
console.log((1.0-0.7).isEqual(0.3)); //true
```

接下来，再来试试浮点数的运算，

```javascript
console.log(1.79+0.12)  //1.9100000000000001
console.log(2.01-0.12)   //1.8899999999999997
console.log(1.01*1.3)    //1.3130000000000002
console.log(0.69/10)     //0.06899999999999999
```

解决方案：

```javascript
//加法函数，用来得到精确的加法结果
//说明：javascript的加法结果会有误差，在两个浮点数相加的时候会比较明显。这个函数返回较为精确的加法结果。
//调用：accAdd(arg1,arg2)
//返回值：arg1加上arg2的精确结果
function accAdd(arg1,arg2){
  var r1,r2,m;
  try{r1=arg1.toString().split(".")[1].length}catch(e){r1=0}
  try{r2=arg2.toString().split(".")[1].length}catch(e){r2=0}
  m=Math.pow(10,Math.max(r1,r2))
  return (arg1*m+arg2*m)/m
}
//给Number类型增加一个add方法，调用起来更加方便。
Number.prototype.add = function (arg){
  return accAdd(arg,this);
}

//减法函数，用来得到精确的减法结果
//说明：javascript的加法结果会有误差，在两个浮点数相加的时候会比较明显。这个函数返回较为精确的减法结果。
//调用：accSub(arg1,arg2)
//返回值：arg1减去arg2的精确结果
function accSub(arg1,arg2){
  var r1,r2,m,n;
  try{r1=arg1.toString().split(".")[1].length}catch(e){r1=0}
  try{r2=arg2.toString().split(".")[1].length}catch(e){r2=0}
  m=Math.pow(10,Math.max(r1,r2));
  //last modify by deeka
  //动态控制精度长度
  n=(r1>=r2)?r1:r2;
  return ((arg1*m-arg2*m)/m).toFixed(n);
}

//除法函数，用来得到精确的除法结果
//说明：javascript的除法结果会有误差，在两个浮点数相除的时候会比较明显。这个函数返回较为精确的除法结果。
//调用：accDiv(arg1,arg2)
//返回值：arg1除以arg2的精确结果
function accDiv(arg1,arg2){
  var t1=0,t2=0,r1,r2;
  try{t1=arg1.toString().split(".")[1].length}catch(e){}
  try{t2=arg2.toString().split(".")[1].length}catch(e){}
  with(Math){
    r1=Number(arg1.toString().replace(".",""))
    r2=Number(arg2.toString().replace(".",""))
    return (r1/r2)*pow(10,t2-t1);
  }
}
//给Number类型增加一个div方法，调用起来更加方便。
Number.prototype.div = function (arg){
  return accDiv(this, arg);
}

//乘法函数，用来得到精确的乘法结果
//说明：javascript的乘法结果会有误差，在两个浮点数相乘的时候会比较明显。这个函数返回较为精确的乘法结果。
//调用：accMul(arg1,arg2)
//返回值：arg1乘以arg2的精确结果
function accMul(arg1,arg2) {
  var m=0,s1=arg1.toString(),s2=arg2.toString();
  try{m+=s1.split(".")[1].length}catch(e){}
  try{m+=s2.split(".")[1].length}catch(e){}
  return  Number(s1.replace(".",""))*Number(s2.replace(".",""))/Math.pow(10,m)
}
//给Number类型增加一个mul方法，调用起来更加方便。
Number.prototype.mul = function (arg){
  return accMul(arg, this);
}
<br>//验证一下：
console.log(accAdd(1.79, 0.12));  //1.91
console.log(accSub(2.01, 0.12));  //1.89
console.log(accDiv(0.69, 10));    //0.069<br>console.log(accMul(1.01, 1.3));   //1.313
```

改造之后，可以愉快地进行浮点数加减乘除操作了~

