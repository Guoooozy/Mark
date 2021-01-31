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

