# Netty

## NIO

NIO三大组件:Selector,Channel,Buffer

### 1.Buffer类及其子类

Buffer类定义了缓冲区都有的四个属性来提供关于其所包含的数据元素信息:

1. Capacity:容量,即可以容纳的最大数据量,在缓冲区创建时被设定且不嗯呢被改变
2. Limit:表示缓冲区的当前终点,不能对缓冲区超过极限的位置进行读写操作.且极限是可以被修改的
3. Position:位置,下一个要被读或写的元素索引,每次读写缓冲区数据时都会改变该值,为下次读写作准备
4. Mark:标记

Buffer类常用函数:

```java
put();//向缓冲数组中放入元素,从当前位置上添加,put之后,position会自动+1
put(int index);//从绝对位置上put
get();//从当前位置position上ger,get之后,position会自动+1
get(int index)//从绝对位置上get
flip();//反转缓冲数组
position(int index);将position指针指向index
```

> 缓冲区(Buffer)

基本介绍:

```txt
缓冲区本质上是一个可以读写数据的内存块,可以理解成是一个容器对象(含数组),该对象提供了一组方法,可以更轻松的使用内存块,缓冲区对象内置了一些机制,能够跟踪好记录缓冲区的状态变化情况.Channel提供从文件,网络读取数据的渠道,但是读取或写入的数据都必须经由Buffer
```

### 2.通道(Channel)

> 基本介绍

- NIO的通道类似于流,但有如下区别:
  1. 通道可以同时进行读写,而流只能读或者写
  2. 通道可以实现异步读写数据
  3. 通道可以从缓冲读数据,也可以写数据到缓冲
- BIO中的stream是单向的,NIO中的通道是单向的,可读可写
- Channel在NIO中是一个接口
  Channel extends Closeable{}
- 常用的Channel类有:FileChannel,DaagramChannel,ServerSocketChannel和SocketChannel..ServerSocketChannel和ServerSocket类似,SocketChannel和Socket类似

### 3.关于Buffer和Channel的注意事项和细节

1. ByteBuffer 支持类型化的put和get,put放入的是什么数据类型,get就应该使用相应的数据类型来取出,否则可能会抛出异常
2. 可以将一个普通的Buffer 转成只读 Buffer
3. NIO还提供了MappedByteBuffer,可以让文件直接在内存中进行修改,而如何同步到文件由NIO来完成
4. NIO还支持通过多个Buffer(即Buffer数组)完成读写操作,即Scattering和Gathering

> MappedByteBuffer使用

```java
RandomAccessFile randomAccessFile = new RandomAccessFile("1.txt", "rw");
//获取对应的通道
FileChannel channel = randomAccessFile.getChannel(  );
/**
         * 优点:可以让文件直接在内存(堆外内存)修改,操作系统不需要拷贝一次,效率较高
         * 参数一:MapMode.READ_WRITE 使用的是读写模式
         * 参数二: 0 可以直接修改的起始位置
         * 参数三: 5 是映射到内存的大小(不是索引位置),即将 1.txt 的多少个字节映射到内存
         * 可以直接修改的范围是 0 - 5
*/
MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
mappedByteBuffer.put(0,(byte)'H');
mappedByteBuffer.put(3,(byte)'9');

randomAccessFile.close();
System.out.println("修改成功");
```

> Buffer的分散和聚集

```java
/**
         * Scattering: 将数据写入到Buffer中时,可以采用Buffer数组,以此写入[分散]
         * Gatering: 从Buffer中读取数据时,可以采用buffer数组,以此读
         */

//使用ServerSocketChannel 和 SocketChannel 网络
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);

//绑定端口到socket,并启动
serverSocketChannel.socket().bind(inetSocketAddress);

//创建一个buffer数组(服务器端)
ByteBuffer[] byteBuffers = new ByteBuffer[2];
byteBuffers[0] = ByteBuffer.allocate(5);
byteBuffers[1] = ByteBuffer.allocate(3);

//等待客户端连接(telnet)
SocketChannel socketChannel = serverSocketChannel.accept();
int messageLength = 8;//假定从客户端接收8个字节

//循环的读取
while (true){
    int byteRead = 0;
    while (byteRead < messageLength) {
        long l = socketChannel.read(byteBuffers);
        byteRead += l;//累计读取的字节数
        System.out.println(byteRead);
        //使用流打印,看看当前的buffer的position 和limit
        Arrays.asList(byteBuffers).stream()
            .map(buffer -> "position=" + buffer.position() + " limit=" + buffer.limit())
            .forEach(System.out::println);
    }

    //将所有的buffer进行flip,开始写操作
    Arrays.asList(byteBuffers).forEach(buffer->buffer.flip());

    //将数据独处显示到客户端
    long byteWirte = 0;
    while (byteWirte<messageLength){
        long l = socketChannel.write(byteBuffers);//
        byteWirte+=l;
    }

    //将所有的buffer 进行clear
    Arrays.asList(byteBuffers).forEach(buffer->{buffer.clear();});
    System.out.println("byteRead:="+byteRead+" byteWrite="+byteWirte+" messageLength="+ messageLength);
```

### 4.Selector

> 基本介绍

1. Java的NIO,用非阻塞的IO方式.可以用一个线程,处理多个的客户端连接,就会使用到Selector
2. Selector能够监测多个注册的通道上是否有事件发生(**注意:多个Channel以事件的方式可以注册到同一个Selector**).如果有事件发生,便获取事件,然后针对每个事件进行相应的处理.这样就可以只用一个单线程去管理多个通道,也就是管理多个连接和请求
3. 只有在连接真正有读写时间发生时,才会进行读写,就打打地减少了系统开销,并且不必为每个连接都创建一个线程,不用去维护多个线程
4. 避免了多线程之间的上下文切换导致的开销

> Selector类的相关方法

Selector是一个抽象类:

public static Selector open();//得到一个选择器对象
public int select(long timeout);//监控所有注册的通道,当其中有IO操作可以进行时,将对应的SelectionKey加入到内部集合中并返回,参数用来设置超时时间
public Set<SelectionKey> selectedKeys();//从内部集合中得到所有的SelectionKey

> 网络编程原理分析

1. 当客户端连接时,会通过ServerSocketCHannel得到SocketChannel
2. 将socketChannel注册到Selector上,register(Selector sel,int ops),一个selector上可以注册多个SocketChannel
3. 注册后返回一个SelectionKey,回合该Selector关联(集合)
4. Selector进行监听select方法,返回有事件发生的通道的个数

### 5.NIO入门程序

> 服务端

```java
package NIO;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

/**
 * @author gzy
 * @create 2021-02-05-10:09
 */
public class NIOServer {
    public static void main(String[] args) throws IOException {
        //创建ServerSocketChannel 等价于  ServerSocket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        //得到一个Selector对象
        Selector selector = Selector.open();

        //绑定一个端口6666,在服务器端监听
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        //设置为非阻塞75
        serverSocketChannel.configureBlocking(false);

        //把serverSocketChannel 注册到  selector 关心 事件为  OP_ACCEPT
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        //循环等待客户端连接
        while(true){

            //这里等待1秒,如果没有事件发生(连接事件),返回
            if(selector.select(1000)==0){//没有事件发生
                System.out.println("服务器等待了1秒,无连接");
                continue;
            }

            //如果返回的>0 ,就获取到相关的selectionKey集合
            //1.如果返回的>0 ,表示已经获取到关注的事件
            //2.selector.selectedKeys() 返回关注事件的集合
            //通过 selectionKeys 可以反向获取通道
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            //遍历 Set<SelectionKey>,使用迭代器遍历
            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
            while (keyIterator.hasNext()){

                //获取到 selectionKey
                SelectionKey key = keyIterator.next();

                //根据key 对应的通道发生的事件做相应处理
                if (key.isAcceptable()){

                    //如果是 OP_ACCEPT,有新的客户端连接
                    //给该客户端生成一个SocketChannel,等待
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("客户端连接成功,生成了一个socketChannel::"+socketChannel.hashCode());

                    //将socketChannel设置为非阻塞
                    socketChannel.configureBlocking(false);

                    //将socketChannel 注册到 selector,关注事件为 OP_READ ,同时给该socketChannel 关联一个Buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }
                if (key.isReadable()){

                    //如果是OP_READ,表示可读
                    //通过key,反向获取到对应channel
                    SocketChannel channel = (SocketChannel)key.channel();

                    //获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer)key.attachment();
                    channel.read(buffer);
                    System.out.println("form客户端"+new String(buffer.array()));
                }

                //手动从集合中移动当前的SelectionKey,防止重复操作
                keyIterator.remove();
            }
        }
    }
}

```

> 客户端

```java
package NIO;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;

/**
 * @author gzy
 * @create 2021-02-06-9:47
 */
public class NIOClient {
    public static void main(String[] args) throws IOException {
        //得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();

        //设置非阻塞
        socketChannel.configureBlocking(false);
        //提供服务器端的ip和端口
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 6666);

        //连接服务器
        if (socketChannel.connect(inetSocketAddress)){
            //如果连接成功,可以发送数据
            String str="hello";
            //wrap方法产生一个字节数组到buffer
            ByteBuffer buffer = ByteBuffer.wrap(str.getBytes(StandardCharsets.UTF_8));
            //发送数据,将buffer数据写入到channel
            socketChannel.write(buffer);
            System.in.read();
        }else{
            //如果连接不成功
            while (!socketChannel.finishConnect()){
                System.out.println("因为连接需要时间,客户端不会阻塞,可以做其他工作");
            }
        }
    }
}

```

### 6.NIO群聊系统

### 7.NIO与零拷贝

> 零拷贝基本介绍

1. 零拷贝是网络编程的关键,很多性能优化都离不开
2. 在Java程序中,常用的零拷贝有mmap(内存映射) 和 sendFile

## 2.Netty

2.1Reactor 模式

> 线程模型的基本介绍

1. 不同的线程模式,对程序的性能有很大影响
2. 目前存在的线程模型有:
   传统阻塞IO服务模型
   Reactor模式
3. 根据Reactor的数量和处理资源池线程的数量不同,有三种典型的实现
   - 单Reactor单线程
   - 单Reactor多线程
   - 主从Reactor多线程
4. Netty线程模式(Netty主要基于主从Reactor多线程模型做了一定的改进,其中主从Reactor多线程模型有多个Reactor)

> 针对传统阻塞IO模式的两个缺点,提出解决方案

1. 基于I/O复用模型:多个连接共用一个阻塞对象,应用程序只需要在一个阻塞对象等待,无需阻塞等待.当某个连接有新的数据可以处理时,操作系统通知应用程序,线程从阻塞状态返回,开始进行业务处理
2. 基于线程池复用线程资源:不必再为每个连接创建线程,将连接完成后的业务处理任务分配给线程进行处理,一个线程可以处理多个连接的业务

> IO复用结合线程池,就是Reactor模式基本设计思想
> 说明:
>
> 1. Reactor模式,通过一个或多个输入同时传递给服务处理器的模式(基于事件驱动)
> 2. 服务器端程序处理传入的多个请求,并将它们同步分派到相应的处理线程,因此Reactor模式也叫Dispatcher模式
> 3. Reactor模式使用IO复用监听事件,收到事件后,分发给某个线程(进程),这点就是网络服务高并发处理关键

