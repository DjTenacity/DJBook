## HashMap底层的数据结构和算法

### 什么是哈希？

最简单形式的 `hash`，是一种在对任何变量/对象的属性应用任何公式/算法后， 为其分配唯一代码的方法。

一个真正的`hash`方法必须遵循下面的原则

> 哈希函数每次在相同或相等的对象上应用哈希函数时, 应每次返回相同的哈希码。换句话说, 两个相等的对象必须一致地生成相同的哈希码。

Java 中所有的对象都有 `Hash` 方法。

Java中的所有对象都继承 Object 类中定义的 hashCode() 函数的默认实现。 此函数通常通过将对象的内部地址转换为整数来生成哈希码，从而为所有不同的对象生成不同的哈希码。

### HashMap 中的 Node 类

Map的定义是： 将键映射到值的对象。

因此，`HashMap` 中必须有一些机制来存储这个键值对。 答案是肯的。 `HashMap` 有一个内部类 Node，如下所示：

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;// 记录hash值， 以便重hash时不需要再重新计算
        final K key; 
        V value;
        Node<K,V> next;
        ...// 其余的代码
    }
```

当然，Node 类具有存储为属性的键和值的映射。 key 已被标记为 final，另外还有两个字段：next 和 hash。

在下面中， 我们将会理解这些属性的必须性。

### 键值对在 HashMap中是如何存储的

键值对在 `HashMap` 中是以 `Node` 内部类的数组存放的，如下所示：

```java
transient Node<K,V>[] table;
```

哈希码计算出来之后， 会转换成该数组的下标， 在该下标中存储对应哈希码的键值对， 在此先不详细讲解hash碰撞的情况。

该数组的长度始终是2的次幂， 通过以下的函数实现该过程

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;// 如果不做该操作， 则如传入的 cap 是 2 的整数幂， 则返回值是预想的 2 倍
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

其原理是将传入参数 `(cap)` 的低二进制全部变为1，最后加1即可获得对应的大于 `cap` 的 2 的次幂作为数组长度。

> 为什么要使用2的次幂作为数组的容量呢？

在此有涉及到 `HashMap` 的 `hash` 函数及数组下标的计算， 键(key)所计算出来的哈希码有可能是大于数组的容量的，那怎么办？ 可以通过简单的求余运算来获得，但此方法效率太低。HashMap中通过以下的方法保证 `hash` 的值计算后都小于数组的容量。

```
(n - 1) & hash
```

这也正好解释了为什么需要2的次幂作为数组的容量。由于n是2的次幂，因此，n-1类似于一个低位掩码。通过与操作，高位的hash值全部归零，保证低位才有效 从而保证获得的值都小于n。

同时，在下一次 `resize()` 操作时， 重新计算每个 `Node` 的数组下标将会因此变得很简单，具体的后文讲解。以默认的初始值16为例

```
   01010011 00100101 01010100 00100101
&   00000000 00000000 00000000 00001111
----------------------------------
    00000000 00000000 00000000 00000101    //高位全部归零，只保留末四位
    // 保证了计算出的值小于数组的长度 n
```

但是，使用了该功能之后，由于只取了低位，因此 hash 碰撞会也会相应的变得很严重。这时候就需要使用「扰动函数」

```
   static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

该函数通过将哈希码的高16位的右移后与原哈希码进行异或而得到，以上面的例子为例

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbucVA6tfccZkiaro32eYsiau8PiaGE6jyEjydhwW4ZMZlDjRl6Mj20ibS0YwhOGtmcIrZyqasdMXY6jOQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此方法保证了高16位不变， 低16位根据异或后的结果改变。计算后的数组下标将会从原先的5变为0。

使用了 「扰动函数」 之后， hash 碰撞的概率将会下降。 有人专门做过类似的测试， 虽然使用该 「扰动函数」 并没有获得最大概率的避免 `hash` 碰撞，但考虑其计算性能和碰撞的概率， JDK 中使用了该方法，且只hash一次。

### 哈希碰撞及其处理

在理想的情况下， 哈希函数将每一个 `key` 都映射到一个唯一的 `bucket`， 然而， 这是不可能的。哪怕是设计在良好的哈希函数，也会产生哈希冲突。

前人研究了很多哈希冲突的解决方法，在维基百科中，总结出了四大类

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbucVA6tfccZkiaro32eYsiau8Pz9B8bvPuicoQLjMiceyQ1rN1fBbtMyzd97LWFm0GJPzqMnU6NSfNROrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在 Java 的 HashMap 中， 采用了第一种 `Separate chaining` 方法(大多数翻译为拉链法)+链表和红黑树来解决冲突。

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbucVA6tfccZkiaro32eYsiau8Pnjc89QSQU1z5ER0vVrVXXz1ojCtCRSzGFg0MaDdrQD0UYS0hApxFkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在 `HashMap` 中， 哈希碰撞之后会通过 `Node` 类内部的成员变量 `Node<K,V> next`; 来形成一个链表(节点小于8)或红黑树（节点大于8， 在小于6时会从新转换为链表）， 从而达到解决冲突的目的。

```
static final int TREEIFY_THRESHOLD = 8;

static final int UNTREEIFY_THRESHOLD = 6;
```

###  HashMap 的初始化

```
 public HashMap();
 public HashMap(int initialCapacity);
 public HashMap(Map<? extends K, ? extends V> m);
 public HashMap(int initialCapacity, float loadFactor); 
```

HashMap 中有四个构造函数， 大多是初始化容量和负载因子的操作。以 `public HashMap(int initialCapacity, float loadFactor)` 为例

```
public HashMap(int initialCapacity, float loadFactor) {
    // 初始化的容量不能小于0
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 初始化容量不大于最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 负载因子不能小于 0
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

通过该函数进行了容量和负载因子的初始化，如果是调用的其他的构造函数， 则相应的负载因子和容量会使用默认值（默认负载因子=0.75， 默认容量=16）。在此时， 还没有进行存储容器 `table` 的初始化， 该初始化要延迟到第一次使用时进行。

###  HashMap 中哈希表的初始化或动态扩容

所谓的哈希表， 指的就是下面这个类型为内部类Node的 table 变量。

```
transient Node<K,V>[] table;
```

作为数组， 其在初始化时就需要指定长度。在实际使用过程中， 我们存储的数量可能会大于该长度，因此 HashMap 中定义了一个阈值参数(threshold)， 在存储的容量达到指定的阈值时， 需要进行扩容。

> 我个人认为初始化也是动态扩容的一种， 只不过其扩容是容量从 0 扩展到构造函数中的数值（默认16）。 而且不需要进行元素的重hash.

#### 7.1 扩容发生的条件

初始化的话只要数值为空或者数组长度为 0 就会进行。 而扩容是在元素的数量大于阈值（threshold）时就会触发。

```
threshold = loadFactor * capacity
```

比如 HashMap 中默认的 loadFactor=0.75, capacity=16, 则

```
threshold = loadFactor * capacity = 0.75 * 16 = 12
```

那么在元素数量大于 12 时， 就会进行扩容。 扩容后的 capacity 和 threshold 也会随之而改变。

负载因子影响触发的阈值，因此，它的值较小的时候，`HashMap` 中的 `hash` 碰撞就很少， 此时存取的性能都很高，对应的缺点是需要较多的内存；而它的值较大时，`HashMap` 中的 `hash` 碰撞就很多，此时存取的性能相对较低，对应优点是需要较少的内存；不建议更改该默认值，如果要更改，建议进行相应的测试之后确定。

#### 7.2 再谈容量为2的整数次幂和数组索引计算

前面说过了数组的容量为 2 的整次幂， 同时， 数组的下标通过下面的代码进行计算

```
index = (table.length - 1) & hash
```

该方法除了可以很快的计算出数组的索引之外， 在扩容之后， 进行重 hash 时也会很巧妙的就可以算出新的 hash 值。 由于数组扩容之后， 容量是现在的 2 倍， 扩容之后 n-1 的有效位会比原来多一位， 而多的这一位与原容量二进制在同一个位置。 示例

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbucVA6tfccZkiaro32eYsiau8PkYnsL9Ek5QxvZGUibF2TGKMsX63aahIyomVofVq26GT20eTw23leCzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样就可以很快的计算出新的索引啦

#### 7.3 步骤

- 先判断是初始化还是扩容， 两者在计算newCap和newThr时会不一样
- 计算扩容后的容量，临界值。
- 将hashMap的临界值修改为扩容后的临界值
- 根据扩容后的容量新建数组，然后将hashMap的table的引用指向新数组。
- 将旧数组的元素复制到table中。在该过程中， 涉及到几种情况， 需要分开进行处理（只存有一个元素， 一般链表， 红黑树）

##### 具体的看代码吧

```java
final Node<K, V>[] resize() {
        //新建oldTab数组保存扩容前的数组table
        Node<K, V>[] oldTab = table;
        //获取原来数组的长度
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //原来数组扩容的临界值
        int oldThr = threshold;
        int newCap, newThr = 0;
        //如果扩容前的容量 > 0
        if (oldCap > 0) {
            //如果原来的数组长度大于最大值(2^30)
            if (oldCap >= MAXIMUM_CAPACITY) {
                //扩容临界值提高到正无穷
                threshold = Integer.MAX_VALUE;
                //无法进行扩容，返回原来的数组
                return oldTab;
                //如果现在容量的两倍小于MAXIMUM_CAPACITY且现在的容量大于DEFAULT_INITIAL_CAPACITY
            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
                //临界值变为原来的2倍
                newThr = oldThr << 1;
        } else if (oldThr > 0) //如果旧容量 <= 0，而且旧临界值 > 0
            //数组的新容量设置为老数组扩容的临界值
            newCap = oldThr;
        else { //如果旧容量 <= 0，且旧临界值 <= 0，新容量扩充为默认初始化容量，新临界值为DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY
            newCap = DEFAULT_INITIAL_CAPACITY;//新数组初始容量设置为默认值
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//计算默认容量下的阈值
        }
        // 计算新的resize上限
        if (newThr == 0) {//在当上面的条件判断中，只有是初始化时(oldCap=0, oldThr > 0)时，newThr == 0
            //ft为临时临界值，下面会确定这个临界值是否合法，如果合法，那就是真正的临界值
            float ft = (float) newCap * loadFactor;
            //当新容量< MAXIMUM_CAPACITY且ft < (float)MAXIMUM_CAPACITY，新的临界值为ft，否则为Integer.MAX_VALUE
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                    (int) ft : Integer.MAX_VALUE);
        }
        //将扩容后hashMap的临界值设置为newThr
        threshold = newThr;
        //创建新的table，初始化容量为newCap
        @SuppressWarnings({"rawtypes", "unchecked"})
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        //修改hashMap的table为新建的newTab
        table = newTab;
        //如果旧table不为空，将旧table中的元素复制到新的table中
        if (oldTab != null) {
            //遍历旧哈希表的每个桶，将旧哈希表中的桶复制到新的哈希表中
            for (int j = 0; j < oldCap; ++j) {
                Node<K, V> e;
                //如果旧桶不为null，使用e记录旧桶
                if ((e = oldTab[j]) != null) {
                    //将旧桶置为null
                    oldTab[j] = null;
                    //如果旧桶中只有一个node
                    if (e.next == null)
                        //将e也就是oldTab[j]放入newTab中e.hash & (newCap - 1)的位置
                        newTab[e.hash & (newCap - 1)] = e;
                        //如果旧桶中的结构为红黑树
                    else if (e instanceof TreeNode)
                        //将树中的node分离
                        ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                    else {  //如果旧桶中的结构为链表,链表重排，jdk1.8做的一系列优化
                        Node<K, V> loHead = null, loTail = null;
                        Node<K, V> hiHead = null, hiTail = null;
                        Node<K, V> next;
                        //遍历整个链表中的节点
                        do {
                            next = e.next;
                            // 原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            } else {// 原索引+oldCap
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap放到bucket里
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
}
```

#### 7.4 注意事项

虽然 `HashMap` 设计的非常优秀， 但是应该尽可能少的避免 `resize()`, 该过程会很耗费时间。

同时， 由于 `hashmap` 不能自动的缩小容量 因此，如果你的 `hashmap` 容量很大，但执行了很多 `remove`操作时，容量并不会减少。如果你觉得需要减少容量，请重新创建一个 hashmap。

###  HashMap.put() 函数内部是如何工作的？

在使用多次 HashMap 之后， 大体也能说出其添加元素的原理：计算每一个key的哈希值， 通过一定的计算之后算出其在哈希表中的位置，将键值对放入该位置，如果有哈希碰撞则进行哈希碰撞处理。

而其工作时的原理如下

![img](https://mmbiz.qpic.cn/mmbiz_jpg/eQPyBffYbucVA6tfccZkiaro32eYsiau8PtNJrygibGKduheyC3ptGwlaPNe5bq1hDsT7IfiaTSoXp7KgMbTJv1TYQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 源码如下：

```java
    /* @param hash         指定参数key的哈希值
     * @param key          指定参数key
     * @param value        指定参数value
     * @param onlyIfAbsent 如果为true，即使指定参数key在map中已经存在，也不会替换value
     * @param evict        如果为false，数组table在创建模式中
     * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K, V>[] tab;
        Node<K, V> p;
        int n, i;
        //如果哈希表为空，调用resize()创建一个哈希表，并用变量n记录哈希表长度
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        /**
         * 如果指定参数hash在表中没有对应的桶，即为没有碰撞
         * Hash函数，(n - 1) & hash 计算key将被放置的槽位
         * (n - 1) & hash 本质上是hash % n，位运算更快
         */
        if ((p = tab[i = (n - 1) & hash]) == null)
            //直接将键值对插入到map中即可
            tab[i] = newNode(hash, key, value, null);
        else {// 桶中已经存在元素
            Node<K, V> e;
            K k;
            // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
                // 当前桶中无该键值对，且桶是红黑树结构，按照红黑树结构插入
            else if (p instanceof TreeNode)
                e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
                // 当前桶中无该键值对，且桶是链表结构，按照链表结构插入到尾部
            else {
                for (int binCount = 0; ; ++binCount) {
                    // 遍历到链表尾部
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 检查链表长度是否达到阈值，达到将该槽位节点组织形式转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 链表节点的<key, value>与put操作<key, value>相同时，不做重复操作，跳出循环
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 找到或新建一个key和hashCode与插入元素相等的键值对，进行put操作
            if (e != null) { // existing mapping for key
                // 记录e的value
                V oldValue = e.value;
                /**
                 * onlyIfAbsent为false或旧值为null时，允许替换旧值
                 * 否则无需替换
                 */
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // 访问后回调
                afterNodeAccess(e);
                // 返回旧值
                return oldValue;
            }
        }
        // 更新结构化修改信息
        ++modCount;
        // 键值对数目超过阈值时，进行rehash
        if (++size > threshold)
            resize();
        // 插入后回调
        afterNodeInsertion(evict);
        return null;
    }
```

在此过程中， 会涉及到哈希碰撞的解决。

###  HashMap.get() 方法内部是如何工作的？

```java
    /**
     * 返回指定的key映射的value，如果value为null，则返回null
     * get可以分为三个步骤：
     * 1.通过hash(Object key)方法计算key的哈希值hash。
     * 2.通过getNode( int hash, Object key)方法获取node。
     * 3.如果node为null，返回null，否则返回node.value。
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        Node<K, V> e;
        //根据key及其hash值查询node节点，如果存在，则返回该节点的value值
        return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

其最终是调用了 `getNode` 函数。 其逻辑如下

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbucVA6tfccZkiaro32eYsiau8Ps6D1fZUmhiayg1iaKTNE8JtBzUAx1D2M7uaxs6UQk9OtFp6W5I8jdkyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 源码如下：

```java
    /**
     * @param hash 指定参数key的哈希值
     * @param key  指定参数key
     * @return 返回node，如果没有则返回null
     */
    final Node<K, V> getNode(int hash, Object key) {
        Node<K, V>[] tab;
        Node<K, V> first, e;
        int n;
        K k;
        //如果哈希表不为空，而且key对应的桶上不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
            //如果桶中的第一个节点就和指定参数hash和key匹配上了
            if (first.hash == hash && // always check first node
                    ((k = first.key) == key || (key != null && key.equals(k))))
                //返回桶中的第一个节点
                return first;
            //如果桶中的第一个节点没有匹配上，而且有后续节点
            if ((e = first.next) != null) {
                //如果当前的桶采用红黑树，则调用红黑树的get方法去获取节点
                if (first instanceof TreeNode)
                    return ((TreeNode<K, V>) first).getTreeNode(hash, key);
                //如果当前的桶不采用红黑树，即桶中节点结构为链式结构
                do {
                    //遍历链表，直到key匹配
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //如果哈希表为空，或者没有找到节点，返回null
        return null;
}
```