---
title: java基础之数据结构1（基础篇）
tags: Interview
categories: Java
copyright: true
---
![java集合框架图](http://upload-images.jianshu.io/upload_images/8926909-080c94b2247e13cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
#### 一、基础类型(Primitives)
##### 1.基础类型(Primitives)与封装类型(Wrappers)的区别在哪里？
**1 传递方式不同**
封装类是引用类型。基本类型（原始数据类型）在传递参数时都是按值传递，而封装类型是按引用传递的(其实“引用也是按值传递的”，传递的是对象的地址)。由于包装类型都是final修饰的不可变量，因此没有提供改变它值的方法，增加了对“按引用传递”的理解难度。
`int`是基本类型，直接存放数值；`Integer`是类，产生对象时用一个引用指向这个对象。
**2 封装类可以有方法和属性**
封装类可以有方法和属性，利用这些方法和属性来处理数据，如`Integer.parseInt(Strings)`。**基本数据类型都是final修饰的**，不能继承扩展新的类、新的方法。
**3 默认值不同**
基本类型跟封装类型的默认值是不一样的。如`int i`,`i`的预设为`0`；`Integer j`，`j`的预设为`null`,因为封装类产生的是对象，对象默认值为null。
**4 存储位置**
基本类型在内存中是存储在栈中，引用类型的引用（值的地址）存储在栈中，而实际的对象（值）是存在堆中。
**基本数据类型的好处就是速度快（不涉及到对象的构造和回收），封装类的目的主要是更好的处理数据之间的转换。**JDK5.0开始可以自动封包了，**基本数据类型可以自动封装成封装类**。
>[《 基础类型(Primitives)与封装类型(Wrappers)的区别》](http://blog.csdn.net/xzp_12345/article/details/79038251)

##### 2.简述九种基本数据类型的大小，以及他们的封装类。
![基本数据类型和包装类](http://upload-images.jianshu.io/upload_images/8926909-6ff5f555b500eab8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 3.int和 Integer 哪个会占用更多的内存？int和 Integer 有什么区别？parseInt()函数在什么时候使用到？
当然是`Integer`会占用更多的内存。以下为`int`和`Integer`的区别：
1、`Integer`是`int`的包装类，`int`则是java的一种基本数据类型
2、`Integer`变量必须实例化后才能使用，而`int`变量不需要
3、`Integer`实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值
4、`Integer`的默认值是`null`，`int`的默认值是0
延伸关于Integer和int的比较 ：
- 由于`Integer`变量实际上是对一个`Integer`对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。
```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```
- `Integer`变量和`int`变量比较时，只要两个变量的值是向等的，则结果为true（因为包装类`Integer`和基本数据类型`int`比较时，java会自动拆包装为`int`，然后进行比较，实际上就变为两个int变量的比较）
```java
Integer i = new Integer(100);
int j = 100；
System.out.print(i == j); //true
```
- 非`new`生成的`Integer`变量和`new Integer()`生成的变量比较时，结果为`false`。（因为非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）
```java
Integer i = new Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```
- 对于两个非`new`生成的`Integer`对象，进行比较时，如果两个变量的值在区间`-128`到`127`之间，则比较结果为`true`，如果两个变量的值不在此区间，则比较结果为`false`
```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true
Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```
对于第4条的原因：
java在编译`Integer i = 100 ;`时，会翻译成为`Integer i = Integer.valueOf(100)；`，而java API中对`Integer`类型的`valueOf`的定义如下：
```java
public static Integer valueOf(int i){
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high){
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
```
java对于`-128`到`127`之间的数，会进行缓存，`Integer i = 127`时，会将`127`进行缓存，下次再写`Integer j = 127`时，就会直接从缓存中取，就不会`new`了。
>[《int和Integer的区别》](http://www.cnblogs.com/guodongdidi/p/6953217.html)

**parseInt()**
`parseInt()`将把该字符之前的字符串转换成数字。`parseInt()`方法还有基模式，可以把二进制、八进制、十六进制或其他任何进制的字符串转换成整数。基是由`parseInt()`方法的第二个参数指定的，所以要解析十六进制的值，当然，对二进制、八进制，甚至十进制（默认模式），都可以这样调用`parseInt()`方法。
如果十进制数包含前导0，那么最好采用基数10，这样才不会意外地得到八进制的值。
```java
static int parseInt(String s)
static int parseInt(String s, int radix)
```
##### 4.如何去小数四舍五入保留小数点后两位？
```java
//使用银行家算法
BigDecimal i = d.multiply(r).setScale(2,RoundingMode.HALF_EVEN);
```
推荐使用`BigDecimal` ，并且采用`setScale`方法来设置精确度，同时使用`RoundingMode.HALF_UP`表示使用最近数字舍入法则来近似计算。在这里我们可以看出`BigDecimal`和四舍五入是绝妙的搭配。
[《java提高篇(三)-----java的四舍五入》](http://www.cnblogs.com/chenssy/p/3366632.html)
##### 5.char 型变量中能不能存贮一个中文汉字，为什么？
`char`型变量是用来存储`Unicode`编码的字符的，`unicode`编码字符集中包含了汉字，所以`char`型变量中当然可以存储汉字啦。不过，如果某个特殊的汉字没有被包含在`unicode`编码字符集中，那么，这个char型变量中就不能存储这个特殊汉字。补充
说明：`unicode`编码占用两个字节，所以，`char`类型的变量也是占用两个字节。
#### 二、类型转换
##### 1.怎样将 bytes 转换为 long 类型
```java
public static long bytes2long(byte[] b) {
    long temp = 0;
    long res = 0;
    for (int i=0;i<8;i++) {
        res <<= 8;
        temp = b[i] & 0xff;
        res |= temp;
    }
    return res;
}
```
##### 2.怎么将 byte 转换为 String**
```java
//string 转成 byte
string s = "Hello!!";
byte[] b = new byte[1024*1024];
b = System.Text.Encoding.ASCII.GetBytes(s);
//当string含有中文字符时用 System.Text.Encoding.UTF8.GetBytes(s);
sock.Send(b);
 ```
```java
//byte 转成 string
byte[] b1 = new byte[1024*1024*2];
sock.Receive(b1);
string s1 = System.Text.Encoding.ASCII.GetString(b1);
// System.Text.Encoding.UTF8.GetString(b1);
```
注意： 在把`byte`数组转换成`string`的时候，由于`byte`数组有2M的字节，所以转换后得到的字符串s1也会填充到2M的字符（用\0来填充）
所以，为了避免这个问题，可以使用`Receive`返回的字节数来确定接收到byte的长度
```java
int length = sock.Receive(b1);
string s1 = System.Text.Encoding.ASCII.GetString(b1, 0, length);
//这样，s1就为byte实际的值
```

##### 3.如何将数值型字符转换为数字
`string `和`int`之间的转换
- `string`转换成`int`  : `Integer.valueOf("12")`
- `int`转换成`string` : `String.valueOf(12)`

`char`转`int`之间的转换
- 首先将char转换成string
`String str=String.valueOf('2')`
`Integer.valueof(str)` 或者`Integer.PaseInt(str)`
Integer.valueof返回的是Integer对象，Integer.paseInt返回的是int


##### 4.我们能将 int 强制转换为 byte 类型的变量吗？如果该值大于 byte 类型的范围，将会出现什么现象
`Byte`转`int`
```java
public static int bytes2int(byte[] bytes) {
        int num = bytes[0] & 0xFF;
        num |= ((bytes[1] << 8) & 0xFF00);
        num |= ((bytes[2] << 16) & 0xFF0000);
        num |= ((bytes[3] << 24) & 0xFF000000);
        return num;
}
```
`int `转 `byte`
```java
public static byte[] int2bytes(int i) {
        byte[] b = new byte[4];
        b[0] = (byte) (0xff&i);
        b[1] = (byte) ((0xff00&i) >> 8);
        b[2] = (byte) ((0xff0000&i) >> 16);
        b[3] = (byte) ((0xff000000&i) >> 24);
        return b;
}
```
##### 5.能在不进行强制转换的情况下将一个 double 值赋值给 long 类型的变量吗?
```java
    public static void main(String[] args) {
        double d = 88.88;
        long l = Math.round(d);
        System.out.println(l);

        long ll = 100L;
        double dd = (double) ll;
        System.out.println(dd);
    }
```
##### 6.类型向下转换是什么?
由低层次类型转换为高层次类型称为向上类型转换。向上类型转换是自动进行的，比如把int型变量赋给为long型变量，把long型变量赋给double型变量，转换都是自动进行的。由派生类转换为基类也是向上提升，也是自动进行的，但转换后，基类的引用符不能应用派生类对象特有的函数。
```java
     Human  jean = new Human();
     Vervebrata   someone = jean;
     some.Work();
```
运行上面语句会出错，虽然`someone`指向了一个`Human`类的对象，但是它不能调用`Work()`函数，因为`someone`的类型为`Vertebrata`，而基类`Vertebrata`中没有申明`Work()`函数。要想通过基类引用符someone调用派生类特有的函数，必须将`someone`的类型强制转换为派生类。这种由基类向派生类转换的过程称为向下类型转换。

#### 三、 数组
##### 1.如何权衡是使用无序的数组还是有序的数组
在数据偏向查找操作的时候用有序数组快一些，在数据偏向插入的时候，无序数组好一些。删除操作效率一样。

##### 2.怎么判断数组是 null 还是为空
（无论使用哪种类型的数组，数组标识符其实只是一个引用，指向在堆中创建的一个真实对象 `Int[] A =new int[10];`new 一下就是实例化了，开辟了内存空间，基本数据类型的元素会被赋初始值，数组建立后长度不能改变，但是还是可以重新赋值）
有如下两个变量定义：
- 1 `int[] zero = new int[0];`
- 2 `int[] nil = null; `
这两种定义有什么区别呢？
 zero是一个长度为0的数组，我们称之为“空数组”，空数组也是一个对象，只是包含元素个数为0。nil是一个数组类型的空引用。

##### 3.怎么打印数组？ 怎样打印数组中的重复元素


##### 4.Array 和 ArrayList有什么区别？什么时候应该使用Array而不是ArrayList?
**1）**精辟阐述：
可以将 `ArrayList`想象成一种“会自动扩增容量的`Array`”。
**2）**`Array（[]）`：最高效；但是其容量固定且无法动态改变；
      `ArrayList`：  容量可动态增长；但牺牲效率；
**3）**建议：
基于效率和类型检验，应尽可能使用Array，无法确定数组大小时才使用ArrayList！
不过当你试着解决更一般化的问题时，Array的功能就可能过于受限。
**4）**Java中一切皆对象，Array也是对象。不论你所使用得Array型别为何，Array名称本身实际上是个reference，指向heap之内得某个实际对象。这个对象可经由“Array初始化语法”被自动产生，也可以以new表达式手动产生。
**5）**Array可做为函数返回值，因为它本身是对象的reference；
**6）**对象数组与基本类型数组在运用上几乎一模一样，唯一差别在于，前者持有得是`reference`，后者直接持有基本型别之值；
例如：
```java
string [] staff=new string[100];
int [] num=new int[10];
```
**7）**容器所持有的其实是一个`reference`指向`Object`，进而才能存储任意型别。当然这不包括基本型别，因为基本型别并不继承自任何`classes`。
**8）**面对`Array`，我们可以直接持有基本型别数值的`Array`（例如：`int [] num;`),也可以持有`reference`（指向对象）的`Array`；但是容器类仅能持有`reference`（指向对象），若要将基本型别置于容器内，需要使用`wrapper`类。但是`wrapper`类使用起来可能不很容易上手，此外，`primitives Array`的效率比起“容纳基本型别之外覆类（的`reference`）”的容器好太多了。
当然，如果你的操作对象是基本型别，而且需要在空间不足时自动扩增容量，`Array`便不适合，此时就得使用外覆类的容器了。
**9）**某些情况下，容器类即使没有转型至原来的型别，仍然可以运作无误。有一种情况尤其特别：编译器对String class提供了一些额外的支持，使它可以平滑运作。
**10）**对数组的一些基本操作，像排序、搜索与比较等是很常见的。因此在Java中提供了`Arrays`类协助这几个操作：`sort(),binarySearch(),equals(),fill(),asList().`
不过`Arrays`类没有提供删除方法，而ArrayList中有remove()方法，不知道是否是不需要在`Array`中做删除等操作的原因（因为此时应该使用链表）。
**11）**ArrayList的使用也很简单：产生`ArrayList`，利用`add()`将对象置入，利用get(i）配合索引值将它们取出。这一切就和Array的使用方式完全相同，只不过少了[]而已。

**换一种简单说法：**
**1）效率：**
数组扩容是对`ArrayList`效率影响比较大的一个因素。
每当执行`Add、AddRange、Insert、InsertRange`等添加元素的方法，都会检查内部数组的容量是否不够了，如果是，它就会以当前容量的两倍来重新构建一个数组，将旧元素Copy到新数组中，然后丢弃旧数组，在这个临界点的扩容操作，应该来说是比较影响效率的。
`ArrayList`是`Array`的复杂版本
`ArrayList`内部封装了一个`Object`类型的数组，从一般的意义来说，它和数组没有本质的差别，甚至于ArrayList的许多方法，如`Index、IndexOf、Contains、Sort`等都是在内部数组的基础上直接调用`Array`的对应方法。
**2）类型识别：**
`ArrayList`存入对象时，抛弃类型信息，所有对象屏蔽为`Object`，编译时不检查类型，但是运行时会报错。但是现在有jdk1.5后引入泛型来进行编译检查类型，如错存入了不同类型会直接报错。
`ArrayList`与数组的区别主要就是由于动态增容的效率问题了
**3）`ArrayList`可以存任何`Object`，如`String`等。**


##### 5.数组和链表数据结构描述，各自的时间复杂度
**数组**是将元素在内存中连续存放，由于每个元素占用内存相同，可以通过下标迅速访问数组中任何元素。但是如果要在数组中增加一个元素，需要移动大量元素，在内存中空出一个元素的空间，然后将要增加的元素放在其中。同样的道理，如果想删除一个元素，同样需要移动大量元素去填掉被移动的元素。如果应用需要快速访问数据，很少插入和删除元素，就应该用数组。
**链表**中的元素在内存中不是顺序存储的，而是通过存在元素中的指针联系到一起，每个结点包括两个部分：一个是存储 数据元素的数据域，另一个是存储下一个结点地址的 指针。 如果要访问链表中一个元素，需要从第一个元素开始，一直找到需要的元素位置。但是增加和删除一个元素对于链表数据结构就非常简单了，只要修改元素中的指针就可以了。如果应用需要经常插入和删除元素你就需要用链表。
- 内存存储区别
数组从栈中分配空间, 对于程序员方便快速,但自由度小。链表从堆中分配空间, 自由度大但申请管理比较麻烦.　
- 逻辑结构区别
数组必须事先定义固定的长度（元素个数），不能适应数据动态地增减的情况。当数据增加时，可能超出原先定义的元素个数；当数据减少时，造成内存浪费。　
链表动态地进行存储分配，可以适应数据动态地增减的情况，且可以方便地插入、删除数据项。（数组中插入、删除数据项时，需要移动其它数据项）　

总结
1、存取方式上，数组可以顺序存取或者随机存取，而链表只能顺序存取；　
2、存储位置上，数组逻辑上相邻的元素在物理存储位置上也相邻，而链表不一定；　
3、存储空间上，链表由于带有指针域，存储密度不如数组大；　
4、按序号查找时，数组可以随机访问，时间复杂度为O(1)，而链表不支持随机访问，平均需要O(n)；　
5、按值查找时，若数组无序，数组和链表时间复杂度均为O(1)，但是当数组有序时，可以采用折半查找将时间复杂度降为O(logn)；　
6、插入和删除时，数组平均需要移动n/2个元素，而链表只需修改指针即可；　
7、空间分配方面：
数组在静态存储分配情形下，存储元素数量受限制，动态存储分配情形下，虽然存储空间可以扩充，但需要移动大量元素，导致操作效率降低，而且如果内存中没有更大块连续存储空间将导致分配失败；
链表存储的节点空间只在需要的时候申请分配，只要内存中有空间就可以分配，操作比较灵活高效；
数组有没有length()这个方法? String有没有length()这个方法

#### 四、队列
##### 1.队列和栈是什么，列出它们的区别
**队列（Queue）**：是限定只能在表的一端进行插入和在另一端进行删除操作的线性表；
**栈（Stack）**：是限定只能在表的一端进行插入和删除操作的线性表。
区别如下：
- 规则不同
       1. 队列：先进先出（First In First Out）FIFO
       2. 栈：先进后出（First In Last Out ）FILO

- 对插入和删除操作的限定不同
       1. 队列：只能在表的一端进行插入，并在表的另一端进行删除；
       2. 栈：只能在表的一端插入和删除。

- 遍历数据速度不同
       1. 队列：基于地址指针进行遍历，而且可以从头部或者尾部进行遍历，但不能同时遍历，无需开辟空间，因为在遍历的过程中不影响数据结构，所以遍历速度要快；
       2. 栈：只能从顶部取数据，也就是说最先进入栈底的，需要遍历整个栈才能取出来，而且在遍历数据的同时需要为数据开辟临时空间，保持数据在遍历前的一致性。


##### 2.BlockingQueue是什么
 阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。
>[《聊聊并发（7）Java中的阻塞队列》](http://www.importnew.com/17537.html)

##### 3.简述 ConcurrentLinkedQueue LinkedBlockingQueue 的用处和不同之处。
- **阻塞队列：线程安全**
按 FIFO（先进先出）排序元素。队列的头部是在队列中时间最长的元素。队列的尾部是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列检索操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。
注意：
1、必须要使用take()方法在获取的时候达成阻塞结果
2、使用`poll()`方法将产生非阻塞效果
- 非阻塞队列
基于链接节点的、无界的、线程安全。此队列按照 FIFO（先进先出）原则对元素进行排序。队列的头部 是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列检索操作从队列头部获得元素。当许多线程共享访问一个公共 `collection` 时，`ConcurrentLinkedQueue `是一个恰当的选择。此队列不允许 null 元素。

在并发编程中，一般推荐使用阻塞队列，这样实现可以尽量地避免程序出现意外的错误。阻塞队列使用最经典的场景就是socket客户端数据的读取和解析，读取数据的线程不断将数据放入队列，然后解析线程不断从队列取数据解析。还有其他类似的场景，只要符合生产者-消费者模型的都可以使用阻塞队列。
使用非阻塞队列，虽然能即时返回结果（消费结果），但必须自行编码解决返回为空的情况处理（以及消费重试等问题）。另外他们都是线程安全的，不用考虑线程同步问题。
> [《JAVA阻塞队列以及非阻塞队列的区别》](http://www.cnblogs.com/starcrm/p/4998067.html)

##### 4.ArrayList、Vector、LinkedList的存储性能和特性
`ArrayList` 和`Vector`他们底层的实现都是一样的，都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢。

`Vector`中的方法由于添加了`synchronized`修饰，因此`Vector`是线程安全的容器，但性能上较`ArrayList`差，因此已经是Java中的遗留容器。

`LinkedList`使用双向链表实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。

`Vector`属于遗留容器（Java早期的版本中提供的容器，除此之外，`Hashtable、Dictionary、BitSet、Stack、Properties`都是遗留容器），已经不推荐使用，但是由于ArrayList和LinkedListed都是非线程安全的，如果遇到多个线程操作同一个容器的场景，则可以通过工具类`Collections`中的`synchronized List`方法将其转换成线程安全的容器后再使用（这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现）。


#### 五、String
##### 1.ByteBuffer 与 StringBuffer有什么区别

#### 六、Collections
##### 1.介绍Java中的Collection FrameWork。集合类框架的基本接口有哪些？
总共有两大接口：`Collection` 和`Map` ，一个元素集合，一个是键值对集合；
- 其中`List`和`Set`接口继承了`Collection`接口，一个是有序元素集合，一个是无序元素集合；
- `ArrayList`和` LinkedList `实现了`List`接口，`HashSet`实现了`Set`接口，这几个都比较常用；
- `HashMap `和`HashTable`实现了`Map`接口，并且`HashTable`是线程安全的，但是`HashMap`性能更好；

##### 2.Collections类是什么？Collection 和 Collections的区别？Collection、Map的实现.
`Collection`是单列集合
 - `List`元素是有序的、可重复。有序的`collection`，可以对列表中每个元素的插入位置进行精确地控制。可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。 可存放重复元素，元素存取是有序的。
- `List`接口中常用类
`Vector`：线程安全，但速度慢，已被ArrayList替代。底层数据结构是数组结构；
`ArrayList`：线程不安全，查询速度快。底层数据结构是数组结构；
`LinkedList`：线程不安全。增删速度快。底层数据结构是列表结构；

 - `Set`(集) 元素无序的、不可重复。取出元素的方法只有迭代器。不可以存放重复元素，元素存取是无序的。
- `Set`接口中常用的类
`HashSet`：线程不安全，存取速度快。它是如何保证元素唯一性的呢？依赖的是元素的`hashCode`方法和`euqals`方法。
`TreeSet`：线程不安全，可以对`Set`集合中的元素进行排序。它的排序是如何进行的呢？通过`compareTo`或者`compare`方法中的来保证元素的唯一性。元素是以二叉树的形式存放的。

`Map` 是一个双列集合
- `Hashtable`:线程安全，速度快。底层是哈希表数据结构。是同步的。不允许null作为键，null作为值。
- `Properties`:用于配置文件的定义和操作，使用频率非常高，同时键和值都是字符串。是集合中可以和IO技术相结合的对象。
- `HashMap`:线程不安全，速度慢。底层也是哈希表数据结构。是不同步的。允许null作为键，null作为值。替代了Hashtable.
- `LinkedHashMap`: 可以保证`HashMap`集合有序。存入的顺序和取出的顺序一致。
- `TreeMap`：可以用来对`Map`集合中的键进行排序.

`Collection`是集合类的上级接口，子接口主要有Set 和List。
`Collections`是针对集合类的一个帮助类，提供了操作集合的工具方法：一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。
>[《介绍Collection框架的结构；Collection 和 Collections的区别》](http://www.cnblogs.com/jinlinFighting/p/5713356.html)

##### 3.集合类框架的最佳实践有哪些
根据应用的需要合理的选择集合的类型对性能非常重要
1.    假如元素的大小是固定的，而且能事先知道，我们就该用Array而不是ArrayList.
2.	有些集合类允许指定初始容量。因此，如果我们能估计出存储元素的数目，我们可以设置初始容量来避免重新计算hash值或者扩容.
3.	为了类型安全，可读性和健壮性的原因总要使用翻新。同时，使用泛型还能皮面运行时的`ClassCastException`.
4.	使用JDK提供的不变类（`immutable class`）作为`Map`的键可以避免为我们自己的类实现`hashCode()`和`equals()`方法。
5.	编程的时候接口优于实现。
6.	底层的集合实际上是空的情况下返回长度是0的集合或者是数组，不要返回null.


##### 4.为什么 Collection 不从 Cloneable 和 Serializable 接口继承?
`Collection`接口指定一组对象，对象即为它的元素。如何维护这些元素由`Collection`的具体实现决定。例如，一些如`List`的`Collection`实现允许重复的元素，而其它的如`Set`就不允许。很多`Collection`实现有一个公有的`clone`方法。然而，把它放到集合的所有实现中也是没有意义的。这是因为`Collection`是一个抽象表现。
重要的是实现，克隆（cloning）或者序列化（serialization）的语义和含义是跟具体的实现相关的。因此应该由**集合类的具体实现类来决定如何被克隆或者序列化**。


##### 5.说出几点 Java 中使用 Collections 的最佳实践？
a）使用正确的集合类，例如，如果不需要同步列表，使用 `ArrayList` 而不是 `Vector`。
b）优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。
c）使用接口代表和访问集合，如使用`List`存储 `ArrayList`，使用 `Map` 存储 `HashMap` 等等。
d）使用迭代器来循环集合。
e）使用集合的时候使用泛型

##### 6.Collections 中 遗留类 (HashTable、Vector) 和 现有类的区别


##### 7.什么是 B+树，B-树，列出实际的使用场景。

