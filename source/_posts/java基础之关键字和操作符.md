---
title: java关键字和操作符的总结
tags: [Interview,java,关键字，操作符]
categories: Java Basic
copyright: true
---
#### final,finalize,finally关键字
##### 1.finalize和final关键字
**什么是finalize()方法？finalize()方法什么时候被调用？**
答： Java允许在类中定义一个名为`finalize()`的方法，一旦垃圾回收器准备好释放对象占用的存储空间，将首先调用`finalize()`方法，并且在下一次垃圾回收动作发生时，才会真正回收对象占用的内存。

**析构函数(finalization)的目的是什么**
析构函数目的是撤销对象前、完成一些清理工作，比如释放资源。释放了之后这些资源可以被回收，重新利用。
<!--more-->

**final关键字有哪些用法**
final关键字主要用于修饰类、类成员、方法、以及方法的形参。
- final修饰成员属性：说明该成员属性是常量，不能被修改；
- final修饰类，该类是最终类，不能被继承。
- final修饰方法：该方法是最终方法，不能被重写。
- final关键字修饰形参：1：当形参被修饰为final,那么该形参所属的方法中不能被篡改。

##### 2. final 与 static 关键字可以用于哪里？它们的作用是什么？
用于修饰成员变量和成员方法，可以理解为“全局常量”，对于变量表示一旦给定值就不可以修改，并且通过类名可以访问；对于方法表示不可覆盖，并且可以通过类名直接访问

##### 3. final, finally, finalize的区别（或者说final、finalize 和 finally 的不同之处？）**
final关键字可以用于类，方法，变量前，用来表示该关键字修饰的类，方法，变量具有不可变的特性。
-  final关键字用于基本数据类型前：这时表明该关键字修饰的变量是一个常量，在定义后该变量的值就不能被修改。
-  final关键字用于方法声明前：这时意味着该方法时最终方法，只能被调用，不能被覆盖，但是可以被重载。
-  final关键字用于类名前：此时该类被称为最终类，该类不能被其他类继承。

**`finalize()`**方法来自于`java.lang.Object`，用于回收资源。可以为任何一个类添加finalize方法。finalize方法将在垃圾回收器清除对象之前调用。在实际应用中，**不要依赖使用该方法回收任何短缺的资源，这是因为很难知道这个方法什么时候被调用**。
**finally**，当代码抛出一个异常时，就会终止方法中剩余代码的处理，并退出这个方法的执行。finally块是程序在正常情况下或异常情况下都会运行的。比较适合用于既要处理异常又有资源释放的代码，保证了资源的合理回收。

##### 4. 能否在运行时向 static  final 类型的赋值
不可以，被`static final`修饰的变量只能在被定义的时候或者类的静态代码块中初始化，一旦赋值后就不能在改变了。`static final`相当于类常量，就是在类被加载进内存的时候就要为属性分配内存，`static`块就是类被加载的时候执行且被执行一次，所以可以在其中进行初始化。

##### 5. 使用final关键字修饰一个变量时，是引用不能变，还是引用的对象不能变?
是引用不能变（final引用恒定不变），引用的对象内容还是可以变的

##### 8. throws, throw分别代表什么意义?
`throw`是指的语句抛出一个异常，`throws`指的是声明方法可能抛出的异常类型

**9、Java 有几种修饰符？分别用来修饰什么**
**类的修饰符：**
- `public`可以在其他任何类中使用，默认为统一包下的任意类。
- `abstract`抽象类，不能被实例化，可以包含抽象方法，抽象方法没有被实现，无具体功能，只能衍生子类。
- `final`不能被继承。

**成员变量**
- 访问修饰符：
![访问修饰符](http://upload-images.jianshu.io/upload_images/8926909-16902c873e4ed9d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  `static`类变量：一个类所拥有的变量，不是类的每个实例有的变量。类变量是指不管类创建了多少对象，系统仅在第一次调用类的时候为类变量分配内存，所有对象共享该类的类变量，因此可以通过类本身或者某个对象来访问类变量。
- `final`：常量。
- `volatile`：声明一个可能同时被并存运行的几个线程所控制和修改的变量。
- `abstract`：只有声明部分，方法体为空，具体在子类中完成。
- `transient`：（过度修饰符）指定该变量是系统保留，暂无特别作用的临时性变量。

**方法修饰符：**

- 访问修饰符
`public`（公共控制符）
`private`（私有控制符）指定此方法只能有自己类等方法访问，其他的类不能访问（包括子类）
`protected`（保护访问控制符）指定该方法可以被它的类和子类进行访问。
- `final`，指定该方法不能被重载。
- `static`，指定不需要实例化就可以激活的一个方法。
- `synchronize`，同步修饰符，在多个线程中，该修饰符用于在运行前，对他所属的方法加锁，以防止其他线程的访问，运行结束后解锁。
- `native`，本地修饰符。指定此方法的方法体是用其他语言在程序外部编写的。

>[《java中的访问修饰符》](https://www.cnblogs.com/tjudzj/p/4443066.html)
>[《java中的类修饰符、成员变量修饰符、方法修饰符》](http://www.cnblogs.com/lixiaolun/p/4311727.html)

#### volatile关键字
`volatile`是一个特殊的修饰符，只有成员变量才能使用它。在Java并发程序缺少同步类的情况下，多线程对成员变量的操作对其它线程是透明的。volatile变量可以保证下一个读取操作会在前一个写操作之后发生，就是上一题的volatile变量规则。
原理以及底层实现可参考：
>[《面试必问的 volatile，你了解多少？》](http://www.importnew.com/27863.html)
[《volatile 关键字实现原理》汇编层面的讲解，推荐！](http://www.importnew.com/27002.html)

##### 1、volatile 修饰符的有过什么实践

##### 2、volatile 变量是什么？volatile 变量和 atomic 变量有什么不同
Java语言提供了一种稍弱的同步机制，即`volatile`变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为`volatile`类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。`volatile`变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取`volatile`类型的变量时总会返回最新写入的值。
在访问`volatile`变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此`volatile`变量是一种比`sychronized`关键字更轻量级的同步机制。

##### 3、volatile 类型变量提供什么保证？能使得一个非原子操作变成原子操作吗?
volatile只提供了保证访问该变量时，每次都是从内存中读取最新值，并不会使用寄存器缓存该值——每次都会从内存中读取。而对该变量的修改，`volatile`并不提供原子性的保证。那么编译器究竟是直接修改内存的值，还是使用寄存器修改都符合volatile的定义。所以，一句话，`volatile`并不提供原子性的保证。

##### 4、能创建 volatile 数组吗？
可以，`volatile`修饰的变量如果是对象或数组之类的，其含义是对象获数组的地址具有可见性，但是数组或对象内部的成员改变不具备可见性

##### 5、transient变量有什么特点?
- 一旦变量被`transient`修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
- `transient`关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被`transient`关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。
- 被`transient`关键字修饰的变量不能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

##### 6、super什么时候使用?
super主要存在于子类方法中，用于指向子类对象中父类对象。
1：访问父类的属性
2：访问父类的函数
3：访问父类的构造函数

##### 7、public static void 写成 static public void会怎样?
一样的

##### 8、说明一下public static void main(String args[])这段声明里每个关键字的作用?
主函数是什么：主函数是一个特殊的函数，作为程序的入口，可以被jvm识别。
主函数的定义：
- `public` ：代表该函数的访问权限是最大的。
- `static` ：代表主函数随着类的加载，就已经存在了。
- `void`：  主函数没有具体的返回值
- `main` ： 不是关键字，是一个特殊的单词可以被jvm识别。
- `(String[] args)` 函数的参数，参数类型是一个数组，该数组中的元素是字符串。字符串类型的数组。
	主函数的格式是固定的：jvm能够识别

##### 9、sizeof 是Java 的关键字吗?
不是，C和C++用`sizeof()`解决移植问题，java不需要。

#### static关键字
##### 1、static class 与 non static class的区别
- 内部静态类不需要有指向外部类的引用。但非静态内部类需要持有对外部类的引用。
- 非静态内部类能够访问外部类的静态和非静态成员。静态类不能访问外部类的非静态成员。他只能访问外部类的静态成员。
- 一个非静态内部类不能脱离外部类实体被创建，一个非静态内部类可以访问外部类的数据和方法，因为他就在外部类里面。

生命周期（Lifecycle）：
- 静态方法（Static Method）与静态成员变量一样，属于类本身，**在类装载的时候被装载到内存**（Memory），不自动进行销毁，会一直存在于内存中，直到JVM关闭。
- 非静态方法（Non-Static Method）又叫实例化方法，属于实例对象，实例化后才会分配内存，必须通过类的实例来引用。不会常驻内存，当实例对象被JVM 回收之后，也跟着消失。

效率：静态方法的使用效率比非静态方法的效率高。

线程安全
- 静态方法是共享代码段，静态变量是共享数据段。既然是“共享”就有并发（Concurrence）的问题。
- 非静态方法是针对确定的一个对象的，所以不会存在线程安全的问题。
- 静态方法和实例方法是一样的，在类型第一次被使用时加载。调用的速度基本上没有差别。

##### 2、static 关键字是什么意思？Java中是否可以覆盖(override)一个private或者是static的方法，静态类型有什么特点？
`static`表示静态的意思，可用于修饰成员变量和成员函数，被静态修饰的成员函数只能访问静态成员，不可以访问非静态成员。静态是随着类的加载而加载的，因此可以直接用类进行访问。 重写是子类中的方法和子类继承的父类中的方法一样（函数名，参数，参数类型，反回值类型），但是子类中的访问权限要不低于父类中的访问权限。重写的前提是必须要继承，**`private`修饰不支持继承，因此被私有的方法不可以被重写**。在Java中，如果父类中含有一个静态方法，且在子类中也含有一个返回类型、方法名、参数列表均与之相同的静态方法，那么该**子类实际上只是将父类中的该同名方法进行了隐藏，而非重写**。换句话说，父类和子类中含有的其实是两个没有关系的方法，它们的行为也并不具有多态性。
>[《static方法能否被重写》](http://blog.csdn.net/xiangwanpeng/article/details/52504274?locationNum=12&fps=1)

##### 3、main() 方法为什么必须是静态的？能不能声明 main() 方法为非静态？
用`static`修饰的就是静态方法。静态方法不依靠对象而存在。其直接与类有关，只要包含在类中，就可以得到执行，而不一定依附于对象的存在而执行。因此，`main`方法作为程序的入口方法，在这之前是不可能有任何对象被建立的，也就在`main`之前包括`main`自身不可能是非静态方法。所以`main`方法一定是静态的，有类就行——从而得到执行，进而有更多静态或非静态方法得到执行。

##### 4、是否可以从一个静态（static）方法内部发出对非静态（non-static）方法的调用
不可以，静态函数中不能访问非静态成员变量，只能访问静态变量。因为静态优先于对象存在.静态方法中更不可以出现this

##### 5、静态变量在什么时候加载？编译期还是运行期？静态代码块加载的时机呢？
静态是随着类的加载而加载的，JVM的代码编译运行顺序是编译、类的加载到执行，属于二者的过渡期。静态代码块也是如此。

##### 6、成员方法是否可以访问静态变量？为什么静态方法不能访问成员变量？
成员方法中可以访问静态成员变量。

**请看下面代码来确定程序的打印先后顺序：**
```java
public class test {

    public static void main(String[] args) {
        new test();
    }

    static int num = 4;

    {
        num += 3;
        System.out.println(b);
    }

    int a = 5;

    {
        System.out.println(c);
    }

    test() {System.out.println(d);}

    static {System.out.println(a);}

    static void run() {System.out.println(e);}
}
```
执行顺序如下：
```java
public class test {        //1.第一步，准备加载类

    public static void main(String[] args) {
        new test();        //4.第四步，new一个类，但在new之前要处理匿名代码块
    }

    static int num = 4;    //2.第二步，静态变量和静态代码块的加载顺序由编写先后决定

    {
        num += 3;
        System.out.println(b);//5.第五步，按照顺序加载匿名代码块，代码块中有打印
    }

    int a = 5;                //6.第六步，按照顺序加载变量

    {  成员变量第三个
        System.out.println(c);    //7.第七步，按照顺序打印c
    }
              //如果将构造函数和构造代码块互换，依旧还是先执行构造代码块。
    test() {  //类的构造函数，第四个加载
        System.out.println(d);     //8.第八步，最后加载构造函数，完成对象的建立
    }

    static {             //3.第三步，静态块，然后执行静态代码块，因为有输出，故打印a
        System.out.println(a);
    }

    static void run()              //静态方法，调用的时候才加载 注意看，e没有加载
    {
        System.out.println(e);
    }
}
```
静态块（静态变量）——成员变量——构造方法——静态方法
1、静态代码块（只加载一次） 2、构造方法（创建一个实例就加载一次）3、静态方法需要调用才会执行.

**如果类还没有被加载：**
- 1、先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。
- 2、执行子类的静态代码块和静态变量初始化。
- 3、执行父类的实例变量初始化
- 4、执行父类的构造函数（有构造代码块则先执行构造代码块）
- 5、执行子类的实例变量初始化
- 6、执行子类的构造函数

**如果类已经被加载：**
则静态代码块和静态变量就不用重复执行，再创建类对象时，只执行与实例相关的变量初始化和构造方法。

补充构造代码块：给对象进行初始化。对象一建立就运行并且优先于构造函数。
构造代码块和构造函数的区别，构造代码块是给所有对象进行统一初始化， 构造函数给对应的对象初始化。
#### switch关键字
##### 1、switch 语句中的表达式可以是什么类型数据？
switch(A),括号中A的取值可以是`byte`、`short`、`int`、`char`、`String`，还有枚举类型。
##### 2、switch 是否能作用在byte 上，是否能作用在long 上，是否能作用在String上？
 Java 7之前，switch后面的括号里面只能放int类型的值，注意是只能放int类型，但是放byte，short，char类型的也可以，是因为byte，short，shar可以自动提升（自动类型转换）为int，不是说就可以放它们，说白了，你放的byte，short，shar类型，然后他们会自动转换为int类型（宽化，自动转换并且安全），其实最后放的还是int类型。String可以了，但是long仍然不行。
>**1.**小的往大的转换(宽化)，自动转换，有些时候就会自动提升为大的类型，比如switch中
**2.**大的往小的转换(窄化)必须强制类型转换所以long不行，要想行就得强转如（int）long。同理，float、double也是不行的，要想行就强转。

##### 3、while 循环和 do 循环有什么不同？
`while`语法格式：
```java
while(布尔表达式){
//语句
}
```
先判断布尔表达式，如果为true就会执行循环体中的语句，然后再判断布尔表达式，如果为true就执行循环体中的语句，一直到布尔表达式为false，然后循环结束。通常用算术运算符（++ -- 累减）
`do/while`语法格式：
```java
do{
//语句
}while(布尔表达式);
```
先执行一次循环体，然后在判断布尔表达式是不是true，如果是就继续执行循环体，在判断布尔表达式，直到为false就结束循环。
两者的区别：`while`是先判断在执行如果判断不成立，就不会执行；`do/while`是先执行在判断，不管判断是否成立都会执行一次
#### 操作符
##### 1、&操作符和&&操作符有什么区别
`&&`运算符是短路与运算。逻辑与跟短路与的差别是非常巨大的，虽然二者都要求运算符左右两端的布尔值都是true整个表达式的值才是true。`&&`之所以称为短路运算是因为，如果`&&`左边的表达式的值是false，右边的表达式会被直接短路掉，不会进行运算。很多时候我们可能都需要用`&&`而不是`&`。
#####  2、a = a + b 与 a += b 的区别？
★ =：**赋值运算符**，在编译器将右边的表达式结果计算出来后，和左边的变量类型比较精度，如果左边的变量精度低于右边的结果的精度，编译器会显式的报错，告诉程序员去强制转型。（若a精度类型弱于b，a = a + b出错，编译检查报错）最后将表达式的结果复制到变量所在的内存区。
★ +=：暂且称之为**运算符**，编译器自动隐式直接将+=运算符后面的操作数强制装换为前面变量的类型，然后在变量所在的内存区上直接根据右边的操作数修改左边变量内存存储的二进制数值最后达到和赋值运算符相同的目的。与前者相比，由于后者是位操作，效率也较前者高。

##### 3、30.1 == 0.3 将会返回什么？true 还是 false？
False，类型不一致。

##### 4、float f=3.4; 是否正确？
不正确。`3.4`是双精度数，将双精度型（double）赋值给浮点型（float）属于下转型（down-casting，也称为窄化）会造成精度损失，因此需要强制类型转换`float f =(float)3.4`; 或者写成`float f =3.4F;`。

##### 5、short s1 = 1; s1 = s1 + 1;有错吗?short s1 = 1; s1 += 1;有错吗？
对于`short s1 = 1; s1 = s1 + 1;`由于1是`int`类型，因此`s1+1`运算结果也是`int` 型，需要强制转换类型才能赋值给`short`型。而`short s1 = 1; s1 += 1;`可以正确编译，因为`s1+= 1;`相当于`s1 = (short)(s1 + 1);`其中有隐含的强制类型转换。