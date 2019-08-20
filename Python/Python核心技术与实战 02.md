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

这样的设计结构显然非常浪费存储空间。为了提高存储空间的利用率，现在的哈希表除了字典本身的结构，会把索引和哈希值、键、值单独分开，也就是下面这样新的结构：

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

可以很清晰地看到，空间利用率得到很大的提高。