---
title: 网络编程之IO、NIO和Netty
tags: [Interview,NIO,Netty]
categories: 网络编程
copyright: true
---
### 一、基本概述
IO 一直是编程学习中的核心部分，在这里所说的IO 不仅仅是对文件的操作，还常常应用在网络编程中，比如 Socket 通信、协议服务器等，都是典型的 IO 操作目标，伴随着海量数据增长和分布式系统的发展，IO 的扩展能力则显得十分重要，Java语言提供了强大的IO机制，基于不同的抽象模型和交互方式，总结之可归纳为三种：
>- 传统的 `java.io` 包，基于流模型实现的BIO
>- 升级的 `java.nio` 包，构建多路复用的、同步非阻塞 NIO 
>- 改造的NIO2，引入了异步非阻塞的AIO

<!--more-->
这里面需要区分和澄清几个关键的概念：
- 区分**同步**或**异步**（`synchronous/asynchronous`）。简单来说，同步是一种可靠的有序运行机制，当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；而异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系。

- 区分**阻塞**与**非阻塞**（`blocking/non-blocking`）。在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如 ServerSocket 新连接建立完毕，或数据读取、写入操作完成；而非阻塞则是不管 IO 操作是否结束，直接返回，相应操作在后台继续处理。

**首先介绍一下IO的基础API设计和概念模型：**
- 1. IO 不仅仅是对文件的操作，网络编程中，比如 `Socket` 通信，都是典型的 IO 操作目标。输入流、输出流（`InputStream / OutputStream`）是用于读取或写入字节的，例如操作图片文件。
- 2. `Reader/Writer` 则是用于操作字符，增加了字符编解码等功能，适用于类似从文件中读取或者写入文本信息。本质上计算机操作的都是字节，不管是网络通信还是文件读取，`Reader/Writer` 相当于构建了应用逻辑和原始数据之间的桥梁。`BufferedOutputStream` 等带缓冲区的实现，可以避免频繁的磁盘读写，进而提高 IO 处理效率。这种设计利用了缓冲区，将批量数据进行一次操作，但在使用中千万别忘了 flush。

- 3. 很多 IO 工具类都实现了 `Closeable` 接口，因为需要进行资源的释放，需要利用try-with-resources、 try-finally 等机制保证 `FileInputStream` 被明确关闭，进而相应文件描述符也会失效，否则将导致资源无法被释放。利用专栏前面的内容提到的 `Cleaner` 或 `finalize` 机制作为资源释放的最后把关，也是必要的。

>IO的具体介绍请戳：[《java基础之IO流（IO篇）》](https://www.jianshu.com/p/48f2fb0b79b5)

**接下来说NIO：**
- 1. `Buffer`，高效的数据容器，除了布尔类型，所有原始数据类型都有相应的 `Buffer` 实现。

- 2. `Channel`，Channel 是操作系统底层的一种抽象，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中，可以通过Socket 获取 Channel，反之亦然。

- 3. `Selector`，是 NIO 实现多路复用的基础，它提供了一种高效的机制，可以检测到注册在，`Selector` 上的多个 `Channel` 中，是否有 `Channel` 处于就绪状态，进而实现了单线程对多Channel 的高效管理。

在 Java 7 中，NIO 有了进一步的改进，也就是 NIO 2，引入了异步非阻塞 IO 方式，也有很多人叫它 AIO（`Asynchronous IO`）。异步 IO 操作基于事件和回调机制，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

>NIO的具体介绍请戳：[《java进阶之NIO》](https://www.jianshu.com/p/f8454c6be204)

### 二、网络编程应用
#### 1. 基于BIO实现
IO和NIO具体的应用在哪里，可结合服务器-客户端（B/S）模式来设计一个极简版的web后台，该后台仅仅能够同时服务多个客户端请求即可。这里有三种思路，首先使用` java.io` 和 `java.net` 中的同步、阻塞式 API，可以简单实现。

服务端的实现：
```java
/**
 * IO服务端实现
 */
public class IOServer {

    public static void main(String[] args) throws Exception {

        ServerSocket serverSocket = new ServerSocket(8000);

        // (1) 接收新连接线程
        new Thread(() -> {
            while (true) {
                try {
                    // (1) 阻塞方法获取新的连接
                    Socket socket = serverSocket.accept();

                    // (2) 每一个新的连接都创建一个线程，负责读取数据
                    new Thread(() -> {
                        try {
                            int len;
                            byte[] data = new byte[1024];
                            InputStream inputStream = socket.getInputStream();
                            // (3) 按字节流方式读取数据
                            while ((len = inputStream.read(data)) != -1) {
                                System.out.println(new String(data, 0, len));
                            }
                        } catch (IOException e) {
                        }
                    }).start();

                } catch (IOException e) {
                }
            }
        }).start();
    }
}
```
客户端的实现：
```java
/**
 * IO客户端
 */
public class IOClient {

    public static void main(String[] args) {
        Executor executor = Executors.newFixedThreadPool(8);
        Thread thread = new Thread(() -> {
            try {
                Socket socket = new Socket("127.0.0.1", 8000);
                while (true) {
                    try {
                        socket.getOutputStream().write((new Date() + ": hello world").getBytes());
                        Thread.sleep(2000);
                    } catch (Exception e) {
                    }
                }
            } catch (IOException e) {
            }
        });
        executor.execute(thread);
    }
}
```
BIO模型实现要点是：
- 服务器端启动 `ServerSocket`，端口 `8000` 表示自动绑定一个空闲端口。
- 调用 `accept` 方法，**阻塞等待客户端连接**。
- 利用 `Socket` 模拟了一个简单的客户端，只进行连接、读取、打印。
- 当连接建立后，启动一个单独线程负责回复客户端请求。

这样，一个简单的 Socket 服务器就被实现出来了。但是我们知道 Java 语言目前的线程实现是比较重量级的，启动或者销毁一个线程是有明显开销的，每个线程都有单独的线程栈等结构，需要占用非常明显的内存，所以，每一个 Client 启动一个线程似乎都有些浪费。可以稍微修正一下引入线程池解决线程开销和切换的问题：
```java
serverSocket = new ServerSocket(8000);
executor = Executors.newFixedThreadPool(8);
while (true) {
       Socket socket = serverSocket.accept();
       ... 开启一个IO线程 ...
       executor.execute(ioThread);
}
```
这样做似乎好了很多，通过一个固定大小的线程池，来负责管理工作线程，避免频繁创建、销毁线程的开销，这是我们构建并发服务的典型方式。这种工作方式，可以参考下图来理解。

![BIO交互模式](https://upload-images.jianshu.io/upload_images/8926909-c1435a8ceb30e2a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果连接数并不是非常多，只有最多几百个连接的普通应用，这种模式往往可以工作的很好。但是，如果连接数量急剧上升，这种实现方式就无法很好地工作了，因为线程上下文切换开销会在高并发时变得很明显，这是同步阻塞方式的低扩展性劣势。

#### 2. 基于NIO实现
NIO 引入的多路复用机制，提供了另外一种思路：
- 线程资源不受限：`NIO`编程模型新来一个连接不再创建一个新的线程，把这条连接直接绑定到某个固定的线程，然后这条连接所有的读写都由该线程来负责，把这么多`while`死循环变成一个死循环，这个死循环由一个线程控制，一条连接来了，不创建一个`while`死循环去监听是否有数据可读,，直接把这条连接注册到`Selector`上，然后通过检查`Selector`批量监测出有数据可读的连接进而读取数据。
- 线程切换效率提高：线程数量大大降低，线程切换效率因此也大幅度提高。
- 数据读写是以字节流为单位效率不高：`NIO`维护一个缓冲区每次从这个缓冲区里面读取一块的数据，数据读写不再以字节为单位，而是以字节块为单位。
```java
public class NIOServer {

    /**
     * serverSelector负责轮询是否有新的连接,clientSelector负责轮询连接是否有数据可读.
     * 服务端监测到新的连接不再创建一个新的线程,而是直接将新连接绑定到clientSelector上,这样不用IO模型中1w个while循环在死等
     * clientSelector被一个while死循环包裹,如果在某一时刻有多条连接有数据可读通过 clientSelector.select(1)方法轮询出来进而批量处理
     * 数据的读写以内存块为单位
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        Selector serverSelector = Selector.open();
        Selector clientSelector = Selector.open();

        new Thread(() -> {
            try {
                ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.socket().bind(new InetSocketAddress(8000));
                serverSocketChannel.configureBlocking(false);
                serverSocketChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                while (true) {
                    // 轮询监测是否有新的连接
                    if (serverSelector.select(1) > 0) {
                        Set<SelectionKey> selectionKeys = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
                        while (keyIterator.hasNext()) {
                            SelectionKey selectionKey = keyIterator.next();
                            if (selectionKey.isAcceptable()) {
                                try {
                                    //(1)每来一个新连接不需要创建一个线程而是直接注册到clientSelector
                                    SocketChannel socketChannel = ((ServerSocketChannel) selectionKey.channel()).accept();
                                    socketChannel.configureBlocking(false);
                                    socketChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove();
                                }
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                while (true) {
                    // (2)批量轮询是否有哪些连接有数据可读
                    if (clientSelector.select(1) > 0) {
                        Set<SelectionKey> selectionKeys = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
                        while (keyIterator.hasNext()) {
                            SelectionKey selectionKey = keyIterator.next();
                            if (selectionKey.isReadable()) {
                                try {
                                    SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    //(3)读取数据以块为单位批量读取
                                    socketChannel.read(byteBuffer);
                                    byteBuffer.flip();
                                    System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer)
                                            .toString());
                                } finally {
                                    keyIterator.remove();
                                    selectionKey.interestOps(SelectionKey.OP_READ);
                                }
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```
首先，通过 `Selector.open()` 创建一个 `Selector`，作为类似调度员的角色。然后，创建一个`ServerSocketChannel`，并且向 `Selector` 注册，通过指定 `SelectionKey.OP_ACCEPT`，告诉调度员，它关注的是新的连接请求。注意，为什么要配置非阻塞模式呢？这是因为阻塞模式下，注册操作是不允许的，会抛出 `IllegalBlockingModeException` 异常。`Selector` 阻塞在 `select` 操作，当有 `Channel` 发生接入请求，就会被唤醒。

**`IO` 都是同步阻塞模式，所以需要多线程以实现多任务处理。而`NIO` 则是利用了单线程轮询事件的机制，通过高效地定位就绪的 `Channel`，来决定做什么，仅仅 `select` 阶段是阻塞的，可以有效避免大量客户端连接时，频繁线程切换带来的问题，应用的扩展能力有了非常大的提高**。下面这张图对这种实现思路进行了形象地说明。
![NIO多路复用](https://upload-images.jianshu.io/upload_images/8926909-f919c016030f2148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 Java 7 引入的 NIO 2 中，又增添了一种额外的异步 IO 模式，利用事件和回调，处理`Accept、Read` 等操作。 AIO 实现看起来是类似这样子：
```java
AsynchronousServerSocketChannel serverSock =        
      AsynchronousServerSocketChannel.open().bind(sockAddr);
serverSock.accept(serverSock, new CompletionHandler<>() { 
    //为异步操作指定 CompletionHandler 回调函数
    @Override
    public void completed(AsynchronousSocketChannel sockChannel, AsynchronousServerSocketChannel serverSock) {
        serverSock.accept(serverSock, this);
        // 另外一个 write（sock，CompletionHandler{}）
        sayHelloWorld(sockChannel, Charset.defaultCharset().encode
                ("Hello World!"));
    }
  // 省略其他路径处理方法...
});
```
#### 3.基于Netty实现
JDK 的 NIO 编程需要了解很多的概念，编程复杂，对 NIO 入门非常不友好，编程模型不友好，ByteBuffer 的 Api 难以掌握，而且JDK 的 NIO 底层由 epoll 实现，该实现饱受诟病的空轮询 bug 会导致 cpu 飙升 100%。而Netty 封装了 JDK 的 NIO，API开箱即用，对开发友好，用netty来实现将比NIO方是简化了很多：

Netty实现服务端：必须要指定三类属性，分别是**线程模型、IO模型、连接读写处理逻辑**。Netty服务端启动流程：1. 创建引导类，2. 指定线程模型、IO模型、连接读写处理逻辑，3. 绑定端口。

```java
public class NettyServer {
    private static final int BEGIN_PORT = 8000;

    public static void main(String[] args) {

        //指定线程模型
        NioEventLoopGroup boosGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        //引导类，负责服务端的启动
        final ServerBootstrap serverBootstrap = new ServerBootstrap();
        //
        final AttributeKey<Object> clientKey = AttributeKey.newInstance("clientKey");
        serverBootstrap
                .group(boosGroup, workerGroup)         //配置线程组，初始化引导类的线程模型
                .channel(NioServerSocketChannel.class) //指定IO模型
                .attr(AttributeKey.newInstance("serverName"), "nettyServer")  //自定义属性
                .childAttr(clientKey, "clientValue")
                .option(ChannelOption.SO_BACKLOG, 1024)         //表示系统用于临时存放已完成三次握手的请求的队列的最大长度，如果连接建立频繁，服务器处理创建新连接较慢，可以适当调大这个参数
                .childOption(ChannelOption.SO_KEEPALIVE, true)  //开启TCP底层心跳机制
                .childOption(ChannelOption.TCP_NODELAY, true)   //开Nagle算法,要求高实时性，有数据发送时就马上发送，就关闭，如果需要减少发送次数减少网络交互，就开启。
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) {
                        System.out.println(ch.attr(clientKey).get());
                        ch.pipeline().addLast(new FirstServerHandler());
                    }
                });

        //异步方法，调用之后立即返回
        bind(serverBootstrap, BEGIN_PORT);
    }

    //自动banding递增端口
    private static void bind(final ServerBootstrap serverBootstrap, final int port) {
        serverBootstrap.bind(port).addListener(future -> {
            if (future.isSuccess()) {
                System.out.println("端口[" + port + "]绑定成功!");
            } else {
                System.err.println("端口[" + port + "]绑定失败!");
                bind(serverBootstrap, port + 1);
            }
        });
    }
}
```
服务端Handler处理类：
```java
public class FirstServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf byteBuf = (ByteBuf) msg;

        System.out.println(new Date() + ": 服务端读到数据 -> " + byteBuf.toString(Charset.forName("utf-8")));

        // 回复数据到客户端
        System.out.println(new Date() + ": 服务端写出数据");
        ByteBuf out = getByteBuf(ctx);
        out.capacity();
        ctx.channel().writeAndFlush(out);
    }

    private ByteBuf getByteBuf(ChannelHandlerContext ctx) {
        byte[] bytes = "Hello,Clinet!".getBytes(Charset.forName("utf-8"));

        ByteBuf buffer = ctx.alloc().buffer();

        buffer.writeBytes(bytes);

        return buffer;
    }
}
```
客户端启动流程：要启动Netty客户端，必须要指定三类属性，分别是线程模型、IO模型、连接读写处理逻辑。Netty客户端启动流程：1. 创建引导类，2. 指定线程模型、IO模型、连接读写处理逻辑，3. 建立连接。

```java
public class NettyClient {

    private static final int MAX_RETRY = 5;

    private static final String HOST = "127.0.0.1";

    private static final int PORT = 8000;

    public static void main(String[] args) {
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                // 1.指定线程模型
                .group(workerGroup)
                // 2.指定 IO 类型为 NIO
                .channel(NioSocketChannel.class)
                // 绑定自定义属性到 channel
                .attr(AttributeKey.newInstance("clientName"), "nettyClient")
                // 设置TCP底层属性
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .option(ChannelOption.TCP_NODELAY, true)
                // 3.IO 处理逻辑
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) {
                        ch.pipeline().addLast(new FirstClientHandler());
                    }
                });

        // 4.建立连接
        connect(bootstrap, HOST, PORT, MAX_RETRY);
    }

    private static void connect(Bootstrap bootstrap, String host, int port, int retry) {
        bootstrap.connect(host, port).addListener(future -> {
            if (future.isSuccess()) {
                System.out.println("连接成功!");
            } else if (retry == 0) {
                System.err.println("重试次数已用完，放弃连接！");
            } else {
                // 第几次重连
                int order = (MAX_RETRY - retry) + 1;
                // 本次重连的间隔
                int delay = 1 << order;
                System.err.println(new Date() + ": 连接失败，第" + order + "次重连……");
                bootstrap.config().group().schedule(() -> connect(bootstrap, host, port, retry - 1), delay, TimeUnit.SECONDS);
            }
        });
    }
}
```
客户端Handler类：
```java
public class FirstClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        System.out.println(new Date() + ": 客户端写出数据");

        // 1.获取数据
        ByteBuf buffer = getByteBuf(ctx);

        // 2.写数据
        ctx.channel().writeAndFlush(buffer);
    }

    private ByteBuf getByteBuf(ChannelHandlerContext ctx) {
        byte[] bytes = "Hello,Server!".getBytes(Charset.forName("utf-8"));
        ByteBuf buffer = ctx.alloc().buffer();
        buffer.writeBytes(bytes);
        return buffer;
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println(new Date() + ": 客户端读到数据 -> " + byteBuf.toString(Charset.forName("utf-8")));
    }

}
```
Netty采用`Reactor`线程模型。这里面主要有三种`Reactor`线程模型。分别是单线程模式、主从`Reactor`模式、多`Reactor`线程模式。其都可以通过初始和`EventLoopGroup`进行设置。其主要区别在于，单`Reactor`模式就是一个线程，既进程处理连接，也处理IO。类似于我们传统的OIO编程。主从`Reactor`模式，其实就是将监听连接和处理IO的分开在不同的线程完成。最后，主从`Reactor`线程模型，为了解决多`Reactor`模型下单一线程性能不足的问题。改为了一组线程池进行处理。官方默认的是采用这种主从`Reactor`模型。其线程数默认为CPU内核的2倍。
***
**读完可以思考：**
- 基础 API 功能与设计， InputStream/OutputStream 和 Reader/Writer 的关系和区别。
- NIO、NIO 2 的基本组成。
- 给定场景，分别用不同模型实现，分析 BIO、NIO 等模式的设计和实现原理。
- NIO 提供的高性能数据操作方式是基于什么原理，如何使用？
- 从开发者的角度来看， NIO 自身实现存在哪些问题？有什么改进的想法吗？