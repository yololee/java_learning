# Java集合容器面试题

### 什么是集合，集合和数组的区别

集合：用于存储数据的容器。

集合和数组的区别

- 数组是固定长度的；集合是可变长度的。
- 数组可以存储基本数据类型，也可以存储引用数据类型；集合只能存储引用数据类型。
- 数组是Java语言中内置的数据类型，是线性排列的，执行效率和类型检查都比集合快，集合提供了众多的属性和方法，方便操作。

### List，Set，Map三者的区别

```java
collection
    -Set:无序,不可以重复，只允许存储一个null元素，保证元素唯一性，不可用迭代器
        -HashSet
        -TreeSet
        -LinkedHashSet
    -List:有序，可以重复，可以插入多个null元素，有索引，可用迭代器
        -ArrayList
        -LinkedList
        -Vector
    -Queue:是 Java 提供的标准队列结构的实现，除了集合的基本功能，它还支持类似先入先出（FIFO， First-in-First-Out）或者后入先出（LIFO，Last-In-First-Out）等特定行为
        - ArrayDeque
        - ArrayBlockingQueue
        - LinkedBlockingDeque
        
map:键值对集合，存储键和值之间的映射.Key无序，唯一；value不要求有序，允许重复，Map没有继承Collection接口
    - HashMap
    - LinkedHashMap
    - ConcurrentHashMap
    - TreeMap
    - HashTable
```

### 集合框架底层数据结构

```java
collection
    -list
    	-ArrayList：Object数组,查询快增删慢
        -LinkedList：双向链表,增删快查询慢
    	-Vector：Object数组,线程安全
    -set
    	-HashSet：基于 HashMap 实现，底层采用 HashMap 的key来保存元素
    	-LinkedHashSet：LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
    	-TreeSet：红黑树(自平衡的排序二叉树)
    
map
    HashMap：JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8），但是数组长度小于64时会首先进行扩容，否则会将链表转化为红黑树，以减少搜索时间。
    
```

### ArrayList、LinkedList、Vector 有何区别？

```java
1.数据结构：ArrayList和Vector是数组，LinkedList是双向链表
    
2.随机访问效率：ArrayList 和 Vector 比 LinkedList查询快，因为LinkedList 是链表数据结构，需要移动指针从前往后依次查找
    
3.增加和删除效率：在非尾部的增加和删除操作，LinkedList 要比 ArrayList 和 Vector 效率要高，因为 ArrayList 和 Vector 增删操作要影响数组内的其他数据的下标，需要进行数据搬移。因为 ArrayList 非线程安全，在增删元素时性能比 Vector 好
    
4.内存空间占用：LinkedList 比 ArrayList 和 Vector 更占内存，因为 LinkedList 的节点除了存储数据，还存储了两个引用，分别是前驱节点和后继节点
    
5.线程安全:ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；Vector 使用了 synchronized 来实现线程同步，是线程安全的    
    
6.扩容:ArrayList 和 Vector 都会根据实际的需要动态的调整容量，只不过在 Vector 扩容每次会增加 1 倍容量，而 ArrayList 只会增加 50%容量

7.使用场景:需要频繁地随机访问集合中的元素时，推荐使用 ArrayList，希望线程安全的对元素进行增删改操作时，推荐使用Vector，而需要频繁插入和删除操作时，推荐使用 LinkedList。    
```

### 源码分析add()方法，remove()方法

```java
1.添加一个特定的元素到list的末尾
    先确保elementData数组的长度足够,不够就扩容

2.在指定位置添加一个元素
    先确保elementData数组的长度足够,不够就扩容
    将数据整体向后移动一位，空出位置之后再插入，效率不太好

3.添加一个集合
    先把该集合转为对象数组，然后扩容，在挨个向后迁移

4.在指定位置，添加一个集合
    扩容，原来的数组挨个向后迁移，把新的集合数组 添加到指定位置

扩容原则：首先创建一个空数组elementData，第一次插入数据时直接扩充至10，然后如果elementData的长度不足，就扩充至1.5倍，如果扩充完还不够，就使用需要的长度作为elementData的长度
    
根据索引删除指定位置的元素，此时会把指定下标到数组末尾的元素挨个向前移动一个单位，并且会把数组最后一个元素设置为null，这样是为了方便之后将整个数组不被使用时，会被GC，可以作为小的技巧使用    
```

### hashMap的底层原理

```java
1.存储方式：  
    Java中的HashMap是以键值对(key-value)的形式存储元素的。

2.调用原理： 
    HashSet数据结构：哈希表结构（数组+链表+红⿊树），HashMap首先会在底层创建一个长度为16，加载因子为0.75的Entry数组，它使用hashCode()equals()方法来向集合中集合添加和检索元素。当调用put()方法的时候，HashMap会计算key的hash值，然后把键值对存储在集合中合适的索引上。如果该索引上没有元素(null)，把元素直接存储在这个位置 如果有元素，继续通过equals⽅法判断元素的属性只是否⼀样 如果equals⽐较为true，就说明元素重复 如果equals⽐较为false，以链表的形式存储在数组的同⼀个索引位置 如果链表的⻓度超过8，就把链表转化为红⿊树（提⾼查询的效率）

3.其他热性：
    HashMap的一些重要的特性是它的容量(capacity)16，负载因子(load factor)0.75和扩容极限(threshold resizing)16*0.75=12，扩容到原来的2倍。
```

### HashMap的put方法的具体流程？

①.判断键值对数组table[i]是否为空或为null，是的话执行resize()进行扩容；

②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

### HashMap常用在什么地方

```java
1、hashMap作为缓存
2、controller层向前台返回数据可以使用map封装数据，还有mybatis中的map可以作为参数或者封装结果集    
```

### 如果使用Object作为HashMap的Key，应该怎么办呢？

```java
重写hashCode()和equals()方法

重写hashCode()是因为需要计算数据的存储位置，需要注意不要试图从散列码计算中排除掉一个对象的关键部分来提高性能，这样虽然能更快，但可能会导致更多的Hash碰撞；

重写equals()方法，需要遵守自反性、对称性、传递性、一致性以及对于任何非null的引用值x，x.equals(null)必须返回false的这几个特性，目的是为了保证key在哈希表中的唯一性；
```

### HashMap和Hashtable的区别

```java
1、hashMap去掉了HashTable中的contains方法，但是加上了containsValue()和containsKey()方法
2、hashtable线程安全，hashmap线程不安全
3、hashMap允允许空键值，而hashtable不允许    
```

### HashMap 和 ConcurrentHashMap 的区别

```java
ConcurrentHashMap对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用lock锁进行保护，相对于HashTable的synchronized锁的粒度更精细了一些，并发性能更好，而HashMap没有锁机制，不是线程安全的。（JDK1.8之后ConcurrentHashMap启用了一种全新的方式实现，利用CAS算法。）

HashMap的键值对允许有null，但是ConCurrentHashMap都不允许。
```

### ConcurrentHashMap 和 Hashtable 的区别？

```java
1.底层数据结构：JDK1.7的 ConcurrentHashMap 底层采用 分段的数组+链表 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 数组+链表 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
    
2.实现线程安全的方式:
	jdk1.7:ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。（默认分配16个Segment，比Hashtable效率提高16倍。）
    jdk1.8:直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。

3.Hashtable(同一把锁) :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈，效率越低。        
```



### TreeSet怎么对集合中的元素进行排序

```java
TreeSet底层结构是二叉树，保证元素唯一性的依据：compareTo方法return 0
    1.让元素自身具备比较性，需要元素对象实现Comparable接口，覆盖comPareTo方法
    2.让集合自身具备比较性，需要定义一个实现Comparator接口的比较器，并覆盖compara方法
```

### map集合的俩种取出方式

```java
1.普遍使用，二次取值。通过map.keySet()遍历可以和value
2.推荐，尤其容量大时，可以通过Map.entrySet遍历key和value
注意：map集合不可以使用迭代器，或者增强for循环遍历    
```

### Conllection和Collections有什么区别

```java
Conllection是集合的上级接口，继承与他的接口主要有Set和List
Collections是工具类，有很多对集合操作的方法    
```

### 如何把集合变成线程安全

```java
Collections.synchronizedCollection(c)
Collections.synchronizedList(list)
Collections.synchronizedSet(set)
Collections.synchronizedMap(map)
就是在集合的核心方法添加上了synchronized关键字    
```

### ConcurrentHashMap的工作原理

```java
 	ConCurrentHashMap把整个Map分为N个Segment，N的默认值为16，加载因子也是0.75，段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容，插入前检测需不需要扩容，有效避免无效扩容；			
     ConCurrentHashMap与HashTable最大的区别就在于ConCurrentHashMap采用synchronized悲观锁锁住单个Segment对象，使用了锁分离技术；同时ConCurrentHashMap也采用了CAS算法（乐观锁），使用 容量大小-1 的哈希地址 计算出待插入键值的下标，如果该下标上的标记为null，则直接调用CAS算法将元素插入到数组中，如果不为空，则根据volatile获取当前位置最新的节点地址值，挂在最新节点下面，变成链表，当链表长度大于等于8时，链表自动转成红黑树，效率较高。
	在不考虑线程安全时，优先使用HashMap；在考虑线程安全时，优先使用ConCurrentHashMap。
```

### 

