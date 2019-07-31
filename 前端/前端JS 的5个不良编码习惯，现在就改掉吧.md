# JS 的5个不良编码习惯，现在就改掉吧

在阅读JavaScript代码时，你是否有过这种感觉

- 你几乎不明白代码的作用？
- 代码使用了很多 JavaScript 技巧？
- 命名和编码风格太过随意？

这些都是不良编码习惯的征兆。

在这篇文章中，我描述了JavaScript中常见的5种不良编码习惯。重要的是，本文会给出一些可行的建议，如何的摆脱摆脱这些习惯。

## 1.不要使用隐式类型转换

JavaScript是一种松散类型的语言。 如果使用得当，这是一个好处，因为它给你带来了灵活性。

大多数运算符`+ - * / ==`(不包括 `===`)在处理不同类型的操作数时会进行隐式转换。

语句`if（condition）{...}`，`while（condition）{...}`隐式地将条件转换为布尔值。

下面的示例依赖于类型的隐式转换，这种有时候会让人感到很困惑：

 ```javascript
console.log("2" + "1");  // => "21"
console.log("2" - "1");  // => 1

console.log('' == 0);    // => true

console.log(true == []); // -> false
console.log(true == ![]); // -> false
 ```

过度依赖隐式类型转换是一个坏习惯。 首先，它使你的代码在边缘情况下不太稳定。 其次，增加了引入难以重现和修复的bug的机会。

现在咱们实现一个获取对象属性的函数。如果属性不存在，函数返回一个默认值

```javascript
function getProp(object, propertyName, defaultValue) {
  if (!object[propertyName]) {
    return defaultValue;
  }
  return object[propertyName];
}
const hero = {
  name: 'Batman',
  isVillian: false
};
console.log(getProp(hero, 'name', 'Unknown'));     // => 'Batman'
```



`getProp()` 读取`name`属性的值，即`'Batman'`。

那么试图访问`isVillian`属性:

```
console.log(getProp(hero, 'isVillian', true)); // => true

```

这是一个错误。即使 `hero` 的属性`isVillian`为`false`，函数`getProp()`也会返回错误的`true`。

这是因为属性存在的验证依赖于`if（！object [propertyName]）{...}`隐式转换的布尔值。

**这些错误很难发现**，要修复该函数，就要明确验证值的类型：

```
function getPropFixed(object, propertyName, defaultValue) {
   if (object[propertyName] === undefined) {
     return defaultValue;
   }
   return object[propertyName];
}

const hero = {
  name: 'Batman',
  isVillian: false
};

console.log(getPropFixed(hero, 'isVillian', true)); // => false

```

`object[propertyName] === undefined`确切地验证属性是否为`undefined`。

**这里建议避免直接使用undefined**。 因此，上述解决方案可以进一步改进：

```
function getPropFixedBetter(object, propertyName, defaultValue) {
  if (!(propertyName in object)) {
    return defaultValue;
  }
  return object[propertyName]
}

```

原谅作者建议是:尽可能不要使用隐式类型转换。相反，请确保变量和函数参数始终具有相同的类型，必要时使用显式类型转换。

**最佳实践列表：**

- 始终使用严格的相等运算符`===`进行比较
- 不要使用松散等式运算符`==`
- 加法运算符 `operand1 + operand2`：两个操作数应该是数字或字符串
- 算术运算符 `- * /％**`：两个操作数都应该是数字
- `if（condition）{...}`，`while（condition）{...}`等语句：`condition` 必须是一个布尔类型值

你可能会说这种方式需要编写更多代码......你是对的！ 但是通过明确的方法，可以控制代码的行为。 此外，显性提高了可读性。

## 2. 不要使用早期的JavaScript技巧

JavaScript的有趣之处在于，它的创建者没有料到这种语言会如此流行。

基于JavaScript构建的应用程序的复杂性比语言发展的速度还要快。这种情况迫使开发人员使用JavaScript技巧和变通方法，只是为了让事情正常运行。

一个典型的例子是查看数组是否包含某个元素。 我从来不喜欢使用`array.indexOf（item）！== -1`来检查。

ES6 及以后版本的功能要强大得多，可以使用新的语言特性安全地重构许多技巧。



![img](https://user-gold-cdn.xitu.io/2019/7/29/16c3b076b3db3ce6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



ES6 中可以使用 `array.includes(item)` 来代替 `array.indexOf(item) !== -1`

## 3. 不要污染函数作用域

在ES2015之前，你可能会养成了将所有变量声明在函数作用域里面。

来看看一个例子：

```
function someFunc(array) {
  var index, item, length = array.length;
  /*
   * Lots of code
   */
  for (index = 0; index < length; index++) {
    item = array[index];
    // Use `item`
  }
  return someResult;
}

```

变量`index、item`和`length` 在函数作用域内。但是这些变量会影响函数作用域，因为它们只在`for()`块作用域内才被需要。

通过引入具有块作用域 `let`和`const`，应该尽可能地限制变量的生命周期。

```
function someFunc(array) {
  /*
   * Lots of code
   */
  const length = array.length;
  for (let index = 0; index < length; index++) {
    const item = array[index];
    // Use `item`
  }
  return someResult;
}

```

`index`和 `item` 变量被限制为`for()`循环块作用域。length 被移动到使用地方的附近。

重构后的代码更容易理解，因为变量不会分散在整个函数作用域内，它们存在于使用地方的附近。

在使用的块作用域定义变量

#### if 块作用域

```
// 不好
let message;
// ...
if (notFound) {
  message = 'Item not found';
  // Use `message`
}

// 好
if (notFound) {
  const message = 'Item not found';
  // Use `message`
}

```

#### for 块作用域

```
// 不好
let item;
for (item of array) {
  // Use `item`
}

// 好
for (const item of array) {
  // Use `item`
}

```

## 4.尽量避免 undefined 和 null

未赋值的变量默认被赋值为`undefined`。例如

```
let count;
console.log(count); // => undefined

const hero = {
  name: 'Batman'
};
console.log(hero.city); // => undefined

```

`count`变量已定义，但尚未使用值初始化。 JavaScript隐式赋值给它`undefined`。

访问不存在的属性`hero.city`时，也会返回`undefined`。

为什么直接使用`undefined`是一个不好习惯？ 因为与`undefined`进行比较时，你正在处理未初始化状态的变量。

变量、对象属性和数组在使用前必须用值初始化

JS 提供了很多避免与`undefined`进行比较方式。

#### **判断属性是否存在**

```
// 不好
const object = {
  prop: 'value'
};
if (object.nonExistingProp === undefined) {
  // ...
}

// 好
const object = {
  prop: 'value'
};
if ('nonExistingProp' in object) {
  // ...
}

```

#### **对象的默认属性**

```
// 不好
function foo(options) {
  if (object.optionalProp1 === undefined) {
    object.optionalProp1 = 'Default value 1';
  }
  // ...
}

// 好
function foo(options) {
  const defaultProps = {
    optionalProp1: 'Default value 1'
  };
  options = {
    ...defaultProps,
    ...options
  }
}

```

#### 默认函数参数

```
// 不好
function foo(param1, param2) {
  if (param2 === undefined) {
    param2 = 'Some default value';
  }
  // ...
}
// 好
function foo(param1, param2 = 'Some default value') {
  // ...
}

```

`null`是一个缺失对象的指示符。应该尽量避免从函数返回 `null`，特别是使用`null`作为参数调用函数。

一旦`null`出现在调用堆栈中，就必须在每个可能访问`null`的函数中检查它的存在，这很容易出错。

```
function bar(something) {
  if (something) {
    return foo({ value: 'Some value' });
  } else {
    return foo(null);
  }
}

function foo(options) {
  let value = null;
  if (options !== null) {
    value = options.value;
    // ...
  }
  return value;
}

```

尝试编写不涉及`null`的代码。 可替代方法是`try /catch`机制，默认对象的使用。

## 5. 不要使用随意的编码风格，执行一个标准

有什么比阅读具有随机编码风格的代码更令人生畏的事情？ 你永远不知道会发生什么！

如果代码库包含许多开发人员的不同编码风格，该怎么办?，这种就像各色人物涂鸦墙。



![img](https://user-gold-cdn.xitu.io/2019/7/29/16c3b07c6e47f39d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



整个团队和应用程序代码库都需要相同的编码风格，它提高了代码的可读性。

一些有用的编码风格的例子：

- [Airbnb JS 风格指南](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fairbnb%2Fjavascript)
- [谷歌 JS 风格指南](https://link.juejin.im?target=https%3A%2F%2Fgoogle.github.io%2Fstyleguide%2Fjsguide.html)

老实说，当我在回家前准备提交时，我可能会忘记设计代码的样式。

我自己总说:保持代码不变，以后再更新它，但是“以后”意味着永远不会。

这里建议使用 eslint 来规范编码风格。

1. 安装[eslint](https://link.juejin.im?target=https%3A%2F%2Feslint.org%2F)
2. 使用最适合自己的编码风格配置 eslint
3. 设置一个预提交钩子，在提交之前运行eslint验证。

## 总结

编写高质量和干净的代码需要纪律，克服不好的编码习惯。

JavaScript是一种宽容的语言，具有很大的灵活性。但是你必须注意你所使用的特性。这里建议是避免使用隐式类型转换，`undefined` 和 `null` 。

现在这种语言发展得相当快。找出复杂的代码，并使用最新 JS 特性来重构。

整个代码库的一致编码风格有益于可读性。良好的编程技能总是一个双赢的解决方案。

作者：前端小智

链接：https://juejin.im/post/5d3e3706f265da1b60294c1f

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。