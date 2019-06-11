### HTML技巧篇：实现元素水平与垂直居中的几种方式

**1）单行文本的居中**

主要实现css代码：

水平居中：text-align:center;垂直居中：line-height:XXpx; /*line-height与元素的height的值一致*/

我们先来看这样一个例子，加入我们这里有一个div，宽度和高度为300px，背景颜色为黑色，然后在div中有**一行简短文字**，我们只需要使用line-height:200px;就可以实现文字的居中效果，具体的代码如下所示：

![img](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=2718996991,3468619810&fm=173&app=49&f=JPEG?w=590&h=537&s=86B055CA92B6966F1C655C0B0000A0C0)

![img](https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=2385258386,112505258&fm=173&app=49&f=JPEG?w=309&h=313&s=2B453B6A53FEB66906E99D0B0000E0C1)

由上图可以看出我们实现了单行文字的垂直居中效果，但是很多时候我们的文字并不知道具体有多少，可能有一行，也可能有很多行，那么遇到多行文字的这种问题我们要如何处理呢。

**2）多行文本的垂直居中**

对于多行文本的垂直居中我们有很多种实现方式，我们这里逐个的来看一下；

1、使用display:table来实现

主要实现代码：

display: table使块状元素成为一个块级表格;

display: table-cell;子元素设置成表格单元格;

vertical-align: middle;使表格内容居中显示，即可实现垂直居中的效果;

具体的html与css的代码就如下所示：

![img](https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=1456261646,3400587635&fm=173&app=49&f=JPEG?w=589&h=605&s=86B055CAD2B6926D5C5D4C070000F0C0)

![img](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=2550155626,1598781602&fm=173&app=49&f=JPEG?w=319&h=311&s=6F70AB425BFFB3CC48E5E50B0000A0C1)

2、使用absolute与transform配合实现

主要实现代码：

position:absolute; 首先给文本绝对定位；

left:50%;top:50%;transform:translate(-50%,-50%); 让文本距离盒子左边和上边分别为50%，再用transform向左（上）平移它自己宽度（高度）的50%，也就达到居中效果了。

具体的html与css的代码就如下所示：

![img](https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=3034476344,3620284357&fm=173&app=49&f=JPEG?w=594&h=642&s=B29055CA82B6936E10E55D0F0000E0C0)

3、使用flex实现

主要实现代码：

display: flex;设置 display 属性的值为 flex 将其定义为弹性容器

align-items: center;定义项目在交叉轴（纵轴）上如何对齐，垂直对齐居中

justify-content: center; 定义了项目在主轴上的对齐方式，水平对齐居中

具体的html与css的代码就如下所示：

![img](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=3405826744,2921948974&fm=173&app=49&f=JPEG?w=605&h=581&s=92B055CA92B4906F5E6D7C030000E0C0)