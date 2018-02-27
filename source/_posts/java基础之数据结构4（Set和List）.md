---
title: 详解Java中的Set和List
tags: [Interview,java,数据结构]
categories: Java Basic
copyright: true
---
#### 一、Set
##### 1.Set 里的元素是不能重复的，那么用什么方法来区分重复与否呢？是用 == 还是 equals()？ 它们有何区别?
<!--more-->
- 如果`hash`码值不相同，说明是一个新元素，存；
如果没有元素和传入对象（也就是`add`的元素）的`hash`值相等，那么就认为这个元素在`table`中不存在，将其添加进`table`；
- 如果`hash`码值相同，且`equles`判断相等，说明元素已经存在，不存；如果`hash`码值相同，且`equles`判断不相等，说明元素不存在，存；

java中的数据类型，可分为两类：
**1.基本数据类型**
也称原始数据类型。**byte,short,char,int,long,float,double,boolean**,他们之间的比较，应用双等号（==），比较的是他们的值。基本数据类型没有equals方法哦。
**2.复合数据类型(类)**
当他们用（==）进行比较的时候，比较的是他们在内存中的存放地址，所以，除非是同一个`new`出来的对象，他们的比较后的结果为`true`，否则比较后结果为`false`。 JAVA当中所有的类都是继承于`Object`这个基类的，在`Object`中的基类中定义了一个`equals`的方法，这个方法的初始行为是比较对象的内存地址，但在一些类库当中这个方法被覆盖掉了，如`String,Integer,Date`在这些类当中`equals`有其自身的实现，而不再是比较类在堆内存中的存放地址了。
  对于复合数据类型之间进行`equals`比较，在没有覆写`equals`方法的情况下，他们之间的比较还是基于他们在内存中的存放位置的地址值的，因为`Object`的`equals`方法也是用双等号（==）进行比较的，所以比较后的结果跟双等号（==）的结果相同。

##### 2.TreeMap：TreeMap 是采用什么树实现的？TreeMap、HashMap、LindedHashMap的区别。
TreeMap 是一个有序的**key-value**集合，它是通过[红黑树](http://www.cnblogs.com/skywang12345/p/3245399.html)实现的。  TreeMap 继承于**AbstractMap**，所以它是一个Map，即一个key-value集合。TreeMap 实现了NavigableMap接口，意味着它**支持一系列的导航方法。**比如返回有序的key集合。TreeMap 实现了Cloneable接口，意味着**它能被克隆**。  TreeMap 实现了java.io.Serializable接口，意味着**它支持序列化**。
TreeMap基于**红黑树**（Red-Black tree）实现。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的** **Comparator** **进行排序**，具体取决于使用的构造方法。  TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。  另外，TreeMap是**非同步**的。  它的iterator 方法返回的**迭代器是fail-fastl**的。

##### 3.TreeMap和TreeSet在排序时如何比较元素？Collections工具类中的sort()方法如何比较元素？
**TreeSet**要求存放的对象所属的类必须实现`Comparable`接口，该接口提供了比较元素的`compareTo()`方法，当插入元素时会回调该方法比较元素的大小。`TreeMap`要求存放的键值对映射的键必须实现`Comparable`接口从而根据键对元素进行排序。`Collections`工具类的`sort`方法有两种重载的形式，第一种要求传入的待排序容器中存放的对象必须实现`Comparable`接口以实现元素的比较；第二种不强制性的要求容器中的元素必须可比较，但是要求传入第二个参数，参数是`Comparator`接口的子类型（需要重写`compare`方法实现元素的比较），相当于一个临时定义的排序规则，其实就是通过接口注入比较元素大小的算法，也是对回调模式的应用（Java中对函数式编程的支持）。

##### 3.TreeSet：一个已经构建好的 TreeSet，怎么完成倒排序。
**1、自然顺序**
即类要实现`Comparable`接口，并重写`compareTo()`方法，`TreeSet`对象调用`add()`方法时，会将存入的对象提升为`Comparable`类型，然后调用对象中的`compareTo()`方法进行比较，根据比较的返回值进行存储。
因为`TreeSet`底层是二叉树，当`compareTo`方法返回`0`时，不存储；当`compareTo`方法返回正数时，存入二叉树的右子树；当`compareTo`方法返回负数时，存入二叉树的左子树。如果一个类没有实现`Comparable`接口就将该类对象存入`TreeSet`集合，会发生类型转换异常。
**2、比较器顺序Comparator**
创建`TreeSet`对象的时候可以指定一个比较器，即传入一个`Comparator`对象，那么`TreeSet`会优先按照`Comparator`中的`compare()`方法排序，`compare`方法中有两个参数，第一个是调用该方法的对象，第二个值集合中已经存入的对象。
##### 4.EnumSet 是什么
##### 5.HashSet和TreeSet有什么区别?
- 底层存储的数据结构不同
`HashSet`底层用的是`HashMap`哈希表结构存储，而`TreeSet`底层用的是`TreeMap`树结构存储
- 存储时保证数据唯一性依据不同
`HashSet`是通过复写`hashCode()`方法和`equals()`方法来保证的，而`HashSet`通过`Compareable`接口的`compareTo()`方法来保证的
- 有序性不一样
`HashSet`无序，`TreeSet`有序
##### 6.HashSet 内部是如何工作的
`HashSet`:底层数据结构是哈希表，本质就是对哈希值的存储，通过判断元素的`hashCode`方法和`equals`方法来保证元素的唯一性，当`hashCode`值不相同，就直接存储了，不用在判断`equals`了，当`hashCode`值相同时，会在判断一次`euqals`方法的返回值是否为`true`，如果为`true`则视为用一个元素，不用存储，如果为`false`，这些相同哈希值不同内容的元素都存放一个`bucket`桶里（当哈希表中有一个桶结构，每一个桶都有一个哈希值）
`TreeSet`:底层的数据结构是二叉树，可以对`Set`集合中的元素进行排序,这种结构，可以提高排序性能, 根据比较方法的返回值确定的,只要返回的是0.就代表元素重复

##### 7.WeakHashMap 是怎么工作的？
#### 二、 List
##### 1.List, Set, Map三个接口，存取元素时各有什么特点？
`List`与`Set`都是单列元素的集合，它们有一个功共同的父接口`Collection`。
**Set**里面不允许有重复的元素，
存元素：`add`方法有一个`boolean`的返回值，当集合中没有某个元素，此时`add`方法可成功加入该元素时，则返回`true`；当集合含有与某个元素`equals`相等的元素时，此时`add`方法无法加入该元素，返回结果为`false`。
取元素：没法说取第几个，只能以`Iterator`接口取得所有的元素，再逐一遍历各个元素。

**List**表示有先后顺序的集合，
存元素：多次调用`add(Object)`方法时，每次加入的对象按先来后到的顺序排序，也可以插队，即调用`add(int index,Object)`方法，就可以指定当前对象在集合中的存放位置。
取元素：
方法1：`Iterator`接口取得所有，逐一遍历各个元素
方法2：调用`get(index i)`来明确说明取第几个。使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在`List`中的位置，类似于数组下标）来访问`List`中的元素，这类似于Java的数组。

**Map**是双列的集合，存放用`put`方法:`put(obj key,obj value)`，每次存储时，要存储一对`key/value`，不能存储重复的`key`，这个重复的规则也是按equals比较相等。
取元素：用`get(Object key)`方法根据`key`获得相应的`value`。
也可以获得所有的`key`的集合，还可以获得所有的`value`的集合，
还可以获得`key`和`value`组合成的`Map.Entry`对象的集合。

**List**以特定次序来持有元素，可有重复元素。`Set `无法拥有重复元素,内部排序。`Map` 保存`key-value`值，`value`可多值。

##### 2.List, Set, Map 是否继承自 Collection 接口
`List`和`Set`是的，而`Map`不是。

##### 3.遍历一个 List 有哪些不同的方式
```java
       Iterator it1 = list.iterator();
       while(it1.hasNext()){
           System.out.println(it1.next());
       }

       //方法2
       for(Iterator it2 = list.iterator();it2.hasNext();){
            System.out.println(it2.next());
       }

       //方法3
       for(String tmp:list){
           System.out.println(tmp);
       }

       //方法4
       for(int i = 0;i < list.size(); i ++){
           System.out.println(list.get(i));
       }
```
#### 三、LinkedList
##### 1.LinkedList 是单向链表还是双向链表
**Linkedlist**，双向链表，优点，增加删除，用时间很短，但是因为没有索引，对索引的操作，比较麻烦，只能循环遍历，但是每次循环的时候，都会先判断一下，这个索引位于链表的前部分还是后部分，每次都会遍历链表的一半 ，而不是全部遍历。
双向链表，都有一个`previous`和`next`， 链表最开始的部分都有一个`fiest`和`last `指向第一个元素，和最后一个元素。增加和删除的时候，只需要更改一个`previous`和`next`，就可以实现增加和删除，所以说，`LinkedList`对于数据的删除和增加相当的方便。

##### 2.LinkedList 与 ArrayList 有什么区别?
-  因为`Array`是基于索引`(index)`的数据结构，它使用索引在数组中搜索和读取数据是很快的。`Array`获取数据的时间复杂度是`O(1)`,但是要删除数据却是开销很大的，因为这需要重排数组中的所有数据。
- 相对于`ArrayList`，`LinkedList`插入是更快的。因为`LinkedList`不像`ArrayList`一样，不需要改变数组的大小，也不需要在数组装满的时候要将所有的数据重新装入一个新的数组，这是`ArrayList`最坏的一种情况，时间复杂度是`O(n)`，而`LinkedList`中插入或删除的时间复杂度仅为`O(1)`。`ArrayList`在插入数据时还需要更新索引（除了插入数组的尾部）。
-  类似于插入数据，删除数据时，`LinkedList`也优于`ArrayList`。
- `LinkedList`需要更多的内存，因为`ArrayList`的每个索引的位置是实际的数据，而`LinkedList`中的每个节点中存储的是实际的数据和前后节点的位置。

##### 3.描述下 Java 中集合（Collections），接口（Interfaces），实现（Implementations）的概念。

##### 4.插入数据时，ArrayList, LinkedList, Vector谁速度较快？
`ArrayList`和`Vector`都是数组实现，但不同的是，`Vector`是线程安全，加了同步，所以原则上`ArrayList`比`Vector`比快；
`LinkekList`是链表实现，增删快，查找慢，所以你要是插入数据时，显然`LinkedList`是最快的，其次是`ArrayList`，再者`Vector`属于遗留容器（Java早期的版本中提供的容器，除此之外，`Hashtable、Dictionary、BitSet、Stack、Properties`都是遗留容器），已经不推荐使用，但是由于`ArrayList`和`LinkedListed`都是非线程安全的，如果遇到多个线程操作同一个容器的场景，则可以通过工具类`Collections`中的`synchronizedList`方法将其转换成线程安全的容器后再使用（这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现）。
#### 四、ArrayList
##### 1.ArrayList 和 HashMap 的默认大小是多数?
这里要讨论这些常用的默认初始容量和扩容的原因是：
当底层实现涉及到扩容时，容器或重新分配一段更大的连续内存（如果是离散分配则不需要重新分配，离散分配都是插入新元素时动态分配内存），要将容器原来的数据全部复制到新的内存上，这无疑使效率大大降低。
加载因子的系数小于等于1，意指  即当元素个数 超过 容量长度*加载因子的系数 时，进行扩容。另外，扩容也是有默认的倍数的，不同的容器扩容情况不同。

`ArrayList、Vector`默认初始容量为`10`。
`Vector`：线程安全，但速度慢。底层数据结构是数组结构，加载因子为1：即当 元素个数 超过 容量长度 时，进行扩容。扩容增量：原容量的 1倍。如 `Vector`的容量为`10`，一次扩容后是容量为`20`。
`ArrayList`：线程不安全，查询速度快。底层数据结构是数组结构，扩容增量：原容量的 `0.5倍+1`，如 `ArrayList`的容量为`10`，一次扩容后是容量为`16`。

`Set`(集) 元素无序的、不可重复。
`HashSet`：线程不安全，存取速度快。
底层实现是一个`HashMap`（保存数据），实现`Set`接口
默认初始容量为`16`（为何是16，见下方对HashMap的描述）
加载因子为`0.75`：即当 元素个数 超过 容量长度的`0.75`倍 时，进行扩容
扩容增量：原容量的` 1 `倍
如 `HashSet`的容量为`16`，一次扩容后是容量为`32`。

`Map`是一个双列集合
`HashMap`：默认初始容量为`16`
（为何是`16`：`16`是`2^4`，可以提高查询效率，另外，`32=16<<1 `      -->至于详细的原因可另行分析，或分析源代码）
加载因子为`0.75`：即当 元素个数 超过 容量长度的`0.75`倍 时，进行扩容
扩容增量：原容量的 `1 `倍
如 `HashSet`的容量为`16`，一次扩容后是容量为`32`。

##### 2.ArrayList 和 Set 的区别？
- `Set` 集合是无序不可以重复的的、`List `集合是有序可以重复的。
- `ArrayList`是数组存储的方式，`HashSet`存储会先进行`HashCode`值得比较(`hashcode`和`equals`方法)，若相同就不会再存储。

补充一下：`Hashset`就是采用哈希算法存取对象的集合，对象用完之后没有回收就是内存泄漏。一个对象一旦`hashCode`生成之后，再对属性值修改后其`Hashcode`值就会发生改变，再通过`hashSet`删除就删除不掉了。

以上的问题还可以继续有如下变形，理解了就能融会贯通：
ArrayList, LinkedList, Vector的区别
ArrayList是如何实现的，ArrayList 和 LinkedList 的区别
ArrayList如何实现扩容
##### 6.Array 和 ArrayList 有何区别？什么时候更适合用Array？
**ArrayList可以算是Array的加强版，（对array有所取舍的加强）。**
**存储内容比较： **
•`Array`数组可以包含基本类型和对象类型，
•`ArrayList`却只能包含对象类型。
但是需要注意的是：`Array`数组在存放的时候一定是同种类型的元素。`ArrayList`就不一定了，因为`ArrayList`可以存储`Object`。

**空间大小比较：**
•	它的空间大小是固定的，空间不够时也不能再次申请，所以需要事前确定合适的空间大小。
•	`ArrayList`的空间是动态增长的，如果空间不够，它会创建一个空间比原空间大一倍的新数组，然后将所有元素复制到新数组中，接着抛弃旧数组。而且，每次添加新的元素的时候都会检查内部数组的空间是否足够。（比较麻烦的地方）。

**方法上的比较： **
`ArrayList`作为`Array`的增强版，当然是在方法上比`Array`更多样化，比如添加全部`addAll()`、删除全部`removeAll()`、返回迭代器`iterator()`等。

适用场景：
如果想要保存一些在整个程序运行期间都会存在而且不变的数据，我们可以将它们放进一个全局数组里，但是如果我们单纯只是想要以数组的形式保存数据，而不对数据进行增加等操作，只是方便我们进行查找的话，那么，我们就选择`ArrayList`。而且还有一个地方是必须知道的，就是如果我们需要对元素进行频繁的移动或删除，或者是处理的是超大量的数据，那么，使用`ArrayList`就真的不是一个好的选择，因为它的效率很低，使用数组进行这样的动作就很麻烦，那么，我们可以考虑选择`LinkedList`。

