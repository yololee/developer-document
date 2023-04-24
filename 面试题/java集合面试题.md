# java集合面试题

## 集合容器概述

### 集合和数组的区别

```
集合：用于存储数据的容器。

集合和数组的区别
1、数组是固定长度的；集合是可变长度的。
2、数组可以存储基本数据类型，也可以存储引用数据类型；集合只能存储引用数据类型。
3、数组是Java语言中内置的数据类型，是线性排列的，执行效率和类型检查都比集合快，集合提供了众多的属性和方法，方便操作。
```

### List，Set，Map三者的区别

```
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

```
collection
    -list
    	-ArrayList：Object数组,查询快增删慢
      -LinkedList：双向链表,增删快查询慢
    	-Vector：Object数组,线程安全
    -set
    	-HashSet（无序，唯一）：基于 HashMap 实现，底层采用 HashMap 的key来保存元素
    	-LinkedHashSet：LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
    	-TreeSet（有序，唯一）：红黑树(自平衡的排序二叉树)
    
map
    -HashMap：JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8），但是数组长度小于64时会首先进行扩容，否则会将链表转化为红黑树，以减少搜索时间。
    -LinkedHashMap：LinkedHashMap 继承自 HashMap，它的底层仍然是数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得LinkedHashMap可以保持键值对的插入顺序
    -HashTable：数组+链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的
    -TreeMap：红黑树（自平衡的排序二叉树
```

## Collection接口

### List接口

#### ArrayList、LinkedList、Vector 有何区别

```
1、数据结构实现：ArrayList 和 Vector 是动态数组的数据结构实现，而 LinkedList 是双向循环链表的数据结构实现。

2、随机访问效率：ArrayList 和 Vector 比 LinkedList 在根据索引随机访问的时候效率要高，因为 LinkedList 是链表数据结构，需要移动指针从前往后依次查找。

3、增加和删除效率：在非尾部的增加和删除操作，LinkedList 要比 ArrayList 和 Vector 效率要高，因为 ArrayList 和 Vector 增删操作要影响数组内的其他数据的下标，需要进行数据搬移。因为 ArrayList 非线程安全，在增删元素时性能比 Vector 好。

4、内存空间占用：一般情况下LinkedList 比 ArrayList 和 Vector 更占内存，因为 LinkedList 的节点除了存储数据，还存储了两个引用，分别是前驱节点和后继节点

5、线程安全：ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；Vector 使用了 synchronized 来实现线程同步，是线程安全的。

6、扩容：ArrayList 和 Vector 都会根据实际的需要动态的调整容量，只不过在 Vector 扩容每次会增加 1 倍容量，而 ArrayList 只会增加 50%容量。

7、使用场景：在需要频繁地随机访问集合中的元素时，推荐使用 ArrayList，希望线程安全的对元素进行增删改操作时，推荐使用Vector，而需要频繁插入和删除操作时，推荐使用 LinkedList。
```

#### Java集合的快速失败机制 “fail-fast”

```
快速失败机制是java集合的一种错误检测机制，当多个线程对集合进行结构上的改变时，有可能会产生 fail-fast 机制。

例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。

原因分析：迭代器在遍历时，ArrayList的父类AbstarctList中有一个modCount变量，每次对集合进行修改时都会modCount++，而foreach的实现原理其实就是Iterator，ArrayList的Iterator中有一个expectedModCount变量，该变量会初始化和modCount相等，每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount的值和expectedmodCount的值是否相等，如果集合进行增删操作，modCount变量就会改变，就会造成expectedModCount!=modCount，此时就会抛出ConcurrentModificationException异常

解决办法：
1、在遍历过程中，所有涉及到改变modCount值的地方全部加上synchronized。
2、使用CopyOnWriteArrayList来替换ArrayList
```

#### 迭代器Iterator是什么？如何一边遍历一边删除Collection中的元素

```
Iterator 接口提供遍历任何 Collection 的接口。我们可以从一个 Collection 中使用迭代器方法来获取迭代器实例。迭代器取代了 Java 集合框架中的 Enumeration，同时迭代器允许调用者在迭代过程中增删元素

边遍历边修改 Collection 的唯一正确方式是使用 Iterator.remove() 方法，如下：
Iterator<Integer> it = list.iterator();
while(it.hasNext()){
  // do something
  it.remove();
}
```

#### ArrayList 的优缺点

```
优点:
1、ArrayList 底层以数组实现，ArrayList 实现了 RandomAccess 接口，根据索引进行随机访问的时候速度非常快
2、ArrayList 在尾部添加一个元素的时候非常方便
缺点：
1、在非尾部的增加和删除操作，影响数组内的其他数据的下标，需要进行数据搬移，比较消耗性能

ArrayList 比较适合顺序添加、随机访问的场景
```

#### 源码分析add()方法，remove()方法

```
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

### Set接口

#### HashSet如何检查重复？HashSet是如何保证数据不可重复的

```
向HashSet 中add ()元素时，判断元素是否存在的依据，不仅要比较hash值，还要结合equles方法比较。HashSet 中的add()方法会使用HashMap 的put()方法

HashMap 的 key 是唯一的，由源码可以看出 HashSet 添加进去的值就是作为HashMap 的key，并且在HashMap中如果K/V相同时，会用新的V覆盖掉旧的V，然后返回旧的V，所以不会重复（ HashMap 比较key是否相等是先比较hashcode 再比较equals ）
```

#### TreeSet怎么对集合中的元素进行排序

```
TreeSet底层结构是二叉树，保证元素唯一性的依据：compareTo方法return 0
    1.让元素自身具备比较性，需要元素对象实现Comparable接口，覆盖comPareTo方法
    2.让集合自身具备比较性，需要定义一个实现Comparator接口的比较器，并覆盖compara方法
```

#### HashSet与HashMap的区别

|                | HashMap                                                | **HashSet**                                                  |
| -------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| 父接口         | 实现了Map接口                                          | 实现Set接口                                                  |
| 存储数据       | 存储键值对                                             | 仅存储对象                                                   |
| 添加元素       | 调用put()向map中添加元素                               | 调用add()方法向Set中添加元素                                 |
| 计算哈希值     | HashMap使用键（Key）计算hashcode                       | HashSet使用对象来计算hashcode值，对于两个对象来说hashcode可能相同，需要用equals()方法用来判断对象的相等性，如果两个对象不同的话，那么返回false |
| 获取元素的速度 | HashMap相对于HashSet较快，因为它是使用唯一的键获取对象 | HashSet较HashMap来说比较慢                                   |

## Map接口

###  HashMap 的实现原理

```
1.存储方式：  
    Java中的HashMap是以键值对(key-value)的形式存储元素的。

2.调用原理： 
    HashSet数据结构：哈希表结构（数组+链表+红⿊树），HashMap首先会在底层创建一个长度为16，加载因子为0.75的Entry数组，它使用hashCode()equals()方法来向集合中集合添加和检索元素。当调用put()方法的时候，HashMap会计算key的hash值，然后把键值对存储在集合中合适的索引上。如果该索引上没有元素(null)，把元素直接存储在这个位置 如果有元素，继续通过equals⽅法判断元素的属性只是否⼀样 如果equals⽐较为true，就说明元素重复 如果equals⽐较为false，以链表的形式存储在数组的同⼀个索引位置 如果链表的⻓度超过8，就把链表转化为红⿊树（提⾼查询的效率）

3.其他热性：
    HashMap的一些重要的特性是它的容量(capacity)16，负载因子(load factor)0.75和扩容极限(threshold resizing)16*0.75=12，扩容到原来的2倍。
```

### HashMap的put方法的具体流程

```
①判断键值对数组table[i]是否为空或为null，是的话执行resize()进行扩容；

②根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

③判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

⑥插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。
```

### HashMap是怎么解决哈希冲突

> 什么是哈希

```
Hash，一般翻译为“散列”，也有直接音译为“哈希”的，这就是把任意长度的输入通过散列算法，变换成固定长度的输出，该输出就是散列值（哈希值）
```

> 什么是哈希冲突

```
当两个不同的输入值，根据同一散列函数计算出相同的散列值的现象，我们就把它叫做碰撞（哈希碰撞）
```

> 总结
>
> 简单总结一下HashMap是使用了哪些方法来有效解决哈希冲突的

```
1. 使用拉链法（使用散列表）来链接拥有相同hash值的数据

2. 使用2次扰动函数（hash函数）来降低哈希冲突的概率，使得数据分布更平均

3. 引入红黑树进一步降低遍历的时间复杂度，使得遍历更快
```

### 为什么HashMap中String、Integer这样的包装类适合作为key

```
String、Integer等包装类的特性能够保证Hash值的不可更改性和计算准确性，能够有效的减少Hash碰撞的几率

1、都是final类型，即不可变性，保证key的不可更改性，不会存在同一对象获取hash值不同的情况
2、内部已重写了equals()、hashCode()等方法，遵守了HashMap内部的规范，不容易出现Hash值计算错误的情况；
```

### 如果使用Object作为HashMap的Key，应该怎么办呢

```
重写hashCode()和equals()方法

重写hashCode()是因为需要计算数据的存储位置，需要注意不要试图从散列码计算中排除掉一个对象的关键部分来提高性能，这样虽然能更快，但可能会导致更多的Hash碰撞；

重写equals()方法，需要遵守自反性、对称性、传递性、一致性以及对于任何非null的引用值x，x.equals(null)必须返回false的这几个特性，目的是为了保证key在哈希表中的唯一性；
```

### ConcurrentHashMap 底层原理

> JDK1.7
>
> 数据结构：Segments数组+HashEntry数组+链表，采用**分段锁**保证安全性

![image-20230424104041883](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230424104041883.png)

```
首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问

一个 ConcurrentHashMap 里包含一个 Segment 数组。Segment 的结构和HashMap类似，是一种数组和链表结构，segment继承了ReentrantLock，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素。当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 的锁
```

> JDK1.8
>
> 数据结构：Node数组+ 链表+红黑树

![image-20230424104440981](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230424104440981.png)

```
在JDK1.8中，放弃了Segment臃肿的设计，取而代之的是采用Node + CAS + Synchronized来保证并发安全，synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍

为什么在有Synchronized 的情况下还要使用CAS

因为CAS是乐观锁,在一些场景中(并发不激烈的情况下)它比Synchronized和ReentrentLock的效率要高,当CAS保障不了线程安全的情况下(扩容或者hash冲突的情况下)转成Synchronized 来保证线程安全,大大提高了低并发下的性能

```

### Hashtable、HashMap、TreeMap的区别

```
Hashtable（数组+链表）:线程安全，无序，不支持 null 键和值
HashMap（数组+链表）:线程不安全，HashMap允许一个空键（其他的空键会覆盖第一个空键）和任意数量的空值，HashMap去掉了HashTable的contains方法，但是加上了containsValue()和containsKey()方法
TreeMap（红黑树）：有序，线程不安全，需要根据key对节点进行排序
```

### HashMap 和 ConcurrentHashMap 的区别

```
1、ConcurrentHashMap对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用lock锁进行保护，相对于HashTable的synchronized锁的粒度更精细了一些，并发性能更好，而HashMap没有锁机制，不是线程安全的。（JDK1.8之后ConcurrentHashMap启用了一种全新的方式实现，利用CAS算法。）

2、HashMap的键值对允许有null，但是ConCurrentHashMap都不允许。
```

### ConcurrentHashMap 和 Hashtable 的区别

```
1、底层数据结构：JDK1.7的 ConcurrentHashMap 底层采用 分段的数组+链表 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 数组+链表 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的

2、实现线程安全的方式：
	- 在JDK1.7的时候，ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。（默认分配16个Segment，比Hashtable效率提高16倍。） 到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作
	- Hashtable(同一把锁) :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈，效率越低
```

## 辅助工具类

### Conllection和Collections有什么区别

```
Conllection是集合的上级接口，继承与他的接口主要有Set和List
Collections是工具类，有很多对集合操作的方法    
```

### 如何把集合变成线程安全

```
Collections.synchronizedCollection(c)
Collections.synchronizedList(list)
Collections.synchronizedSet(set)
Collections.synchronizedMap(map)
就是在集合的核心方法添加上了synchronized关键字    
```

