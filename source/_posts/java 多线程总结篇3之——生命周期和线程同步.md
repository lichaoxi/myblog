
---
title: 多线程从入门到放弃【3】
tags: [Interview,java,多线程]
categories: 多线程
copyright: true
---
线程的生命周期全在一张图中，理解此图是基本：
![线程生命状态](http://upload-images.jianshu.io/upload_images/8926909-b9592af555b7ecda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
#### 一、新建和就绪状态
当程序使用`new`关键字创建了一个线程之后，该线程就处于新建状态，此时它和其他的Java对象一样，仅仅由Java虚拟机为其分配内存，并初始化其成员变量的值。此时的线程对象没有表现出任何线程的动态特征，程序也不会执行线程的线程执行体。

当线程对象调用了`start()`方法之后，该线程处于就绪状态。Java虚拟机会为其创建方法调用栈和程序计数器，处于这个状态中的线程并没有开始运行，只是表示该线程可以运行了。至于该线程何时开始运行，取决于JVM里线程调度器的调度。

注意：启动线程使用`start()`方法，而不是`run()`方法。永远不要调用线程对象的`run()`方法。调用`start`方法来启动线程，系统会把该`run()`方法当成线程执行体来处理；但如果直按调用线程对象的`run()`方法，则`run()`方法立即就会被执行，而且在`run()`方法返回之前其他线程无法并发执行。也就是说，系统把线程对象当成一个普通对象，而`run()`方法也是一个普通方法，而不是线程执行体。需要指出的是，调用了线程的`run()`方法之后，该线程已经不再处于新建状态，不要再次调用线程对象的`start()`方法。只能对处于新建状态的线程调用`start()`方法，否则将引发`IllegaIThreadStateExccption`异常。
调用线程对象的`start()`方法之后，该线程立即进入就绪状态——就绪状态相当于"等待执行"，但该线程并未真正进入运行状态。如果希望调用子线程的`start()`方法后子线程立即开始执行，程序可以使用`Thread.sleep(1) `来让当前运行的线程（主线程）睡眠1毫秒，1毫秒就够了，因为在这1毫秒内CPU不会空闲，它会去执行另一个处于就绪状态的线程，这样就可以让子线程立即开始执行。
#### 二、运行和阻塞状态
##### 2.1 线程调度
如果处于就绪状态的线程获得了CPU，开始执行`run()`方法的线程执行体，则该线程处于运行状态，如果计算机只有一个CPU。那么在任何时刻只有一个线程处于运行状态，当然在一个多处理器的机器上，将会有多个线程并行执行；当线程数大于处理器数时，依然会存在多个线程在同一个CPU上轮换的现象。

当一个线程开始运行后，它不可能一直处于运行状态（除非它的线程执行体足够短，瞬间就执行结束了）。线程在运行过程中需要被中断，目的是使其他线程获得执行的机会，线程调度的细节取决于底层平台所采用的策略。对于采用抢占式策略的系统而言，系统会给每个可执行的线程一个小时间段来处理任务；当该时间段用完后，系统就会剥夺该线程所占用的资源，让其他线程获得执行的机会。在选择下一个线程时，系统会考虑线程的优先级。

所有现代的桌面和服务器操作系统都采用抢占式调度策略，但一些小型设备如手机则可能采用协作式调度策略，在这样的系统中，只有当一个线程调用了它的`sleep()`或`yield()`方法后才会放弃所占用的资源——也就是必须由该线程主动放弃所占用的资源。

##### 2.2 线程阻塞
当发生如下情况时，线程将会进入阻塞状态
① 线程调用`sleep()`方法主动放弃所占用的处理器资源
② 线程调用了一个阻塞式IO方法，在该方法返回之前，该线程被阻塞
③ 线程试图获得一个同步监视器，但该同步监视器正被其他线程所持有。关于同步监视器的知识、后面将存更深入的介绍
④ 线程在等待某个通知（notify）
⑤ 程序调用了线程的`suspend()`方法将该线程挂起。但这个方法容易导致死锁，所以应该尽量避免使用该方法

当前正在执行的线程被阻塞之后，其他线程就可以获得执行的机会。被阻塞的线程会在合适的时候重新进入就绪状态，注意是就绪状态而不是运行状态。也就是说，被阻塞线程的阻塞解除后，必须重新等待线程调度器再次调度它。
##### 2.3 解除阻塞
针对上面几种情况，当发生如下特定的情况时可以解除上面的阻塞，让该线程重新进入就绪状态：
① 调用`sleep()`方法的线程经过了指定时间。
② 线程调用的阻塞式IO方法已经返回。
③ 线程成功地获得了试图取得的同步监视器。
④ 线程正在等待某个通知时，其他线程发出了个通知。
⑤ 处于挂起状态的线程被调甩了`resume()`恢复方法。
线程从阻塞状态只能进入就绪状态，无法直接进入运行状态。而就绪和运行状态之间的转换通常不受程序控制，而是由系统线程调度所决定。当处于就绪状态的线程获得处理器资源时，该线程进入运行状态；当处于运行状态的线程失去处理器资源时，该线程进入就绪状态。但有一个方法例外，调用`yield()`方法可以让运行状态的线程转入就绪状态。关于`yield()`方法后面有更详细的介纽。
#### 三、线程死亡
##### 3.1 死亡状态
线程会以如下3种方式结束，结束后就处于死亡状态：
① `run()`或`call()`方法执行完成，线程正常结束。
② 线程抛出一个未捕获的`Exception`或`Error`。
③ 直接调用该线程`stop()`方法来结束该线程——该方法容易导致死锁，通常不推荐使用。
#### 四、重难点考察
##### 1. 有哪些不同的线程生命周期？
当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，它要经过新建(`New`)、就绪（`Runnable`）、运行（`Running`）、阻塞(`Blocked`)和死亡(`Dead`)5种状态。尤其是当线程启动以后，它不可能一直"霸占"着CPU独自运行，所以CPU需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换

**1. 新建状态**，当程序使用new关键字创建了一个线程之后，该线程就处于新建状态，此时仅由JVM为其分配内存，并初始化其成员变量的值

**2. 就绪状态**，当线程对象调用了start()方法之后，该线程处于就绪状态。Java虚拟机会为其创建方法调用栈和程序计数器，等待调度运行

**3. 运行状态**，如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态

**4. 阻塞状态**，当处于运行状态的线程失去所占用资源之后，便进入阻塞状态.

**5. 转换过程**, 在线程的生命周期当中，线程的各种状态的**转换过程**
##### 2.线程状态，BLOCKED 和 WAITING 有什么区别
线程可以通过`wait,join,LockSupport.park`方式进入`wating`状态，进入`wating`状态的线程等待唤醒(`notify`或`notifyAll`)才有机会获取cpu的时间片段来继续执行。线程的 `blocked`状态往往是无法进入同步方法/代码块来完成的。这是因为无法获取到与同步方法/代码块相关联的锁。与`wating`状态相关联的是等待队列，与`blocked`状态相关的是同步队列，一个线程由等待队列迁移到同步队列时，线程状态将会由`wating`转化为`blocked`。可以这样说，`blocked`状态是处于`wating`状态的线程重新焕发生命力的必由之路。

不过个人觉得实际上不用可以区分两者, 因为两者都会暂停线程的执行. 两者的区别是: 进入`waiting`状态是线程主动的, 而进入`blocked`状态是被动的. 更进一步的说, 进入`blocked`状态是在同步(`synchronized`代码之外), 而进入`waiting`状态是在同步代码之内.
##### 3.ThreadLocal用途是什么，原理是什么，用的时候要注意什么？

在多线程程序中，同一个线程在某个时间段只能处理一个任务，我们希望在这个时间段内,任务的某些变量能够和处理它的线程进行绑定，,在任务需要使用这个变量的时候，这个变量能够方便的从线程中取出来。`ThreadLocal`能很好的满足这个需求，用`ThreadLocal`变量的程序看起来也会简洁很多，因为减少了变量在程序中的传递，每个运行的线程都会有一个类型为`ThreadLocal`。`ThreadLocalMap`的`map`，这个`map`就是用来存储与这个线程绑定的变量,`map`的`key`就是`ThreadLocal`对象，`value`就是线程正在执行的任务中的某个变量的包装类`Entry`、在使用`ThreadLocal`对象，尽量使用`static`，不然会使线程的`ThreadLocalMap`产生太多`Entry`，从而造成内存泄露。
##### 4.Java中用到的线程调度算法是什么？

##### 5.什么是多线程中的上下文切换？

即使是单核CPU也支持多线程执行代码，CPU通过给每个线程分配CPU时间片来实现这个机制。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU通过不停地切换线程执行，让我们感觉多个线程时同时执行的，时间片一般是几十毫秒（ms）。CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再次加载这个任务的状态，从任务保存到再加载的过程就是一次上下文切换。

##### 6、你对线程优先级的理解是什么？

每一个线程都是有优先级的, 一般来说, 高优先级的线程在运行时会具有优先权, 但这依赖于线程调度的实现, 这个实现是和操作系统相关的. 我们可以定义线程的优先级`getPriority() setPriority()`, 但是这并不能保证高优先级的线程会在低优先级的线程前执行. 县城优先级是一个`int`变量, `1`代表最低, `10`代表最高。
##### 7、什么是线程调度器 (Thread Scheduler) 和时间分片 (Time Slicing)?

线程调度器是一个操作系统服务, 它负责为`Runnable`状态的线程分配CPU时间, 一旦我们创建一个线程并启动它, 它的执行便依赖于线程调度器的实现。时间分片是指将可用的CPU时间分配给可用的`Runnable`线程的过程。分配的CPU时间可以基于线程的优先级或者线程的等待时间, 线程调度并不受到Java虚拟机控制, 所以由应用程序来控制它是更好的选择(也就是说不要让你的程序依赖于线程优先级)。

#### 二、线程同步
java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查）， 将会导致数据不准确，相互之间产生冲突，因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用， 从而保证了该变量的唯一性和准确性。
##### 1、 请说出你所知的线程同步的方法
**1.同步方法。**即有synchronized关键字修饰的方法。 由于java的每个对象都有一个内置锁，当用此关键字修饰方法时， 内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。代码如：
```java
public synchronized void save(){}
//注：synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类
```
**2.同步代码块**，即有`synchronized`关键字修饰的语句块。 被该关键字修饰的语句块会自动被加上内置锁，从而实现同步
```java
//代码如：
synchronized(object){
}
```
***注：同步是一种高开销的操作，因此应该尽量减少同步的内容。 通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。***
```java
/**
     * 线程同步的运用
     */
    public class SynchronizedThread {

        class Bank {

            private int account = 100;

            public int getAccount() {
                return account;
            }

            /**
             * 用同步方法实现
             * @param money
             */
            public synchronized void save(int money) {
                account += money;
            }

            /**
             * 用同步代码块实现
             * @param money
             */
            public void save1(int money) {
                synchronized (this) {
                    account += money;
                }
            }
        }

        class NewThread implements Runnable {
            private Bank bank;

            public NewThread(Bank bank) {
                this.bank = bank;
            }

            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    // bank.save1(10);
                    bank.save(10);
                    System.out.println(i + "账户余额为：" + bank.getAccount());
                }
            }
        }

        /**
         * 建立线程，调用内部类
         */
        public void useThread() {
            Bank bank = new Bank();
            NewThread new_thread = new NewThread(bank);
            System.out.println("线程1");
            Thread thread1 = new Thread(new_thread);
            thread1.start();
            System.out.println("线程2");
            Thread thread2 = new Thread(new_thread);
            thread2.start();
        }

        public static void main(String[] args) {
            SynchronizedThread st = new SynchronizedThread();
            st.useThread();
        }
    }
```
**3.使用特殊域变量(volatile)实现线程同步**
a.`volatile`关键字为域变量的访问提供了一种免锁机制，
b.使用`volatile`修饰域相当于告诉虚拟机该域可能会被其他线程更新，
c.因此每次使用该域就要重新计算，而不是使用寄存器中的值
d.`volatile`不会提供任何原子操作，它也不能用来修饰final类型的变量
```java
//只给出要修改的代码，其余代码与上同
        class Bank {
            //需要同步的变量加上volatile
            private volatile int account = 100;

            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized
            public void save(int money) {
                account += money;
            }
        ｝
```
***注：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。 用final域，有锁保护的域和volatile域可以避免非同步的问题。***
**4.使用重入锁实现线程同步**，在JavaSE5.0中新增了一个`java.util.concurrent`包来支持同步。 `ReentrantLock`类是可重入、互斥、实现了`Lock`接口的锁， 它与使用`synchronized`方法和快具有相同的基本行为和语义，并且扩展了其能力`ReenreantLock`类的常用方法有：
```java
ReentrantLock() : 创建一个ReentrantLock实例
lock() : 获得锁
unlock() : 释放锁
```
***注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用***
**5.使用局部变量实现线程同步**。如果使用`ThreadLocal`管理变量，则每一个使用该变量的线程都获得该变量的副本， 副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。
`ThreadLocal` 类的常用方法
```java
ThreadLocal() : 创建一个线程本地变量
get() : 返回此线程局部变量的当前线程副本中的值
initialValue() : 返回此线程局部变量的当前线程的"初始值"
set(T value) : 将此线程局部变量的当前线程副本中的值设置为value
```
***注：ThreadLocal与同步机制：a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题。 b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式***
 **6.使用阻塞队列实现线程同步，**7.使用原子变量实现线程同步****

>[《关于线程同步的7种方式》](https://www.cnblogs.com/XHJT/p/3897440.html "java笔记--关于线程同步（7种同步方式）")

##### 2、 synchronized 的原理是什么

`Synchronized`的语义底层是通过一个`monitor`（监视器锁）来实现的。`Synchronized`进过编译，会在同步块的前后分别形成`monitorenter`和`monitorexit`这个两个字节码指令。在执行`monitorenter`指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行`monitorexit`指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。
>monitorenter ：
Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes monitorenter attempts to gain ownership of the monitor associated with objectref, as follows:
• If the entry count of the monitor associated with objectref is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
• If the thread already owns the monitor associated with objectref, it reenters the monitor, incrementing its entry count.
• If another thread already owns the monitor associated with objectref, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.

每个对象有一个监视器锁（`monitor`）。当`monitor`被占用时就会处于锁定状态，线程执行`monitorenter`指令时尝试获取`monitor`的所有权，过程如下：
1、如果`monitor`的进入数为`0`，则该线程进入`monitor`，然后将进入数设置为1，该线程即为`monitor`的所有者。
2、如果线程已经占有该`monitor`，只是重新进入，则进入`monitor`的进入数加1.
3.如果其他线程已经占用了`monitor`，则该线程进入阻塞状态，直到`monitor`的进入数为0，再重新尝试获取`monitor`的所有权。
>monitorexit：　
The thread that executes monitorexit must be the owner of the monitor associated with the instance referenced by objectref.
The thread decrements the entry count of the monitor associated with objectref. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

执行`monitorexit`的线程必须是`objectref`所对应的`monitor`的所有者。指令执行时，`monitor`的进入数减1，如果减1后进入数为0，那线程退出`monitor`，不再是这个`monitor`的所有者。其他被这个`monitor`阻塞的线程可以尝试去获取这个 `monitor` 的所有权。
>[《深入分析Synchronized》](http://blog.csdn.net/shandian000/article/details/54927876) [《Java并发编程之——Synchronized》](https://www.cnblogs.com/paddix/p/5367116.html)

##### 3、 synchronized 和ReentrantLock有什么不同

相似点：这两种同步方式有很多相似之处，它们都是加锁方式同步，而且都是阻塞式的同步，也就是说当如果一个线程获得了对象锁，进入了同步块，其他访问该同步块的线程都必须阻塞在同步块外面等待，而进行线程阻塞和唤醒的代价是比较高的（操作系统需要在用户态与内核态之间来回切换，代价很高，不过可以通过对锁优化进行改善）。

区别：这两种方式最大区别就是对于`Synchronized`来说，它是java语言的关键字，是原生语法层面的互斥，需要jvm实现。而`ReentrantLock`它是JDK 1.5之后提供的API层面的互斥锁，需要`lock()`和`unlock()`方法配合`try/finally`语句块来完成。

总的来说，`Lock`提供了比`synchronized`更多的功能。但是要注意以下几点：

1）`Lock`不是`Java`语言内置的，`synchronized`是Java语言的关键字，因此是内置特性。`Lock`是一个类，通过这个类可以实现同步访问；

2）`Lock`和`synchronized`有一点非常大的不同，采用`synchronized`不需要用户去手动释放锁，当`synchronized`方法或者`synchronized`代码块执行完之后，系统会自动让线程释放对锁的占用；而`Lock`则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
***synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；***

3）`Lock`可以让等待锁的线程响应中断，而`synchronized`却不行，使用`synchronized`时，等待的线程会一直等待下去，不能够响应中断；

4）通过`Lock`可以知道有没有成功获取锁，而`synchronized`却无法办到。

5）`Lock`可以提高多个线程进行读操作的效率。

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时`Lock`的性能要远远优于`synchronized`。所以说，在具体使用时要根据适当情况选择。
>[《java两种线程同步方式的区别——Synchronized和ReentrantLock》](http://blog.csdn.net/chenchaofuck1/article/details/51045134)

##### 4、什么场景下可以使用 volatile 替换 synchronized
只需要保证共享资源的可见性的时候可以使用`volatile`替代，`synchronized`保证可操作的原子性一致性和可见性。` volatile`适用于新值不依赖于旧值的情形

##### 5、有T1，T2，T3三个线程，怎么确保它们按顺序执行？怎样保证T2在T1执行完后执行，T3在T2执行完后执行

这里考察的主要知识点就是线程同步机制和锁的问题，[《Java 指定线程执行顺序（三种方式）》](http://blog.csdn.net/difffate/article/details/63684290)
另外关于执行顺序的问题，可以有很多的考察点，可以刷刷这个博客[《java指定线程执行的顺序》](http://blog.csdn.net/BeauXie/article/details/53018570)

##### 6、同步块内的线程抛出异常会发生什么

这个问题坑了很多Java程序员，若你能想到锁是否释放这条线索来回答还有点希望答对。无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我更喜欢同步块，因为它不用我花费精力去释放锁，该功能可以在`finally block`里释放锁实现。

##### 7、当一个线程进入一个对象的`synchronized` 方法A 之后，其它线程是否可进入此对象的` synchronized` 方法B
不能。其它线程只能访问该对象的非同步方法，同步方法则不能进入。因为非静态方法上的`synchronized`修饰符要求执行方法时要获得对象的锁，如果已经进入A方法说明对象锁已经被取走，那么试图进入B方法的线程就只能在等锁池（注意不是等待池哦 ）中等待对象的锁。


##### 8、使用 synchronized 修饰静态方法和非静态方法有什么区别

在`static`方法前加`synchronizedstatic`：静态方法属于类方法，它属于这个类，获取到的锁，是属于类的锁。 在普通方法前加`synchronizedstatic`：非`static`方法获取到的锁，是属于当前对象的锁。 结论：类锁和对象锁不同，他们之间不会产生互斥。

##### 9、如何从给定集合那里创建一个 synchronized 的集合
我们可以使用`Collections.synchronizedCollection(Collection c)`根据指定集合来获取一个`synchronized`（线程安全的）集合。比如`HashMap`可以这样来实现线程安全：
```java
Map m = Collections.synchronizedMap(new HashMap);
```