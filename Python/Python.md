### Python

dir(对象) , 遍历对象属性
对象.__dict__   对象的字典键值对

vim : i 插入   :+W+Q 关闭当前文件
	


#coding:utf-8
 告诉Python解释器 当前文件 编码格式

Python 创建类是 class Foo (object)  里面有object 说明是新式类,
否则是jiushilei

 cookie   以键值对的格式 存储在浏览器当中的一段文本信息 ,
再次请求这个网站时这个cookie信息就会自动加到请求报文的头里面发送到服务器里面


​    
break/continue只能用在循环中，除此以外不能单独使用
break/continue在嵌套循环中，只对最近的一层循环起作用
continue的作用：用来结束本次循环，紧接着执行下一次的循环

切片是指对操作的对象截取其中一部分的操作。字符串、列表、元组都支持切片操作。

切片的语法：[起始:结束:步长]
注意：选取的区间属于左闭右开型，即从"起始"位开始，到"结束"位的前一位结束（不包含结束位本身)。
翻转字符串  : a="abvcy"    a=a[4:0:-1]