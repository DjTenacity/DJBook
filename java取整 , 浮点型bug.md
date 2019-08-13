```java
有float类型的

向上取整:Math.ceil()   //只要有小数都+1
向下取整:Math.floor()   //不取小数
四舍五入:Math.round()  //四舍五入

```

[android6.0中floatmath用什么替代](https://zhidao.baidu.com/question/938772510489341692.html) 
解决方案是使用数学类:(float) Math,例如(float)Math.sqrt(),我也是刚遇到这...Android6.0使用 Math.floor 代替 FloatMath.floor 即可



float a =1.0f - 0.9f ;

float b = 0.9f - 0.8f;



a==b ?

![](D:\DJGitBook\img\float.png)

java里面。浮点计算都会用BigDecimal。 或者一般折扣系数，金额。后台设计里面都是用整数。

转换的时候容易精度丢失

价格用分做单位。

![](D:\DJGitBook\img\float2.jpg)

float 和 double  需要转成了二进制计算的  

包装类也不相等   精度在小数点后18位

精度丢失mongodb里面也会出现。95折在mongodb里面成了94折