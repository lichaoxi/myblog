---
title: 多线程从入门到放弃【4】
tags: [Interview,java,多线程]
categories: 多线程
copyright: true
---
在开发Java多线程应用程序中，各个线程之间由于要共享资源，必须用到锁机制。Java提供了多种多线程锁机制的实现方式，常见的有·synchronized、ReentrantLock、Semaphore、AtomicInteger·等。每种机制都有优缺点与各自的适用场景，必须熟练掌握他们的特点才能在Java多线程应用开发时得心应手。——[《Java锁机制详解》](https://www.cnblogs.com/hanganglin/p/3577096.html)。
<!--more-->
线程同步有关的类图关系可用以下的图总结：
![线程同步的类图](http://upload-images.jianshu.io/upload_images/8926909-8ffafbe023d370cf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 1、Java Concurrency API 中的 Lock接口是什么？对比同步它有什么优势？
`Lock`接口比同步方法和同步块（这里的同步就是考察`Synchronized`关键字）提供了更具扩展性的锁操作。
如果一个代码块被`synchronized`修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：
- 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
- 线程执行发生异常，此时JVM会让线程自动释放锁。

那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能阻塞等待，非常影响效率。因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过`Lock`就可以办到。

再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。
但是采用synchronized关键字来实现同步的话，就会导致一个问题：如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过`Lock`就可以办到。

`Lock`不是Java语言内置的，`synchronized`是Java语言的关键字，因此是内置特性，`Lock`是一个类，通过这个类可以实现同步访问；他们允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。它的优势有：可以使锁更公平；可以使线程在等待锁的时候响应中断；可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间；可以在不同的范围，以不同的顺序获取和释放锁。

关于API及代码的例子请移步：[《java并发编程Lock》](https://www.cnblogs.com/dolphin0520/p/3923167.html)。常用接口方法如下：
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```
首先`lock()`方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。由于在前面讲到如果采用`Lock`，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用`Lock`必须在`try{}catch{}`块中进行，并且将释放锁的操作放在`finally`块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用`Lock`来进行同步的话，是以下面这种形式去使用的：
```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```
`tryLock()`方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回`true`，如果获取失败（即锁已被其他线程获取），则返回`false`，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。`tryLock(long time, TimeUnit unit)`方法和`tryLock()`方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回`false`。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回`true`。
```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     }
}else {
    //如果不能获取锁，则直接做其他事情
}
```
`lockInterruptibly()`方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过`lock.lockInterruptibly()`想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用`threadB.interrupt()`方法能够中断线程B的等待过程。由于`lockInterruptibly()`的声明中抛出了异常，所以`lock.lockInterruptibly()`必须放在try块中或者在调用`lockInterruptibly()`的方法外声明抛出`InterruptedException`。
```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {
     //.....
    }
    finally {
        lock.unlock();
    }
```
`ReentrantLock`，意思是“可重入锁”，`ReentrantLock`是唯一实现了`Lock`接口的类，并且`ReentrantLock`提供了更多的方法。以下给出一个`ReentrantLock`的运行实例：
```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();    //注意这个地方，声明为类的属性
    public static void main(String[] args)  {
        final Test test = new Test();

        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         //可以用Java箭头函数特性改写上述冗余代码：
         // new Thread(){()->Thread.currentThread}.start();
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }

    public void insert(Thread thread) {
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
```
上文中提到了Lock接口以及对象，使用它，很优雅的控制了竞争资源的安全访问，但是这种锁不区分读写，称这种锁为普通锁。为了提高性能，Java提供了读写锁，在读的地方使用读锁，在写的地方使用写锁，灵活控制，如果没有写锁的情况下，读是无阻塞的,在一定程度上提高了程序的执行效率。Java中读写锁有个接口`java.util.concurrent.locks. ReadWriteLock`，也有具体的实现`ReentrantReadWriteLock`，因而会有下面的提问：
##### 2、ReadWriteLock是什么？
当有写线程时，则写线程独占同步状态，当没有写线程时只有读线程时，则多个读线程可以共享同步状态。读写锁就是为了实现这种效果而生。

读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，我们只要上好相应的锁即可。如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！读写锁接口：`ReadWriteLock`，它的具体实现类为：`ReentrantReadWriteLock`。
[《ReadWriteLock场景应用》](https://www.cnblogs.com/liang1101/p/6475555.html?utm_source=itdadao&utm_medium=referral)：在多线程的环境下，对同一份数据进行读写，会涉及到线程安全的问题。比如在一个线程读取数据的时候，另外一个线程在写数据，而导致前后数据的不一致性；一个线程在写数据的时候，另一个线程也在写，同样也会导致线程前后看到的数据的不一致性。这时候可以在读写方法中加入互斥锁，任何时候只能允许一个线程的一个读或写操作，而不允许其他线程的读或写操作，这样是可以解决这样以上的问题，但是效率却大打折扣了。因为在真实的业务场景中，一份数据，读取数据的操作次数通常高于写入数据的操作，而线程与线程间的读读操作是不涉及到线程安全的问题，没有必要加入互斥锁，只要在读-写，写-写期间上锁就行了。[API调用请移步](https://www.cnblogs.com/sheeva/p/6480116.html)

构造了一个线程安全的缓存，先创建一个`ReentrantReadWriteLock `对象，构造函数 `false` 代表是非公平的（非公平的含义和ReentrantLock相同）。然后通过`readLock`、`writeLock`方法分别获取读锁和写锁。在做读操作的时候，也就是`get`方法，我们要先获取读锁；在做写操作的时候，即put方法，我们要先获取写锁。
```java
public class ReadWriteCache {
    private static Map<String, Object> data = new HashMap<>();
    private static ReadWriteLock lock = new ReentrantReadWriteLock(false);
    private static Lock rlock = lock.readLock();
    private static Lock wlock = lock.writeLock();

    public static Object get(String key) {
        rlock.lock();
        try {
            return data.get(key);
        } finally {
            rlock.unlock();
        }
    }

    public static Object put(String key, Object value) {
        wlock.lock();
        try {
            return data.put(key, value);
        } finally {
            wlock.unlock();
        }
    }
}
```

##### 3、锁机制有什么用

有些业务逻辑在执行过程中要求对数据进行排他性的访问，于是需要通过一些机制保证在此过程中数据被锁住不会被外界修改，这就是所谓的锁机制。

##### 4、什么是乐观锁（Optimistic Locking）？如何实现乐观锁？如何避免ABA问题

悲观锁(`Pessimistic Lock`), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

乐观锁(`Optimistic Lock`), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库如果提供类似于`write_condition`机制的其实都是提供的乐观锁。

##### 5、解释以下名词：重排序，自旋锁，偏向锁，轻量级锁，可重入锁，公平锁，非公平锁，乐观锁，悲观锁

**重入锁（ReentrantLock）**是一种递归无阻塞的同步机制。重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。在JAVA环境下 `ReentrantLock` 和`synchronized `都是 可重入锁。

**自旋锁**，由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率远高于互斥锁。如何旋转呢？何为自旋锁，就是如果发现锁定了，不是睡眠等待，而是采用让当前线程不停地的在循环体内执行实现的，当循环的条件被其他线程改变时 才能进入临界区。

**偏向锁(Biased Locking)**是Java6引入的一项多线程优化，它会偏向于第一个访问锁的线程，如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，则线程是不需要触发同步的，这种情况下，就会给线程加一个偏向锁。 如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，将锁恢复到标准的轻量级锁。

**轻量级锁**是由偏向所升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁。

**重入锁（ReentrantLock）**是一种递归无阻塞的同步机制，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。 在JAVA环境下 `ReentrantLock` 和`synchronized `都是 可重入锁。

**公平锁**，就是很公平，在并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程线程是等待队列的第一个，就占有锁，否则就会加入到等待队列中，以后会按照FIFO的规则从队列中取到自己

**非公平锁**比较粗鲁，上来就直接尝试占有锁。在公平的锁上，线程按照他们发出请求的顺序获取锁，但在非公平锁上，则允许‘插队’：当一个线程请求非公平锁时，如果在发出请求的同时该锁变成可用状态，那么这个线程会跳过队列中所有的等待线程而获得锁。     非公平的ReentrantLock 并不提倡 插队行为，但是无法防止某个线程在合适的时候进行插队。
##### 6、什么时候应该使用可重入锁？

场景1：如果已加锁，则不再重复加锁。a、忽略重复加锁。b、用在界面交互时点击执行较长时间请求操作时，防止多次点击导致后台重复执行（忽略重复触发）。以上两种情况多用于进行非重要任务防止重复执行，（如：清除无用临时文件，检查某些资源的可用性，数据备份操作等）

场景2：如果发现该操作已经在执行，则尝试等待一段时间，等待超时则不执行（尝试等待执行）这种其实属于场景2的改进，等待获得锁的操作有一个时间的限制，如果超时则放弃执行。用来防止由于资源处理不当长时间占用导致死锁情况（大家都在等待资源，导致线程队列溢出）。

场景3：如果发现该操作已经加锁，则等待一个一个加锁（同步执行，类似`synchronized`）这种比较常见大家也都在用，主要是防止资源使用冲突，保证同一时间内只有一个操作可以使用该资源。但与synchronized的明显区别是性能优势（伴随jvm的优化这个差距在减小）。同时Lock有更灵活的锁定方式，公平锁与不公平锁，而synchronized永远是公平的。这种情况主要用于对资源的争抢（如：文件操作，同步消息发送，有状态的操作等）

场景4：可中断锁。`synchronized`与`Lock`在默认情况下是不会响应中断(interrupt)操作，会继续执行完。`lockInterruptibly()`提供了可中断锁来解决此问题。（场景3的另一种改进，没有超时，只能等待中断或执行完毕）这种情况主要用于取消某些操作对资源的占用。如：（取消正在同步运行的操作，来防止不正常操作长时间占用造成的阻塞）
##### 7、简述锁的等级方法锁、对象锁、类锁

方法锁（`synchronized`修饰方法时）通过在方法声明中加入 `synchronized`关键字来声明 `synchronized `方法。`synchronized `方法控制对类成员变量的访问： 每个类实例对应一把锁，每个` synchronized `方法都必须获得调用该方法的类实例的锁方能执行，否则所属线程阻塞，方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。这种机制确保了同一时刻对于每一个类实例，其所有声明为 `synchronized `的成员函数中至多只有一个处于可执行状态，从而有效避免了类成员变量的访问冲突。

对象锁（`synchronized`修饰方法或代码块）当一个对象中有`synchronized method`或`synchronized block`的时候调用此对象的同步方法或进入其同步区域时，就必须先获得对象锁。如果此对象的对象锁已被其他调用者占用，则需要等待此锁被释放。（方法锁也是对象锁）。java的所有对象都含有1个互斥锁，这个锁由JVM自动获取和释放。线程进入`synchronized`方法的时候获取该对象的锁，当然如果已经有线程获取了这个对象的锁，那么当前线程会等待；`synchronized`方法正常返回或者抛异常而终止，JVM会自动释放对象锁。这里也体现了用`synchronized`来加锁的1个好处，方法抛异常的时候，锁仍然可以由JVM来自动释放。　

类锁(`synchronized`修饰静态的方法或代码块)，由于一个`class`不论被实例化多少次，其中的静态方法和静态变量在内存中都只有一份。所以，一旦一个静态的方法被申明为`synchronized`。此类所有的实例化对象在调用此方法，共用同一把锁，我们称之为类锁。对象锁是用来控制实例方法之间的同步，类锁是用来控制静态方法（或静态变量互斥体）之间的同步。类锁只是一个概念上的东西，并不是真实存在的，它只是用来帮助我们理解锁定实例方法和静态方法的区别的。java类可能会有很多个对象，但是只有1个`Class`对象，也就是说类的不同实例之间共享该类的`Class`对象。`Class`对象其实也仅仅是1个java对象，只不过有点特殊而已。由于每个java对象都有1个互斥锁，而类的静态方法是需要`Class`对象。所以所谓的类锁，不过是`Class`对象的锁而已。获取类的`Class`对象有好几种，最简单的就是［类名.class］的方式。
##### 8、Java中活锁和死锁有什么区别？

死锁：是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。死锁发生的四个条件

1、互斥条件：线程对资源的访问是排他性的，如果一个线程对占用了某资源，那么其他线程必须处于等待状态，直到资源被释放。

2、请求和保持条件：线程T1至少已经保持了一个资源R1占用,但又提出对另一个资源R2请求，而此时，资源R2被其他线程T2占用，于是该线程T1也必须等待，但又对自己保持的资源R1不释放。

3、不剥夺条件：线程已获得的资源，在未使用完之前，不能被其他线程剥夺，只能在使用完以后由自己释放。

4、环路等待条件：在死锁发生时，必然存在一个“进程-资源环形链”，即：`{p0,p1,p2,...pn}`,进程p0（或线程）等待p1占用的资源，p1等待p2占用的资源，pn等待p0占用的资源。（最直观的理解是，p0等待p1占用的资源，而p1而在等待p0占用的资源，于是两个进程就相互等待）

活锁：是指线程1可以使用资源，但它很礼貌，让其他线程先使用资源，线程2也可以使用资源，但它很绅士，也让其他线程先使用资源。这样你让我，我让你，最后两个线程都无法使用资源。

##### 9、如何确保 N 个线程可以访问 N 个资源同时又不导致死锁？

预防死锁，预先破坏产生死锁的四个条件。互斥不可能破坏，所以有如下3种方法：

1.破坏，请求和保持条件1.1）进程等所有要请求的资源都空闲时才能申请资源，这种方法会使资源严重浪费（有些资源可能仅在运行初期或结束时才使用，甚至根本不使用）1.2）允许进程获取初期所需资源后，便开始运行，运行过程中再逐步释放自己占有的资源。比如有一个进程的任务是把数据复制到磁盘中再打印，前期只需要获得磁盘资源而不需要获得打印机资源，待复制完毕后再释放掉磁盘资源。这种方法比上一种好，会使资源利用率上升。

2.破坏，不可抢占条件。这种方法代价大，实现复杂

3.破坏，循坏等待条件。对各进程请求资源的顺序做一个规定，避免相互等待。这种方法对资源的利用率比前两种都高，但是前期要为设备指定序号，新设备加入会有一个问题，其次对用户编程也有限制

##### 10、死锁与饥饿的区别？

相同点：二者都是由于竞争资源而引起的。

不同点：
- 从进程状态考虑，死锁进程都处于等待状态，忙等待(处于运行或就绪状态)的进程并非处于等待状态，但却可能被饿死；
- 死锁进程等待永远不会被释放的资源，饿死进程等待会被释放但却不会分配给自己的资源，表现为等待时限没有上界(排队等待或忙式等待)；
- 死锁一定发生了循环等待，而饿死则不然。这也表明通过资源分配图可以检测死锁存在与否，但却不能检测是否有进程饿死；
- 死锁一定涉及多个进程，而饥饿或被饿死的进程可能只有一个。
- 在饥饿的情形下，系统中有至少一个进程能正常运行，只是饥饿进程得不到执行机会。而死锁则可能会最终使整个系统陷入死锁并崩溃
##### 11、怎么检测一个线程是否拥有锁？
`java.lang.Thread`中有一个方法叫`holdsLock()`，它返回true如果当且仅当当前线程拥有某个具体对象的锁
```java
Object o = new Object();

@Test
public void test1() throws Exception {

    new Thread(new Runnable() {

        @Override
        public void run() {
            synchronized(o) {
                System.out.println("child thread: holdLock: " +
                    Thread.holdsLock(o));
            }
        }
    }).start();

    System.out.println("main thread: holdLock: " + Thread.holdsLock(o));
    Thread.sleep(2000);

}
```
```java
main thread: holdLock: false
child thread: holdLock: true
```
##### 12、如何实现分布式锁？
基于数据库实现分布式锁
基于缓存（redis，memcached，tair）实现分布式锁
基于Zookeeper实现分布式锁
可以参考详情[《分布式锁的几种实现方式》](http://www.hollischuang.com/archives/1716) 、 [《分布式锁的3种方式》](https://www.cnblogs.com/rwxwsblog/p/6046034.html)
##### 13、有哪些无锁数据结构，他们实现的原理是什么？

java 1.5提供了一种无锁队列（`wait-free/lock-free`）`ConcurrentLinkedQueue`，可支持多个生产者多个消费者线程的环境：网上别人自己实现的一种[无锁算法队列](http://aigo.iteye.com/blog/2292229)，原理和jdk官方的`ConcurrentLinkedQueue`相似：通过`volatile`关键字来保证数据唯一性（注：java的volatile和c++的volatile关键字是两码事！），但是里面又用到atomic，感觉有点boost::lockfree::queue的风格，估计参考了boost的代码来编写这个java无锁队列。
##### 14、Executors类是什么？ Executor和Executors的区别

正如上面所说，这三者均是 Executor 框架中的一部分。Java 开发者很有必要学习和理解他们，以便更高效的使用 Java 提供的不同类型的线程池。总结一下这三者间的区别，以便大家更好的理解：

`Executor `和 `ExecutorService` 这两个接口主要的区别是：`ExecutorService` 接口继承了` Executor` 接口，是` Executor `的子接口
`Executor `和` ExecutorService` 第二个区别是：`Executor `接口定义了 `execute()`方法用来接收一个`Runnable`接口的对象，而 `ExecutorService `接口中的 `submit()`方法可以接受`Runnable`和`Callable`接口的对象。
`Executor` 和 `ExecutorService` 接口第三个区别是` Executor `中的 `execute() `方法不返回任何结果，而 `ExecutorService `中的 `submit()`方法可以通过一个 `Future `对象返回运算结果。
`Executor` 和 `ExecutorService` 接口第四个区别是除了允许客户端提交一个任务，`ExecutorService` 还提供用来控制线程池的方法。比如：调用 `shutDown() `方法终止线程池。可以通过 《Java Concurrency in Practice》 一书了解更多关于关闭线程池和如何处理 `pending` 的任务的知识。
`Executors` 类提供工厂方法用来创建不同类型的线程池。比如: `newSingleThreadExecutor()` 创建一个只有一个线程的线程池，`newFixedThreadPool(int numOfThreads)`来创建固定线程数的线程池，`newCachedThreadPool()`可以根据需要创建新的线程，但如果已有线程是空闲的会重用已有线程。

Executor	ExecutorService
Executor 是 Java 线程池的核心接口，用来并发执行提交的任务	ExecutorService 是 Executor 接口的扩展，提供了异步执行和关闭线程池的方法
提供execute()方法用来提交任务	提供submit()方法用来提交任务
execute()方法无返回值	submit()方法返回Future对象，可用来获取任务执行结果
不能取消任务	可以通过Future.cancel()取消pending中的任务
没有提供和关闭线程池有关的方法	提供了关闭线程池的方法
![区别](http://upload-images.jianshu.io/upload_images/8926909-d2a25c310db90f5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 16、什么是Java线程转储(Thread Dump)，如何得到它？

线程转储是一个JVM活动线程的列表，它对于分析系统瓶颈和死锁非常有用。
有很多方法可以获取线程转储——使用Profiler，Kill -3命令，jstack工具等等。我更喜欢jstack工具，因为它容易使用并且是JDK自带的。由于它是一个基于终端的工具，所以我们可以编写一些脚本去定时的产生线程转储以待分析。
##### 17、如何在Java中获取线程堆栈？
Java虚拟机提供了线程转储（thread dump）的后门，通过这个后门可以把线程堆栈打印出来。通常我们将堆栈信息重定向到一个文件中，便于我们分析，由于信息量太大，很可能超出控制台缓冲区的最大行数限制造成信息丢失。这里介绍一个jdk自带的打印线程堆栈的工具，jstack用于打印出给定的Java进程ID或core file或远程调试服务的Java堆栈信息。（[Java问题定位之Java线程堆栈分析](http://blog.csdn.net/weiweicao0429/article/details/53185999)）
```java
示例：$jstack –l 23561 >> xxx.dump
命令 : $jstack [option] pid >> 文件
```
表示输出到文件尾部，实际运行中，往往一次dump的信息，还不足以确认问题，建议产生三次dump信息，如果每次dump都指向同一个问题，我们才确定问题的典型性。

##### 18、说出 3 条在 Java 中使用线程的最佳实践
给你的线程起个有意义的名字。这样可以方便找bug或追踪。`OrderProcessor, QuoteProcessor or TradeProcessor`这种名字比`Thread-1. Thread-2 and Thread-3`好多了，给线程起一个和它要完成的任务相关的名字，所有的主要框架甚至JDK都遵循这个最佳实践。

避免锁定和缩小同步的范围锁花费的代价高昂且上下文切换更耗费时间空间，试试最低限度的使用同步和锁，缩小临界区。因此相对于同步方法我更喜欢同步块，它给我拥有对锁的绝对控制权。

多用同步类少用wait和notify首先，`CountDownLatch, Semaphore, CyclicBarrier`和`Exchanger`这些同步类简化了编码操作，而用`wait`和`notify`很难实现对复杂控制流的控制。其次，这些类是由最好的企业编写和维护在后续的JDK中它们还会不断优化和完善，使用这些更高等级的同步工具你的程序可以不费吹灰之力获得优化。

多用并发集合少用同步集合，这是另外一个容易遵循且受益巨大的最佳实践，并发集合比同步集合的可扩展性更好，所以在并发编程时使用并发集合效果更好。如果下一次你需要用到map，你应该首先想到用`ConcurrentHashMap`。

##### 19、【情景开放题】

##### 实际项目中使用多线程举例。你在多线程环境中遇到的常见的问题是什么？你是怎么解决它的？

##### 请说出与线程同步以及线程调度相关的方法

##### 程序中有3个 socket，需要多少个线程来处理

##### 假如有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有 10 个线程同时调用它，如何做到

##### 如何在 Windows 和 Linux 上查找哪个线程使用的 CPU 时间最长

##### 如何确保 main() 方法所在的线程是 Java 程序最后结束的线程

##### 非常多个线程（可能是不同机器），相互之间需要等待协调才能完成某种工作，问怎么设计这种协调方案

##### 你需要实现一个高效的缓存，它允许多个用户读，但只允许一个用户写，以此来保持它的完整性，你会怎样去实现它



