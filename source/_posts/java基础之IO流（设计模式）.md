---
title: 结合IO类讲讲装饰者模式
tags: [Interview,java,IO,设计模式]
categories: Java IO
copyright: true
---
java IO流的设计是基于装饰者模式&适配模式，面对IO流庞大的包装类体系，核心是要抓住其功能所对应的装饰类。

装饰模式又名包装（Wrapper）模式。装饰模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。装饰模式通过创建一个包装对象，也就是装饰，来包裹真实的对象。装饰模式以对客户端透明的方式动态地给一个对象附加上更多的责任。换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不创造更多子类的情况下，将对象的功能加以扩展。装饰模式把客户端的调用委派到被装饰类。装饰模式的关键在于这种扩展是完全透明的。
![装饰者模式](http://upload-images.jianshu.io/upload_images/8926909-dc3cf3d3ce99df2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
装饰者的角色：
**抽象构件角色（Component）**：给出一个抽象接口，以规范准备接收附加责任的对象。
**具体构件角色（Concrete Component）**：定义将要接收附加责任的类。
**装饰角色（Decorator）**：持有一个构件（Component）对象的引用，并定义一个与抽象构件接口一致的接口。
**具体装饰角色（Concrete Decorator）**：负责给构件对象“贴上”附加的责任。

实现案例：
```java
//抽象构件角色
public interface Component
{
    public void doSomething();
}
```
```java
//具体构件角色
public class ConcreteComponent implements Component
{
    @Override
    public void doSomething()
    {
        System.out.println("功能A");
    }
}
```
```java
//装饰者角色
public class Decorator implements Component
{
    //维护一个抽象构件角色
    private Component component;

    public Decorator(Component component)
    {
        this.component = component;
    }

    @Override
    public void doSomething()
    {
        component.doSomething();
    }
}
```
```java
//具体装饰者角色
public class ConcreteDecorator1 extends Decorator
{
    public ConcreteDecorator1(Component component)
    {
        super(component);
    }

    @Override
    public void doSomething()
    {
        super.doSomething();

        this.doAnotherThing();
    }

    private void doAnotherThing()
    {
        System.out.println("功能B");
    }
}
```
以上就是装饰者模式的一个极简代码思路，实际上IO流的装饰体系也是在对上面思路的一中具体实现。
**JAVA-IO流体系：**
在IO中，具体构件角色是节点流，装饰角色是过滤流。
1、继承自`InputStream/OutputStream`的流都是用于向程序中输入/输出数据，且数据的单位都是字节(byte=8bit)，如图，深色的为节点流，浅色的为过滤流。
![类继承图](http://upload-images.jianshu.io/upload_images/8926909-88c33f33bf9b3218.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、继承自`Reader/Writer`的流都是用于向程序中输入/输出数据，且数据的单位都是字符(2byte=16bit)，如图，深色的为节点流，浅色的为过滤流。
![类继承图](http://upload-images.jianshu.io/upload_images/8926909-1f9a7b90439b0784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从图中可以看出，`InputStream`就是装饰者模式中的超类`（Component）`，`ByteArrayInputStream，FileInputStream`相当于被装饰者`（ConcreteComponent）`，这些类都提供了最基本的字节读取功能。而另外一个和这两个类是同一级的类`FilterInputStream`即是装饰者`（Decorator）`，`BufferedInputStream，DataInputStream，PushbackInputStream`…这些都是被装饰者装饰后形成的成品。为什么可以说：装饰模式可以在不创造更多子类的情况下，将对象的功能加以扩展，能理解这一点就能很好的掌握装饰者设计模式的精髓，如果在`InputStream`这里扩展出`FilterInputStream`类下面的装饰类，那么针对`FileInputStream`和`ByteArrayInputStream`就都要去实现一次`BufferedInputStream`了，那么可能就会衍生出`BufferedFileInputStream`和`BufferedByteArrayInputStream`这样的类，如果按照这样的扩展方式去添加功能，对于添加功能的子类来说简直是一场噩梦，好在装饰着模式很好的解决了这个问题，现在我们只需要在过滤流类这里维护一个超类，不论传入的是什么具体的节点流，那么都只要套一层装饰，就能对功能方法进行加强。
如果想要对文件输入流进行缓存加强可以这样装饰：
```java
File file = new File ("hello.txt");
BufferedInputStream inBuffered=new BufferedInputStream (new FileInputStream(file));
```
如果想要对字节数组输入流进行缓存加强可以这样装饰：
```java
byte[] byts="Hello".getBytes();
BufferedInputStream bf=new BufferedInputStream(new ByteArrayInputStream(byts));
```
那么节点流上的类就可以平行扩展，而装饰者同样可以按照功能进行另外一个维度的扩展，调用的时候就可以按需进行组合装饰，这样就可以减少了子类还将对象的功能进行扩展，不得不佩服前人在该设计模式上的智慧，理解了这装饰着模式后，就应该对java中IO流的体系进行梳理：
节点流类型
对文件操作的字符流有`FileReader/FileWriter`，
字节流有`FileInputStream/FileOutputStream`。
过滤流类型

缓冲流：缓冲流要“套接”在相应的节点流之上，对读写的数据提供了缓冲的功能，提高了读写效率，同时增加了一些新的方法。
字节缓冲流有`BufferedInputStream / BufferedOutputStream`，字符缓冲流有`BufferedReader / BufferedWriter`，字符缓冲流分别提供了读取和写入一行的方法`ReadLine`和`NewLine`方法。
对于输出的缓冲流，写出的数据，会先写入到内存中，再使用`flush`方法将内存中的数据刷到硬盘。所以，在使用字符缓冲流的时候，一定要先`flush`，然后再`close`，避免数据丢失。
转换流：用于字节数据到字符数据之间的转换。
字符流`InputStreamReader / OutputStreamWriter`。其中，`InputStreamReader`需要与`InputStream`“套接”，`OutputStreamWriter`需要与`OutputStream`“套接”。
数据流：提供了读写Java中的基本数据类型的功能。
`DataInputStream`和`DataOutputStream`分别继承自`InputStream`和`OutputStream`，需要“套接”在`InputStream`和`OutputStream`类型的节点流之上。
对象流：用于直接将对象写入写出。
流类有`ObjectInputStream`和`ObjectOutputStream`，本身这两个方法没什么，但是其要写出的对象有要求，该对象必须实现`Serializable`接口，来声明其是可以序列化的。否则，不能用对象流读写。(api以及demo在文末)
重点梳理一下：Java中Inputstream/OutputStream与Reader/Writer的区别

`Reader/Writer`和`InputStream/OutputStream`分别是I/O库提供的两套平行独立的等级机构，
- `InputStream、OutputStream`是用来处理8位元的流，也就是用于读写ASCII字符和二进制数据；`Reader、Writer`是用来处理16位元的流，也就是用于读写Unicode编码的字符。
- 在JAVA语言中，byte类型是8位的，char类型是16位的，所以在处理中文的时候需要用`Reader`和`Writer`。
- 两种等级机构下，有一道桥梁`InputStreamReader`、`OutputStreamWriter`负责进行`InputStream`到`Reader`的适配和由`OutputStream`到`Writer`的适配。

在Java中，有不同类型的`Reader/InputStream`输入流对应于不同的数据源：
- `FileReader/FileInputStream` 用于从文件输入；
`CharArrayReader/ByteArrayInputStream `用于从程序中的字符数组输入；
- `StringReader/StringBufferInputStream` 用于从程序中的字符串输入；
`PipedReader/PipeInputStream `用于读取从另一个线程中的 -
- `PipedWriter/PipeOutputStream `写入管道的数据。
- 相应的也有不同类型的`Writer/OutputStream`输出流对应于不同的数据源：`FileWriter/FileOutputStream，CharArrayWriter/ByteArrayOutputStream，StringWriter，PipeWriter/PipedOutputStream`。

#### IO流的应用选择

##### 1、确定选用流对象的步骤
确定原始数据的格式
确定是输入还是输出
是否需要转换流
数据的来源（去向）
是否需要缓冲
是否需要格式化输出

##### 按照数据格式分
二进制格式（只要确定不是纯文本格式的），`InputStream`, `OutputStream`, 及其所有带`Stream`子类
- 纯文本格式（比如英文/汉字/或其他编码文字）：`Reader, Writer`, 及其相关子类

##### 按照输入输出分
输入：`Reader， InputStream`，及其相关子类
输出：`Writer，OutputStream`，及其相关子类

##### 按缓冲功能分
要缓冲：`BufferedInputStream, BufferedOuputStream, BuffereaReader, BufferedWriter`

##### 按照格式化输出

需要格式化输出：`PrintStream`（输出字节），`PrintWriter`（输出字符）
##### 特殊需求

从`Stream`转化为`Reader，Writer：InputStreamReader，OutputStreamWriter`
对象输入输出流：`ObjectInputStream，ObjectOutputStream`
进程间通信：`PipeInputStream，PipeOutputStream，PipeReader，PipeWriter`
合并输入：`SequenceInputStream`
更特殊的需要：`PushbackInputStream, PushbackReader, LineNumberInputStream, LineNumberReader`
```java
//对象流案例
public class Demo3 {
    public static void main(String[] args) throws IOException,
            ClassNotFoundException {
        Cat cat = new Cat("tom", 3);
        FileOutputStream fos = new FileOutputStream(new File("c:\\Cat.txt"));
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(cat);
        System.out.println(cat);
        oos.close();
        // 反序列化
        FileInputStream fis = new FileInputStream(new File("c:\\Cat.txt"));
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object readObject = ois.readObject();
        Cat cat2 = (Cat) readObject;
        System.out.println(cat2);
        fis.close();
}

class Cat implements Serializable {
    public String name;
    public int age;

    public Cat() {

    }

    public Cat(String name, int age) {

        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Cat [name=" + name + ", age=" + age + "]";
    }
}
```
```java
//DataInputStream 基本数据类型和String
//操作基本数据类型的方法：
int readInt()://一次读取四个字节，并将其转成int值。
boolean readBoolean()://一次读取一个字节。
short readShort();
long readLong();
//剩下的数据类型一样。
String readUTF()://按照utf-8修改版读取字符。注意，它只能读writeUTF()//写入的字符数据。
DataOutputStream
DataOutputStream(OutputStream):
//操作基本数据类型的方法:
writeInt(int)：//一次写入四个字节。
//注意和write(int)不同。write(int)只将该整数的最低一个8位写入。剩余三个8位丢弃。
writeBoolean(boolean);
writeShort(short);
writeLong(long);
//剩下是数据类型也也一样。
writeUTF(String)://按照utf-8修改版将字符数据进行存储。只能通过readUTF读取。
```
转换流：
`InputStreamReader`：字节到字符的桥梁。
`OutputStreamWriter`：字符到字节的桥梁。
```java
//从字节流中读取字符信息
BufferedReader bf=new InputStreamReader(new FileInputStream("src"));

//将字符信息用指定字节编码写出
OutputStreamWriter bw=new OutputStreamWriter(new FileOutputStream("target"),"utf-8");
        bw.write("Hello");
```
```java
public class TestIo {
    public class Demo4 {
    public static void main(String[] args) throws IOException {
        File file = new File("c:\\a.txt");
        File fileGBK = new File("c:\\gbk.txt");
        File fileUTF = new File("c:\\utf.txt");

        // 写入
        // 使用系统默认码表写入
        testWriteFile(file);
        // 使用gbk编码向gbk文件写入信息
        testWriteFile(fileGBK, "gbk");
        // 使用utf-8向utf-8文件中写入信息
        testWriteFile(fileUTF, "utf-8");

        // 读取
        // 默认编码
        testReadFile(file);
        // 传入gbk编码文件,使用gbk解码
        testReadFile(fileGBK, "gbk");
        // 传入utf-8文件,使用utf-8解码
        testReadFile(fileUTF, "utf-8");

    }

    // 使用系统码表将信息写入到文件中
    private static void testWriteFile(File file) throws IOException {
        FileOutputStream fos = new FileOutputStream(file);
        OutputStreamWriter ops = new OutputStreamWriter(fos);
        ops.write("中国");
        ops.close();
    }

    // 使用指定码表,将信息写入到文件中
    private static void testWriteFile(File file, String encod)
            throws IOException {
        FileOutputStream fos = new FileOutputStream(file);
        OutputStreamWriter ops = new OutputStreamWriter(fos, encod);
        ops.write("中国");
        ops.close();
    }

    // 该方法中nputStreamReader使用系统默认编码读取文件.
    private static void testReadFile(File file) throws IOException {
        FileInputStream fis = new FileInputStream(file);
        InputStreamReader ins = new InputStreamReader(fis);
        int len = 0;
        while ((len = ins.read()) != -1) {
            System.out.print((char) len);
        }
        ins.close();
    }

    // 该方法适合用指定编码读取文件
    private static void testReadFile(File file, String encod)
            throws IOException {
        FileInputStream fis = new FileInputStream(file);
        InputStreamReader ins = new InputStreamReader(fis, encod);
        int len = 0;
        while ((len = ins.read()) != -1) {
            System.out.print((char) len);
        }
        ins.close();
    }
}
```