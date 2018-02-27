---
title: Java里必须要会的Hash
tags: [Interview,java,数据结构]
categories: Java Basic
copyright: true
---
##### 1.Hashcode 的作用
对于包含容器类型的程序设计语言来说，基本上都会涉及到 `hashCode`。在Java中也一样，`hashCode`方法的主要作用是为了配合基于散列的集合一起正常运行，这样的散列集合包括`HashSet、HashMap`以及`HashTable`。<!--more-->
为什么这么说呢？考虑一种情况，当向集合中插入对象时，如何判别在集合中是否已经存在该对象了？（注意：集合中不允许重复的元素存在）
也许大多数人都会想到调用`equals`方法来逐个进行比较，这个方法确实可行。但是如果集合中已经存在一万条数据或者更多的数据，如果采用`equals`方法去逐一比较，效率必然是一个问题。此时`hashCode`方法的作用就体现出来了，当集合要添加新的对象时，先调用这个对象的`hashCode`方法，得到对应的`hashcode`值，实际上在`HashMap`的具体实现中会用一个`table`保存已经存进去的对象的`hashcode`值，如果`table`中没有该`hashcode`值，它就可以直接存进去，不用再进行任何比较了；如果存在该`hashcode`值， 就调用它的`equals`方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址，所以这里存在一个冲突解决的问题，这样一来实际调用`equals`方法的次数就大大降低了，说通俗一点：Java中的`hashCode`方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为散列值。
```java
//java.util.HashMap的中put方法的具体实现：
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
}
```
`put`方法是用来向`HashMap`中添加新的元素，从`put`方法的具体实现可知，会先调用`hashCode`方法得到该元素的`hashCode`值，然后查看`table`中是否存在该`hashCode`值，如果存在则调用`equals`方法重新确定是否存在该元素，如果存在，则更新`value`值，否则将新的元素添加到`HashMap`中。从这里可以看出，`hashCode`方法的存在是为了减少`equals`方法的调用次数，从而提高程序效率。

>[《浅谈Java中的hashcode方法》](http://www.importnew.com/18851.html)

##### 2.简述一致性 Hash 算法
一致性哈希修正了CARP使用的简单哈希算法带来的问题，在分布式系统中也得到了广泛应用。一致性hash算法（DHT）通过减少影响范围的方式解决了增减服务器导致的数据散列问题，从而解决了分布式环境下负载均衡问题，如果存在热点数据，那么通过增添节点的方式，对热点区间进行划分，将压力分配至其他服务器。重新达到负载均衡的状态。

>[《一致性哈希算法1》](https://www.cnblogs.com/color-my-life/p/5799903.html)
[《一致性哈希算法2》](https://www.cnblogs.com/lpfuture/p/5796398.html)

##### 3.有没有可能两个不相等的对象有相同的 hashcode？当两个对象hashcode 相同怎么办？如何获取值对象？
对于两个对象：
- 如果调用`equals`方法得到的结果为`true`，则两个对象的`hashcode`值必定相等；
- 如果`equals`方法得到的结果为`false`，则两个对象的`hashcode`值不一定不同；
- 如果两个对象的`hashcode`值不等，则`equals`方法得到的结果必定为`false`；
- 如果两个对象的`hashcode`值相等，则`equals`方法得到的结果未知。

总之一句话：**等价对象产生相同整数的哈希码，不同对象不一定要不同的哈希码**。后面两个命题其实就是这句话的逆否命题。

所以，针对上面问题提到的，两个不相等的对象其实就是问的`equals`为`false`，那么`hashcode`不一定是不同，也就是有可能会相同了。为什呢会这样呢？`hashCode`是所有java对象的固有方法，如果不重载的话，返回的实际上是该对象在jvm的堆上的内存地址，而不同对象的内存地址肯定不同，所以这个`hashCode`也就肯定不同了。如果重载了的话，由于采用的算法的问题，有可能导致两个不同对象的`hashCode`相同。
后面的问题其实会比较多的出现在**Map**的面试考察中，
当我们调用`Map`的`get()`方法，`HashMap`会使用键对象的`hashcode`找到`bucket`位置，然后获取值对象。面试官提醒他如果有两个值对象储存在同一个`bucket`，他给出答案：将会遍历链表直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？除非面试者直`HashMap`在链表中存储的是键值对，否则他们不可能回答出这一题。其中一些记得这个重要知识点的面试者会说，找到bucket位置之后，会调用`keys.equals()`方法去找到链表中正确的节点，最终找到要找的值对象。完美的答案！
##### 4.为什么在重写 equals 方法的时候需要重写 hashCode 方法？equals与hashCode 的异同点在哪里?
这里头牵扯到另外隐含的问题，一并拿出来解决：
>1、首先我们为什么需要重写`hashCode()`方法和`equals()`方法
2、为什么在重写 equals 方法的时候需要重写 hashCode 方法？
3、如何重写这两个方法?

**回答第一个问题：**
Java中的超类`Object`类中定义的`equals()`方法是用来比较两个引用所指向的对象的内存地址是否一致，`Object`类中`equals()`方法的源码
```java
public boolean equals(Object obj) {
       return (this == obj);
}
```
Object类中的`hashCode()`方法，用`native`关键字修饰，说明这个方法是个原生函数，也就说这个方法的实现不是用java语言实现的，是使用c/c++实现的，并且被编译成了DLL，由java去调用，jdk源码中不包含。对于不同的平台它们是不同的，java在不同的操作系统中调用不同的`native`方法实现对操作系统的访问，因为java语言不能直接访问操作系统底层，因为它没有指针。
Java的API文档对`hashCode()`方法做了详细的说明，这也是我们重写`hashCode(`)方法时的原则【Object类】
```java
public native int hashCode();
```
**我们在定义类时，我们经常会希望两个不同对象的某些属性值相同时就认为他们相同，所以我们要重写equals()方法，但同时也要改写`hashCode()`方法，所以java中的很多类都重写了这两个方法,例如String类，包装类。**
**第二个问题：为什么在重写 equals 方法的时候需要重写 hashCode 方法？**
Java 对于hashCode方法的规约：
- 在java应用程序运行时，无论何时多次调用同一个对象时的`hsahCode()`方法，这个对象的`hashCode()`方法的返回值必须是同一个`int`值
-  如果两个对象`equals()`返回值为`true`,则他们的`hashCode()`也必须返回相同的int值
- 如果两个对象根据`equals()`比较是不等的，则`hashCode()`方法不一定得返回不同的整数。

根据规约，为了保证同一个对象，`equals`相同的情况下`hashcode`值必定相同，如果重写了`equals`而未重写`hashcode`方法，可能就会出现两个对象`equals`相同的（因为equal都是根据对象的特征进行重写的），但`hashcode`确实不相同的。如果不这样做程序也可以执行，只不过会隐藏bug。比如重写了`equals`方法，属性相同就认为相同，但不重写`hashcode`，那么我们再`new`一个新的对象，当原对象`equals`（新对象）等于`true`时，两者的`hashcode`却是不一样的，由此将产生了理解的不一致，如在存储散列集合时（如`Set`类），将会存储了两个值一样的对象，导致混淆，因此就也需要重写`hashcode()`。【一句话，容易在要求散列存储的时候，把相同对象给放到一个集合】
有这个要求的症结在于，要考虑到类似HashMap、HashTable、HashSet的这种散列的数据类型的运用。

##### 5.a.hashCode() 有什么用？与 a.equals(b) 有什么关系？
- 如果调用`equals`方法得到的结果为`true`，则两个对象的`hashcode`值必定相等；
- 如果`equals`方法得到的结果为`false`，则两个对象的`hashcode`值不一定不同；
- 如果两个对象的`hashcode`值不等，则`equals`方法得到的结果必定为`false`；
- 如果两个对象的`hashcode`值相等，则`equals`方法得到的结果未知。

总之一句话：**等价对象产生相同整数的哈希码，不同对象不一定要不同的哈希码**。后面两个命题其实就是这句话的逆否命题。

##### 6.hashCode() 和 equals() 方法的重要性体现在什么地方？
散列存储集合如`HashMap`的很多函数要基于`equal()`函数和`hashCode()`函数。`hashCode()`用来定位要存放的位置，`equal()`用来判断是否**相等**。 
那么，*相等的概念是什么？* 
Object版本的equal只是简单地判断是不是**同一个**实例。但是有的时候，我们想要的的是逻辑上的相等。比如有一个学生类student，有一个属性studentID，只要studentID相等，不是同一个实例我们也认为是同一学生。当我们认为判定equals的相等应该是逻辑上的相等而不是只是判断是不是内存中的同一个东西的时候，就需要重写equal()。**而涉及到HashMap的时候，重写了equals()，就需要重写hashCode()**

##### 7.Object类hashcode,equals 设计原则？ sun为什么这么设计？
>- 在程序执行期间，只要`equals`方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，`hashCode`方法必须始终如一地返回同一个整数。
>- 如果两个对象根据`equals`方法比较是相等的，那么调用两个对象的`hashCode`方法必须返回相同的整数结果。
>- 如果两个对象根据`equals`方法比较是不等的，则`hashCode`方法不一定得返回不同的整数。

**以上是摘自Effective Java的原话，下面进行具体解读：**
- 1.同一个对象（没有发生过修改）无论何时调用`hashCode()`得到的返回值必须一样。 
如果一个`key`对象在`put`的时候调用`hashCode()`决定了存放的位置，而在`get`的时候调用`hashCode()`得到了不一样的返回值，这个值映射到了一个和原来不一样的地方，那么肯定就找不到原来那个键值对了。
- 2.`hashCode()`的返回值相等的对象不一定相等，通过`hashCode()`和`equals()`必须能唯一确定一个对象 
不相等的对象的`hashCode()`的结果可以相等。`hashCode()`在注意关注碰撞问题的时候，也要关注生成速度问题，完美`hash`不现实。
- 3.一旦重写了`equals()`函数（重写`equals`的时候还要注意要满足**自反性、对称性、传递性、一致性**），就必须重写`hashCode()`函数。而且`hashCode()`的生成哈希值的依据应该是`equals()`中用来比较是否相等的字段 。如果两个由`equals()`规定相等的对象生成的`hashCode`不等，对于`hashMap`来说，他们很可能分别映射到不同位置，没有调用`equals()`比较是否相等的机会，两个实际上相等的对象可能被插入不同位置，出现错误。其他一些基于哈希方法的集合类可能也会有这个问题。
[【牛客网面试题】](https://www.nowcoder.net/questionTerminal/040f6133720941b3b782b5cceaa27198)

##### 8.Object：Object有哪些公用方法？Object类的概述。
`Object`是所有类的父类，任何类都默认继承`Object`。
`clone`：保护方法，实现对象的浅复制，只有实现了`Cloneable`接口才可以调用该方法，否则抛出`CloneNotSupportedException`异常
`equals`：在`Object`中与`==`是一样的，子类一般需要重写该方法
`hashCode`：该方法用于哈希查找，重写了`equals`方法一般都要重写`hashCode`方法。这个方法在一些具有哈希功能的`Collection`中用到
`getClass`：`final`方法，获得运行时类型
`wait`：使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。`wait()`方法一直等待，直到获得锁或者被中断。`wait(long timeout)`设定一个超时间隔，如果在规定时间内没有获得锁就返回。 调用该方法后当前线程进入睡眠状态，直到以下事件发生：
- 1. 其他线程调用了该对象的`notify`方法
- 2. 其他线程调用了该对象的`notifyAll`方法
- 3. 其他线程调用了`interrupt`中断该线程
- 4. 时间间隔到了，此时该线程就可以被调度了，如果是被中断的话就抛出一个`InterruptedException`异常

`notify`：唤醒在该对象上等待的某个线程
`notifyAll`：唤醒在该对象上等待的所有线程
`toString`：转换成字符串，一般子类都有重写，否则打印句柄

##### 如何在父类中为子类自动完成所有的 hashcode 和 equals 实现？这么做有何优劣。
##### 可以在 hashcode() 中使用随机数字吗？
