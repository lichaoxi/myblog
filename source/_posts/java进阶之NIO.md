---
title: NIO的基础总结
tags: [IO,NIO,并发]
categories: 网络
copyright: true
---
Java NIO(New IO)是一个可以替代标准Java IO API的IO API（从Java 1.4开始)，Java NIO提供了与标准IO不同的IO工作方式。
<!--more-->
**Java NIO: Channels and Buffers（通道和缓冲区）**
标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

**Java NIO: Non-blocking IO（非阻塞IO）**
Java NIO可以让你非阻塞的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。

**Java NIO: Selectors（选择器）**
Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

Java NIO 由以下几个核心部分组成：
- Channels
- Buffers
- Selectors

基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。
![Channel和Buffer的读写](http://upload-images.jianshu.io/upload_images/8926909-fb3310e57d93894e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Channel通道涵盖了UDP 和 TCP 网络IO，以及文件IO
```java
FileChannel
DatagramChannel
SocketChannel
ServerSocketChannel
```
Buffer实现，覆盖了能通过IO发送的基本数据类型：`byte, short, int, long, float, double` 和 `char`。
```java
ByteBuffer
CharBuffer
DoubleBuffer
FloatBuffer
IntBuffer
LongBuffer
ShortBuffer
```
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。
![单线程使用Selector处理多Channel](http://upload-images.jianshu.io/upload_images/8926909-5e50515713aa3509.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 一、NIO的使用
##### 1.Channel
`FileChannel` 从文件中读写数据。
`DatagramChannel `能通过UDP读写网络中的数据。
`SocketChannel` 能通过TCP读写网络中的数据。
`ServerSocketChannel`可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个`SocketChannel`。

使用`FileChannel`读取数据到Buffer中的示例：
```java
        RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        ByteBuffer buf = ByteBuffer.allocate(48);

        int bytesRead = inChannel.read(buf);
        while (bytesRead != -1) {

            System.out.println("Read " + bytesRead);
            buf.flip();

            while(buf.hasRemaining()){
                System.out.print((char) buf.get());
            }

            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        aFile.close();
```
##### 2.Buffer的使用
使用Buffer一般遵循下面几个步骤：
- 分配空间（`ByteBuffer buf = ByteBuffer.allocate(1024);` 还有一种allocateDirector）
- 写入数据到Buffer(`int bytesRead = fileChannel.read(buf);`)
- 调用`filp()`方法（ `buf.flip();`）
- 从Buffer中读取数据（`System.out.print((char)buf.get());`）
- 调用`clear()`方法或者`compact()`方法

Buffer顾名思义：缓冲区，实际上是一个容器，一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer。如下图：
![channel和buffer的读写](http://upload-images.jianshu.io/upload_images/8926909-ac43d8ab02212cbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

向Buffer中写数据：
- 从Channel写到Buffer (`fileChannel.read(buf)`)
- 通过Buffer的`put()`方法 （`buf.put(…)`）

从Buffer中读取数据：
- 从Buffer读取到Channel (`channel.write(buf)`)
- 使用`get()`方法从Buffer中读取数据 （`buf.get()`）

可以把Buffer简单地理解为一组基本数据类型的元素列表，它通过几个变量来保存这个数据的当前位置状态：`capacity, position, limit, mark`

#### 3.Selector
Selector的创建：
```java
Selector selector = Selector.open();
```
为了将Channel和Selector配合使用，必须将Channel注册到Selector上，通过SelectableChannel.register()方法来实现：
```java
ssc= ServerSocketChannel.open();
ssc.socket().bind(new InetSocketAddress(PORT));
ssc.configureBlocking(false);
ssc.register(selector, SelectionKey.OP_ACCEPT);
```
>[《Java NIO 系列教程》](http://ifeve.com/java-nio-all/)
>[《攻破JAVA NIO技术壁垒》](http://www.importnew.com/19816.html)