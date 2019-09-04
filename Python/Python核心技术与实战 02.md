# Python核心技术与实战 

## 01 | 列表和元组，到底用哪一个？

### 列表 list 和元组 tuple 基础

列表和元组，都是**一个可以放置任意数据类型的有序集合**。

- **列表是动态的**，长度大小不固定，可以随意地增加、删减或者改变元素（mutable）。
- **元组是静态的**，长度大小固定，无法增加删减或者改变（immutable）。
  - 如果想对已有的元组做任何"改变"，就只能重新开辟一块内存，创建新的元组了。

```python
tup = (1, 2, 3, 4)
new_tup = tup + (5, ) # 创建新的元组 new_tup，并依次填充原元组的值
new _tup
(1, 2, 3, 4, 5)
 
l = [1, 2, 3, 4]
l.append(5) # 添加元素 5 到原列表的末尾
l
[1, 2, 3, 4, 5]
```

**Python 中的列表和元组都支持负数索引**，-1 表示最后一个元素，-2 表示倒数第二个元素

除了基本的初始化，索引外，**列表和元组都支持切片操作**

```python
l = [1, 2, 3, 4]
l[1:3] # 返回列表中索引从 1 到 2 的子列表
[2, 3]
 
tup = (1, 2, 3, 4)
tup[1:3] # 返回元组中索引从 1 到 2 的子元组
(2, 3) 
```

列表和元组都**可以随意嵌套**

两者也可以通过 list() 和 tuple() 函数相互转换：

```python
list((1, 2, 3))
[1, 2, 3]
 
tuple([1, 2, 3])
(1, 2, 3)
```

列表和元组常用的内置函数：

```python
l = [3, 2, 3, 7, 8, 1]
l.count(3) 
2
l.index(7)
3
l.reverse()
l
[1, 8, 7, 3, 2, 3]
l.sort()
l
[1, 2, 3, 3, 7, 8]
 
tup = (3, 2, 3, 7, 8, 1)
tup.count(3)
2
tup.index(7)
3
list(reversed(tup))
[1, 8, 7, 3, 2, 3]
sorted(tup)
[1, 2, 3, 3, 7, 8]
```

- count(item) 表示统计列表 / 元组中 item 出现的次数。
- index(item) 表示返回列表 / 元组中 item 第一次出现的索引。
- list.reverse() 和 list.sort() 分别表示原地倒转列表和排序（注意，元组没有内置的这两个函数)。
- reversed() 和 sorted() 同样表示对列表 / 元组进行倒转和排序，但是会返回一个倒转后或者排好序的新的列表 / 元组。

### 列表和元组存储方式的差异

列表是动态的、可变的，而元组是静态的、不可变的。这样的差异，势必会影响两者存储方式

```python
l = [1, 2, 3]
l.__sizeof__()
64
tup = (1, 2, 3)
tup.__sizeof__()
48
```

由于列表是动态的，所以它需要存储指针，来指向对应的元素（上述例子中，对于 int 型，8 字节）。另外，由于列表可变，所以需要额外存储已经分配的长度大小（8 字节），这样才可以实时追踪列表空间的使用情况，当空间不足时，及时分配额外空间。

```python
l = []
l.__sizeof__() // 空列表的存储空间为 40 字节
40
l.append(1)
l.__sizeof__() 
72 // 加入了元素 1 之后，列表为其分配了可以存储 4 个元素的空间 (72 - 40)/8 = 4
l.append(2) 
l.__sizeof__()
72 // 由于之前分配了空间，所以加入元素 2，列表空间不变
l.append(3)
l.__sizeof__() 
72 // 同上
l.append(4)
l.__sizeof__() 
72 // 同上
l.append(5)
l.__sizeof__() 
104 // 加入元素 5 之后，列表的空间不足，所以又额外分配了可以存储 4 个元素的空间
```



可以看到，为了减小每次增加 / 删减操作时空间分配的开销，Python 每次分配空间时都会额外多分配一些，这样的机制（over-allocating）保证了其操作的高效性：增加 / 删除的时间复杂度均为 O(1)。

而元组长度大小固定，元素不可变，所以存储空间固定。

### 列表和元组的性能

可以得出结论：元组要比列表更加轻量级一些，所以总体上来说，元组的性能速度要略优于列表。

另外，Python 会在后台，对静态数据做一些**资源缓存**（resource caching）。通常来说，因为垃圾回收机制的存在，如果一些变量不被使用了，Python 就会回收它们所占用的内存，返还给操作系统，以便其他变量或其他应用使用。

但是对于一些静态变量，比如元组，如果它不被使用并且占用空间不大时，Python 会暂时缓存这部分内存。这样，下次我们再创建同样大小的元组时，Python 就可以不用再向操作系统发出请求，去寻找内存，而是可以直接分配之前缓存的内存空间，这样就能大大加快程序的运行速度。

例子，是计算**初始化**一个相同元素的列表和元组分别所需的时间。我们可以看到，元组的初始化速度，要比列表快 5 倍。

```python
python3 -m timeit 'x=(1,2,3,4,5,6)'
20000000 loops, best of 5: 9.97 nsec per loop
python3 -m timeit 'x=[1,2,3,4,5,6]'
5000000 loops, best of 5: 50.1 nsec per loop
```

但如果是**索引操作**的话，两者的速度差别非常小，几乎可以忽略不计。

```python
python3 -m timeit -s 'x=[1,2,3,4,5,6]' 'y=x[3]'
10000000 loops, best of 5: 22.2 nsec per loop
python3 -m timeit -s 'x=(1,2,3,4,5,6)' 'y=x[3]'
10000000 loops, best of 5: 21.9 nsec per loop

```

如果想要增加、删减或者改变元素，那么列表显然更优。原因你现在肯定知道了，那就是对于元组，你必须得通过新建一个元组来完成。



### 列表和元组的使用场景

 如果存储的数据和数量不变， 那么肯定选用元组更合适, 否则用列表更合适。

### 总结

列表和元组都是有序的，可以存储任意数据类型的集合，区别主要在于下面这两点。

- 列表是动态的，长度可变，可以随意的增加、删减或改变元素。列表的存储空间略大于元组，性能略逊于元组。
- 元组是静态的，长度大小固定，不可以对元素进行增加、删减或者改变操作。元组相对于列表更加轻量级，性能稍优。





## 02 | 字典、集合

### 字典、集合基础

​	字典是一系列由键（key）和值（value）配对组成的元素的集合，在 Python3.7+，字典被确定为有序（注意：在 3.6 中，字典有序是一个 implementation detail，在 3.7 才正式成为语言特性，因此 3.6 中无法 100% 确保其有序性），而 3.6 之前是无序的，其长度大小可变，元素可以任意地删减和改变。

​	**相比于列表和元组，字典的性能更优，特别是对于查找、添加和删除操作，字典都能在常数时间复杂度内完成。**

​	而**集合和字典基本相同，唯一的区别，就是集合没有键和值的配对，是一系列无序的、唯一的元素组合。**

#### 创建和访问

```python
d1 = {'name': 'jason', 'age': 20, 'gender': 'male'}
d2 = dict({'name': 'jason', 'age': 20, 'gender': 'male'})
d3 = dict([('name', 'jason'), ('age', 20), ('gender', 'male')])
d4 = dict(name='jason', age=20, gender='male') 
d1 == d2 == d3 ==d4
True
 
s1 = {1, 2, 3}
s2 = set([1, 2, 3])
s1 == s2
True
```

**Python 中字典和集合，无论是键还是值，都可以是混合类型**

   集合:  `s = {1, 'hello', 5.0}`

**字典访问可以直接索引键，如果不存在，就会抛出异常**

```python
d = {'name': 'jason', 'age': 20}
d['name']
'jason'
d['location']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'location'
    
```

**字典也可以使用 get(key, default) 函数来进行索引。如果键不存在，调用 get() 函数可以返回一个默认值**。比如下面这个示例，返回了`'null'`

```python
d = {'name': 'jason', 'age': 20}
d.get('name')
'jason'
d.get('location', 'null')
'null'
```

**集合并不支持索引操作，因为集合本质上是一个哈希表，和列表不一样**(会抛出异常)。



判断一个元素在不在字典或集合内，我们可以用 value in dict/set 来判断。

```python
d = {'name': 'jason', 'age': 20}
'name' in d
True
'location' in d
False
```

#### 增加、删除、更新等操作

```python
d = {'name': 'jason', 'age': 20}
d['gender'] = 'male' # 增加元素对'gender': 'male'
d['dob'] = '1999-02-01' # 增加元素对'dob': '1999-02-01'
d
{'name': 'jason', 'age': 20, 'gender': 'male', 'dob': '1999-02-01'}
d['dob'] = '1998-01-01' # 更新键'dob'对应的值 
d.pop('dob') # 删除键为'dob'的元素对
'1998-01-01'
d
{'name': 'jason', 'age': 20, 'gender': 'male'}
 
s = {1, 2, 3}
s.add(4) # 增加元素 4 到集合
s
{1, 2, 3, 4}
s.remove(4) # 从集合中删除元素 4
s
{1, 2, 3}
```

**集合的 pop() 操作是删除集合中最后一个元素，可是集合本身是无序的，你无法知道会删除哪个元素，因此这个操作得谨慎使用**



字典 根据键或值，进行升序或降序排序：

```python
d = {'b': 1, 'a': 2, 'c': 10}
d_sorted_by_key = sorted(d.items(), key=lambda x: x[0]) # 根据字典键的升序排序
d_sorted_by_value = sorted(d.items(), key=lambda x: x[1]) # 根据字典值的升序排序
d_sorted_by_key
[('a', 2), ('b', 1), ('c', 10)]
d_sorted_by_value
[('b', 1), ('a', 2), ('c', 10)]
```

这里返回了一个列表。列表中的每个元素，是由原字典的键和值组成的元组



对于集合，其排序和前面讲过的列表、元组很类似，直接调用 sorted(set) 即可，结果会返回一个排好序的列表。

```python
s = {3, 4, 2, 1}
sorted(s) # 对集合的元素进行升序排序
[1, 2, 3, 4]
```



### 字典和集合性能

典和集合是进行过性能高度优化的数据结构，特别是对于查找、添加和删除操作



```python
def find_product_price(products, product_id):
    for id, price in products:
        if id == product_id:
            return price
    return None 
     
products = [
    (143121312, 100), 
    (432314553, 30),
    (32421912367, 150) 
]
 
print('The price of product 432314553 is {}'.format(find_product_price(products, 432314553)))
 
# 输出
The price of product 432314553 is 30
```



假设列表有 n 个元素，而查找的过程要遍历列表，那么时间复杂度就为 O(n)。即使我们先对列表进行排序，然后使用二分查找，也会需要 O(logn) 的时间复杂度，更何况，列表的排序还需要 O(nlogn) 的时间。

但如果我们用字典来存储这些数据，那么查找就会非常便捷高效，只需 O(1) 的时间复杂度就可以完成。原因也很简单，刚刚提到过的，**字典的内部组成是一张哈希表，你可以直接通过键的哈希值，找到其对应的值。**



给定某件商品的 ID，我们**要找出其价格**。

```python
products = {
  143121312: 100,
  432314553: 30,
  32421912367: 150
}
print('The price of product 432314553 is {}'.format(products[432314553])) 
 
# 输出
The price of product 432314553 is 30
```

现在需求变成，要找出这些**商品有多少种不同的价格**



如果还是选择使用列表，对应的代码如下，其中，A 和 B 是两层循环。同样假设原始列表有 n 个元素，那么，在最差情况下，需要 O(n^2) 的时间复杂度。

```python
# list version
def find_unique_price_using_list(products):
    unique_price_list = []
    for _, price in products: # A
        if price not in unique_price_list: #B
            unique_price_list.append(price)
    return len(unique_price_list)
 
products = [
    (143121312, 100), 
    (432314553, 30),
    (32421912367, 150),
    (937153201, 30)
]
print('number of unique price is: {}'.format(find_unique_price_using_list(products)))
 
# 输出
number of unique price is: 3
```

但如果我们选择使用集合这个数据结构，由于集合是高度优化的哈希表，里面元素不能重复，并且其添加和查找操作只需 O(1) 的复杂度，那么，总的时间复杂度就只有 O(n)

```python
# set version
def find_unique_price_using_set(products):
    unique_price_set = set()
    for _, price in products:
        unique_price_set.add(price)
    return len(unique_price_set)        
 
products = [
    (143121312, 100), 
    (432314553, 30),
    (32421912367, 150),
    (937153201, 30)
]
print('number of unique price is: {}'.format(find_unique_price_using_set(products)))
 
# 输出
number of unique price is: 3
```



下面的代码，初始化了含有 100,000 个元素的产品，并分别计算了使用列表和集合来统计产品价格数量的运行时间：

```python
import time
id = [x for x in range(0, 100000)]
price = [x for x in range(200000, 300000)]
products = list(zip(id, price))
 
# 计算列表版本的时间
start_using_list = time.perf_counter()
find_unique_price_using_list(products)
end_using_list = time.perf_counter()
print("time elapse using list: {}".format(end_using_list - start_using_list))
## 输出
time elapse using list: 41.61519479751587
 
# 计算集合版本的时间
start_using_set = time.perf_counter()
find_unique_price_using_set(products)
end_using_set = time.perf_counter()
print("time elapse using set: {}".format(end_using_set - start_using_set))
# 输出
time elapse using set: 0.008238077163696289
```

以看到，仅仅十万的数据量，两者的速度差异就如此之大。

### 字典和集合的工作原理

通过举例以及与列表的对比，看到了字典和集合操作的高效性。不过，字典和集合为什么能够如此高效，特别是查找、插入和删除操作？

这当然和字典、集合内部的数据结构密不可分。不同于其他数据结构，字典和集合的内部结构都是一张哈希表。

- 对于字典而言，这张表存储了哈希值（hash）、键和值这 3 个元素。
- 而对集合来说，区别就是哈希表内没有键和值的配对，只有单一的元素了。

老版本 Python 的哈希表结构如下所示：

```
--+-------------------------------+
  | 哈希值 (hash)  键 (key)  值 (value)
--+-------------------------------+
0 |    hash0      key0    value0
--+-------------------------------+
1 |    hash1      key1    value1
--+-------------------------------+
2 |    hash2      key2    value2
--+-------------------------------+
. |           ...
__+_______________________________+
 

```

不难想象，随着哈希表的扩张，它会变得越来越稀疏。举个例子，比如我有这样一个字典：`{'name': 'mike', 'dob': '1999-01-01', 'gender': 'male'}`

它会存储为类似下面的形式：

```
entries = [
['--', '--', '--']
[-230273521, 'dob', '1999-01-01'],
['--', '--', '--'],
['--', '--', '--'],
[1231236123, 'name', 'mike'],
['--', '--', '--'],
[9371539127, 'gender', 'male']
]
```

一个位置要同时分配哈希值，键和值的空间 , 这样的设计结构显然非常浪费存储空间。为了提高存储空间的利用率，现在的哈希表除了字典本身的结构，会把索引和哈希值、键、值单独分开，也就是下面这样新的结构：

```
Indices
----------------------------------------------------
None | index | None | None | index | None | index ...
----------------------------------------------------
 
Entries
--------------------
hash0   key0  value0
---------------------
hash1   key1  value1
---------------------
hash2   key2  value2
---------------------
        ...
---------------------
```

在新的哈希表结构下的存储形式，就会变成下面这样：

```
indices = [None, 1, None, None, 0, None, 2]
entries = [
[1231236123, 'name', 'mike'],
[-230273521, 'dob', '1999-01-01'],
[9371539127, 'gender', 'male']
]

```

None表示indices这个array上对应的位置没有元素，index表示有元素，并且对应entries这个array index位置上的元素

可以很清晰地看到，空间利用率得到很大的提高。

#### 插入操作

每次向字典或集合插入一个元素时，Python 会首先计算键的哈希值（hash(key)），再和 mask = PyDicMinSize - 1 做与操作，**计算这个元素应该插入哈希表的位置** index = hash(key) & mask。**如果哈希表中此位置是空的，那么这个元素就会被插入其中。**

而如果**此位置已被占用**，Python 便会比较两个元素的**哈希值和键是否相等**。

- 若两者都相等，则表明这个元素已经存在，如果**值不同，则更新值**。
- 若两者中有一个不相等，这种情况我们通常称为**哈希冲突**（hash collision），意思是**两个元素的键不相等，但是哈希值相等**。这种情况下，Python 便**会继续寻找表中空余的位置，直到找到位置为止**。

值得一提的是，通常来说，遇到这种情况，最简单的方式是**线性寻找**，即从这个位置开始，挨个往后寻找空位。当然，Python 内部对此进行了优化（这一点无需深入了解，你有兴趣可以查看源码，我就不再赘述），让这个步骤更加高效。 ( 哈希表常与链表一起使用，譬如用来解决哈希冲突。线性寻找是最简单的，但是不是最高效的 )

#### 查找操作

和前面的插入操作类似，Python 会**根据哈希值**，找到其应该处于的位置；然后，比较哈希表这个位置中元素的哈希值和键，与需要查找的元素是否相等。如果相等，则直接返回；如果不等，则继续查找，直到找到空位或者抛出异常为止。

#### 删除操作

对于删除操作，Python **会暂时对这个位置的元素，赋于一个特殊的值，等到重新调整哈希表的大小时，再将其删除。**

不难理解，**哈希冲突的发生，往往会降低字典和集合操作的速度**。因此，为了保证其高效性，字典和集合内的哈希表，通常会保证其至少留有 1/3 的剩余空间。随着元素的不停插入，**当剩余空间小于 1/3 时，Python 会重新获取更大的内存空间，扩充哈希表。不过，这种情况下，表内所有的元素位置都会被重新排放**。随着哈希表的扩张，它会变得越来越稀疏。

虽然哈希冲突和哈希表大小的调整，都会导致速度减缓，但是这种情况发生的次数极少。所以，平均情况下，这仍能保证插入、查找和删除的时间复杂度为 O(1)。

### 总结

这节课，我们一起学习了字典和集合的基本操作，并对它们的高性能和内部存储结构进行了讲解。

字典在 Python3.7+ 是有序的数据结构，而集合是无序的，其内部的哈希表存储结构，保证了其查找、插入、删除操作的高效性。所以，字典和集合通常运用在对元素的高效查找、去重等场景。

**1.** 下面初始化字典的方式，哪一种更高效？

```python
# Option A
d = {'name': 'jason', 'age': 20, 'gender': 'male'}
 
# Option B
d = dict({'name': 'jason', 'age': 20, 'gender': 'male'})

```

**2.** 字典的键可以是一个列表吗？下面这段代码中，字典的初始化是否正确呢？如果不正确，可以说出你的原因吗？

```python
d = {'name': 'jason', ['education']: ['Tsinghua University', 'Stanford University']}
```

思考题 1：
第一种方法更快，原因感觉上是和之前一样，就是不需要去调用相关的函数，而且  {} 应该是关键字，内部会去直接调用底层C写好的代码  .   直接｛｝的方式，更高效。可以使用dis分析其字节码

思考题 2:
用列表作为 Key 在这里是不被允许的**，因为列表是一个动态变化的数据结构，字典当中的 key 要求是不可变的，**原因也很好理解，key 首先是不重复的，如果 Key 是可以变化的话，那么随着 Key 的变化，这里就有可能就会有重复的 Key，那么这就和字典的定义相违背；如果把**这里的列表换成之前我们讲过的元组是可以的，因为元组不可变** 

 Python 3.7 以后插入有序变为字典的特性。构造新字典的方式：
\1. double star
\>>> d1 = {'name': 'jason', 'age': 20, 'gender': 'male'}
\>>> d2 = {'hobby': 'swim', **d1}
\2. update 函数：
\>>> d3 = {'hobby': 'swim'}
\>>> d3.update(d1)
我们可以看到这两种方式得到的字典是满足插入有序的：
\>>> d3
{'hobby': 'swim', 'name': 'jason', 'age': 20, 'gender': 'male'}

 

# 03 | 深入浅出字符串

## 字符串基础

什么是字符串呢？字符串是由独立字符组成的一个序列，通常包含在单引号（`''`）双引号（`""`）或者三引号之中（`''' '''`或`""" """`，两者一样）

Python 中单引号、双引号和三引号的字符串是一模一样的，没有区别，比如下面这个例子中的 s1、s2、s3 完全一样。

```python
s1 = 'hello'
s2 = "hello"
s3 = """hello"""
s1 == s2 == s3
True
```

Python 同时支持这三种表达方式，很重要的一个原因就是，这样方便你在字符串中，内嵌带引号的字符串。比如：

```
"I'm a student"复制代码
```

Python 的三引号字符串，则主要应用于多行字符串的情境，比如函数的注释等等。

```python
def calculate_similarity(item1, item2):
    """
    Calculate similarity between two items
    Args:
        item1: 1st item
        item2: 2nd item
    Returns:
      similarity score between item1 and item2
    """
```

#### 常见的的转义字符

![](D:\DJGitBook\img\python\常见的的转义字符.png)

```python
s = 'a\nb\tc'
print(s)
a
b	c
# 打印出来的输出，就是字符 a，换行，字符 b，然后制表符，最后打印字符 c
len(s)
5
# 整个字符串 s 仍然只有 5 个元素。
```

可以把字符串想象成一个由单个字符组成的数组，所以，Python 的字符串同样支持索引，切片和遍历等等

```python
name = 'jason'
name[0]
'j'
name[1:3]
'as'

for char in name:
    print(char)   
```

Python 的字符串是不可变的

```python
s = 'hello'
s[0] = 'H'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment

```



Python 中字符串的改变，通常只能通过创建新的字符串来完成。

```python
s = 'H' + s[1:]
s = s.replace('h', 'H')
# 第一种方法，是直接用大写的'H'，通过加号'+'操作符，与原字符串切片操作的子字符串拼接而成新的字符串。
# 第二种方法，是直接扫描原字符串，把小写的'h'替换成大写的'H'，得到新的字符串。
```

Java，有可变的字符串类型，比如 StringBuilder，每次添加、改变或删除字符（串），无需创建新的字符串，时间复杂度仅为 O(1)。这样就大大提高了程序的运行效率。

```python
str1 += str2  # 表示 str1 = str1 + str2

```



```python
s = ''
for n in range(0, 100000):
    s += str(n)
```

每次创建一个新的字符串，都需要 O(n) 的时间复杂度。因此，总的时间复杂度就为 O(1) + O(2) + … + O(n) = O(n^2)。这个结论只适用于老版本的 Python 了

自从 Python2.5 开始，每次处理字符串的拼接操作时（str1 += str2），Python 首先会检测 str1 还**有没有其他的引用**。如果没有的话，就会尝试原地**扩充字符串 buffer 的大小**，而不是重新分配一块内存来创建新的字符串并拷贝。上述例子中的时间复杂度就仅为 O(n) 了。

以后在写程序遇到字符串拼接时，如果使用’+='更方便，就放心地去用吧，不用过分担心效率问题了。

字符串拼接问题，除了使用加法操作符，我们还可以使用字符串内置的 join 函数。string.join(iterable)，表示把每个元素都按照指定的格式连接起来。

```python
l = []
for n in range(0, 100000):
    l.append(str(n))
l = ' '.join(l) 
```

列表的 append 操作是 O(1) 复杂度，字符串同理。因此，这个含有 for 循环例子的时间复杂度为 n*O(1)=O(n)。



#### 分割函数 split()

- string.strip(str)，表示去掉首尾的 str 字符串；
- string.lstrip(str)，表示只去掉开头的 str 字符串；
- string.rstrip(str)，表示只去掉尾部的 str 字符串。

## 字符串的格式化

  string.format()，就是所谓的格式化函数；而大括号{}就是所谓的格式符，用来为后面的真实值——变量 name 预留位置。

```python
print('no data available for person with id: {}, name: {}'.format(id, name))
# 如果id = '123'、name='jason'
# 输出'no data available for person with id: 123, name: jason'

```



string.format() 是最新的字符串格式函数与规范。自然，我们还有其他的表示方法，比如在 Python 之前版本中，字符串格式化通常用 % 来表示，那么上述的例子，就可以写成下面这样：

```python
print('no data available for person with id: %s, name: %s' % (id, name))
```



写程序时， 推荐使用 format 函数，毕竟这是最新规范，也是官方文档推荐的规范。

## 总结

- Python 中字符串使用单引号、双引号或三引号表示，三者意义相同，并没有什么区别。其中，三引号的字符串通常用在多行字符串的场景。
- Python 中字符串是不可变的（前面所讲的新版本 Python 中拼接操作’+='是个例外）。因此，随意改变字符串中字符的值，是不被允许的。
- Python 新版本（2.5+）中，字符串的拼接变得比以前高效了许多，你可以放心使用。
- Python 中字符串的格式化（string.format）常常用在输出、日志的记录等场景。



## 思考题

在新版本的 Python（2.5+）中，下面的两个字符串拼接操作，你觉得哪个更优呢？

```python
s = ''
for n in range(0, 100000):
    s += str(n)
l = []
****
for n in range(0, 100000):
    l.append(str(n))
    
s = ' '.join(l)
```

关于思考题，如果字符串拼接的次数较少，比如range(100)，那么方法一更优，因为时间复杂度精确的来说第一种是O(n)，第二种是O(2n)，如果拼接的次数较多，比如range(1000000)，方法二稍快一些，虽然方法二会遍历两次，但是join的速度其实很快，列表append和join的开销要比字符串+=小一些。



个人提一个更加pythonic，更加高效的办法
s = " ".join(map(str, range(0, 10000)))*



# 04 | Python “黑箱”：输入与输出

## 输入输出基础

```python
name = input('your name:')
gender = input('you are a boy?(y/n)')
 
###### 输入 ######
your name:Jack
you are a boy?
 
welcome_str = 'Welcome to the matrix {prefix} {name}.'
welcome_dic = {
    'prefix': 'Mr.' if gender == 'y' else 'Mrs',
    'name': name
}
 
print('authorizing...')
print(welcome_str.format(**welcome_dic))
 
########## 输出 ##########
authorizing...
Welcome to the matrix Mr. Jack.
```

input() 函数暂停程序运行，同时等待键盘输入；直到回车被按下，函数的参数即为提示语，**输入的类型永远是字符串型（str）**

print() 函数则接受字符串、数字、字典、列表甚至一些自定义类的输出。

```python
a = input()
1
b = input()
2
 
print('a + b = {}'.format(a + b))
########## 输出 ##############
a + b = 12
print('type of a is {}, type of b is {}'.format(type(a), type(b)))
########## 输出 ##############
type of a is <class 'str'>, type of b is <class 'str'>
print('a + b = {}'.format(int(a) + int(b)))
########## 输出 ##############
a + b = 3
```

**把 str 强制转换为 int 请用 int()，转为浮点数请用 float()。而在生产环境中使用强制转换时，请记得加上 try except**

**Python 对 int 类型没有最大限制**（相比之下， C++ 的 int 最大为 2147483647，超过这个数字会产生溢出），但是对 float 类型依然有精度限制

相当比例的安全漏洞，都来自随意的 I/O 处理。



## 文件输入输出

命令行的输入输出，只是 Python 交互的最基本方式，适用一些简单小程序的交互。而生产级别的 Python 代码，大部分 I/O 则来自于文件、网络、其他进程的消息等等。

#### 简单的 NLP（自然语言处理）任务

 NLP 任务的基本步骤，也就是下面的四步：

1. 读取文件；
2. 去除所有标点符号和换行符，并把所有大写变成小写；
3. 合并相同的词，统计每个词出现的频率，并按照词频从大到小排序；
4. 将结果按行输出到文件 out.txt。

```python
import re
 
# 你不用太关心这个函数
def parse(text):
    # 使用正则表达式去除标点符号和换行符
    text = re.sub(r'[^\w ]', ' ', text)
 
    # 转为小写
    text = text.lower()
    
    # 生成所有单词的列表
    word_list = text.split(' ')
    
    # 去除空白单词
    word_list = filter(None, word_list)
    
    # 生成单词和词频的字典
    word_cnt = {}
    for word in word_list:
        if word not in word_cnt:
            word_cnt[word] = 0
        word_cnt[word] += 1
    
    # 按照词频排序
    sorted_word_cnt = sorted(word_cnt.items(), key=lambda kv: kv[1], reverse=True)
    
    return sorted_word_cnt
 
with open('in.txt', 'r') as fin:
    text = fin.read()
 
word_and_freq = parse(text)
 
with open('out.txt', 'w') as fout:
    for word, freq in word_and_freq:
        fout.write('{} {}\n'.format(word, freq))
```

用 open() 函数拿到文件的指针。其中，第一个参数指定文件位置（相对位置或者绝对位置）；第二个参数，如果是 `'r'`表示读取，如果是`'w'` 则表示写入，当然也可以用 `'rw'`，表示读写都要。a 则是一个不太常用（但也很有用）的参数，表示追加（append），这样打开的文件，如果需要写入，会从原始文件的最末尾开始写入。



在拿到指针后，我们可以通过 read() 函数，来读取文件的全部内容。代码 text = fin.read() ，即表示把文件所有内容读取到内存中，并赋值给变量 text。这么做自然也是有利有弊：

- 优点是方便，接下来我们可以很方便地调用 parse 函数进行分析；
- 缺点是如果文件过大，一次性读取可能造成内存崩溃。

我们可以给 read 指定参数 size ，用来表示读取的最大长度。还可以通过 readline() 函数，每次读取一行，这种做法常用于数据挖掘（Data Mining）中的数据清洗，在写一些小的程序时非常轻便。

如果每行之间没有关联，这种做法也可以降低内存的压力。而 write() 函数，可以把参数中的字符串输出到文件中，也很容易理解。

**open() 函数对应于 close() 函数，也就是说，如果你打开了文件，在完成读取任务后，就应该立刻关掉它。而如果你使用了 with 语句，就不需要显式调用 close()。在 with 的语境下任务执行完毕后，close() 函数会被自动调用**

所有 I/O 都应该进行错误处理。因为 I/O 操作可能会有各种各样的情况出现，而一个健壮（robust）的程序，需要能应对各种情况的发生，而不应该崩溃（故意设计的情况除外）。

**代码权限管理非常重要**





## JSON 序列化与实战

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，它的设计意图是把所有事情都用设计的字符串来表示，这样既方便在互联网上传递信息，也方便人进行阅读

```python
import json
 
params = {
    'symbol': '123456',
    'type': 'limit',
    'price': 123.4,
    'amount': 23
}
 
params_str = json.dumps(params)
 
print('after json serialization')
print('type of params_str = {}, params_str = {}'.format(type(params_str), params))
 
original_params = json.loads(params_str)
 
print('after json deserialization')
print('type of original_params = {}, original_params = {}'.format(type(original_params), original_params))
 
########## 输出 ##########
 
after json serialization
type of params_str = <class 'str'>, params_str = {'symbol': '123456', 'type': 'limit', 'price': 123.4, 'amount': 23}
after json deserialization
type of original_params = <class 'dict'>, original_params = {'symbol': '123456', 'type': 'limit', 'price': 123.4, 'amount': 23}

```

- json.dumps() 这个函数，接受 Python 的基本数据类型，然后将其序列化为 string；
- json.loads() 这个函数，接受一个合法字符串，然后将其反序列化为 Python 的基本数据类型。

**记得加上错误处理**



输出字符串到文件，或者从文件中读取 JSON 字符串

```python
import json
 
params = {
    'symbol': '123456',
    'type': 'limit',
    'price': 123.4,
    'amount': 23
}
 
with open('params.json', 'w') as fout:
    params_str = json.dump(params, fout)
 
with open('params.json', 'r') as fin:
    original_params = json.load(fin)
 
print('after json deserialization')
print('type of original_params = {}, original_params = {}'.format(type(original_params), original_params))
 
########## 输出 ##########
 
after json deserialization
type of original_params = <class 'dict'>, original_params = {'symbol': '123456', 'type': 'limit', 'price': 123.4, 'amount': 23}
```

可以通过 JSON 将用户的个人配置输出到文件，方便下次程序启动时自动读取。这也是现在普遍运用的成熟做法。

## 总结

- I/O 操作需谨慎，一定要进行充分的错误处理，并细心编码，防止出现编码漏洞；
- 编码时，对内存占用和磁盘占用要有充分的估计，这样在出错时可以更容易找到原因；
- JSON 序列化是很方便的工具，要结合实战多多练习；
- 代码尽量简洁、清晰，哪怕是初学阶段，也要有一颗当元帅的心。

filter(None, Iterable) 是一种容易出错的用法，这里不止过滤空字符串，还能过滤 0，None，空列表等值。这里的 None，严格意义上等于 lambda x: x, 是一个 callable。

