# Netty框架基本介绍

> https://blog.csdn.net/zhijingzhi/article/details/125197108

## 关于NIO

1. 概述：NIO全称java non-blocking IO ，是指JDK1.4开始，java提供了一系列改进的输入/输出的新特性，被统称为NIO(即New IO )。新增了许多用于处理输入输出的类，这些类都被放在java.nio包及子包下，并且对java.io包中的许多类进行了改写，新增了满足NIO的功能。

2. NIO和BIO有着相同的目的和作用，但是他们的实现方式完全不同，BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多。另外，NIO是非阻塞式的，这一点跟BIO也很不相同，使用它可以提供非阻塞式的高伸缩网络。

3. NIO主要有三大核心部分：
   - Channel（通道），
   - Buffer（缓冲区）,
   - Selector（选择器）。

4. 传统的的BIO基于字节流或者字符流进行操作 ，而NIO基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。  
   Selector用于监听多个通道的事件（比如连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

## 与传统IO的区别：

|  #   | IO     | NIO        | 备注 |
| :--: | ------ | ---------- | ---- |
|  1   | 面向流 | 面向缓冲区 |      |
|  2   | 阻塞IO | 非阻塞IO   |      |
|  3   | 无     | 选择器     |      |

### **Buffer**

缓冲区：实际上是一个容器，是一个特殊的数组，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，
但是读取或写入的数据都必须经由 Buffer。

- 如下图所示：

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919204035969.png#pic_center)

在 NIO 中，Buffer 是一个顶层父类，它是一个抽象类，常用的 Buffer 子类有：

- ByteBuffer，存储字节数据到缓冲区
- ShortBuffer，存储字符串数据到缓冲区
- CharBuffer，存储字符数据到缓冲区
- IntBuffer，存储整数数据到缓冲区
- LongBuffer，存储长整型数据到缓冲区
- DoubleBuffer，存储小数到缓冲区
- FloatBuffer，存储小数到缓冲区

对于 Java 中的基本数据类型，都有一个 Buffer 类型与之相对应，最常用的自然是ByteBuffer 类（二进制数据），该类的主要方法如下所示：

```java
public abstract ByteBuffer put(byte[] b); 存储字节数据到缓冲区
public abstract byte[] get(); 从缓冲区获得字节数据
public final byte[] array(); 把缓冲区数据转换成字节数组
public static ByteBuffer allocate(int capacity); 设置缓冲区的初始容量
public static ByteBuffer wrap(byte[] array); 把一个现成的数组放到缓冲区中使用
public final Buffer flip(); 翻转缓冲区，重置位置到初始位
```

### Channel

- 通道：类似于 BIO 中的 stream，例如 FileInputStream 对象，用来建立到目标（文件，网络套接字，硬件设备等）的一个连接，但是需要注意：BIO 中的 stream 是单向
  的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道(Channel)是双向的，既可以用来进行读操作，也可以用来进行写操作。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919204133165.png#pic_center)

 

常用的 Channel 类有：FileChannel、DatagramChannel、ServerSocketChannel 和 SocketChannel。

- FileChannel 用于文件的数据读写，
- DatagramChannel 用于 UDP 的数据读写，
- ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。

```java
public int read(ByteBuffer dst) ，从通道读取数据并放到缓冲区中
public int write(ByteBuffer src) ，把缓冲区的数据写到通道中
public long transferFrom(ReadableByteChannel src, long position, long count)，从目标通道中复制数据到当前通道
public long transferTo(long position, long count, WritableByteChannel target)，把数据从当前通道复制给目标通道
```

### **Selector**

- 选择器

  能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也
  就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919204200107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzc3OTc4,size_16,color_FFFFFF,t_70#pic_center)

- 该类的常用方法如下所示：

  ```java
  public static Selector open()，得到一个选择器对象
  public int select(long timeout)，监控所有注册的通道，当其中有 IO 操作可以进行时，将对应的SelectionKey 加入到内部集合中并返回，参数用来设置超时时间
  
  public Set<SelectionKey> selectedKeys()，从内部集合中得到所有的 SelectionKey
  ```

- SelectionKey，代表了 Selector 和网络通道的注册关系,一共四种：

  ```java
  int OP_ACCEPT：有新的网络连接可以 accept，值为 16
  int OP_CONNECT：代表连接已经建立，值为 8
  int OP_READ 和 int OP_WRITE：代表了读、写操作，值为 1 和 4 该类的常用方法如下所示：
  
  public abstract Selector selector()，得到与之关联的 Selector 对象
  public abstract SelectableChannel channel()，得到与之关联的通道
  public final Object attachment()，得到与之关联的共享数据
  public abstract SelectionKey interestOps(int ops)，设置或改变监听事件
  public final boolean isAcceptable()，是否可以 accept
  public final boolean isReadable()，是否可以读
  public final boolean isWritable()，是否可以写
  ```

- ServerSocketChannel，用来在服务器端监听新的客户端 Socket 连接，常用方法如下所示：

  ```java
  public static ServerSocketChannel open()，得到一个 ServerSocketChannel 通道
  public final ServerSocketChannel bind(SocketAddress local)，设置服务器端端口号
  public final SelectableChannel configureBlocking(boolean block)，设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
  public SocketChannel accept()，接受一个连接，返回代表这个连接的通道对象
  public final SelectionKey register(Selector sel, int ops)，注册一个选择器并设置监听事件
  ```

- SocketChannel，网络 IO 通道，具体负责进行读写操作。NIO 总是把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。常用方法如下所示：

  ```java
  public static SocketChannel open()，得到一个 SocketChannel 通道
  public final SelectableChannel configureBlocking(boolean block)，设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
  public boolean connect(SocketAddress remote)，连接服务器
  public boolean finishConnect()，如果上面的方法连接失败，接下来就要通过该方法完成连接操作
  public int write(ByteBuffer src)，往通道里写数据
  public int read(ByteBuffer dst)，从通道里读数据
  public final SelectionKey register(Selector sel, int ops, Object att)，注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
  public final void close()，关闭通道
  ```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919204256836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzc3OTc4,size_16,color_FFFFFF,t_70#pic_center)

 



## **IO 对比总结**

IO 的方式通常分为几种：同步阻塞的 BIO、同步非阻塞的 NIO、异步非阻塞的 AIO。

- BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。
- NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。
- AIO 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用 OS 参与并发操作，编程比较复杂，JDK7 开始支持。

**举个例子再理解一下：**

- 同步阻塞：你到饭馆点餐，然后在那等着，啥都干不了，饭馆没做好，你就必须等着！
- 同步非阻塞：你在饭馆点完餐，就去玩儿了。不过玩一会儿，就回饭馆问一声：好了没啊！
- 异步非阻塞：饭馆打电话说，我们知道您的位置，一会给你送过来，安心玩儿就可以了，类似于现在的外卖。

| 对比总结     | BIO      | NIO                    | AIO        |
| ------------ | -------- | ---------------------- | ---------- |
| IO 方式      | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| API 使用难度 | 简单     | 复杂                   | 复杂       |
| 可靠性       | 差       | 好                     | 好         |
| 吞吐量       | 低       | 高                     | 高         |

## Netty

Netty 是一个基于 NIO 的网络编程框架，使用 Netty 可以帮助你快速、简单的开发出一个网络应用，相当于简化和流程化了 NIO 的开发过程。作为当前最流行的 NIO 框架，Netty 在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，知名的 Elasticsearch 、Dubbo 框架内部都采用了 Netty。

### **Netty和Tomcat的区别：**

 最大的区别在于通信协议，Tomcat是基于Http协议的，他的实质是一个基于http协议的web容器，但是Netty不一样，他能通过编程定义各种协议，因为netty能够通过codec自己来编码/解码字节流，完成类似redis访问的功能，这就是netty和tomcat的最大不同。

### **Netty三大特性：**

1.并发高
​ Netty是一款基于NIO（Nonblocking IO），非阻塞开发的网络通信框架，对比BIO(Blocking IO),他的并发性能得到了很大的提高。

2.传输快
​ Java的内存有堆内存、栈内存和字符串常量池等等，其中堆内存是占用内存空间最大的一块，也是Java存放对象的地方。一般我们的数据如果需要IO读取到堆内存，中间需要经过Socket的缓冲区，也就是说一个数据会被拷贝两次才能到达他的终点，如果数据量大，就会造成不必要的资源浪费。

 Netty针对这种情况，使用了NIO中的另一大特性–零拷贝，当他需要接受数据时，他就会直接从IO读到了那块内存中去，在netty里面通过ByteBuf可以直接对这些数据进行直接操作，从而加快了传输速度。

3.封装好
​ 经过BIO，NIO，Netty分别实现网络编程代码，区别非常明显

## **核心API**

### ChannelHandler 及其实现类

 ChannelHandler 接口定义了许多事件处理的方法，我们可以通过重写这些方法去实现具体的业务逻辑。API 关系如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919204343316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzc3OTc4,size_16,color_FFFFFF,t_70#pic_center)

 

 我们经常需要自定义一个 Handler 类去继承ChannelHandlerAdapter，然后通过

重写相应方法实现业务逻辑，我们接下来看看一般都需要重写哪些方法：

```java
 public void channelActive(ChannelHandlerContext ctx)，通道就绪事件
 public void channelRead(ChannelHandlerContext ctx, Object msg)，通道读取数据事件
 public void channelReadComplete(ChannelHandlerContext ctx) ，数据读取完毕事件
 public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)，通道发生异常事件
```



### ChannelPipeline

 ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于一个贯穿 Netty 的链。

```java
ChannelPipeline addFirst(ChannelHandler... handlers)，把一个业务处理类（handler）添加到链中的第一个位置
ChannelPipeline addLast(ChannelHandler... handlers)，把一个业务处理类（handler）添加到链中的最后一个位置
```



### ChannelHandlerContext

 这是事件处理器上下文对象，Pipeline 链中的实际处理节点。每个处理节点ChannelHandlerContext 中 包含 一 个 具 体 的 事 件 处 理 器 ChannelHandler ， 同时ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler进行调用。常用方法如下所示：

```java
ChannelFuture close()，关闭通道
ChannelOutboundInvoker flush()，刷新
ChannelFuture writeAndFlush(Object msg)，将 数 据 写 到 ChannelPipeline中当前ChannelHandler 的下一个 ChannelHandler 开始处理（出站）
```



### ChannelOption

Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数。ChannelOption 是
Socket 的标准参数，而非 Netty 独创的。常用的参数配置有：

```java
1. ChannelOption.SO_BACKLOG
      对应 TCP/IP 协议 listen 函数中的 backlog 参数，用来初始化服务器可连接队列大小。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog 参数指定了队列的大小。
2. ChannelOption.SO_KEEPALIVE ，一直保持连接活动状态。
```



### **ChannelFuture**

 表示 Channel 中异步 I/O 操作的结果，在 Netty 中所有的 I/O 操作都是异步的，I/O 的调用会直接返回，调用者并不能立刻获得结果，但是可以通过 ChannelFuture 来获取 I/O 操作的处理状态。
常用方法如下所示：

```java
 Channel channel()，返回当前正在进行 IO 操作的通道



 ChannelFuture sync()，等待异步操作执行完毕
```



### **Unpooled**

 这是 Netty 提供的一个专门用来操作缓冲区的工具类，常用方法如下所示：

public static ByteBuf copiedBuffer(CharSequence string, Charset charset)，通过给定的数据和字符编码返回一个 ByteBuf 对象（类似于 NIO 中的 ByteBuffer 对象）
1

#### EventLoopGroup 和其实现类 NioEventLoopGroup

 EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个EventLoop 同时工作，每个 EventLoop 维护着一个 Selector 实例。
EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop来处理任务。在 Netty 服务器端编程中，我们一般都需要提供两个 EventLoopGroup，例如：BossEventLoopGroup 和 WorkerEventLoopGroup。

***\* 通常一个服务端口即一个ServerSocketChannel对应一个Selector和一个EventLoop线程。\****BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919203928342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzc3OTc4,size_16,color_FFFFFF,t_70#pic_center)
​ BossEventLoopGroup 通常是一个单线程的 EventLoop，EventLoop 维护着一个注册了
ServerSocketChannel 的 Selector 实例，BossEventLoop 不断轮询 Selector 将连接事件分离出来，
通常是 OP_ACCEPT 事件，然后将接收到的 SocketChannel 交给 WorkerEventLoopGroup，
WorkerEventLoopGroup 会由 next 选择其中一个 EventLoopGroup 来将这个 SocketChannel 注
册到其维护的 Selector 并对其后续的 IO 事件进行处理。

 

常用方法如下所示：

```scss
 public NioEventLoopGroup()，构造方法



 public Future<?> shutdownGracefully()，断开连接，关闭线程
```



#### ServerBootstrap 和 Bootstrap

ServerBootstrap 是 Netty 中的服务器端启动助手，通过它可以完成服务器端的各种配置；常用方法如下
所示：

```scss
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个 EventLoopB   channel(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现

public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)，用来给接收到的通道添加配置
public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的 handler）
public ChannelFuture bind(int inetPort) ，该方法用于服务器端，用来设置占用的端口号
```


Bootstrap 是 Netty 中的客户端启动助手，通过它可以完成客户端的各种配置。常用方法如下所示：

```java
public B  group(EventLoopGroup group) ，该方法用于客户端，用来设置一个 EventLooppublic B  channel(Class<? extends C> channelClass)，该方法用来设置一个客户端的通道实现
public <T> B option(ChannelOption<T> option, T value)，用来给 ServerChannel 添加配置
public B handler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的 handler）
public ChannelFuture connect(String inetHost, int inetPort) ，该方法用于客户端，用来连接服务器端
```


## NettyServer

```java
public class NettyServer {
        public static void main(String[] args) throws Exception {
            //1.创建一个线程组：用来处理网络事件（接受客户端连接）
            EventLoopGroup bossGroup = new NioEventLoopGroup();
            //2.创建一个线程组：用来处理网络事件（处理通道 IO 操作）
            EventLoopGroup workerGroup = new NioEventLoopGroup();
            //3.创建服务器端启动助手来配置参数
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup) //4.设置两个线程组 EventLoopGroup
                    .channel(NioServerSocketChannel.class) //5.使用 NioServerSocketChannel 作为服务器端通道实现
                    .option(ChannelOption.SO_BACKLOG, 128) //6.设置线程队列中等待连接的个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true) //7.保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() { //8.创建一个通道初始化对象
                        public void initChannel(SocketChannel sc) { //9.往 Pipeline 链中添加自定义的业务处理 handler
                            sc.pipeline().addLast(new NettyServerHandler()); //服务器端业务处理类
                     //NettyServerHandler,继承自ChannelHandlerAdapter，自定义channelRead等方法
                            System.out.println(".......Server is ready.......");
                        }
                    });
            //10.启动服务器端并绑定端口，等待接受客户端连接(非阻塞)
            ChannelFuture cf = b.bind(9999).sync();
            System.out.println("......Server is Starting......");
            //11.关闭通道，关闭线程池
            cf.channel().closeFuture().sync();
           //12.优雅的退出线程组
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
```



## NettyClient

```java
public class NettyClient {
        public static void main(String[] args) throws Exception {
            //1.创建一个 EventLoopGroup 线程组
            EventLoopGroup group = new NioEventLoopGroup();
            //2.创建客户端启动助手
            Bootstrap b = new Bootstrap();
            b.group(group) //3.设置 EventLoopGroup 线程组
                    .channel(NioSocketChannel.class) //4.使用 NioSocketChannel 作为客户端通道实现
                    .handler(new ChannelInitializer<SocketChannel>() { //5.创建一个通道初始化对象
                        @Override
                        protected void initChannel(SocketChannel sc) { //6.往 Pipeline 链中添加自定义的业务处理 handler
                            sc.pipeline().addLast(new NettyClientHandler()); //客户端业务处理类
                       //NettyServerHandler,继承自ChannelHandlerAdapter，自定义channelRead等方法
                            System.out.println("......Client is ready.......");
                        }
                    });
            //7.启动客户端,等待连接上服务器端(非阻塞)
            ChannelFuture cf = b.connect("127.0.0.1", 9999).sync();
            //8.等待连接关闭(非阻塞)
            cf.channel().closeFuture().sync();
        }
    }
```

