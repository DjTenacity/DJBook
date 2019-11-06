####  List里面常用的属性和方法：

    常用属性：
        length          长度
        reversed        翻转
        isEmpty         是否为空
        isNotEmpty      是否不为空
    常用方法：
        add         增加
        addAll      拼接数组
        indexOf     查找  传入具体值
        remove      删除  传入具体值
        removeAt    删除  传入索引值
        fillRange   修改
        insert(index,value);            指定位置插入
        insertAll(index,list)           指定位置插入List
        toList()    其他类型转换成List
        join()      List转换成字符串
        split()     字符串转化成List
        forEach
        map
        where
        any
        every
 ```dart
  List myList = ['香蕉', '苹果', '西瓜'];
  print(myList.length);
  print(myList.isEmpty);
  print(myList.isNotEmpty);
  // 返回   Iterable<E> get reversed;
  print(myList.reversed); //对列表倒序排序
  var newMyList = myList.reversed.toList();
  print(newMyList);

  myList.insert(1, 'aaa'); //插入  一个
  print(myList);

  myList.insertAll(1, ['aaa', 'bbb']); //插入 多个
  print(myList); 


  var str=myList.join('-');   //list转换成字符串
  print(str);
  var list=str.split('-');
  print(list);
//[香蕉, aaa, bbb, aaa, 苹果, 西瓜]
//香蕉-aaa-bbb-aaa-苹果-西瓜
//[香蕉, aaa, bbb, aaa, 苹果, 西瓜]
 ```



#### Set 

 用它最主要的功能就是去除数组重复内容

 Set是没有顺序且不能重复的集合，所以不能通过索引去获取值

```dart
  var s = new Set();
  s.add('香蕉');
  s.add('苹果');
  s.add('苹果');

  print(s); //{香蕉, 苹果} 
  print(s.toList());

  myList = ['香蕉', '苹果', '西瓜', '香蕉', '苹果', '香蕉', '苹果']; 
  s = new Set();

  s.addAll(myList); 
  print(s); 
  print(s.toList());
```



#### 映射(Maps)是无序的键值对

```
  常用属性：
        keys            获取所有的key值
        values          获取所有的value值
        isEmpty         是否为空
        isNotEmpty      是否不为空
  常用方法:
        remove(key)     删除指定key的数据
        addAll({...})   合并映射  给映射内增加属性
        containsValue   查看映射内的值  返回true/false
        forEach   
        map
        where
        any
        every
```

```dart
//创建方式
 Map person={
    "name":"张三",
    "age":20,
    "sex":"男"
  };

   var m=new Map();
   m["name"]="李四";

   print(person);
   print(m); 

  //常用方法：
  person.addAll({
    "work": ['敲代码', '送外卖'],
    "height": 160
  });

  print(person); 
  person.remove("sex");
  print(person);
  print(person.containsValue('张三'));
```

#### forEach map where any every

```dart
 for (var i = 0; i < myList.length; i++) {
    print(myList[i]);
  }

  for (var item in myList) {
    print(item);
  }

  myList.forEach((value) {
    print("$value");
  });

  myList = [1, 3, 4];
  List newList = new List();
  for (var i = 0; i < myList.length; i++) {
    newList.add(myList[i] * 2);
  }
  print(newList);

  newList = myList.map((value) {
    return value * 2;
  });
  print(newList.toList());

  myList = [1, 3, 4, 5, 7, 8, 9];

  newList = myList.where((value) {
    return value > 5;
  });
  print(newList.toList());

  myList = [1, 3, 4, 5, 7, 8, 9];

  var f = myList.any((value) {
    //只要集合里面有满足条件的就返回true

    return value > 5;
  });
  print(f);

  myList = [1, 3, 4, 5, 7, 8, 9];

  f = myList.every((value) {
    //每一个都满足条件返回true  否则返回false

    return value > 5;
  });
  print(f);

  // set

  s = new Set();
  s.addAll([1, 222, 333]);
  s.forEach((value) => print(value));

  //map

  person = {"name": "张三", "age": 20};

  person.forEach((key, value) {
    print("$key---$value");
  });
```

