# JS 中有趣的事实

## NaN 是一个 number 类型

`NaN`是一个 `number` 类型。 而且，`NaN` 不等于它自己。 实际上`NaN`不等于任何东西，验证一个变量是否是 `NaN` 可以使用 `isNaN()` 方法来判断。

```
> typeof(NaN)
"number"

> NaN === NaN
false

```

## null 是一个对象

`null`是一个对象。 听起来奇怪！ 对？ 但这是事实。

```
> typeof(null)
"object"

```

在这种情况下，`null`表示没有值。因此，`null`不应该是`Object`的实例。

```
> null instanceof Object
false    

```

## undefined 可以被定义

```
undefined`不是 JS 中的保留关键字， 你可以为其指定值也不会报错，如果声明一个变量没有赋值，默认为 `undefined
> var some_var;
undefined
> some_var == undefined
true
> undefined = 'i am undefined'   

```

## 0.1 + 0.2 不等于 to 0.3

在JavaScript中，`0.1 +0.2 == 0.3`返回`false`。 事实是，javascript 将浮点数存储为二进制。

```
> 0.1 + 0.2
0.30000000000000004
> 0.1 + 0.2 == 0.3
false    

```

## Math.max() 比 Math.min() 小

`Math.max() > Math.min()`返回`false`的事实看起来是错误的，但实际上它是正确的。

如果没有参数传给`min()`或`max()`，那么它将返回以下值。

```
> Math.max()
-Infinity
> Math.min()
Infinity    

```

## 018 - 045 = -19

在JavaScript中，前缀`0`会把任何数字转换成八进制。但是，八进制中不使用`8`，任何包含`8`的数字都将被无声地转换为常规的十进制数字。

```
> 018 - 045
-19    

```

因此，`018-019`实际上等于十进制表达式`18-37`，因为`045`是八进制，但`018`是十进制。

## 函数可以自执行

只需创建一个函数，并在调用其他函数时立即调用它，并使用 `()` 语法

```
> (function()  { console.log('I am self executing');  })();
I am self executing    

```

## 括号的位置问题

```
`return` 语句后面没有东西的时候它什么都不返回。 实际上，JS 后面 `return` 添加一个 `;`。

> function foo() {
   return
   {
      foo: 'bar'
   }
}
> foo(); 
undefined

> function foo() {
   return {
      foo: 'bar'
   }
}
> foo(); 
{foo: "bar"}

```

## 没有整数数据类型

在 JS 中，没有`int`（整数）数据类型。 所有数字均为 `Number` 类型。 实际上它将`int`数的浮点值存储在内存上。

## sort() 函数自动类型转换

`sort()` 函数自动将值转换为字符串，这就会导致奇怪的事情发生。

```
> [1,5,20,10].sort()
(4) [1, 10, 20, 5]

```

但是，它可以通过比较来解决:

```
> [1,5,20,10].sort(function(a, b){return a - b});
(4) [1, 5, 10, 20]

```

## 数组和对象的和

```
> !+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]
9
> {} + []
0
> [] + {}
"[object Object]"
> [] + []
""
> {} + {}
"[object Object][object Object]"
> {} + [] == [] + {}
true
```

作者：前端小智

链接：https://juejin.im/post/5d364cbdf265da1bc07e798f

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。