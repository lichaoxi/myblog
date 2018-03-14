
---
title: 多线程从入门到放弃【2】
tags: [Interview,java,多线程]
categories: 多线程
copyright: true
---
此片文章主要总结的是Thread类及相关的基础概念和API，首先需要厘清线程调度中的几个基本概念：
<!--more-->
#### 一、线程调度的基本方法
##### 1、调整线程优先级：
Java线程有优先级，优先级高的线程会获得较多的运行机会。Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：
```java
static int MAX_PRIORITY    //线程可以具有的最高优先级，取值为10。

static int MIN_PRIORITY    //线程可以具有的最低优先级，取值为1。

static int NORM_PRIORITY  //分配给线程的默认优先级，取值为5。
```
`Thread`类的`setPriority()`和`getPriority()`方法分别用来设置和获取线程的优先级。每个线程都有默认的优先级。主线程的默认优先级为`Thread.NORM_PRIORITY`。线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。如果希望程序能移植到各个操作系统中，应该仅仅使用`Thread`类有以下三个静态常量作为优先级，这样能保证同样的优先级采用了同样的调度方式。
##### 2、线程睡眠：
`Thread.sleep(long millis)`方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（`Runnable`）状态。`sleep()`平台移植性好。

##### 3、线程等待：
`Object`类中的`wait()`方法，导致当前的线程等待，直到其他线程调用此对象的` notify() `方法或 `notifyAll()` 唤醒方法。这个两个唤醒方法也是`Object`类中的方法，行为等价于调用 `wait(0)` 一样。

##### 4、线程让步：
`Thread.yield() `方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

##### 5、线程加入：
`join()`方法，等待其他线程终止。在当前线程中调用另一个线程的`join()`方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

##### 6、线程唤醒：
`Object`类中的`notify()`方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 `wait `方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个`notifyAll()`，唤醒在此对象监视器上等待的所有线程。

注意：`Thread`中`suspend()`和`resume()`两个方法在JDK1.5中已经废除，不再介绍。因为有死锁倾向。

#### 二、常用函数
**①sleep(long millis)**: 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）
**②join()**:指等待t线程终止。join是`Thread`类的一个方法，启动线程后直接调用，即`join()`的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了`join()`方法后面的代码，只有等到子线程结束了才能执行。
```java
Thread t = new AThread();
t.start();
t.join();
```
为什么要用`join()`方法?在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到`join()`方法了。
```java
public class Main2 {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName()+"主线程运行开始!");
        Thread1 mTh1=new Thread1("A");
        Thread1 mTh2=new Thread1("B");
        mTh1.start();
        mTh2.start();
        try {
            mTh1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            mTh2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");
    }
}
class Thread1 extends Thread{
    private String name;
    public Thread1(String name) {
        super(name);
        this.name=name;
    }
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 线程运行开始!");
        for (int i = 0; i < 5; i++) {
            System.out.println("子线程"+name + "运行 : " + i);
            try {
                sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName() + " 线程运行结束!");
    }
}
```
```java
main主线程运行开始!
A 线程运行开始!
B 线程运行开始!
子线程A运行 : 0
子线程B运行 : 0
子线程A运行 : 1
子线程A运行 : 2
子线程A运行 : 3
子线程A运行 : 4
A 线程运行结束!
子线程B运行 : 1
子线程B运行 : 2
子线程B运行 : 3
子线程B运行 : 4
B 线程运行结束!
main主线程运行结束!
```
**③yield():**暂停当前正在执行的线程对象，并执行其他线程。

`Thread.yield()`方法作用是：暂停当前正在执行的线程对象，并执行其他线程。`yield()`应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用`yield()`的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证`yield()`达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

结论：`yield()`从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，`yield()`将导致线程从运行状态转到可运行状态，但有可能没有效果（注意线程的生命周期）。
```java
public class Main2 {
    public static void main(String[] args) {
        ThreadYield yt1 = new ThreadYield("张三");
        ThreadYield yt2 = new ThreadYield("李四");
        yt1.start();
        yt2.start();
    }
}
class ThreadYield extends Thread{
    public ThreadYield(String name) {
        super(name);
    }
    @Override
    public void run() {
        for (int i = 1; i <= 50; i++) {
            System.out.println("" + this.getName() + "-----" + i);
            // 当i为30时，该线程就会把CPU时间让掉，让其他或者自己的线程执行（也就是谁先抢到谁执行）
            if (i ==30) {
                this.yield();
            }
        }
    }
}
```
```java
运行结果：
第一种情况：李四（线程）当执行到30时会CPU时间让掉，这时张三（线程）抢到CPU时间并执行。
第二种情况：李四（线程）当执行到30时会CPU时间让掉，这时李四（线程）抢到CPU时间并执行。
```
**④setPriority():** 更改线程的优先级。
```java
MIN_PRIORITY = 1
NORM_PRIORITY = 5
MAX_PRIORITY = 10
```

用法：
```java
Thread4 t1 = new Thread4("t1");
Thread4 t2 = new Thread4("t2");
t1.setPriority(Thread.MAX_PRIORITY);
t2.setPriority(Thread.MIN_PRIORITY);
```
**⑤interrupt():**不要以为它是中断某个线程！它只是线线程发送一个中断信号，让线程在无限等待时（如死锁时）能抛出抛出，从而结束线程，但是如果你吃掉了这个异常，那么这个线程还是不会中断的！

**⑥wait()：**`Obj.wait()`，与`Obj.notify()`必须要与`synchronized(Obj)`一起使用，也就是`wait,`与`notify`是针对已经获取了Obj锁进行操作，从语法角度来说就是`Obj.wait(),Obj.notify`必须在`synchronized(Obj){...}`语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的`notify()`唤醒该线程，才能继续获取对象锁，并继续执行。相应的`notify()`就是对对象锁的唤醒操作。但有一点需要注意的是`notify()`调用后，并不是马上就释放对象锁的，而是在相应的`synchronized(){}`语句块执行结束，自动释放锁后，JVM会在`wait()`对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。`Thread.sleep()`与`Object.wait()`二者都可以暂停当前线程，释放CPU控制权，主要的区别在于`Object.wait()`在释放CPU同时，释放了对象锁的控制。

单单在概念上理解清楚了还不够，需要在实际的例子中进行测试才能更好的理解。对`Object.wait()，Object.notify()`的应用最经典的例子，应该是三线程打印ABC的问题了吧，这是一道比较经典的面试题，题目要求如下：建立三个线程，A线程打印10次A，B线程打印10次B,C线程打印10次C，要求线程同时运行，交替打印10次ABC。这个问题用Object的`wait()，notify()`就可以很方便的解决。代码如下：
```java
public class MyThreadPrinter2 implements Runnable {

    private String name;
    private Object prev;
    private Object self;

    private MyThreadPrinter2(String name, Object prev, Object self) {
        this.name = name;
        this.prev = prev;
        this.self = self;
    }

    @Override
    public void run() {
        int count = 10;
        while (count > 0) {
            synchronized (prev) {
                synchronized (self) {
                    System.out.print(name);
                    count--;

                    self.notify();
                }
                try {
                    prev.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();
        MyThreadPrinter2 pa = new MyThreadPrinter2("A", c, a);
        MyThreadPrinter2 pb = new MyThreadPrinter2("B", a, b);
        MyThreadPrinter2 pc = new MyThreadPrinter2("C", b, c);

        new Thread(pa).start();
        Thread.sleep(100);  //确保按顺序A、B、C执行
        new Thread(pb).start();
        Thread.sleep(100);
        new Thread(pc).start();
        Thread.sleep(100);
        }
}
```
先来解释一下其整体思路，从大的方向上来讲，该问题为三线程间的同步唤醒操作，主要的目的就是ThreadA->ThreadB->ThreadC->ThreadA循环执行三个线程。为了控制线程执行的顺序，那么就必须要确定唤醒、等待的顺序，所以每一个线程必须同时持有两个对象锁，才能继续执行。一个对象锁是prev，就是前一个线程所持有的对象锁。还有一个就是自身对象锁。主要的思想就是，为了控制执行的顺序，必须要先持有prev锁，也就前一个线程要释放自身对象锁，再去申请自身对象锁，两者兼备时打印，之后首先调用`self.notify()`释放自身对象锁，唤醒下一个等待线程，再调用`prev.wait()`释放prev对象锁，终止当前线程，等待循环结束后再次被唤醒。运行上述代码，可以发现三个线程循环打印ABC，共10次。程序运行的主要过程就是A线程最先运行，持有C,A对象锁，后释放A,C锁，唤醒B。线程B等待A锁，再申请B锁，后打印B，再释放B，A锁，唤醒C，线程C等待B锁，再申请C锁，后打印C，再释放C,B锁，唤醒A。看起来似乎没什么问题，但如果你仔细想一下，就会发现有问题，就是初始条件，三个线程按照A,B,C的顺序来启动，按照前面的思考，A唤醒B，B唤醒C，C再唤醒A。但是这种假设依赖于JVM中线程调度、执行的顺序。
#### 三、深入考察及疑难点
##### 1、启动一个线程是调用 run() 还是 start() 方法？start() 和 run() 方法有什么区别
相当于玩游戏机，只有一个游戏机（cpu），可是有很多人要玩，于是，start是排队！等CPU选中你就是轮到你，你就`run（）`，当CPU的运行的时间片执行完，这个线程就继续排队，等待下一次的`run（）`。调用`start（）`后，线程会被放到等待队列，等待CPU调度，并不一定要马上开始执行，只是将这个线程置于可动行状态。然后通过JVM，线程Thread会调用`run（）`方法，执行本线程的线程体。先调用start后调用run，这么麻烦，为了不直接调用run？就是为了实现多线程的优点，没这个start不行。

**（1）.start（）**方法来启动线程，真正实现了多线程运行。这时无需等待run方法体代码执行完毕，可以直接继续执行下面的代码；通过调用Thread类的start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。 然后通过此Thread类调用方法run()来完成其运行操作的， 这里方法run()称为线程体，它包含了要执行的这个线程的内容， Run方法运行结束， 此线程终止。然后CPU再调度其它线程。

**（2）.run（）**方法当作普通方法的方式调用。程序还是要顺序执行，要等待run方法体执行完毕后，才可继续执行下面的代码； 程序中只有主线程——这一个线程， 其程序执行路径还是只有一条， 这样就没有达到写线程的目的。
记住：多线程就是分时利用CPU，宏观上让所有线程一起执行 ，也叫并发

**start() :**它的作用是启动一个新线程，新线程处于就绪状态，拿到cpu执行权就会执行相应的run()方法。start()不能被重复调用。
**run():** run()就和普通的成员方法一样，可以被重复调用。单独调用run()的话，会在当前线程中执行run()，而并不会启动新线程！（为什么不直接调用run方法的原因也在此。）
##### 2、sleep() 方法和对象的 wait() 方法都可以让线程暂停执行，它们有什么区别？
**sleep（）方法**
`sleep()`使当前线程进入停滞状态（阻塞当前线程），让出CUP的使用、目的是不让当前线程独自霸占该进程所获的CPU资源，以留一定时间给其他线程执行的机会;`sleep()`是`Thread`类的`Static`(静态)的方法；因此他不能改变对象的机锁，所以当在一个`Synchronized`块中调用`Sleep()`方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。在`sleep()`休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。

**wait（）方法**
`wait()`方法是Object类里的方法；当一个线程执行到`wait()`方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，`wait(long timeout)`超时时间到后还需要返还对象锁）；其他线程可以访问；`wait()`使用`notify`或者`notifyAlll`或者指定睡眠时间来唤醒当前等待池中的线程。`wiat()`必须放在`synchronized block`中，否则会在`program runtime`时扔出”`java.lang.IllegalMonitorStateException`“异常。
共同点：

1. 他们都是在多线程的环境下，都可以在程序的调用处阻塞指定的毫秒数，并返回。

2. `wait()`和`sleep()`都可以通过`interrupt()`方法 打断线程的暂停状态，从而使线程立刻抛出`InterruptedException`。

   如果线程A希望立即结束线程B，则可以对线程B对应的`Thread`实例调用`interrupt`方法。如果此刻线程B正在`wait/sleep /join`，则线程B会立刻抛出`InterruptedException`，在`catch() {}` 中直接`return`即可安全地结束线程。 需要注意的是，`InterruptedException`是线程自己从内部抛出的，并不是`interrupt()`方法抛出的。对某一线程调用 `interrupt()`时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出`InterruptedException`。但是，一旦该线程进入到 `wait()/sleep()/join()`后，就会立刻抛出`InterruptedException` 。

不同点：
1. `Thread`类的方法：`sleep(),yield()`等。`Object`的方法：`wait()`和`notify()`等

2. 每个对象都有一个锁来控制同步访问。`Synchronized`关键字可以和对象的锁交互，来实现线程的同步。`sleep`方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。

3. `wait，notify`和`notifyAll`只能在同步控制方法或者同步控制块里面使用，而`sleep`可以在任何地方使用

4. `sleep`必须捕获异常，而`wait，notify`和`notifyAll`不需要捕获异常

所以`sleep()`和`wait()`方法的最大区别是：`sleep()`睡眠时，保持对象锁，仍然占有该锁；而`wait()`睡眠时，释放对象锁。但是`wait()`和`sleep()`都可以通过`interrupt()`方法打断线程的暂停状态，从而使线程立刻抛出`InterruptedException`（但不建议使用该方法）。
##### 3、yield方法有什么作用？sleep() 方法和 yield()方法有什么区别?

**sleep()和yield()**的区别:`sleep()`使当前线程进入停滞状态，所以执行`sleep()`的线程在指定的时间内肯定不会被执行；`yield()`只是使当前线程重新回到可执行状态，所以执行`yield()`的线程有可能在进入到可执行状态后马上又被执行。

`sleep` 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，`yield` 方法使当前线程让出 CPU 占有权，但让出的时间是不可设定的。实际上，`yield()`方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU  的占有权交给此线程，否则，继续运行原来的线程。所以`yield()`方法称为“退让”，它把运行机会让给了同等优先级的其他线程

另外，sleep 方法允许较低优先级的线程获得运行机会，但 `yield() ` 方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。在一个运行系统中，如果较高优先级的线程没有调用 sleep 方法，又没有受到 I\O 阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。

##### 4、Java 中如何停止一个线程？**
停止一个线程意味着在任务处理完任务之前停掉正在做的操作，也就是放弃当前的操作。停止一个线程可以用`Thread.stop()`方法，但最好不要用它。虽然它确实可以停止一个正在运行的线程，但是这个方法是不安全的，而且是已被废弃的方法。在java中有以下3种方法可以终止正在运行的线程：
1.使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2.使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。
3.使用interrupt方法中断线程。具体案例和api可以参考文章[《java多线程之——interrupt深入研究》](https://www.cnblogs.com/carmanloneliness/p/3516405.html)。

##### 5、stop()和 suspend()方法为何不推荐使用?

`Stop()`方法作为一种粗暴的线程终止行为，在线程终止之前没有对其做任何的清除操作，因此具有固有的不安全性。 用`Thread.stop()`方法来终止线程将会释放该线程对象已经锁定的所有监视器。如果以前受这些监视器保护的任何对象都处于不连贯状态，那么损坏的对象对其他线程可见，这有可能导致不安全的操作。 由于上述原因，因此不应该使用`stop()`方法，而应该在自己的`Thread`类中置入一个标志，用于控制目标线程是活动还是停止。如果该标志指示它要停止运行，可使其结束`run（）`方法。如果目标线程等待很长时间，则应使用`interrupt()`方法来中断该等待。

`suspend()`方法 该方法已经遭到反对，因为它具有固有的死锁倾向。调用`suspend（）`方法的时候，目标线程会停下来。如果目标线程挂起时在保护关键系统资源的监视器上保持有锁，则在目标线程重新开始以前，其他线程都不能访问该资源。除非被挂起的线程恢复运行。对任何其他线程来说，如果想恢复目标线程，同时又试图使用任何一个锁定的资源，就会造成死锁。由于上述原因，因此不应该使用`suspend（）`方法，而应在自己的thread类中置入一个标志，用于控制线程是活动还是挂起。如果标志指出线程应该挂起，那么用`wait（）`方法命令其进入等待状态。如果标志指出线程应当恢复，那么用`notify()`方法重新启动线程。
##### 6、如何在两个线程间共享数据?

通过在线程之间共享对象就可以了，然后通过`wait/notify/notifyAll、await/signal/signalAll`进行唤起和等待，比方说阻塞队列`BlockingQueue`就是为线程之间共享数据而设计的。

##### 7、强制启动一个线程？
- 实现Runnable接口优势：
1）适合多个相同的程序代码的线程去处理同一个资源。
2）可以避免java中的单继承的限制。
3）增加程序的健壮性，代码可以被多个线程共享，代码和数据独立。
- 继承Thread类优势：
1）可以将线程类抽象出来，当需要使用抽象工厂模式设计时。
2）多线程同步
- 在函数体使用
1）无需继承thread或者实现Runnable，缩小作用域。
##### 8、如何让正在运行的线程暂停一段时间
1、`sleep()`方法（休眠）是线程类（`Thread`）的静态方法，调用此方法会让当前线程暂停执行指定的时间，将执行机会（CPU）让给其他线程，但是对象的锁依然保持，因此休眠时间结束后会自动恢复（线程回到就绪状态，请参考第66题中的线程状态转换图）。
2、`wait()`是Object类的方法，调用对象的wait()方法导致当前线程放弃对象的锁（线程暂停执行），进入对象的等待池（wait pool），只有调用对象的`notify()`方法（或`notifyAll()`方法）时才能唤醒等待池中的线程进入等锁池（lockpool），如果线程重新获得对象的锁就可以进入就绪状态
##### 9、什么是线程组，为什么在Java中不推荐使用？
`ThreadGroup`是一个类, 它的目的是提供关于线程组的信息.
`ThreadGroup `API比较薄弱, 它并没有为`Thread`提供了更多的功能. 它主要有两个功能: 一是获取线程组中处于活跃状态线程的列表; 二是甚至为线程设置未捕获异常处理器(`uncaught exception handler`) . 但在java 1.5中Thread类也添加了`setUncaughtExceptionHandler(UncaughtExceptionHandler eh)`方法, 所以`ThreadGroup`是已经过时的, 不建议使用.
##### 10、你是如何调用 wait（方法的）？使用 if 块还是循环？为什么

`wait()` 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。