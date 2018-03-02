---
title: 详解IO的使用
tags: [Interview,java,IO]
categories: Java IO
copyright: true
---
I/O类库中使用“流”这个抽象概念。Java对设备中数据的操作是通过流的方式。表示任何有能力产出数据的数据源对象，或者是有能力接受数据的接收端对象。“流”屏蔽了实际的I/O设备中处理数据的细节。IO流用来处理设备之间的数据传输。设备是指硬盘、内存、键盘录入、网络等。
<!--more-->
IO的分类可以为：
- 流按操作数据类型的不同分为两种：字节流与字符流。
- 流按流向分为：输入流，输出流（以程序为参照物，输入到程序，或是从程序输出）
![IO流继承体系](http://upload-images.jianshu.io/upload_images/8926909-b2b4859ffeb3ae26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 一、字节流
##### 1、Inpustream
`InputStream `有`read`方法，一次读取一个字节，`OutputStream`的`write`方法一次写一个`int`。这两个类都是抽象类。意味着不能创建对象，那么需要找到具体的子类来使用。操作流的步骤都是：

第一步：1：打开流（即创建流）
第二步：2：通过流读取内容
第三步：3：用完后，关闭流资源

案例一：使用 `read()`方法，一次读取一个字节,读到文件末尾返回-1.
```java
 private static void showContent(String path) throws IOException {
        // 打开流
        FileInputStream fis = new FileInputStream(path);

        int len;
        while ((len = fis.read()) != -1) {
            System.out.print((char) len);
        }
        // 使用完关闭流
        fis.close();
```
案例二：使用`read()`方法的时候，可以将读到的数据装入到字节数组中，一次性的操作数组，可以提高效率。
```java
private static void showContent2(String path) throws IOException {
        // 打开流
        FileInputStream fis = new FileInputStream(path);

        // 通过流读取内容
        byte[] byt = new byte[1024];
        int len = fis.read(byt);
        for (int i = 0; i <len; i++) {
            System.out.print(byt[i]);
    　　}

        // 使用完关闭流
        fis.close();
    }
```
案例三：使用`read(byte[] b,int off,int len)`
```java
  /**
     * 把数组的一部分当做流的容器来使用
     * read(byte[] b,int off,int len)
     */
    private static void showContent3(String path) throws IOException {
        // 打开流
        FileInputStream fis = new FileInputStream(path);

        // 通过流读取内容
        byte[] byt = new byte[1024];
        // 从什么地方开始存读到的数据
        int start = 5;

        // 希望最多读多少个(如果是流的末尾，流中没有足够数据)
        int maxLen = 6;

        // 实际存放了多少个
        int len = fis.read(byt, start, maxLen);

        for (int i = start; i < start + maxLen; i++) {
            System.out.print((char) byt[i]);
        }

        // 使用完关闭流
        fis.close();
    }
```
案例四（推荐使用）：使用缓冲(提高效率),并循环读取(读完所有内容).
 ```java
    /**
     * 使用字节数组当缓冲
     * */
    private static void showContent7(String path) throws IOException {
        FileInputStream fis = new FileInputStream(path);
        byte[] byt = new byte[1024];
        int len = 0;
        while ((len = fis.read(byt)) != -1) {
            System.out.println(new String(byt, 0, len));
        }
        fis.close();
}
```
##### 2、OutputStream
`OutputStram` 的`write`方法，一次只能写一个字节。成功的向文件中写入了内容。但是并不高效，如何提高效率呢？可以使用缓冲，在`OutputStram`类中有`write(byte[] b)`方法，将 `b.length `个字节从指定的 `byte` 数组写入此输出流中。
```java
private static void writeTxtFile(String path) throws IOException {
        // 1：打开文件输出流，流的目的地是指定的文件
        FileOutputStream fos = new FileOutputStream(path,true);

        // 2：通过流向文件写数据
        byte[] byt = "java".getBytes();
        fos.write(byt);

        // 3:用完流后关闭流
        fos.close();
    }
```
##### 3、输入输出流综合使用——文件拷贝实现
```java
public static void copyFile(String srcPath, String destPath) throws IOException {
        // 打开输入流，输出流
        FileInputStream fis = new FileInputStream(srcPath);
        FileOutputStream fos = new FileOutputStream(destPath);

        // 读取和写入信息
        int len = 0;

        // 使用字节数组，当做缓冲区
        byte[] byt = new byte[1024];
        while ((len = fis.read(byt)) != -1) {
            fos.write(byt, 0, len);
        }

        // 关闭流
        fis.close();
        fos.close();
    }
```
可以根据拷贝的需求调整数组的大小，一般是1024的整数倍。使用缓冲后效率大大提高。目前我们是抛出处理，一旦出现了异常，`close`就没有执行，也就没有释放资源。那么为了保证`close`的执行该如何处理呢。那么就需要使用`try{} catch(){}finally{}`语句。`try`中放入可能出现异常的语句，`catch`是捕获异常对象，`fianlly`是一定要执行的代码。
```java
public static void copyFile(String srcPath, String destPath) {

        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            fis = new FileInputStream(srcPath);
            fos = new FileOutputStream(destPath);

            byte[] byt = new byte[1024 * 1024];
            int len = 0;
            while ((len = fis.read(byt)) != -1) {

                fos.write(byt, 0, len);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            try {
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                throw new RuntimeException(e);
            } finally {
                if (fos != null) {
                    try {
                        fos.close();
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }
    }
```
大量的异常捕获代码使得以上代码变得十分臃肿难看，好在java7提供了**TWR（try-with-resource）**语法糖，由于很多外部资源类都间接的实现了`AutoCloseable`接口（单方法回调接口），因此可以利用TWR语法在`try`结束的时候通过回调的方式自动调用外部资源类的`close()`方法，避免书写冗长的`finally`代码块。`
```java
    public static void copyByTWR(String srcPath, String destPath){
        try( FileInputStream fis = new FileInputStream(srcPath);
             FileOutputStream fos = new FileOutputStream(destPath)){
            byte[] byt = new byte[1024 * 1024];
            int len = 0;
            while ((len = fis.read(byt)) != -1) {
                fos.write(byt, 0, len);
            }
        }catch (IOException e){
            throw new RuntimeException(e);
        }
    }
```
当`try`语句块运行结束时，`FileInputStream / FileOutputStream`会被自动关闭。这是因为`FileInputStream `实现了java中的`java.lang.AutoCloseable`接口。所有实现了这个接口的类都可以在`try-with-resources`结构中使用上面的例子在try关键字后的括号里创建了两个资源——`FileInputStream` 和`FileOutputStream`。当程序运行离开try语句块时，这两个资源都会被自动关闭,这些资源将按照他们被创建顺序的逆序来关闭。首先`FileOutputStream`会被关闭，然后`FileInputStream`会被关闭。
怎么实现对资源的关闭呢？当`try-with-resources`结构中抛出一个异常，同时`FileInputStream`被关闭时（调用了其`close`方法）也抛出一个异常，`try-with-resources`结构中抛出的异常会向外传播，而`FileInputStream`被关闭时抛出的异常被抑制了。这与文章开始处利用旧风格代码的例子（在finally语句块中关闭资源）相反。
>具体参考：[Java 7中的Try-with-resources](http://ifeve.com/java-7%E4%B8%AD%E7%9A%84try-with-resources/)

##### 4、缓冲流
Java其实提供了专门的字节流缓冲来提高效率。`BufferedInputStream` 和 `BufferedOutputStream。BufferedOutputStream`和`BufferedOutputStream`类可以通过减少读写次数来提高输入和输出的速度。它们内部有一个缓冲区，用来提高处理效率。查看API文档，发现可以指定缓冲区的大小。其实内部也是封装了字节数组。没有指定缓冲区大小，默认的字节是8192。显然缓冲区输入流和缓冲区输出流要配合使用。首先缓冲区输入流会将读取到的数据读入缓冲区，当缓冲区满时，或者调用`flush`方法，缓冲输出流会将数据写出。
注意：当然使用缓冲流来进行提高效率时，对于小文件可能看不到性能的提升。但是文件稍微大一些的话，就可以看到实质的性能提升了。
```java
public class Test {
    public static void main(String[] args) throws IOException {
        String srcPath = "c:\\a.mp3";
        String destPath = "d:\\copy.mp3";
        copyFile(srcPath, destPath);
    }

    public static void copyFile(String srcPath, String destPath)
            throws IOException {
        // 打开输入流，输出流
        FileInputStream fis = new FileInputStream(srcPath);
        FileOutputStream fos = new FileOutputStream(destPath);

        // 使用缓冲流
        BufferedInputStream bis = new BufferedInputStream(fis);
        BufferedOutputStream bos = new BufferedOutputStream(fos);

        // 读取和写入信息
        int len = 0;

        while ((len = bis.read()) != -1) {
            bos.write(len);
        }

        // 关闭流
        bis.close();
        bos.close();
    }
}
```
#### 二、字符流
 计算机并不区分二进制文件与文本文件。所有的文件都是以二进制形式来存储的，因此，从本质上说，所有的文件都是二进制文件。所以字符流是建立在字节流之上的，它能够提供字符层次的编码和解码。可以说字符流就是：字节流 + 编码表，为了更便于操作文字数据。字符流的抽象基类：`Reader ， Writer`。由这些类派生出来的子类名称都是以其父类名作为子类名的后缀，如`FileReader、FileWriter`。
##### 1、Reader
```java
int read()
```
读取一个字符。返回的是读到的那个字符。如果读到流的末尾，返回-1.
```java
int read(char[])
```
将读到的字符存入指定的数组中，返回的是读到的字符个数，也就是往数组里装的元素的个数。如果读到流的末尾，返回-1.
```java
close()
```
读取字符其实用的是window系统的功能，就希望使用完毕后，进行资源的释放
由于`Reader`也是抽象类，所以想要使用字符输入流需要使用`Reader`的实现类——`FileReader`。1，用于读取文本文件的流对象。2，用于关联文本文件。
构造函数：在读取流对象初始化的时候，必须要指定一个被读取的文件。如果该文件不存在会发生FileNotFoundException。
```java
    /**
     * 使用字符流读取文件内容
     */
    public static void readFileByReader(String path) throws Exception {
        Reader reader = new FileReader(path);
        int len = 0;
        while ((len = reader.read()) != -1) {
            System.out.print((char) len);
        }
        reader.close();
    }
```
##### 2、Writer
```java
write(ch): //将一个字符写入到流中。

write(char[]):// 将一个字符数组写入到流中。

write(String): //将一个字符串写入到流中。

flush(): //刷新流，将流中的数据刷新到目的地中，流还存在。

close(): //关闭资源：在关闭前会先调用flush()，刷新流中的数据去目的地。然流关闭。
```
基本方法和`OutputStream` 类似，有`write`方法，功能更多一些。可以接收字符串。`Writer`是抽象类无法创建对象，`Writer`的实现子类为`FileWriter`。默认的`FileWriter`方法新值会覆盖旧值，想要实现追加功能需要使用如下构造函数创建输出流 `append`值为`true`了。
```java
FileWriter(String fileName, boolean append)
FileWriter(File file, boolean append)
```
##### 3、文件拷贝——完善异常处理风格
一次读一个字符就写一个字符，效率不高。把读到的字符放到字符数组中，再一次性的写出，缓冲数组可以提高效率。
```java
 /**
     * 使用字符流拷贝文件，有完善的异常处理
     */
    public static void copyFile2(String path1, String path2) {
        Reader reader = null;
        Writer writer = null;
        try {
            // 打开流
            reader = new FileReader(path1);
            writer = new FileWriter(path2);

            // 进行拷贝
            int ch = -1;
　　　　　　　char [] arr=new char[1024];
            while ((ch = reader.read(arr)) != -1) {
                writer.write(ch);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            // 关闭流，注意一定要能执行到close()方法，所以都要放到finally代码块中
            try {
                if (reader != null) {
                    reader.close();
                }
            } catch (Exception e) {
                throw new RuntimeException(e);
            } finally {
                try {
                    if (writer != null) {
                        writer.close();
                    }
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
```
同样可以用TWR风格进行简写，将大量finally中的资源关闭操作省略掉，声明赋值语句放入到try()中即可。

##### 4、缓冲流
使用字符流缓冲区拷贝文本文件可以提高效率，`Reader`有一个子类`BufferedReader`， 子类继承父类显然子类可以重写父类的方法可以增加自己的新方法。例如一次读一行就是常用的操作.那么`BufferedReader `类就提供了这个方法,可以查看`readLine()`方法具备 一次读取一个文本行的功能。很显然,该子类可以对功能进行增强。
```java
private static void copyFile(File srcFile, File destFile)throws IOException {
        // 创建字符输入流
        FileReader fr = new FileReader(srcFile);
        // 创建字符输出流
        FileWriter fw = new FileWriter(destFile);

        // 字符输入流的缓冲流
        BufferedReader br = new BufferedReader(fr);
        // 字符输出流的缓冲流
        BufferedWriter bw = new BufferedWriter(fw);

        String line = null;
        // 一次读取一行
        while ((line = br.readLine()) != null) {
            // 一次写出一行.
            bw.write(line);
            // 刷新缓冲
            bw.flush();
            // 进行换行,由于readLine方法默认没有换行.需要手动换行
            bw.newLine();
        }
        // 关闭流
        br.close();
        bw.close();
    }
```
#### 三、装饰器模式：
使用分层对象来动态透明的向单个对象中添加责任（功能）。装饰器指定包装在最初的对象周围的所有对象都具有相同的基本接口。某些对象是可装饰的，可以通过将其他类包装在这个可装饰对象的四周，来将功能分层。装饰器必须具有和他所装饰的对象相同的接口。Java I/O类库需要多种不同的功能组合，所以使用了装饰器模式。`Filterxxx`类是`JavaIO`提供的装饰器基类，即我们要想实现一个新的装饰器，就要继承这些类。

继承实现的增强类：
优点：代码结构清晰，而且实现简单
缺点：对于每一个的需要增强的类都要创建具体的子类来帮助其增强，这样会导致继承体系过于庞大。

修饰模式实现的增强类：
优点：内部可以通过多态技术对多个需要增强的类进行增强
缺点：需要内部通过多态技术维护需要增强的类的实例。进而使得代码稍微复杂。

#### 四、面试总结：
##### 1、为了提高读写性能，可以采用什么流
针对读写对象的不同，字节流可以采用带缓冲区的BufferedInputStream和BufferedOutputStream，字符流可以采用带缓冲区的BufferedReader和BufferedWriter。

##### 2、Java中有几种类型的流
字节流和字符流。字节流继承于`InputStream、OutputStream`，字符流继承于`Reader、Writer`。在`java.io `包中还有许多其他的流，主要是为了提高性能和使用方便。关于Java的I/O需要注意的有两点：一是两种对称性（输入和输出的对称性，字节和字符的对称性）；二是两种设计模式（适配器模式和装潢模式）。另外Java中的流不同于C#的是它只有一个维度一个方向。

##### 3、JDK 为每种类型的流提供了一些抽象类以供继承，分别是哪些类
Java中的流分为两种，一种是字节流，另一种是字符流，分别由四个抽象类来表示（每种流包括输入和输出两种所以一共四个）:`InputStream，OutputStream，Reader，Writer`。Java中其他多种多样变化的流均是由它们派生出来的.

4、对文本文件操作用什么I/O流？
`FileReader/FileWriter`

5、对各种基本数据类型和String类型的读写，采用什么流？ `DataInputStream、DataOutputStream`

6、能指定字符编码的 I/O 流类型是什么？
```java
BufferedReader/BufferedWriter
BufferedInputStream/BufferedOutputStream
```
回答以上问题需要分清各个IO流子类的应用场景：
>`FileInputStream/FileOutputStream`  需要逐个字节处理原始二进制流的时候使用，效率低下。
`FileReader/FileWriter` 需要组个字符处理的时候使用。
`StringReader/StringWriter` 需要处理字符串的时候，可以将字符串保存为字符数组。
`PrintStream/PrintWriter` 用来包装`FileOutputStream` 对象，方便直接将`String`字符串写入文件 。
`Scanner`　用来包装`System.in`流，很方便地将输入的`String`字符串转换成需要的数据类型。
`InputStreamReader/OutputStreamReader` ,  字节和字符的转换桥梁，在网络通信或者处理键盘输入的时候用。
`BufferedReader/BufferedWriter `， `BufferedInputStream/BufferedOutputStream `， 缓冲流用来包装字节流后者字符流，提升IO性能，`BufferedReader`还可以方便地读取一行，简化编程。
`SequenceInputStream(InputStream s1, InputStream s2)`序列流，合并流对象时使用.
>`ObjectInputStream、ObjectOutputStream`，方法用于序列化对象并将它们写入一个流，另一个方法用于读取流并反序列化对象。
`ByteArrayInputStream、ByteArrayOutputStream`，操作数组
`DataInputStream、DataOutputStream`操作基本数据类型和字符串。

##### 7、写一个方法，输入一个文件名和一个字符串，统计这个字符串在这个文件中出现的次数。
```java
public final class MyUtil {

    // 工具类中的方法都是静态方式访问的因此将构造器私有不允许创建对象(绝对好习惯)
    private MyUtil() {
        throw new AssertionError();
    }

    /**
     * 统计给定文件中给定字符串的出现次数
     *
     * @param filename  文件名
     * @param word 字符串
     * @return 字符串在文件中出现的次数
     */
    public static int countWordInFile(String filename, String word) {
        int counter = 0;
        try (FileReader fr = new FileReader(filename)) {
            try (BufferedReader br = new BufferedReader(fr)) {
                String line = null;
                while ((line = br.readLine()) != null) {
                    int index = -1;
                    while (line.length() >= word.length() && (index = line.indexOf(word)) >= 0) {
                        counter++;
                        line = line.substring(index + word.length());
                    }
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return counter;
    }
}
```

