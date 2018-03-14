---
title: 多线程从入门到放弃【1】
tags: [Interview,java,多线程]
categories: 多线程
copyright: true
---

##### 1、什么是线程
进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）
线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）
<!--more-->
##### 2、为什么要使用多线程？或者说使用多线程的好处

（1）发挥多核CPU的优势
随着工业的进步，现在的笔记本、台式机乃至商用的应用服务器至少也都是双核的，4核、8核甚至16核的也都不少见，如果是单线程的程序，那么在双核CPU上就浪费了50%，在4核CPU上就浪费了75%。单核CPU上所谓的”多线程”那是假的多线程，同一时间处理器只会处理一段逻辑，只不过线程之间切换得比较快，看着像多个线程”同时”运行罢了。多核CPU上的多线程才是真正的多线程，它能让你的多段逻辑同时工作，多线程，可以真正发挥出多核CPU的优势来，达到充分利用CPU的目的。
（2）防止阻塞
从程序运行效率的角度来看，单核CPU不但不会发挥出多线程的优势，反而会因为在单核CPU上运行多线程导致线程上下文的切换，而降低程序整体的效率。但是单核CPU我们还是要应用多线程，就是为了防止阻塞。试想，如果单核CPU使用单线程，那么只要这个线程阻塞了，比方说远程读取某个数据吧，对端迟迟未返回又没有设置超时时间，那么你的整个程序在数据返回回来之前就停止运行了。多线程可以防止这个问题，多条线程同时运行，哪怕一条线程的代码执行读取数据阻塞，也不会影响其它任务的执行。
（3）便于建模
这是另外一个没有这么明显的优点了。假设有一个大的任务A，单线程编程，那么就要考虑很多，建立整个程序模型比较麻烦。但是如果把这个大的任务A分解成几个小任务，任务B、任务C、任务D，分别
建立程序模型，并通过多线程分别运行这几个任务，那就简单很多了。

##### 3、多线程的几种实现方式？
（1）继承`Thread`类
（2）实现`Runnable`接口
```java
//继承Thread类
class Thread1 extends Thread{
    private String name;
    public Thread1(String name) {
       this.name=name;
    }
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(name + "运行  :  " + i);
            try {
                sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public class Main {
    public static void main(String[] args) {
        Thread1 mTh1=new Thread1("A");
        Thread1 mTh2=new Thread1("B");
        mTh1.start();
        mTh2.start();
    }
}
```
说明：
程序启动运行main时候，java虚拟机启动一个进程，主线程main在`main()`调用时候被创建。随着调用两个对象的`start`方法，另外两个线程也启动了，这样，整个应用就在多线程下运行。

注意：`start()`方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（`Runnable`），什么时候运行是由操作系统决定的。从程序运行的结果可以发现，多线程程序是乱序执行。因此，只有乱序执行的代码才有必要设计为多线程。`Thread.sleep()`方法调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留出一定时间给其他线程执行的机会。实际上所有的多线程代码执行顺序都是不确定的，每次执行的结果都是随机的。

实现接口的方式比继承类的方式更灵活，也能减少程序之间的耦合度，**面向接口编程**也是[设计模式](http://www.amazon.cn/gp/product/B001130JN8/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=importnew-23&linkCode=as2&camp=536&creative=3200&creativeASIN=B001130JN8 "设计模式:可复用面向对象软件的基础")6大原则的核心。
```java
//实现Runnable接口
class Thread2 implements Runnable{
    private String name;
    public Thread2(String name) {
        this.name=name;
    }

    @Override
    public void run() {
          for (int i = 0; i < 5; i++) {
                System.out.println(name + "运行  :  " + i);
                try {
                    Thread.sleep((int) Math.random() * 10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    }
}

public class Main {
    public static void main(String[] args) {
        new Thread(new Thread2("C")).start();
        new Thread(new Thread2("D")).start();
    }
}
```
说明：
`Thread2`类通过实现`Runnable`接口，使得该类有了多线程类的特征。`run（）`方法是多线程程序的一个约定。所有的多线程代码都在run方法里面。`Thread`类实际上也是实现了`Runnable`接口的类。
在启动的多线程的时候，需要先通过`Thread`类的构造方法`Thread(Runnable target) `构造出对象，然后调用`Thread`对象的`start()`方法来运行多线程代码。
实际上所有的多线程代码都是通过运行`Thread`的`start()`方法来运行的。因此，不管是扩展`Thread`类还是实现`Runnable`接口来实现多线程，最终还是通过`Thread`的对象的API来控制线程的，熟悉`Thread`类的API是进行多线程编程的基础。
##### 4、用Thread和Runnable哪种方式更好？区别是什么？
显然是用`Runnable`更好，实现`Runnable`接口比继承`Thread`类所具有的优势：
1）：适合多个相同的程序代码的线程去处理同一个资源
2）：可以避免java中的单继承的限制
3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
4）：线程池只能放入实现`Runable`或`callable`类线程，不能直接放入继承`Thread`的类
##### 5、什么是线程安全？
线程安全就是多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，进行保护，其他线程不能进行访问直到该线程读取完，其他线程才可使用。不会出现数据不一致或者数据污染。线程不安全就是不提供数据访问保护，有可能出现多个线程先后更改数据造成所得到的数据是脏数据
还有一种通俗的解释：如果你的代码在多线程下执行和在单线程下执行永远都能获得一样的结果，那么你的代码就是线程安全的。
这个问题有值得一提的地方，就是线程安全也是有几个级别的：
（1）不可变
像`String、Integer、Long`这些，都是`final`类型的类，任何一个线程都改变不了它们的值，要改变除非新创建一个，因此这些不可变对象不需要任何同步手段就可以直接在多线程环境下使用
（2）绝对线程安全
不管运行时环境如何，调用者都不需要额外的同步措施。要做到这一点通常需要付出许多额外的代价，Java中标注自己是线程安全的类，实际上绝大多数都不是线程安全的，不过绝对线程安全的类，Java中也有，比方说`CopyOnWriteArrayList、CopyOnWriteArraySet`
（3）相对线程安全
相对线程安全也就是我们通常意义上所说的线程安全，像`Vector`这种，`add、remove`方法都是原子操作，不会被打断，但也仅限于此，如果有个线程在遍历某个Vector、有个线程同时在add这个Vector，99%的情况下都会出现`ConcurrentModificationException`，也就是`fail-fast`机制。
（4）线程非安全
这个就没什么好说的了，`ArrayList、LinkedList、HashMap`等都是线程非安全的类.
##### 6、Vector, SimpleDateFormat是线程安全类吗？
`SimpleDateFormate`不是线程安全的，如果我们把`SimpleDateFormat`定义成`static`成员变量，那么多个`thread`之间会共享这个sdf对象， 所以`Calendar`对象也会共享。假定线程A和线程B都进入了`parse(text, pos) `方法， 线程B执行到`calendar.clear()`后，线程A执行到`calendar.getTime()`, 那么就会有问题。

Vector 在方法上使用同步这样做本身没有解决多线程问题，反而，在引入了概念的混乱的同时，导致性能问题，因为 `synchronized` 的开销是巨大的：阻止编译器乱序 。详情请看[《Vector是线程安全吗？》](http://blog.csdn.net/xdonx/article/details/9465489)。看下面Vector代码就知道了：
```java
if (!vector.contains(element))
    vector.add(element);
    ...
}
```
这是经典的 `put-if-absent` 情况，尽管` contains, add `方法都正确地同步了，但作为 `vector `之外的使用环境，仍然存在 ` race condition`: 因为虽然条件判断`if (!vector.contains(element))`与方法调用 `vector.add(element)`;  都是原子性的操作 (atomic)，但在 if 条件判断为真后，那个用来访问`vector.contains `方法的锁已经释放，在即将的 `vector.add `方法调用 之间有间隙，在多线程环境中，完全有可能被其他线程获得 `vector`的 `lock `并改变其状态, 此时当前线程的`vector.add(element)`;  正在等待（只不过我们不知道而已）。只有当其他线程释放了 `vector` 的 `lock` 后，`vector.add(element)`; 继续，但此时它已经基于一个错误的假设了。单个的方法`synchronized` 了并不代表组合`（compound）`的方法调用具有原子性，使 `compound actions`成为线程安全的可能解决办法之一还是离不开`intrinsic lock `(这个锁应该是 `vector `的，但由 `client` 维护)：
```java
 // Vector v = ...
    public  boolean putIfAbsent(E x) {
        synchronized(v) {
            boolean absent = !contains(x);
            if (absent) {
                add(x);
            }
        }
        return absent;
    }
```
所以，正确地回答那个“愚蠢”的问题是：`Vector` 和 `ArrayList` 实现了同一接口 `List`, 但所有的 `Vector` 的方法都具有 `synchronized` 关键修饰。但对于复合操作，`Vector `仍然需要进行同步处理。
##### 7、什么 Java 原型不是线程安全的？

##### 8、哪些集合类是线程安全的？
![map类](http://upload-images.jianshu.io/upload_images/8926909-5a588cedfa1a46ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1.在Map类中，提供两种线程安全容器。
**(1)java.util.Hashtable**
`Hashtable`和`HashMap`类似，都是散列表，存储键值对映射。主要区别在于`Hashtable`是线程安全的。当我们查看`Hashtable`源码的时候，可以看到`Hashtable`的方法都是通过`synchronized`来进行方法层次的同步，以达到线程安全的作用。
**(2)java.util.concurrent.ConcurrentHashMap**
`ConcurrentHashMap`是性能更好的散列表。在兼顾线程安全的同时，相对于`Hashtable`，在效率上有很大的提高。我们可以猜想，`Hashtable`的线程安全实现是对方法进行`synchronized`，很明显可以通过其他并发方式，如`ReentrantLock`进行优化。而`ConcurrentHashMap`正是采用了`ReentrantLock`。运用锁分离技术，即在代码块上加锁，而不是方法上加。同时`ConcurrentHashMap`的一个特色是允许多个修改并发操作。这就有意思了，我们知道一般写都是互斥的，为什么这个还能多个同时写呢？那是因为`ConcurrentHashMap`采用了内部使用段机制，将`ConcurrentHashMap`分成了很多小段。只要不在一个小段上写就可以并发写
![collection类图](http://upload-images.jianshu.io/upload_images/8926909-8416895cf65f47c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.`Collection`部分主要是运用的`CopyOnWrite`机制，即写时复制机制。从字面上就能理解什么意思，就是当我们往一个容器里添加元素的时候，先对这个容器进行一次复制，对副本进行写操作。写操作结束后，将原容器的引用指向新副本容器，就完成了写的刷新。
从它的实现原理，我们可以看出这种机制是存在缺点的。
(1).内存占用：毫无疑问，每次写时需要首先复制一遍原容器，假如复制了很多，或者本身原容器就比较大，那么肯定会占用很多内存。可以采用压缩容器中的元素来防止内存消耗过大。
(2).数据一致性问题：当我们在副本中进行写操组时，只能在最终结束后使数据同步，不能实时同步
可以看到，这种机制适用于读操作多，写操作少的应用场景。
-  **java.util.concurrent.CopyOnWriteArrayList**
`Collection`类的线程安全容器主要都是利用的`ReentrantLock`实现的线程安全，`CopyOnWriteArrayList`也不例外。在并发写的时候，需要获取`lock`。读的时候不需要进行`lock`

-  **java.util.concurrent.CopyOnWriteArraySet**
`CopyOnWriteArraySet`的实现就是基于`CopyOnWriteArrayList`实现的，采用的装饰器进行实现。二者的区别和`List`和`Set`的区别一样。
- **Vector**
一般我们都不用`Vector`了，不过它确实也是线程安全的。相对于其他容器，能够提供随机访问功能。
##### 9、多线程中的忙循环是什么?
忙循环就是程序员用循环让一个线程等待，不像传统方法`wait(), sleep() `或 `yield() `它们都放弃了CPU控制，而忙循环不会放弃CPU，它就是在运行一个空循环。这么做的目的是为了保留CPU缓存，在多核系统中，一个等待线程醒来的时候可能会在另一个内核运行，这样会重建缓存。为了避免重建缓存和减少等待重建的时间就可以使用它了。

##### 10、什么是线程局部变量
`ThreadLocal`，很多地方叫做线程本地变量，也有些地方叫做线程本地存储。变量值的共享可以使用`public static`变量的形式，所有的线程都使用同一个`public static`变量，但是如果每一个线程都有自己的变量该如何共享呢，就是通过`ThreadLocal`，`ThreadLocal`为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。
最常见的`ThreadLocal`使用场景为 用来解决 数据库连接、`Session`管理等。譬如在数据库链接的时候可以这样实现：
```java
class ConnectionManager {
    private  Connection connect = null;
    public Connection openConnection() {
        if(connect == null){
            connect = DriverManager.getConnection();
        }
        return connect;
    }
    public void closeConnection() {
        if(connect!=null)
            connect.close();
    }
}
class Dao{
    public void insert() {
        ConnectionManager connectionManager = new ConnectionManager();
        Connection connection = connectionManager.openConnection();
        //使用connection进行操作
        connectionManager.closeConnection();
    }
}
```
每次都是在方法内部创建的连接，那么线程之间避免了线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不尽严重影响程序执行效率，还可能导致服务器压力巨大。　那么这种情况下使用`ThreadLocal`是再适合不过的了，因为`ThreadLocal`在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。但是要注意，虽然`ThreadLocal`能够解决上面说的问题，但是由于在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用会比不使用`ThreadLocal`要大。
关于`ThreadLocal`的API：
```java
//关于ThreadLocal的API：
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```
`get()`方法是用来获取`ThreadLocal`在当前线程中保存的变量副本，`set()`用来设置当前线程中变量的副本，`remove()`用来移除当前线程中变量的副本，`initialValue()`是一个`protected`方法，一般是用来在使用时进行重写的，它是一个延迟加载方法。这部分可以网上参考：[《Java并发编程：深入剖析ThreadLocal》](https://www.cnblogs.com/dolphin0520/p/3920407.html) ，往上关于此部分知识的有几篇高访问量博文都有争议，切勿照搬，可看评论区。
##### 11、线程和进程有什么区别？进程间如何通讯，线程间如何通讯
进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）
线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）
**一、进程间的通讯**
**管道( pipe )**：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
**有名管道 (namedpipe)**： 有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。
**信号量(semophore )**： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
**消息队列( messagequeue )** ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
**信号**： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
**共享内存(shared memory )**：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。
**套接字(socket )** ： 套解口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。

**二、线程间的通信方式**
**锁机制**：包括互斥锁、条件变量、读写锁
互斥锁提供了以排他方式防止数据结构被并发修改的方法。
读写锁允许多个线程同时读共享数据，而对写操作是互斥的。
条件变量可以以原子的方式阻塞进程，直到某个特定条件为真为止。对条件的测试是在互斥锁的保护下进行的。条件变量始终与互斥锁一起使用。
**信号量机制(Semaphore)**：包括无名线程信号量和命名线程信号量
**信号机制(Signal)**：类似进程间的信号处理
线程间的通信目的主要是用于线程同步，所以线程没有像进程通信中的用于数据交换的通信机制。
**管道( pipe )**：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
**有名管道 (namedpipe)**： 有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。
**信号量(semophore )** ： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
**消息队列( messagequeue )** ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
**信号 (sinal )** ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
**共享内存(shared memory )** ：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。
**套接字(socket )** ： 套解口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。

##### 12、什么是多线程环境下的伪共享（false sharing）
共享就是一个内存区域的数据被多个处理器访问，伪共享就是不是真的共享。这里的共享这个概念是基于逻辑层面的。实际上伪共享与共享在cache line 上实际都是共享的。CPU访问的数据都是从cache line中读取的。如果cpu 在cache 中找不到需要的变量，则称缓存未命中。
未命中时，需要通过总线从内存中读取进cache 中。每次读取的内存大小就是一个cache line 的大小。如果多个CPU访问的不同内存变量被装载到了同一个cache line 中，则从程序逻辑层上讲，并没有共享变量，但实际上在cache line 上他们是共享访问的，这个就是典型的伪共享。伪共享与共享 在 cache line 的层面上必须都是共享的。多个CPU对共享内存的访问安全通过缓存一致性来保证。伪共享问题很难被发现，因为线程可能访问完全不同的全局变量，内存中却碰巧在很相近的位置上。如其他诸多的并发问题，避免伪共享的最基本方式是仔细审查代码，根据缓存行来调整你的数据结构。
##### 13、同步和异步有何异同，在什么情况下分别使用他们？举例说明
如果数据将在线程间共享。例如正在写的数据以后可能被另一个线程读到，或者正在读的数据可能已经被另一个线程写过了，那么这些数据就是共享数据，必须进行同步存取。当应用程序在对象上调用了一个需要花费很长时间来执行的方法，并且不希望让程序等待方法的返回时，就应该使用异步编程，在很多情况下采用异步途径往往更有效率。
Java中交互方式分为同步和异步两种：
- 同步交互：指发送一个请求,需要等待返回,然后才能够发送下一个请求，有个等待过程；
- 异步交互：指发送一个请求,不需要等待返回,随时可以再发送下一个请求，即不需要等待。

区别：一个需要等待，一个不需要等待，在部分情况下，我们的项目开发中都会优先选择不需要等待的异步交互方式。
哪些情况建议使用同步交互呢？比如银行的转账系统，对数据库的保存操作等等，都会使用同步交互操作，其余情况都优先使用异步交互
