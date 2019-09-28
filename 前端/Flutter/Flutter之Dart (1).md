## Dart 变量

  dart是一个强大的脚本类语言，可以不预先定义变量类型 ，自动会类型推导

  dart中定义变量可以通过var关键字可以通过类型来申明变量

  如：

```dart
var str='this is var';

String str='this is var';

int str=123;
//字符串
String str='你好dart';
print(str);
//数字类型
int myNum=12354;
 print(myNum);

//dart里面有类型校验
var str='';
str=1234;
print(str);
```

  注意： var 后就不要写类型 ，  写了类型 不要var   两者都写   var  a int  = 5;  报错



####  final 和 const修饰符  

const值不变 一开始就得赋值
final 可以开始不赋值 只能赋一次 ; 而final不仅有const的编译时常量的特性，最重要的它是运行时常量，并且final是**惰性初始化**，即在运行时第一次使用前才初始化


永远不改量的量，请使用final或const修饰它，而不是使用var或其他变量类型。

```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';
    
const bar = 1000000; // Unit of pressure (dynes/cm2)
const double atm = 1.01325 * bar; // Standard atmosphere
```



#### Dart的命名规则

    1、变量名称必须由数字、字母、下划线和美元符($)组成。
    2.注意：标识符开头不能是数字
    3.标识符不能是保留字和关键字。   
    4.变量的名字是区分大小写的如: age和Age是不同的变量。在实际的运用中,也建议,不要用一个单词大小写区分两个变量。
    5、标识符(变量名称)一定要见名思意 ：变量名称建议用名词，方法名称建议用动词  
###  



#### 入口方法的两种定义方式

```dart
main(){
  print('你好dart');
}
  ///这也是一个注释

//表示main方法没有返回值
void main(){
 print('你好dart');
}
  
```



## Dart 数据类型

Dart中支持以下数据类型：

  常用数据类型：
      Numbers（数值）:
          int
          double
      Strings（字符串）
          String
      Booleans(布尔)
          bool
      List（数组）
          在Dart中，数组是列表对象，所以大多数人只是称它们为列表
      Maps（字典）
          通常来说，Map 是一个键值对相关的对象。 键和值可以是任何类型的对象。**每个 键 只出现一次， 而一个值则可以出现多次**

  项目中用不到的数据类型 （用不到）：

+ Runes 
          Rune是UTF-32编码的字符串。它可以通过文字转换成符号表情或者代表特定的文字。

        main() {
          var clapping = '\u{1f44f}';
          print(clapping);
          print(clapping.codeUnits);
          print(clapping.runes.toList());
      
          Runes input = new Runes(
              '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
          print(new String.fromCharCodes(input));
        }


+ Symbols
      Symbol对象表示在Dart程序中声明的运算符或标识符。您可能永远不需要使用符号，但它们对于按名称引用标识符的API非常有用，因为缩小会更改标识符名称而不会更改标识符符号。要获取标识符的符号，请使用符号文字，它只是＃后跟标识符：

   ​    在 Dart 中符号用 # 开头来表示，入门阶段不需要了解这东西，可能永远也用不上。
     http://dart.goodev.org/guides/libraries/library-tour#dartmirrors---reflection



#### 字符串类型

1、字符串定义的几种方式

```dart
var str1='this is str1';
String str2="this is str2";

print(str1);
print(str2);
```

2、字符串的拼接

```dart
String str1='你好';
var str2='Dart';

print("$str1 $str2");
print(str1 + str2);
print(str1 +" "+ str2);
```



#### 数值类型

1、int   必须是整型


    int a=123;
    
    a=45;
    
    print(a);

2、double  既可以是整型 也可是浮点型

    double b=23.5;
    
    b=24;
    
    print(b);

3、运算符

    // + - * / %
    
    var c=a+b;
    print(c);


#### 布尔类型

1、bool

    bool flag1=true; 

2、条件判断语句


      var a2 = 123;
      var b2 = '123';
      var b3 = 123;
    
      print(a2 == b2);
      print(b3 == a2);



#### List（数组/集合）

1、第一种定义List的方式

```dart
  var l1=['aaa','bbbb','cccc'];
  print(l1);
  print(l1.length);
  print(l1[1]);
```



2、第二种定义List的方式

```dart
   var l2=new List();

   l2.add('张三'); 

   print(l2);
   print(l2[0]);
```

3、定义List指定类型

```dart
  var l3=new List<String>();
  l3.add('张三');
```

#### Maps（字典）

```dart
//第一种定义 Maps的方式
   var person={
     "name":"张三",
     "age":20,
     "work":["程序员","送外卖"]
   };

   print(person);
   print(person["name"]);
   print(person["age"]);
   print(person["work"]);
```

```dart
 //第二种定义 Maps的方式 
  var p=new Map();

  p["name"]="李四";
  p["age"]=22;
  p["work"]=["程序员","送外卖"];
  print(p);
  print(p["age"]);
```

#### Dart判断数据类型    

is 关键词来判断类型

```dart 
  // var str='1234';
  var str=123;

  if(str is String){
    print('是string类型');
  }else if(str is int){
    print('int');
  }else{
    print('其他类型');
  } 
```

#### ++  --   表示自增 自减 1

```
在赋值运算里面 如果++ -- 写在前面 这时候先运算 再赋值，如果++ --写在后面 先赋值后运行运算
```

```dart
    var a=10;
    var b=a--;

    print(a);  //9
    print(b);  //10

   var a=10;
   a++;   //a=a+1;
   var a=10;
   a--;    //a=a-1;

   var a=10;
   var b=a++;
   print(a);  //11
   print(b);  //10

   var a=10;
   var b=++a;
   print(a);  //11
   print(b);  //11 

   var a=10;
   var b=--a;
   print(a);  //9
   print(b);  //9
 
   var a=10;
   var b=a--;
   print(a);  //9
   print(b);  //10
```



### Dart循环

####  for基本语法

        	  for (int i = 1; i<=100; i++) {   
                print(i);
              }
        	//第一步，声明变量int i = 1;
        	//第二步，判断i <=100
        	//第三步，print(i);
        	//第四步，i++
        	//第五步 从第二步再来，直到判断为false


#### do{  }while( );

	语法格式:
		
		while(表达式/循环条件){			
			
		} do{
			语句/循环体
			
		}while(表达式/循环条件);


注意： 1、最后的分号不要忘记
			 2、循环条件中使用的变量需要经过初始化
		     3、循环体中，应有结束循环的条件，否则会造成死循环。



#### break 和 continue

	break语句功能:
	      1、在switch语句中使流程跳出switch结构。
	      2、在循环语句中使流程跳出当前循环,遇到break 循环终止，后面代码也不会执行
	      
	      强调:
	      1、如果在循环中已经执行了break语句,就不会执行循环体中位于break后的语句。
	      2、在多层循环中,一个break语句只能向外跳出一层
	
	    break可以用在switch case中 也可以用在 for 循环和 while循环中
	
	
	continue语句的功能:
			  
	    【注】只能在循环语句中使用,使本次循环结束，即跳过循环体重下面尚未执行的语句，接着进行下次的是否执行循环的判断。
	
	    continue可以用在for循环以及 while循环中，但是不建议用在while循环中，不小心容易死循环
