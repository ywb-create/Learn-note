# IO与Netty

## 一.IO模型

### 1.BIO

#### 1.说明

​	同步并阻塞(**传统阻塞型**)，服务器实现模式为**一个连接一个线程**，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销

#### 2.图示

![image-20200517214721510](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200517214721510.png)

#### 3.流程

1. 服务器端启动一个ServerSocket
2. 客户端启动Socket对服务器进行通信，默认情况下服务器端需要对**每个客户 建立一个线程**与之通讯
3. 客户端发出请求后, 先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝
4. 如果有响应，客户端线程会等待请求结束后，在继续执行

#### 4.应用实例

```java
/*
	监听端口6666，如果有客户端连接，就创建一个线程，与之通讯
*/
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class BIOServer {
    public static void main(String[] args) throws Exception {

        //线程池机制

        //思路
        //1. 创建一个线程池
        //2. 如果有客户端连接，就创建一个线程，与之通讯(单独写一个方法)

        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();

        //创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);

        System.out.println("服务器启动了");

        while (true) {

            System.out.println("线程信息 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());
            //监听，等待客户端连接
            System.out.println("等待连接....");
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");

            //就创建一个线程，与之通讯(单独写一个方法)
            newCachedThreadPool.execute(new Runnable() {
                public void run() { //我们重写
                    //可以和客户端通讯
                    handler(socket);
                }
            });

        }


    }

    //编写一个handler方法，和客户端通讯
    public static void handler(Socket socket) {

        try {
            System.out.println("线程信息 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            //通过socket 获取输入流
            InputStream inputStream = socket.getInputStream();

            //循环的读取客户端发送的数据
            while (true) {

                System.out.println("线程信息 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());

                System.out.println("read....");
               int read =  inputStream.read(bytes);
               if(read != -1) {
                   System.out.println(new String(bytes, 0, read
                   )); //输出客户端发送的数据
               } else {
                   break;
               }
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            System.out.println("关闭和client的连接");
            try {
                socket.close();
            }catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
}

```

#### 5.产生问题

1. 每个请求都需要**创建独立的线程**，与对应的客户端进行数据 Read，业务处理，数据 Write 
2. 当并发数较大时，需要**创建大量线程来处理连接**，系统资源占用较大。
3. 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在 Read 操作上，造成线程资源浪费

### 2.NIO

#### 1.说明

​	**同步非阻塞**，服务器实现模式为**一个线程处理多个请求**(连接)，即客户端发送的连接请求都会**注册到多路复用器**上，多路复用器轮询到连接有I/O请求就进行处理 

​	Java NIO有三大核心组件，Selector、Channel、Buffer

#### 2.图示

![image-20200514164319744](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200514164319744.png)

#### 3.核心组件

##### 1.关系图示

![image-20200514164432702](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200514164432702.png)

 说明：

1. 每个channel 都会对应一个Buffer
2. Selector 对应一个线程， 一个线程对应多个channel(连接)
3. 该图反应了有三个channel 注册到 该selector 
4. 程序切换到哪个channel 是有事件决定的, Event 就是一个重要的概念
5. Selector 会根据不同的事件，在各个通道上切换
6. Buffer 就是一个内存块 ， 底层是有一个数组
7. 数据的读取写入是通过Buffer, 这个和BIO , BIO 中要么是输入流，或者是
    输出流, 不能双向，但是NIO的Buffer 是可以读也可以写, 需要 flip 方法切换
8. channel 是双向的, 可以返回底层操作系统的情况, 比如Linux ， 底层的操作系统
    通道就是双向的.

##### 2.buffer

###### 1.概述

​	缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个**容器对象**(**含数组**)，该对象提供了一组方法，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer。

###### 2.属性

 mark <= position <= limit <= capacity

| **属性** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| Capacity | 容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变 |
| Limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的 |
| Position | 位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变改值，为下次读写作准备 |
| Mark     | 标记                                                         |

###### 3.api

```java
	public final int capacity( )//返回此缓冲区的容量
  public final int position( )//返回此缓冲区的位置
  public final Buffer position (int newPositio)//设置此缓冲区的位置
  public final int limit( )//返回此缓冲区的限制
  public final Buffer limit (int newLimit)//设置此缓冲区的限制
  public final Buffer mark( )//在此缓冲区的位置设置标记
  public final Buffer reset( )//将此缓冲区的位置重置为以前标记的位置
  public final Buffer clear( )//清除此缓冲区, 即将各个标记恢复到初始状态，但是数据并没有真正擦除, 后面操作会覆盖
  public final Buffer flip( )//反转此缓冲区
  public final Buffer rewind( )//重绕此缓冲区
  public final int remaining( )//返回当前位置与限制之间的元素数
  public final boolean hasRemaining( )//告知在当前位置和限制之间是否有元素
  public abstract boolean isReadOnly( );//告知此缓冲区是否为只读缓冲区
  public abstract boolean hasArray();//告知此缓冲区是否具有可访问的底层实现数组
  public abstract Object array();//返回此缓冲区的底层实现数组
  public abstract int arrayOffset();//返回此缓冲区的底层实现数组中第一个缓冲区元素的偏移量
  public abstract boolean isDirect();//告知此缓冲区是否为直接缓冲区
```

###### 4.bytebuffer

```java
		public static ByteBuffer allocateDirect(int capacity)//创建直接缓冲区
    public static ByteBuffer allocate(int capacity)//设置缓冲区的初始容量
    public static ByteBuffer wrap(byte[] array)//把一个数组放到缓冲区中使用
    public static ByteBuffer wrap(byte[] array,int offset, int length)//构造初始化位置offset和上界length的缓冲区
      
     //缓存区存取相关API
    public abstract byte get( );//从当前位置position上get，get之后，position会自动+1
    public abstract byte get (int index);//从绝对位置get
    public abstract ByteBuffer put (byte b);//从当前位置上添加，put之后,position会自动+1
    public abstract ByteBuffer put (int index, byte b);//从绝对位置index上put



//使用OS内存创建直接缓冲区
ByteBuffer：
	public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }
    
	DirectByteBuffer(int cap) {
        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;
    }


```



##### 3.channel

###### 1.概述

NIO的通道类似于流，但有些区别如下：

- 通道可以**同时进行读写**，而流只能读或者只能写
- 通道可以实现**异步**读写数据
- 通道可以从缓冲读数据，也可以写数据到缓冲

常见Channel类

- FileChannel：用于文件的数据读写

- DatagramChannel：用于UDP的数据读写
- ServerSocketChannel ： 类似 ServerSocket，用于TCP的数据读写
-  SocketChannel： 类似Socket

###### 2.FileChannel

```java
//fileChannel主要用于对本地文件进行io读写

public int read(ByteBuffer dst) //从通道读取数据并放到缓冲区中
public int write(ByteBuffer src) //把缓冲区的数据写到通道中
public long transferFrom(ReadableByteChannel src, long position, long count)//从目标通道中复制数据到当前通道
public long transferTo(long position, long count, WritableByteChannel target)//把数据从当前通道复制给目标通道
```

示例：

**1.本地数据读写文件**

```java

//实现目的：将“hello” 写入a.txt
public class TestFileChannel{
	public static void main(String args[]){
    String str="hello";
    //创建一个输出流
    FileOutputStream os=new FileOutputStream("a.txt");
    //通过输出流获取对应的channel
    FileChannel fc=os.getChannel();
    //创建一个缓冲区
    ByteBuffer b=ByteBuffer.allocate(1024);
    //将str放入到缓冲区
    b.put(str.getByte());
    //对byteBuffer进行filp()
    b.filp();
    //将缓冲区数据写入到通道
    fc.write(b);
    //关闭通道
    fc.close();
  }
}

//实现目的：读取a.txt
public class TestFileChannel{
	public static void main(String args[]){
    String str="hello";
    //创建一个输入流
    File file=new File("a.txt");
    FileInputStream is=new FileInputStream(file);
    //通过输入流获取对应的channel
    FileChannel fc=is.getChannel();
    //创建一个缓冲区
    ByteBuffer b=ByteBuffer.allocate((int)file.length());
    //将通道中数据读到缓冲区
    fc.read(b);
    //将byteBuffer的字节数据转成String
    System.out.print(new String(b.array()));
    //关闭通道
    fc.close();
  }
}

//使用一个buffer完后数据读写
public class NIOFileChannel03 {
    public static void main(String[] args) throws Exception {

        FileInputStream fileInputStream = new FileInputStream("1.txt");
        FileChannel fileChannel01 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("2.txt");
        FileChannel fileChannel02 = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(512);

        while (true) { //循环读取

            //这里有一个重要的操作，一定不要忘了
            /*
             public final Buffer clear() {
                position = 0;
                limit = capacity;
                mark = -1;
                return this;
            }
             */
            byteBuffer.clear(); //清空buffer
            int read = fileChannel01.read(byteBuffer);
            System.out.println("read =" + read);
            if(read == -1) { //表示读完
                break;
            }
            //将buffer 中的数据写入到 fileChannel02 -- 2.txt
            byteBuffer.flip();
            fileChannel02.write(byteBuffer);
        }

        //关闭相关的流
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```



**2.拷贝文件**

```java
public class NIOFileChannel04 {
    public static void main(String[] args)  throws Exception {

      //创建相关流
      FileInputStream is = new FileInputStream("d:\\a.jpg");
      FileOutputStream os = new FileOutputStream("d:\\a2.jpg");

      //获取各个流对应的filechannel
      FileChannel sourceCh = is.getChannel();
      FileChannel destCh = os.getChannel();

      //使用transferForm完成拷贝
      destCh.transferFrom(sourceCh,0,sourceCh.size());
      //关闭相关通道和流
      sourceCh.close();
      destCh.close();
      is.close();
      os.close();
    }
}
```



**3.Scattering和Gathering：通过buffer数组完成数据读写**

```java
/**
 * Scattering：将数据写入到buffer时，可以采用buffer数组，依次写入  [分散]
 * Gathering: 从buffer读取数据时，可以采用buffer数组，依次读
 */
public class ScatteringAndGatheringTest {
    public static void main(String[] args) throws Exception {

        //使用 ServerSocketChannel 和 SocketChannel 网络
        ServerSocketChannel ssc = ServerSocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);

        //绑定端口到socket
        ssc.socket().bind(inetSocketAddress);

        //创建buffer数组
        ByteBuffer[] byteBuffers = new ByteBuffer[2];
        byteBuffers[0] = ByteBuffer.allocate(5);
        byteBuffers[1] = ByteBuffer.allocate(3);

        //等客户端连接(telnet)
        SocketChannel socketChannel = ssc.accept();
        int messageLength = 8;   //假定从客户端接收8个字节
        //循环的读取
        while (true) {
            int byteRead = 0;
            while (byteRead < messageLength ) {
                long l = socketChannel.read(byteBuffers);
                byteRead += l; //累计读取的字节数
                System.out.println("byteRead=" + byteRead);
                //使用流打印, 看看当前的这个buffer的position 和 limit
                Arrays.asList(byteBuffers).stream().map(buffer -> "postion=" + buffer.position() + ", limit=" + buffer.limit()).forEach(System.out::println);
            }
            //将所有的buffer进行flip
            Arrays.asList(byteBuffers).forEach(buffer -> buffer.flip());
            //将数据读出显示到客户端
            long byteWirte = 0;
            while (byteWirte < messageLength) {
                long l = socketChannel.write(byteBuffers); //
                byteWirte += l;
            }
            //将所有的buffer 进行clear
            Arrays.asList(byteBuffers).forEach(buffer-> {
                buffer.clear();
            });
            System.out.println("byteRead:=" + byteRead + " byteWrite=" + byteWirte + ", messagelength" + messageLength);
        }
    }
}

```



##### 4.selector

###### 1.概述

​	Selector能够检测**多个**注册的通道上**是否有事件发生**，多个Channel以事件的方式可以注册到同一个Selector)，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求

好处：

1. 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程
2. 避免了多线程之间的上下文切换导致的开销

###### 2.api

```java
public abstract class Selector implements Closeable { 
  public static Selector open();//得到一个选择器对象
  public int select(long timeout);//监控所有注册的通道，当其中有 IO 操作可以进行时，将对应的 SelectionKey 加入到内部集合中并返回，参数用来设置超时时间
  public Set<SelectionKey> selectedKeys();//从内部集合中得到所有的 SelectionKey
}

//select()说明
selector.select()//阻塞
selector.select(1000);//阻塞1000毫秒，在1000毫秒后返回
selector.wakeup();//唤醒selector
selector.selectNow();//不阻塞，立马返还
```



#### 4.NIO网络编程原理

##### 1.图示

![image-20200517164107722](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200517164107722.png)

##### 2.说明

1. 当客户端连接时，会通过ServerSocketChannel 得到 SocketChannel
2. Selector 进行监听  select 方法, 返回有事件发生的通道的个数.
3. 将socketChannel注册到Selector上, register(Selector sel, **int** ops), 一个selector上可以注册多个SocketChannel
4. 注册后返回一个 SelectionKey, 会和该Selector 关联(集合)
5. 进一步得到各个 SelectionKey (有事件发生)
6. 在通过 SelectionKey  反向获取 SocketChannel , 方法 channel()
7. 可以通过  得到的 channel  , 完成业务处理



##### 3.SelectionKey

​	SelectionKey表示 Selector 和网络通道的**注册关系**，共四种

- int OP_ACCEPT：有新的网络连接可以 accept，值为 16（**OP_ACCEPT = 1 << 4**）
- int OP_CONNECT：代表连接已经建立，值为 8（**OP_CONNECT = 1 << 3**）
- int OP_READ：代表读操作，值为 1 （**OP_READ = 1 << 0**）
- int OP_WRITE：代表写操作，值为 4（**OP_WRITE = 1 << 2**）

```java
//api
public abstract class SelectionKey {
  public abstract Selector selector();//得到与之关联的Selector对象
  public abstract SelectableChannel channel();//得到与之关联的通道
  public final Object attachment();//得到与之关联的共享数据
  public abstract SelectionKey interestOps(int ops);//设置或改变监听事件
  public final boolean isAcceptable();//是否可以 accept
  public final boolean isReadable();//是否可以读
  public final boolean isWritable();//是否可以写
}
```

##### 4.ServerSocketChannel

​	ServerSocketChannel 在**服务器端监听新的客户端** **Socket** **连接**

```java
//api
public abstract class ServerSocketChannel extends AbstractSelectableChannel implements NetworkChannel{
  public static ServerSocketChannel open();//得到一个 ServerSocketChannel 通道
  public final ServerSocketChannel bind(SocketAddress local);//设置服务器端端口号
  public final SelectableChannel configureBlocking(boolean block);//设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
  public SocketChannel accept();//接受一个连接，返回代表这个连接的通道对象
  public final SelectionKey register(Selector sel, int ops);//注册一个选择器并设置监听事件
}
```

##### 5.SocketChannel

​	SocketChannel，网络 IO 通道，**具体负责进行读写操作**。NIO 把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区

```java
//api
  public abstract class SocketChannel extends AbstractSelectableChannel implements ByteChannel, ScatteringByteChannel, GatheringByteChannel, NetworkChannel{
  public static SocketChannel open();//得到一个 SocketChannel 通道
  public final SelectableChannel configureBlocking(boolean block);//设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
  public boolean connect(SocketAddress remote);//连接服务器
  public boolean finishConnect();//如果上面的方法连接失败，接下来就要通过该方法完成连接操作
  public int write(ByteBuffer src);//往通道里写数据
  public int read(ByteBuffer dst);//从通道里读数据
  public final SelectionKey register(Selector sel, int ops, Object att);//注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
  public final void close();//关闭通道
}
```

##### 6.案例

```java
//服务端
public class NIOServer {
    public static void main(String[] args) throws Exception{

        //创建ServerSocketChannel -> ServerSocket

        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        //得到一个Selecor对象
        Selector selector = Selector.open();

        //绑定一个端口6666, 在服务器端监听
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        //设置为非阻塞
        serverSocketChannel.configureBlocking(false);

        //把 serverSocketChannel 注册到  selector 关心 事件为 OP_ACCEPT
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("注册后的selectionkey 数量=" + selector.keys().size()); // 1
        //循环等待客户端连接
        while (true) {

            //这里我们等待1秒，如果没有事件发生, 返回
            if(selector.select(1000) == 0) { //没有事件发生
                System.out.println("服务器等待了1秒，无连接");
                continue;
            }

            //如果返回的>0, 就获取到相关的 selectionKey集合
            //1.如果返回的>0， 表示已经获取到关注的事件
            //2. selector.selectedKeys() 返回关注事件的集合
            //   通过 selectionKeys 反向获取通道
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            System.out.println("selectionKeys 数量 = " + selectionKeys.size());

            //遍历 Set<SelectionKey>, 使用迭代器遍历
            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();

            while (keyIterator.hasNext()) {
                //获取到SelectionKey
                SelectionKey key = keyIterator.next();
                //根据key 对应的通道发生的事件做相应处理
                if(key.isAcceptable()) { //如果是 OP_ACCEPT, 有新的客户端连接
                    //该该客户端生成一个 SocketChannel
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("客户端连接成功 生成了一个 socketChannel " + socketChannel.hashCode());
                    //将  SocketChannel 设置为非阻塞
                    socketChannel.configureBlocking(false);
                    //将socketChannel 注册到selector, 关注事件为 OP_READ， 同时给socketChannel关联一个Buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));

                    System.out.println("客户端连接后 ，注册的selectionkey 数量=" + selector.keys().size());
                }
                if(key.isReadable()) {  //发生 OP_READ

                    //通过key 反向获取到对应channel
                    SocketChannel channel = (SocketChannel)key.channel();

                    //获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer)key.attachment();
                    channel.read(buffer);
                    System.out.println("form 客户端 " + new String(buffer.array()));

                }

                //手动从集合中移动当前的selectionKey, 防止重复操作
                keyIterator.remove();
            }
        }
    }
}


//客户端
public class NIOClient {
    public static void main(String[] args) throws Exception{

        //得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();
        //设置非阻塞
        socketChannel.configureBlocking(false);
        //提供服务器端的ip 和 端口
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 6666);
        //连接服务器
        if (!socketChannel.connect(inetSocketAddress)) {

            while (!socketChannel.finishConnect()) {
                System.out.println("因为连接需要时间，客户端不会阻塞，可以做其它工作..");
            }
        }
        //如果连接成功，就发送数据
        String str = "hello";
        //Wraps a byte array into a buffer
        ByteBuffer buffer = ByteBuffer.wrap(str.getBytes());
        //发送数据，将 buffer 数据写入 channel
        socketChannel.write(buffer);
        System.in.read();
    }
}

```

#### 5.NIO与零拷贝

##### 1.概述

​	零拷贝的指计算机操作的过程中，**CPU不需要为数据在内存之间的拷贝消耗资源**。而它通常是指计算机在网络上发送文件时，不需要将文件内容拷贝到用户空间（User Space）而直接在内核空间（Kernel Space）中传输到网络的方式

​	在 Java 程序中，**常用的零拷贝有 mmap(内存映射) 和 sendFile。**Java中FileChannel的transferTo()就使用到了零拷贝【**sendfile()**】；MappedByteBuffer【直接内存：**mmap**】申请堆外内存，因是堆外内存所以不受Minor GC控制，只能在发生Full GC时才能被回收。而其子类DirectByteBuffer，维护一个Cleaner对象来完成内存回收，既可以通过Full GC来回收内存，也可以调用clean()方法来进行回收。

##### 2.好处

- 减少甚至完全避免CPU拷贝，从而让CPU解脱出来去执行其他的任务
- 减少内存带宽的占用
- 减少用户空间和操作系统内核空间之间的上下文切换

##### 3.具体实现

###### 1.传统io

```java
File file = new File("index.html");
RandomAccessFile raf = new RandomAccessFile(file, "rw");

byte[] arr = new byte[(int) file.length()];
raf.read(arr);

Socket socket = new ServerSocket(8080).accept();
socket.getOutputStream().write(arr);
```

以上传统io代码对应的linux底层数据拷贝：

![零拷贝1](/Users/yinwenbo/Documents/img/零拷贝1.jpeg)

说明

1. read 调用导致用户态到内核态的一次变化，同时，第一次复制开始：DMA（Direct Memory Access，直接内存存取，即不使用 CPU 拷贝数据到内存，而是 DMA 引擎传输数据到内存，用于解放 CPU） 引擎从磁盘读取 index.html 文件，并将数据放入到内核缓冲区。
2. 发生第二次数据拷贝，即：将内核缓冲区的数据拷贝到用户缓冲区，同时，发生了一次用内核态到用户态的上下文切换。
3. 发生第三次数据拷贝，我们调用 write 方法，系统将用户缓冲区的数据拷贝到 Socket 缓冲区。此时，又发生了一次用户态到内核态的上下文切换。
4. 第四次拷贝，数据异步的从 Socket 缓冲区，使用 DMA 引擎拷贝到网络协议引擎。这一段，不需要进行上下文切换。
5. write 方法返回，再次从内核态切换到用户态

###### 2.mmap

​	mmap 通过**内存映射**，将文件映射到内核缓冲区，同时，**用户空间可以共享内核空间的数据**。这样，在进行网络传输时，就可以减少内核空间到用户控件的拷贝次数。

![零拷贝2](/Users/yinwenbo/Documents/img/零拷贝2.jpeg)

​	user buffer 和 kernel buffer 共享 index.html。如果你想把硬盘的 index.html 传输到网络中，再也不用拷贝到用户空间，再从用户空间拷贝到 Socket 缓冲区。

​	现在，只需要从内核缓冲区拷贝到 Socket 缓冲区即可，这将减少一次内存拷贝（从 4 次变成了 3 次），但不减少上下文切换次数。

###### 3.sendFile()

​	数据不经过用户态，直接从内核缓冲区进入到 Socket Buffer，同时，由于和用户态完全无关，就减少了一次上下文切换。数据经过了 3 次拷贝，3 次上下文切换。

![零拷贝3](/Users/yinwenbo/Documents/img/零拷贝3.jpeg)



###### 4.sendFile()升级版

​	Linux 在 2.4 版本中，做了一些修改，避免了从内核缓冲区拷贝到 Socket buffer 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝

![零拷贝4](/Users/yinwenbo/Documents/img/零拷贝4.jpeg)

​	文件进入到网络协议栈，只需 2 次拷贝：第一次使用 DMA 引擎从文件拷贝到内核缓冲区，第二次从内核缓冲区将数据拷贝到网络协议栈；内核缓存区只会拷贝一些 offset 和 length 信息到 SocketBuffer，基本无消耗。

#### 6.NIO与BIO比较

1. BIO 以流的方式处理数据,而 NIO 以块的方式处理数据,块 I/O 的效率比流 I/O 高很多
2. BIO 是阻塞的，NIO 则是非阻塞的
3. BIO基于字节流和字符流进行操作，而 NIO 基于 Channel(通道)和 Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择器)用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用**单个线程就可以监听多个客户端**通道 

### 3.AIO

#### 1.说明

​	**异步非阻塞**，AIO引入异步通道的概念，采用Proactor模式，简化了程序的编写，有效的请求才启动线程，它的特点是现有操作系统完成后才通知服务端程序启动线程取处理，一般适用于**连接数比较多且连接时间长**的应用。

### 4.应用场景及对比

#### 1.对比

|          | **BIO**  | **NIO**                | **AIO**    |
| -------- | -------- | ---------------------- | ---------- |
| IO 模型  | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

#### 2.应用场景

​	BIO方式适用于**连接数目比较小且固定的架构**，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。

​	NIO方式适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK1.4开始支持。

​	AIO方式使用于**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。



## 二.Netty

### 1.概述

​	Netty是一个**异步的**、**基于事件驱动**的网络应用框架，用以开发高性能、高可靠的网络IO程序。Netty本质是一个NIO框架，主要封装了TCP协议，适用于服务器通讯相关的多种应用场景。

![image-20200517215705107](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200517215705107.png)



### 2.架构设计

#### 1.介绍

目前存在的线程模型有：

 	**传统阻塞** **I/O** **服务模型** 
 	**Reactor** **模式**

**根据** **Reactor** **的数量和处理资源池线程的数量不同，有** **3** **种典型的实现** 

- 单 Reactor 单线程；
- 单 Reactor 多线程
- 主从 Reactor 多线程 



​	Netty 线程模式(Netty 主要**基于主从** **Reactor** **多线程模型**做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor)

#### 2.传统阻塞I/O 服务模型

![image-20200518105139233](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518105139233.png)

产生问题：

1. 当并发数很大，就会创建大量的线程，占用很大系统资源
2. 连接创建后，如果当前线程暂时没有数据可读，该线程
    会阻塞在read 操作，造成线程资源浪费

#### 3.Reactor模式

问题解决：

1. 基于 I/O 复用模型：多个连接**共用一个阻塞对象**，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理

2. 基于线程池复用线程资源：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。

   ![image-20200518172311440](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518172311440.png)

##### 1.单 Reactor 单线程

![image-20200518172503059](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518172503059.png)

优缺点分析：

1. **优点：**模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成
2. **缺点：**性能问题，只有一个线程，无法完全发挥多核 CPU 的性能。Handler 在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
3. **缺点：**可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障
4. **使用场景：**客户端的数量有限，业务处理非常快速，比如 Redis在业务处理的时间复杂度 O(1) 的情况

##### 2.单 Reactor 多线程

![image-20200518172703513](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518172703513.png)

说明：

1. Reactor 对象通过select 监控客户端请求事件, 收到事件后，通过dispatch进行分发
2. 如果建立连接请求, 则右Acceptor 通过accept 处理连接请求, 然后创建一个Handler对象处理完成连接后的各种事件
3. 如果不是连接请求，则由reactor分发调用连接对应的handler 来处理
4. handler 只负责响应事件，不做具体的业务处理, 通过read 读取数据后，会分发给后面的worker线程池的某个线程处理业务
5. worker 线程池会分配独立线程完成真正的业务，并将结果返回给handler
6. handler收到响应后，通过send 将结果返回给client



优缺点分析：

1. **优点：**可以充分的利用多核cpu 的处理能力，增加的Worker线程池**主要处理非IO操作**
2. **缺点：**多线程数据共享和访问比较复杂， reactor 处理所有的事件的监听和响应，在单线程运行， 在高并发场景容易出现性能瓶颈.

##### 3.主从 Reactor 多线程 

![image-20200518175703812](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518175703812.png)

说明：

1. Reactor主线程 MainReactor 对象通过select 监听连接事件, 收到事件后，通过Acceptor 处理连接事件
2. 当 Acceptor  处理连接事件后，MainReactor 将连接分配给SubReactor （多个）
3. subreactor 将连接加入到连接队列进行监听,并创建handler进行各种事件处理
4. 当有新事件发生时， subreactor 就会调用对应的handler处理
5. handler 通过read 读取数据，分发给后面的worker 线程处理
6. worker 线程池分配独立的worker 线程进行业务处理，并返回结果
7. handler 收到响应的结果后，再通过send 将结果返回给client
8. Reactor 主线程可以对应多个Reactor 子线程, 即MainRecator 可以关联多个SubReactor



方案优缺点说明：

1. **优点：**父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理。
2. **优点：**父线程与子线程的数据交互简单，Reactor 主线程只需要把新连接传给子线程，子线程无需返回数据。
3. **缺点：**编程复杂度较高

##### 4.Netty模型

![Netty模型图](/Users/yinwenbo/Documents/img/Netty模型图.png)

说明：

1. Netty抽象出两组线程池 BossGroup 专门负责接收客户端的连接, WorkerGroup 专门负责网络的读写(ChannelHandler处理非io操作)
2. BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup
3. NioEventLoopGroup 相当于一个事件循环组, 这个组中含有多个事件循环 ，每一个事件循环是 NioEventLoop
4. NioEventLoop 表示一个不断循环的执行处理任务的线程， 每个NioEventLoop 都有一个selector , 用于监听绑定在其上的socket的网络通讯
5. NioEventLoopGroup 可以有多个线程, 即可以含有多个NioEventLoop
6. 每个Boss NioEventLoop 循环执行的步骤有3步

- 轮询accept 事件
- 处理accept 事件 , 与client建立连接 , 生成NioScocketChannel , 并将其注册到某个worker NIOEventLoop 上的 selector 
- 处理任务队列的任务 ， 即 runAllTasks

  7 .每个 Worker NIOEventLoop 循环执行的步骤

- 轮询read, write 事件
- 处理i/o事件， 即read , write 事件，在对应NioScocketChannel 处理
- 处理任务队列的任务 ， 即 runAllTasks

  8 .每个Worker NIOEventLoop  处理业务时，会使用pipeline(管道), pipeline 中包含了 channel , 即通过pipeline 可以获取到对应通道, 管道中维护了很多的处理器

#### 4.Task与异步模型

##### 1.Task

​	当有一个耗时操作时最好异步执行，此时可以提交到对应的NioEventLoop中的TastQueue中执行

**任务队列中的 Task 有 3 种典型使用场景**：

- 用户程序自定义的普通任务 

```java
//在handler中（extends ChannelInboundHandlerAdapter）重写channelRead方法
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
      //比如这里我们有一个非常耗时长的业务-> 异步执行 -> 提交该channel 对应的
      //NIOEventLoop 的 taskQueue中,

      //解决方案1 用户程序自定义的普通任务---->taskQueue
      ctx.channel().eventLoop().execute(new Runnable() {
        @Override
        public void run() {
          try {
            Thread.sleep(5 * 1000);
            ctx.wraiteAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));
            System.out.println("channel code=" + ctx.channel().hashCode());
          } catch (Exception ex) {
            System.out.println("发生异常" + ex.getMessage());
          }
        }
      });
    }
```

- 用户自定义定时任务

```java
//在handler中（extends ChannelInboundHandlerAdapter）重写channelRead方法
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception { 
  	////解决方案2 : 用户自定义定时任务 -> 该任务是提交到 scheduleTaskQueue中
		ctx.channel().eventLoop().schedule(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(5 * 1000);
                    ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵4", CharsetUtil.UTF_8));
                    System.out.println("channel code=" + ctx.channel().hashCode());
                } catch (Exception ex) {
                    System.out.println("发生异常" + ex.getMessage());
                }
            }
     }, 5, TimeUnit.SECONDS);
}
```

- 非当前 Reactor 线程调用 Channel 的各种方法

  ​		例如在**推送系统**的业务线程里面，根据**用户的标识**，找到对应的 **Channel** **引用**，然后调用 Write 类方法向该用户推送消息，就会进入到这种场景。最终的 Write 会提交到任务队列中后被**异步消费** 

###### NioEventLoop说明：

​	NioEventLoop 内部采用串行化设计，从消息的读取->解码->处理->编码->发送，始终由 IO 线程 NioEventLoop 负责

- NioEventLoopGroup 下包含多个 NioEventLoop
- 每个 NioEventLoop 中包含有一个 Selector，一个 taskQueue
- 每个 NioEventLoop 的 Selector 上可以注册监听多个 NioChannel
- 每个 NioChannel 只会绑定在唯一的 NioEventLoop 上
- 每个 NioChannel 都绑定有一个自己的 ChannelPipeline

##### 2.异步模型

###### 1.介绍

1. 异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在完成后，通过状态、通知和回调来通知调用者。
2. Netty 中的 I/O 操作是异步的，包括 Bind、Write、Connect 等操作会简单的返回一个 ChannelFuture。
3. 调用者并不能立刻获得结果，而是通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果
4. Netty 的异步模型是建立在 future 和 callback 的之上的。callback 就是回调。重点说 Future，它的核心思想是：假设一个方法 fun，计算过程可能非常耗时，等待 fun返回显然不合适。那么可以在调用 fun 的时候，立马返回一个 Future，后续可以通过 Future去监控方法 fun 的处理过程(即 ： Future-Listener 机制)

###### Future说明

1. 表示异步的执行结果, 可以通过它提供的方法来检测执行是否完成，比如检索计算等等.
2. ChannelFuture 是一个接口 ： **public interface** ChannelFuture **extends** Future<Void>
    我们可以添加监听器，当监听的事件发生时，就会通知到监听器. 案例说明

###### 3.原理图示

![image-20200518201354038](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518201354038.png)

说明：

1. 在使用 Netty 进行编程时，拦截操作和转换出入站数据只需要您提供 callback 或利用future 即可。这使得**链式操作**简单、高效, 并有利于编写可重用的、通用的代码。
2. Netty 框架的目标就是让你的业务逻辑从网络基础应用编码中分离出来、解脱出来

###### 4.**Future-Listener** 机制

1. 当 Future 对象刚刚创建时，处于非完成状态，调用者可以通过返回的 ChannelFuture 来获取操作执行的状态，注册监听函数来执行完成后的操作。
2. 常见有如下操作

- 通过 isDone 方法来判断当前操作是否完成；
- 通过 isSuccess 方法来判断已完成的当前操作是否成功；
- 通过 getCause 方法来获取已完成的当前操作失败的原因；
- 通过 isCancelled 方法来判断已完成的当前操作是否被取消；
- 通过 addListener 方法来注册监听器，当操作已完成(isDone 方法返回完成)，将会通知指定的监听器；如果 Future 对象已完成，则通知指定的监听器

###### 5.Future-Listener示例

```java
//绑定一个端口并且同步, 生成了一个 ChannelFuture 对象
//启动服务器(并绑定端口)
ChannelFuture cf = bootstrap.bind(6668).sync();
//给cf 注册监听器，监控我们关心的事件
cf.addListener(new ChannelFutureListener() {
  @Override
  public void operationComplete(ChannelFuture future) throws Exception {
    if (cf.isSuccess()) {
      System.out.println("监听端口成功");
    } else {
      System.out.println("监听端口失败");
    }
  }
});
```

​		相比传统阻塞 I/O，执行 I/O 操作后线程会被阻塞住, 直到操作完成；**异步处理的好处是不会造成线程阻塞**，线程在 I/O 操作期间可以执行别的程序，在高并发情形下会更稳定和更高的吞吐量

#### 5.案例-HTTP服务

##### 1.要求

1. Netty 服务器在 6668 端口监听，浏览器发出请求 "http://localhost:6668/ "
2. 服务器可以回复消息给客户端 "Hello! 我是服务器 5 " ,  并对特定请求资源进行过滤.

##### 2.代码

###### 1.Server

```java
public class TestServer {
    public static void main(String[] args) throws Exception {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).childHandler(new TestServerInitializer());

            ChannelFuture channelFuture = serverBootstrap.bind(6668).sync();
            
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

###### 2.ChannelInitializer

```java
public class TestServerInitializer extends ChannelInitializer<SocketChannel> {
  @Override
  protected void initChannel(SocketChannel ch) throws Exception {

    //向管道加入处理器

    //得到管道
    ChannelPipeline pipeline = ch.pipeline();

    //加入一个netty 提供的httpServerCodec codec =>[coder - decoder]
    //HttpServerCodec 说明
    //1. HttpServerCodec 是netty 提供的处理http的 编-解码器
    pipeline.addLast("MyHttpServerCodec",new HttpServerCodec());
    //2. 增加一个自定义的handler
    pipeline.addLast("MyTestHttpServerHandler", new TestHttpServerHandler());

    System.out.println("ok~~~~");

  }
}
```

###### 3.handler

```java
/*
说明
1. SimpleChannelInboundHandler 是 ChannelInboundHandlerAdapter
2. HttpObject 客户端和服务器端相互通讯的数据被封装成 HttpObject
 */
public class TestHttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {
    //channelRead0 读取客户端数据
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
      System.out.println("对应的channel=" + ctx.channel() + " pipeline=" + ctx.pipeline() + " 通过pipeline获取channel" + ctx.pipeline().channel());

      System.out.println("当前ctx的handler=" + ctx.handler());

      //判断 msg 是不是 httprequest请求
      if(msg instanceof HttpRequest) {

        System.out.println("ctx 类型="+ctx.getClass());

        System.out.println("pipeline hashcode" + ctx.pipeline().hashCode() + " TestHttpServerHandler hash=" + this.hashCode());

        System.out.println("msg 类型=" + msg.getClass());
        System.out.println("客户端地址" + ctx.channel().remoteAddress());

        //获取到
        HttpRequest httpRequest = (HttpRequest) msg;
        //获取uri, 过滤指定的资源
        URI uri = new URI(httpRequest.uri());
        if("/favicon.ico".equals(uri.getPath())) {
          System.out.println("请求了 favicon.ico, 不做响应");
          return;
        }
        //回复信息给浏览器 [http协议]
        ByteBuf content = Unpooled.copiedBuffer("hello, 我是服务器", CharsetUtil.UTF_8);

        //构造一个http的相应，即 httpresponse
        FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);

        response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
        response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());

        //将构建好 response返回
        ctx.writeAndFlush(response);
      }
    }
}

```



### 3.核心模块

#### 1.Bootstrap、ServerBootstrap

##### 1.介绍

​	Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是**配置整个 Netty 程序**，串联各个组件，Netty 中 Bootstrap 类是**客户端程序的启动引导类**，ServerBootstrap 是**服务端启动引导类**

##### 2.api

```java
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)//该方法用于服务器端，用来设置两个 EventLoop
public B group(EventLoopGroup group) //该方法用于客户端，用来设置一个 EventLoop
public B channel(Class<? extends C> channelClass)//该方法用来设置一个服务器端的通道实现
public <T> B option(ChannelOption<T> option, T value)//用来给 ServerChannel 添加配置
public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)//用来给接收到的通道添加配置
public ServerBootstrap childHandler(ChannelHandler childHandler)//该方法用来设置业务处理类（自定义的 handler）

```

#### 2.Future、ChannelFuture

##### 1.介绍

​	 Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

##### 2.api

```java
Channel channel()//返回当前正在进行 IO 操作的通道
ChannelFuture sync()//等待异步操作执行完毕
```

#### 3.Channel

##### 1.介绍

1.  Netty 网络通信的组件，能够用于执行网络 I/O 操作。
2. 通过Channel 可获得当前网络连接的通道的状态
3. 通过Channel 可获得 网络连接的配置参数 （例如接收缓冲区大小）
4. Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
5. 调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方
6. 支持关联 I/O 操作与对应的处理程序
7. 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型:

- NioSocketChannel，异步的客户端 TCP Socket 连接。
- NioServerSocketChannel，异步的服务器端 TCP Socket 连接。
- NioDatagramChannel，异步的 UDP 连接。
- NioSctpChannel，异步的客户端 Sctp 连接。
- NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。

#### 4.Selector

##### 1.介绍

1. Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。
2. 当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel 

#### 5.ChannelHandler

##### 1.介绍

1. ChannelHandler 是一个接口，**处理 I/O 事件或拦截 I/O 操作**，并将其**转发**到其 ChannelPipeline(业务处理链)中的下一个处理程序。
2. ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类

##### 2.实现类

![image-20200518203934854](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518203934854.png)

说明：

- ChannelInboundHandler 用于处理入站 I/O 事件。
- ChannelOutboundHandler 用于处理出站 I/O 操作。

- ChannelInboundHandlerAdapter 用于处理入站 I/O 事件。
- ChannelOutboundHandlerAdapter 用于处理出站 I/O 操作。
- ChannelDuplexHandler 用于处理入站和出站事件。

##### 2.api

```java
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {
  public ChannelInboundHandlerAdapter() { }
  public void channelRegistered(ChannelHandlerContext ctx) throws Exception{
    ctx.fireChannelRegistered();    
  }    
  public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelUnregistered();    
  }
  //通道就绪事件
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelActive();  
  }    
  public void channelInactive(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelInactive();   
  }    
  //通道读取数据事件
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    ctx.fireChannelRead(msg);    
  }
  //数据读取完毕事件
  public void channelReadComplete(ChannelHandlerContext ctx) throws Exception 
  {
    ctx.fireChannelReadComplete();
  } 
  public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
    ctx.fireUserEventTriggered(evt);
  }
  public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelWritabilityChanged();
  }
  //通道发生异常事件
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.fireExceptionCaught(cause);
  }
}
```

#### 6.Pipeline、ChannelPipeline

##### 1.介绍

1. ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于一个贯穿 Netty 的链。(**也可以这样理解ChannelPipeline是 **保存 **ChannelHandler** **的** List，用于处理或拦截Channel 的入站事件和出站操作)
2. ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互
3. 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应

##### 2.图示

![image-20200518204751332](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518204751332.png)

说明：

- 一个 Channel 包含了一个 ChannelPipeline，而 ChannelPipeline 中又维护了一个由 ChannelHandlerContext 组成的双向链表，并且每个 ChannelHandlerContext 中又关联着一个 ChannelHandler
- ChannelPipeline提供了ChannelHandler链的容器。以客户端应用程序为例，如果事件的运动方向是从客户端到服务端的，那么我们称这些事件为出站的，即客户端发送给服务端的数据会通过pipeline中的一系列ChannelOutboundHandler，并被这些Handler处理，反之则称为入站的
- 入站事件和出站事件在一个双向链表中，入站事件会从链表 head 往后传递到最后一个入站的 handler，出站事件会从链表 tail 往前传递到最前一个出站的 handler，两种类型的 handler 互不干扰

##### 3.api

```java
ChannelPipeline addFirst(ChannelHandler... handlers)//把一个业务处理类（handler）添加到链中的第一个位置
ChannelPipeline addLast(ChannelHandler... handlers)//把一个业务处理类（handler）添加到链中的最后一个位置
```

#### 7.ChannelHandlerContext

##### 1.介绍

1. 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象
2. 即ChannelHandlerContext 中 包 含 一 个 具 体 的 事 件 处 理 器 ChannelHandler ， 同 时ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler进行调用.

##### 2.api

```java
ChannelFuture close()//关闭通道
ChannelOutboundInvoker flush()//刷新
ChannelFuture writeAndFlush(Object msg)// 将 数 据 写 到 ChannelPipeline 中 当 前ChannelHandler 的下一个 ChannelHandler 开始处理（出站）
```

#### 8.ChannelOption

##### 1.介绍

1. Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数。

2. ChannelOption 参数如下:

   **ChannelOption.SO_BACKLOG:**对应 TCP/IP 协议 listen 函数中的 backlog 参数，*用来初始化服务器可连接队列大小*。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog 参数指定了队列的大小。

   **ChannelOption.SO_KEEPALIVE:**:一直保持连接活动状态

#### 9.EventLoopGroup、 **NioEventLoopGroup**

##### 1.介绍

1. EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop 同时工作，每个 EventLoop 维护着一个 Selector 实例。
2. EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop来处理任务。在 Netty 服务器端编程中，我们一般都需要提供两个 EventLoopGroup，例如：BossEventLoopGroup 和 WorkerEventLoopGroup。
3. 通常一个服务端口即一个 ServerSocketChannel对应一个Selector 和一个EventLoop线程。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示

![image-20200518205622789](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200518205622789.png)

说明：

- BossEventLoopGroup 通常是一个单线程的 EventLoop，EventLoop 维护着一个注册了ServerSocketChannel 的 Selector 实例，BossEventLoop 不断轮询 Selector 将连接事件分离出来
- 通常是 OP_ACCEPT 事件，然后将接收到的 SocketChannel 交给 WorkerEventLoopGroup
- WorkerEventLoopGroup 会由 next 选择其中一个 EventLoop来将这个 SocketChannel 注册到其维护的 Selector 并对其后续的 IO 事件进行处理

##### 2.api

```java
public NioEventLoopGroup()//构造方法
public Future<?> shutdownGracefully()//断开连接，关闭线程
```

#### 10.Unpooled

##### 1.介绍

​	Netty 提供一个专门用来操作缓冲区ByteBuf(即Netty的数据容器)的工具类

ByteBuf:

![截屏2020-05-18 下午9.05.15](/Users/yinwenbo/Documents/img/截屏2020-05-18 下午9.05.15.png)

​	0-reader:已经读取的区域

​	reader-writer:可读区域

​	writer-capacity:可写区域

##### 2.api

```java
//通过给定的数据和字符编码返回一个 ByteBuf 对象（类似于NIO中的ByteBuffer）
public static ByteBuf copiedBuffer(CharSequence string, Charset charset)
```

### 4.应用实例

#### 1.群聊系统

##### 1.要求

1. 编写一个 Netty 群聊系统，实现服务器端和客户端之间的数据简单通讯（非阻塞）
2. 实现多人群聊
3. 服务器端：可以监测用户上线，离线，并实现消息转发功能
4. 客户端：通过channel 可以无阻塞发送消息给其它所有用户，同时可以接受其它用户发送的消息(有服务器转发得到)

##### 2.代码

###### 1.Server

```java
public class GroupChatServer {

    private int port; //监听端口


    public GroupChatServer(int port) {
        this.port = port;
    }

    //编写run方法，处理客户端的请求
    public void run() throws  Exception{

        //创建两个线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); //8个NioEventLoop

        try {
            ServerBootstrap b = new ServerBootstrap();

            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {

                            //获取到pipeline
                            ChannelPipeline pipeline = ch.pipeline();
                            //向pipeline加入解码器
                            pipeline.addLast("decoder", new StringDecoder());
                            //向pipeline加入编码器
                            pipeline.addLast("encoder", new StringEncoder());
                            //加入自己的业务处理handler
                            pipeline.addLast(new GroupChatServerHandler());

                        }
                    });

            System.out.println("netty 服务器启动");
            ChannelFuture channelFuture = b.bind(port).sync();

            //监听关闭
            channelFuture.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }

    public static void main(String[] args) throws Exception {

        new GroupChatServer(7000).run();
    }
}

```

###### 2.ServerHandler

```java
public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    //public static List<Channel> channels = new ArrayList<Channel>();

    //使用一个hashmap 管理
    //public static Map<String, Channel> channels = new HashMap<String,Channel>();

    //定义一个channle 组，管理所有的channel
    //GlobalEventExecutor.INSTANCE) 是全局的事件执行器，是一个单例
    private static ChannelGroup  channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");


    //handlerAdded 表示连接建立，一旦连接，第一个被执行
    //将当前channel 加入到  channelGroup
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        //将该客户加入聊天的信息推送给其它在线的客户端
        /*
        该方法会将 channelGroup 中所有的channel 遍历，并发送 消息，
        我们不需要自己遍历
         */
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 加入聊天" + sdf.format(new java.util.Date()) + " \n");
        channelGroup.add(channel);
    }

    //断开连接, 将xx客户离开信息推送给当前在线的客户
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        Channel channel = ctx.channel();
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 离开了\n");
        System.out.println("channelGroup size" + channelGroup.size());

    }

    //表示channel 处于活动状态, 提示 xx上线
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 上线了~");
    }

    //表示channel 处于不活动状态, 提示 xx离线了
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 离线了~");
    }

    //读取数据
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {

        //获取到当前channel
        Channel channel = ctx.channel();
        //这时我们遍历channelGroup, 根据不同的情况，回送不同的消息

        channelGroup.forEach(ch -> {
            if(channel != ch) { //不是当前的channel,转发消息
                ch.writeAndFlush("[客户]" + channel.remoteAddress() + " 发送了消息" + msg + "\n");
            }else {//回显自己发送的消息给自己
                ch.writeAndFlush("[自己]发送了消息" + msg + "\n");
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //关闭通道
        ctx.close();
    }
}

```

###### 3.Client

```java
public class GroupChatClient {

    //属性
    private final String host;
    private final int port;

    public GroupChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() throws Exception{
        EventLoopGroup group = new NioEventLoopGroup();

        try {


        Bootstrap bootstrap = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>() {

                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {

                        //得到pipeline
                        ChannelPipeline pipeline = ch.pipeline();
                        //加入相关handler
                        pipeline.addLast("decoder", new StringDecoder());
                        pipeline.addLast("encoder", new StringEncoder());
                        //加入自定义的handler
                        pipeline.addLast(new GroupChatClientHandler());
                    }
                });

        ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
        //得到channel
            Channel channel = channelFuture.channel();
            System.out.println("-------" + channel.localAddress()+ "--------");
            //客户端需要输入信息，创建一个扫描器
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNextLine()) {
                String msg = scanner.nextLine();
                //通过channel 发送到服务器端
                channel.writeAndFlush(msg + "\r\n");
            }
        }finally {
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new GroupChatClient("127.0.0.1", 7000).run();
    }
}

```

###### 4.ClientHandler

```java
public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    //public static List<Channel> channels = new ArrayList<Channel>();

    //使用一个hashmap 管理
    //public static Map<String, Channel> channels = new HashMap<String,Channel>();

    //定义一个channle 组，管理所有的channel
    //GlobalEventExecutor.INSTANCE) 是全局的事件执行器，是一个单例
    private static ChannelGroup  channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    //handlerAdded 表示连接建立，一旦连接，第一个被执行
    //将当前channel 加入到  channelGroup
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        //将该客户加入聊天的信息推送给其它在线的客户端
        /*
        该方法会将 channelGroup 中所有的channel 遍历，并发送 消息，
        我们不需要自己遍历
         */
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 加入聊天" + sdf.format(new java.util.Date()) + " \n");
        channelGroup.add(channel);
    }

    //断开连接, 将xx客户离开信息推送给当前在线的客户
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        Channel channel = ctx.channel();
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 离开了\n");
        System.out.println("channelGroup size" + channelGroup.size());

    }

    //表示channel 处于活动状态, 提示 xx上线
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 上线了~");
    }

    //表示channel 处于不活动状态, 提示 xx离线了
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 离线了~");
    }

    //读取数据
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {

        //获取到当前channel
        Channel channel = ctx.channel();
        //这时我们遍历channelGroup, 根据不同的情况，回送不同的消息

        channelGroup.forEach(ch -> {
            if(channel != ch) { //不是当前的channel,转发消息
                ch.writeAndFlush("[客户]" + channel.remoteAddress() + " 发送了消息" + msg + "\n");
            }else {//回显自己发送的消息给自己
                ch.writeAndFlush("[自己]发送了消息" + msg + "\n");
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //关闭通道
        ctx.close();
    }
}

```

###### 5.User

```java
public class User {
    private int id;
    private String pwd;
}

```

#### 2.心跳检测

##### 1.要求

1. 编写一个 Netty心跳检测机制案例, 当服务器超过3秒没有读时，就提示读空闲
2. 当服务器超过5秒没有写操作时，就提示写空闲
3. 实现当服务器超过7秒没有读或者写操作时，就提示读写空闲

##### 2.代码

###### 1.Server

```java
public class MyServer {
    public static void main(String[] args) throws Exception{


        //创建两个线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); //8个NioEventLoop
        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup, workerGroup);
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();
                    //加入一个netty 提供 IdleStateHandler
                    /*
                    说明
                    1. IdleStateHandler 是netty 提供的处理空闲状态的处理器
                    2. long readerIdleTime : 表示多长时间没有读, 就会发送一个心跳检测包检测是否连接
                    3. long writerIdleTime : 表示多长时间没有写, 就会发送一个心跳检测包检测是否连接
                    4. long allIdleTime : 表示多长时间没有读写, 就会发送一个心跳检测包检测是否连接
                    5. 文档说明
                    triggers an {@link IdleStateEvent} when a {@link Channel} has not performed
 * read, write, or both operation for a while.
 *                  6. 当 IdleStateEvent 触发后 , 就会传递给管道 的下一个handler去处理
 *                  通过调用(触发)下一个handler 的 userEventTiggered , 在该方法中去处理 IdleStateEvent(读空闲，写空闲，读写空闲)
                     */
                    pipeline.addLast(new IdleStateHandler(7000,7000,10, TimeUnit.SECONDS));
                    //加入一个对空闲检测进一步处理的handler(自定义)
                    pipeline.addLast(new MyServerHandler());
                }
            });

            //启动服务器
            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

###### 2.ServerHandler

```java
public class MyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     *
     * @param ctx 上下文
     * @param evt 事件
     * @throws Exception
     */
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {

        if(evt instanceof IdleStateEvent) {

            //将  evt 向下转型 IdleStateEvent
            IdleStateEvent event = (IdleStateEvent) evt;
            String eventType = null;
            switch (event.state()) {
                case READER_IDLE:
                  eventType = "读空闲";
                  break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }
            System.out.println(ctx.channel().remoteAddress() + "--超时时间--" + eventType);
            System.out.println("服务器做相应处理..");

            //如果发生空闲，我们关闭通道
           // ctx.channel().close();
        }
    }
}

```

###### 3.Main

```java
public class Test {
    public static void main(String[] args) throws Exception {
        System.out.println(System.nanoTime()); //纳秒  10亿分之1
        Thread.sleep(1000);
        System.out.println(System.nanoTime());

    }
}
```

#### 3.WebSocket

##### 1.要求

**实例要求**:  

1. Http协议是无状态的, 浏览器和服务器间的请求响应一次，下一次会重新创建连接
2. 要求：实现基于webSocket的长连接的全双工的交互
3. 改变Http协议多次请求的约束，实现长连接了， 服务器可以发送消息给浏览器
4. 客户端浏览器和服务器端会相互感知，比如服务器关闭了，浏览器会感知，同样浏览器关闭了，服务器会感知

##### 2.代码

###### 1.Server

```java
public class MyServer {
    public static void main(String[] args) throws Exception{


        //创建两个线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); //8个NioEventLoop
        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup, workerGroup);
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();

                    //因为基于http协议，使用http的编码和解码器
                    pipeline.addLast(new HttpServerCodec());
                    //是以块方式写，添加ChunkedWriteHandler处理器
                    pipeline.addLast(new ChunkedWriteHandler());

                    /*
                    说明
                    1. http数据在传输过程中是分段, HttpObjectAggregator ，就是可以将多个段聚合
                    2. 这就就是为什么，当浏览器发送大量数据时，就会发出多次http请求
                     */
                    pipeline.addLast(new HttpObjectAggregator(8192));
                    /*
                    说明
                    1. 对应websocket ，它的数据是以 帧(frame) 形式传递
                    2. 可以看到WebSocketFrame 下面有六个子类
                    3. 浏览器请求时 ws://localhost:7000/hello 表示请求的uri
                    4. WebSocketServerProtocolHandler 核心功能是将 http协议升级为 ws协议 , 保持长连接
                    5. 是通过一个 状态码 101
                     */
                    pipeline.addLast(new WebSocketServerProtocolHandler("/hello2"));
                    //自定义的handler ，处理业务逻辑
                    pipeline.addLast(new MyTextWebSocketFrameHandler());
                }
            });

            //启动服务器
            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

```

###### 2.ServerHandler

```java
//这里 TextWebSocketFrame 类型，表示一个文本帧(frame)
public class MyTextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame>{
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        System.out.println("服务器收到消息 " + msg.text());

        //回复消息
        ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间" + LocalDateTime.now() + " " + msg.text()));
    }

    //当web客户端连接后， 触发方法
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        //id 表示唯一的值，LongText 是唯一的 ShortText 不是唯一
        System.out.println("handlerAdded 被调用" + ctx.channel().id().asLongText());
        System.out.println("handlerAdded 被调用" + ctx.channel().id().asShortText());
    }


    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        System.out.println("handlerRemoved 被调用" + ctx.channel().id().asLongText());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("异常发生 " + cause.getMessage());
        ctx.close(); //关闭连接
    }
}
```

###### 3.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    var socket;
    //判断当前浏览器是否支持websocket
    if(window.WebSocket) {
        //go on
        socket = new WebSocket("ws://localhost:7000/hello2");
        //相当于channelReado, ev 收到服务器端回送的消息
        socket.onmessage = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + ev.data;
        }

        //相当于连接开启(感知到连接开启)
        socket.onopen = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = "连接开启了.."
        }

        //相当于连接关闭(感知到连接关闭)
        socket.onclose = function (ev) {

            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + "连接关闭了.."
        }
    } else {
        alert("当前浏览器不支持websocket")
    }

    //发送消息到服务器
    function send(message) {
        if(!window.socket) { //先判断socket是否创建好
            return;
        }
        if(socket.readyState == WebSocket.OPEN) {
            //通过socket 发送消息
            socket.send(message)
        } else {
            alert("连接没有开启");
        }
    }
</script>
    <form onsubmit="return false">
        <textarea name="message" style="height: 300px; width: 300px"></textarea>
        <input type="button" value="发生消息" onclick="send(this.form.message.value)">
        <textarea id="responseText" style="height: 300px; width: 300px"></textarea>
        <input type="button" value="清空内容" onclick="document.getElementById('responseText').value=''">
    </form>
</body>
</html>
```



### 5.编码与解码

#### 1.介绍

1. 编写网络应用程序时，因为数据在网络中传输的都是二进制字节码数据，在发送数据时就需要编码，接收数据时就需要解码 
2. codec(编解码器) 的组成部分有两个：decoder(解码器)和 encoder(编码器)。encoder 负责把业务数据转换成字节码数据，decoder 负责把字节码数据转换成业务数据

![image-20200519182505666](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519182505666.png)

##### 1.Netty的编解码器

Netty 提供的编码器

- StringEncoder，对字符串数据进行编码
- ObjectEncoder，对 Java 对象进行编码
-  ...

Netty 提供的解码器

- StringDecoder, 对字符串数据进行解码
- ObjectDecoder，对 Java 对象进行解码
- ...

##### 2.问题

​	Netty 本身自带的 ObjectDecoder 和 ObjectEncoder 可以用来实现 POJO 对象或各种业务对象的编码和解码，底层使用的仍是 Java 序列化技术 , 而Java 序列化技术本身效率就不高，存在如下问题：

- 无法跨语言
- 序列化后的体积太大，是二进制编码的 5 倍多。
- 序列化性能太低

#### 2.Protobuf

##### 1.介绍

1. Protobuf 是 Google 发布的开源项目，全称 Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 **RPC[远程过程调用**remote procedure call ]** **数据交换格式** 。
    目前很多公司 http+json ➔ tcp+protobuf
2. 参考文档 : https://developers.google.com/protocol-buffers/docs/proto   语言指南
3. Protobuf 是以 message 的方式来管理数据的.
4. 支持跨平台、**跨语言**，即[客户端和服务器端可以是不同的语言编写的] （**支持目前绝大多数语言**，例如 C++、C#、Java、python 等）
5. 高性能，高可靠性
6. 使用 protobuf 编译器能自动生成代码，Protobuf 是将类的定义使用.proto 文件进行描述。说明，在idea 中编写 .proto 文件时，会自动提示是否**下载** **.ptotot** **编写插件**. 可以让**语法高亮**。
7. 然后通过 protoc.exe 编译器根据.proto 自动生成.java 文件

![image-20200519190519063](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519190519063.png)

##### 2.案例

要求：

1. 客户端可以随机发送Student  PoJo/ Worker PoJo 对象到服务器 (通过 Protobuf 编码) 
2. 服务端能接收Student PoJo/ Worker PoJo 对象(需要判断是哪种类型)，并显示信息(通过 Protobuf 解码)



代码：

###### 1.Server

```java
public class NettyServer {
    public static void main(String[] args) throws Exception {


        //创建BossGroup 和 WorkerGroup
        //说明
        //1. 创建两个线程组 bossGroup 和 workerGroup
        //2. bossGroup 只是处理连接请求 , 真正的和客户端业务处理，会交给 workerGroup完成
        //3. 两个都是无限循环
        //4. bossGroup 和 workerGroup 含有的子线程(NioEventLoop)的个数
        //   默认实际 cpu核数 * 2
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); //8
        try {
            //创建服务器端的启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();

            //使用链式编程来进行设置
            bootstrap.group(bossGroup, workerGroup) //设置两个线程组
                    .channel(NioServerSocketChannel.class) //使用NioSocketChannel 作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true) //设置保持活动连接状态
//                    .handler(null) // 该 handler对应 bossGroup , childHandler 对应 workerGroup
                    .childHandler(new ChannelInitializer<SocketChannel>() {//创建一个通道初始化对象(匿名对象)
                        //给pipeline 设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {


                            ChannelPipeline pipeline = ch.pipeline();
                            //在pipeline加入ProtoBufDecoder
                            //指定对哪种对象进行解码
                            pipeline.addLast("decoder", new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()));
                            pipeline.addLast(new NettyServerHandler());
                        }
                    }); // 给我们的workerGroup 的 EventLoop 对应的管道设置处理器

            System.out.println(".....服务器 is ready...");

            //绑定一个端口并且同步, 生成了一个 ChannelFuture 对象
            //启动服务器(并绑定端口)
            ChannelFuture cf = bootstrap.bind(6668).sync();

            //给cf 注册监听器，监控我们关心的事件

            cf.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if (cf.isSuccess()) {
                        System.out.println("监听端口 6668 成功");
                    } else {
                        System.out.println("监听端口 6668 失败");
                    }
                }
            });
            //对关闭通道进行监听
            cf.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }

}

```

###### 2.ServerHandler

```java
//自定义一个Handler 需要继续netty 规定好的某个HandlerAdapter(规范)
//public class NettyServerHandler extends ChannelInboundHandlerAdapter {
public class NettyServerHandler extends SimpleChannelInboundHandler<MyDataInfo.MyMessage> {


    //读取数据实际(这里我们可以读取客户端发送的消息)
    /*
    1. ChannelHandlerContext ctx:上下文对象, 含有 管道pipeline , 通道channel, 地址
    2. Object msg: 就是客户端发送的数据 默认Object
     */
    @Override
    public void channelRead0(ChannelHandlerContext ctx, MyDataInfo.MyMessage msg) throws Exception {

        //根据dataType 来显示不同的信息

        MyDataInfo.MyMessage.DataType dataType = msg.getDataType();
        if(dataType == MyDataInfo.MyMessage.DataType.StudentType) {

            MyDataInfo.Student student = msg.getStudent();
            System.out.println("学生id=" + student.getId() + " 学生名字=" + student.getName());

        } else if(dataType == MyDataInfo.MyMessage.DataType.WorkerType) {
            MyDataInfo.Worker worker = msg.getWorker();
            System.out.println("工人的名字=" + worker.getName() + " 年龄=" + worker.getAge());
        } else {
            System.out.println("传输的类型不正确");
        }


    }

//    //读取数据实际(这里我们可以读取客户端发送的消息)
//    /*
//    1. ChannelHandlerContext ctx:上下文对象, 含有 管道pipeline , 通道channel, 地址
//    2. Object msg: 就是客户端发送的数据 默认Object
//     */
//    @Override
//    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
//
//        //读取从客户端发送的StudentPojo.Student
//
//        StudentPOJO.Student student = (StudentPOJO.Student) msg;
//
//        System.out.println("客户端发送的数据 id=" + student.getId() + " 名字=" + student.getName());
//    }

    //数据读取完毕
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {

        //writeAndFlush 是 write + flush
        //将数据写入到缓存，并刷新
        //一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵1", CharsetUtil.UTF_8));
    }

    //处理异常, 一般是需要关闭通道

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

###### 3.Client

```java
public class NettyClient {
    public static void main(String[] args) throws Exception {

        //客户端需要一个事件循环组
        EventLoopGroup group = new NioEventLoopGroup();


        try {
            //创建客户端启动对象
            //注意客户端使用的不是 ServerBootstrap 而是 Bootstrap
            Bootstrap bootstrap = new Bootstrap();

            //设置相关参数
            bootstrap.group(group) //设置线程组
                    .channel(NioSocketChannel.class) // 设置客户端通道的实现类(反射)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            //在pipeline中加入 ProtoBufEncoder
                            pipeline.addLast("encoder", new ProtobufEncoder());
                            pipeline.addLast(new NettyClientHandler()); //加入自己的处理器
                        }
                    });

            System.out.println("客户端 ok..");
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6668).sync();
            //给关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        }finally {

            group.shutdownGracefully();

        }
    }
}
```

###### 4.ClientHandler

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    //当通道就绪就会触发该方法
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        //随机的发送Student 或者 Workder 对象
        int random = new Random().nextInt(3);
        MyDataInfo.MyMessage myMessage = null;

        if(0 == random) { //发送Student 对象

            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.StudentType).setStudent(MyDataInfo.Student.newBuilder().setId(5).setName("玉麒麟 卢俊义").build()).build();
        } else { // 发送一个Worker 对象

            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.WorkerType).setWorker(MyDataInfo.Worker.newBuilder().setAge(20).setName("老李").build()).build();
        }

        ctx.writeAndFlush(myMessage);
    }

    //当通道有读取事件时，会触发
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务器回复的消息:" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("服务器的地址： "+ ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

###### 5.Student.proto

```java
syntax="proto3";//版本
option optiomize_for=SPEED;//加快解析
//可以使用option java_package="xx.xx"指定生成位置到特定包下
option java_outer_classname="MyDataInfo";//生成的外部类名，也是文件名

message MyMessage{
  //定义一个枚举类型
  enum DataType{
  	StudentType=0;
    WorkerType=1;
  }
  DataType data_type=1;//表示传输的枚举类型
  //表示出现的枚举类型最多只出现一个
  oneof dataBody{
    Student student=2;
    Worker worker=3;
  }
}


//protobuf使用message管理数据
message Student {//会在StudentDomain生成一个内部类Student，是真正的发送的对象
  int32 id=1;//此类中有一个属性名称为id，类型的int32，1表示属性序号
  string name=2;
}
message Worker {
  string name=1;
  int32 age=2;
}

//之后编译 使用protoc.exe --java_out=.Student.proto 命令生成StudentDomain.java放到项目中使用
```

###### 6.MyDataInfo

```java
//由protoc.exe生成的MyDataInfo
```

#### 3.编码解码器

##### 1.介绍

1. 当Netty发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种格式（比如java对象）；如果是出站消息，它会被编码成字节。
2. Netty提供一系列实用的编解码器，他们都实现了ChannelInboundHadnler或者ChannelOutboundHandler接口。在这些类中，channelRead方法已经被重写了。以入站为例，对于每个从入站Channel读取的消息，这个方法会被调用。随后，它将调用由解码器所提供的decode()方法进行解码，并将已经解码的字节转发给ChannelPipeline中的下一个ChannelInboundHandler。

![image-20200519203730655](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519203730655.png)

##### 2.ByteToMessageDecoder

![image-20200519194924446](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519194924446.png)

问题：

​	由于不可能知道远程节点是否会一次性发送一个完整的信息，**tcp有可能出现粘包拆包**的问题，这个类会对入站数据进行缓冲，直到它准备好被处理.

```java
public class ToIntegerDecoder extends ByteToMessageDecoder {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        if (in.readableBytes() >= 4) {
            out.add(in.readInt());
        }
    }
}
```

说明：

​	这个例子，每次入站从ByteBuf中读取4字节，将其解码为一个int，然后将它添加到下一个List中。当没有更多元素可以被添加到该List中时，它的内容将会被发送给下一个ChannelInboundHandler。int在被添加到List中时，会被自动装箱为Integer。在调用readInt()方法前必须验证所输入的ByteBuf是否具有足够的数据

![image-20200519195918858](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519195918858.png)

##### 3.ReplayingDecoder

​	public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder

​	ReplayingDecoder扩展了ByteToMessageDecoder类，使用这个类，我们不必调用readableBytes()方法。参数S指定了用户状态管理的类型，其中Void代表不需要状态管理

```java
public class ToIntegerDecoder2 extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
      //不需判断，内部自行处理 
     	out.add(in.readInt());
        
    }
}
```

局限性：

- 并不是所有的 ByteBuf 操作都被支持，如果调用了一个不被支持的方法，将会抛出一个 UnsupportedOperationException。
- ReplayingDecoder 在某些情况下可能稍慢于 ByteToMessageDecoder，例如网络缓慢并且消息格式复杂时，消息会被拆成了多个碎片，速度变慢

##### 4.其他编解码器

###### 1.解码器decoder

1. LineBasedFrameDecoder：这个类在Netty内部也有使用，它使用行尾控制字符（\n或者\r\n）作为分隔符来解析数据。
2. DelimiterBasedFrameDecoder：使用自定义的特殊字符作为消息的分隔符。
3. HttpObjectDecoder：一个HTTP数据的解码器
4. LengthFieldBasedFrameDecoder：通过指定长度来标识整包消息，这样就可以自动的处理黏包和半包消息。

###### 2.编码器encoder

![image-20200519200501907](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519200501907.png)

#### 4.TCP粘包拆包

##### 1.介绍

1. TCP是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的socket，因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（Nagle算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于分辨出完整的数据包了，因为**面向流的通信是无消息保护边界**的
2. 由于TCP无消息保护边界, 需要在接收端处理消息边界问题，也就是粘包、拆包问题

##### 2.图示

![image-20200519200904476](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200519200904476.png)

说明：

​	假设客户端分别发送了两个数据包D1和D2给服务端，由于服务端一次读取到字节数是不确定的，故可能存在以下四种情况：

1. 服务端分两次读取到了两个独立的数据包，分别是D1和D2，没有粘包和拆包
2. 服务端一次接受到了两个数据包，D1和D2粘合在一起，称之为TCP粘包
3. 服务端分两次读取到了数据包，第一次读取到了完整的D1包和D2包的部分内容，第二次读取到了D2包的剩余内容，这称之为TCP拆包
4. 服务端分两次读取到了数据包，第一次读取到了D1包的部分内容D1_1，第二次读取到了D1包的剩余部分内容D1_2和完整的D2包。

##### 3.解决

1. 使用自定义协议 + 编解码器 来解决
2. 关键就是要解决 **服务器端每次读取数据长度的问题**, 这个问题解决，就不会出现服务器多读或少读数据的问题，从而避免的TCP 粘包、拆包 。

##### 4.实例

要求：

1. 要求客户端发送 5 个 Message 对象, 客户端每次发送一个 Message 对象
2. 服务器端每次接收一个Message, 分5次进行解码， 每读取到 一个Message , 会回复一个Message 对象 给客户端.

代码:

###### 1.Protocol 

```java
//协议包
public class MessageProtocol {
    private int len; //关键
    private byte[] content;

    public int getLen() {
        return len;
    }

    public void setLen(int len) {
        this.len = len;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }
}

```

###### 2.Decoder

```java
public class MyMessageDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyMessageDecoder decode 被调用");
        //需要将得到二进制字节码-> MessageProtocol 数据包(对象)
        int length = in.readInt();

        byte[] content = new byte[length];
        in.readBytes(content);

        //封装成 MessageProtocol 对象，放入 out， 传递下一个handler业务处理
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(length);
        messageProtocol.setContent(content);

        out.add(messageProtocol);

    }
}
```

###### 3.Encoder

```java
public class MyMessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext ctx, MessageProtocol msg, ByteBuf out) throws Exception {
        System.out.println("MyMessageEncoder encode 方法被调用");
        out.writeInt(msg.getLen());
        out.writeBytes(msg.getContent());
    }
}
```

###### 4.Initializer;

```java
public class MyInitializer extends ChannelInitializer<SocketChannel>{
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {

        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new MyMessageEncoder()); //加入编码器
        pipeline.addLast(new MyMessageDecoder()); //加入解码器
        pipeline.addLast(new MyClientHandler());
    }
}
```

###### 5.Server&&Client

```java
public class MyServer {
    public static void main(String[] args) throws Exception{

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup).channel(NioServerSocketChannel.class).childHandler(new MyInitializer()); //自定义一个初始化类
            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}

public class MyClient {
    public static void main(String[] args)  throws  Exception{

        EventLoopGroup group = new NioEventLoopGroup();

        try {

            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group).channel(NioSocketChannel.class)
                    .handler(new MyInitializer()); //自定义一个初始化类

            ChannelFuture channelFuture = bootstrap.connect("localhost", 7000).sync();

            channelFuture.channel().closeFuture().sync();

        }finally {
            group.shutdownGracefully();
        }
    }
}
```

###### 7.Handler

```java
//处理业务的handler
public class MyServerHandler extends SimpleChannelInboundHandler<MessageProtocol>{
    private int count;

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //cause.printStackTrace();
        ctx.close();
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol msg) throws Exception {

        //接收到数据，并处理
        int len = msg.getLen();
        byte[] content = msg.getContent();

        System.out.println();
        System.out.println();
        System.out.println();
        System.out.println("服务器接收到信息如下");
        System.out.println("长度=" + len);
        System.out.println("内容=" + new String(content, Charset.forName("utf-8")));

        System.out.println("服务器接收到消息包数量=" + (++this.count));

        //回复消息

        String responseContent = UUID.randomUUID().toString();
        int responseLen = responseContent.getBytes("utf-8").length;
        byte[]  responseContent2 = responseContent.getBytes("utf-8");
        //构建一个协议包
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(responseLen);
        messageProtocol.setContent(responseContent2);

        ctx.writeAndFlush(messageProtocol);
    }
}

public class MyClientHandler extends SimpleChannelInboundHandler<MessageProtocol> {

    private int count;
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //使用客户端发送10条数据 "今天天气冷，吃火锅" 编号

        for(int i = 0; i< 5; i++) {
            String mes = "今天天气冷，吃火锅";
            byte[] content = mes.getBytes(Charset.forName("utf-8"));
            int length = mes.getBytes(Charset.forName("utf-8")).length;

            //创建协议包对象
            MessageProtocol messageProtocol = new MessageProtocol();
            messageProtocol.setLen(length);
            messageProtocol.setContent(content);
            ctx.writeAndFlush(messageProtocol);
        }
    }
  
		@Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol msg) throws Exception {

        int len = msg.getLen();
        byte[] content = msg.getContent();

        System.out.println("客户端接收到消息如下");
        System.out.println("长度=" + len);
        System.out.println("内容=" + new String(content, Charset.forName("utf-8")));
        System.out.println("客户端接收消息数量=" + (++this.count));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("异常消息=" + cause.getMessage());
        ctx.close();
    }
}
```

### 6.源码剖析

#### 1.EventLoopGroup

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
//bossGroup 用于接收TCP请求，然后将请求交给workerGroup，workerGroup获取后和连接进行通讯。
```

##### 1.设置线程数

当new NioEventLoopGroup()时会传到其父类MultithreadEventLoopGroup执行构造方法

```java
/*
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));
*/
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
  			//空参则设置为cpu核数的2倍，有参就直接传入参数
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
    }
```

##### 2.初始化线程池

确定线程个数后会在MultithreadEventExecutorGroup中初始化线程池

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                            EventExecutorChooserFactory chooserFactory, Object... args) {
        if (nThreads <= 0) {
            throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
        }

        if (executor == null) {
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }
				//1.创建EventExecutor数组
        children = new EventExecutor[nThreads];

        for (int i = 0; i < nThreads; i ++) {
            boolean success = false;
            try {
              	//2.创建NioEventLoop
                children[i] = newChild(executor, args);
                success = true;
            } catch (Exception e) {
                // TODO: Think about if this is a good exception type
                throw new IllegalStateException("failed to create a child event loop", e);
            } finally {
              	//3.创建失败，优雅关闭
                if (!success) {
                    for (int j = 0; j < i; j ++) {
                        children[j].shutdownGracefully();
                    }

                    for (int j = 0; j < i; j ++) {
                        EventExecutor e = children[j];
                        try {
                            while (!e.isTerminated()) {
                                e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                            }
                        } catch (InterruptedException interrupted) {
                            // Let the caller handle the interruption.
                            Thread.currentThread().interrupt();
                            break;
                        }
                    }
                }
            }
        }

        chooser = chooserFactory.newChooser(children);
				
        final FutureListener<Object> terminationListener = new FutureListener<Object>() {
            @Override
            public void operationComplete(Future<Object> future) throws Exception {
                if (terminatedChildren.incrementAndGet() == children.length) {
                    terminationFuture.setSuccess(null);
                }
            }
        };
				//3.为线程池添加一个关闭监听器
        for (EventExecutor e: children) {
            e.terminationFuture().addListener(terminationListener);
        }
				//4.将所有的单例线程池添加到一个HashSet中
        Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
        Collections.addAll(childrenSet, children);
        readonlyChildren = Collections.unmodifiableSet(childrenSet);
    }
```

##### 3.优雅关闭线程池

在SingleThreadEventExecutor中优雅关闭线程池

```java
public Future<?> shutdownGracefully(long quietPeriod, long timeout, TimeUnit unit) {
        if (quietPeriod < 0) {
            throw new IllegalArgumentException("quietPeriod: " + quietPeriod + " (expected >= 0)");
        }
        if (timeout < quietPeriod) {
            throw new IllegalArgumentException(
                    "timeout: " + timeout + " (expected >= quietPeriod (" + quietPeriod + "))");
        }
        if (unit == null) {
            throw new NullPointerException("unit");
        }

        if (isShuttingDown()) {
            return terminationFuture();
        }

        boolean inEventLoop = inEventLoop();
        boolean wakeup;
        int oldState;
        for (;;) {
            if (isShuttingDown()) {
                return terminationFuture();
            }
            int newState;
            wakeup = true;
            oldState = state;
            if (inEventLoop) {
                newState = ST_SHUTTING_DOWN;
            } else {
                switch (oldState) {
                    case ST_NOT_STARTED:
                    case ST_STARTED:
                        newState = ST_SHUTTING_DOWN;
                        break;
                    default:
                        newState = oldState;
                        wakeup = false;
                }
            }
            if (STATE_UPDATER.compareAndSet(this, oldState, newState)) {
                break;
            }
        }
        gracefulShutdownQuietPeriod = unit.toNanos(quietPeriod);
        gracefulShutdownTimeout = unit.toNanos(timeout);

        if (oldState == ST_NOT_STARTED) {
            try {
                doStartThread();
            } catch (Throwable cause) {
                STATE_UPDATER.set(this, ST_TERMINATED);
                terminationFuture.tryFailure(cause);

                if (!(cause instanceof Exception)) {
                    // Also rethrow as it may be an OOME for example
                    PlatformDependent.throwException(cause);
                }
                return terminationFuture;
            }
        }

        if (wakeup) {
            wakeup(inEventLoop);
        }

        return terminationFuture();
    }
```

修改状态后执行doStartThread()方法，此方法会调用run()

```java
 private void doStartThread() {
        assert thread == null;
        executor.execute(new Runnable() {
            @Override
            public void run() {
                thread = Thread.currentThread();
                if (interrupted) {
                    thread.interrupt();
                }

                boolean success = false;
                updateLastExecutionTime();
                try {
                    SingleThreadEventExecutor.this.run();
                    success = true;
                  			.....
                }
            }
        }
  }                                 
```

这个run会到NioEventLoop执行run方法

```java
//NioEventLoop
protected void run() {
        for (;;) {
            try {
                switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
                    case SelectStrategy.CONTINUE:
                        continue;
                    case SelectStrategy.SELECT:
                        select(wakenUp.getAndSet(false));
                        if (wakenUp.get()) {
                            selector.wakeup();
                        }
                        // fall through
                    default:
                }
                cancelledKeys = 0;
                needsToSelectAgain = false;
                final int ioRatio = this.ioRatio;
                if (ioRatio == 100) {
                    try {
                        processSelectedKeys();
                    } finally {
                        // Ensure we always run tasks.
                        runAllTasks();
                    }
                } else {
                    final long ioStartTime = System.nanoTime();
                    try {
                        processSelectedKeys();
                    } finally {
                        // Ensure we always run tasks.
                        final long ioTime = System.nanoTime() - ioStartTime;
                        runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                    }
                }
            } catch (Throwable t) {
                handleLoopException(t);
            }
            // Always handle shutdown even if the loop processing threw an exception.
            try {
                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        return;
                    }
                }
            } catch (Throwable t) {
                handleLoopException(t);
            }
        }
    }
```

这个run方法里会执行**closeAll()、confirmShutdown()**

```java
/*
（1）取消NioTask的任务
（2）有请求正在响应，等待响应
（3）关闭channel的连接，触发pipeline inActive事件
（4）释放已经flush和未flush的消息对象
（5）取消channel在selector上的注册，触发pipeline deregister事件
*/
private void closeAll() {
  selectAgain();
  Set<SelectionKey> keys = selector.keys();
  Collection<AbstractNioChannel> channels = new ArrayList<AbstractNioChannel>(keys.size());
  for (SelectionKey k: keys) {
    Object a = k.attachment();
    if (a instanceof AbstractNioChannel) {
      channels.add((AbstractNioChannel) a);
    } else {
      k.cancel();
      @SuppressWarnings("unchecked")
      NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
      invokeChannelUnregistered(task, k, null);
    }
  }

  for (AbstractNioChannel ch: channels) {
    ch.unsafe().close(ch.unsafe().voidPromise());
  }
}


//closeAll() -> close()  
private void close(final ChannelPromise promise, final Throwable cause,
                           final ClosedChannelException closeCause, final boolean notify) {
  if (!promise.setUncancellable()) {
    return;
  }
  if (closeInitiated) {
    if (closeFuture.isDone()) {
     	//已经关闭
      safeSetSuccess(promise);
    } else if (!(promise instanceof VoidChannelPromise)) { 
      closeFuture.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
          promise.setSuccess();
        }
      });
    }
    return;
  }

  closeInitiated = true;

  final boolean wasActive = isActive();
  final ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
  this.outboundBuffer = null; 
  Executor closeExecutor = prepareToClose();
  if (closeExecutor != null) {
    closeExecutor.execute(new Runnable() {
      @Override
      public void run() {
        try {
          // Execute the close.
          doClose0(promise);
        } finally {
          invokeLater(new Runnable() {
            @Override
            public void run() {
              if (outboundBuffer != null) {
                // Fail all the queued messages
                outboundBuffer.failFlushed(cause, notify);
                outboundBuffer.close(closeCause);
              }
              fireChannelInactiveAndDeregister(wasActive);
            }
          });
        }
      }
    });
  } else {
    try {
      // 执行socketChannel或者serverSocketChannel的close方法，真正的将底层channel关闭
      doClose0(promise);
    } finally {
      if (outboundBuffer != null) {
        outboundBuffer.failFlushed(cause, notify);
        outboundBuffer.close(closeCause);
      }
    }
    if (inFlush0) {
      invokeLater(new Runnable() {
        @Override
        public void run() {
          fireChannelInactiveAndDeregister(wasActive);
        }
      });
    } else {
      fireChannelInactiveAndDeregister(wasActive);
    }
  }
}


/*
（1）将所有的定时任务取消
（2）执行完成所有的task任务
（3）执行完所有的shutdownhook任务
（4）等待quiteperiod时间（默认2s），如果有新的任务提交，重新执行主流程操作，也就会重新从步骤4开始执行（4已经执行完成后，再次执行也是直接退出，其实就是重新执行步骤5）。如果在quiteperiod时间段内没有新的任务提交，则返回true确认关闭，否则等待到 shutdowntimeout超时（默认15s）
*/
protected boolean confirmShutdown() {
  if (!isShuttingDown()) {
    return false;
  }

  if (!inEventLoop()) {
    throw new IllegalStateException("must be invoked from an event loop");
  }

  cancelScheduledTasks();

  if (gracefulShutdownStartTime == 0) {
    gracefulShutdownStartTime = ScheduledFutureTask.nanoTime();
  }

  if (runAllTasks() || runShutdownHooks()) {
    if (isShutdown()) {
      // Executor shut down - no new tasks anymore.
      return true;
    }

    // There were tasks in the queue. Wait a little bit more until no tasks are queued for the quiet period or
    // terminate if the quiet period is 0.
    // See https://github.com/netty/netty/issues/4241
    if (gracefulShutdownQuietPeriod == 0) {
      return true;
    }
    wakeup(true);
    return false;
  }

  final long nanoTime = ScheduledFutureTask.nanoTime();

  if (isShutdown() || nanoTime - gracefulShutdownStartTime > gracefulShutdownTimeout) {
    return true;
  }

  if (nanoTime - lastExecutionTime <= gracefulShutdownQuietPeriod) {
    // Check if any tasks were added to the queue every 100ms.
    // TODO: Change the behavior of takeTask() so that it returns on timeout.
    wakeup(true);
    try {
      Thread.sleep(100);
    } catch (InterruptedException e) {
      // Ignore
    }

    return false;
  }

  // No tasks were added for last quiet period - hopefully safe to shut down.
  // (Hopefully because we really cannot make a guarantee that there will be no execute() calls by a user.)
  return true;
}
```

执行后返回到SingleThreadEventExecutor执行doStartThread方法中的cleanup();

```java
try {
  // Run all remaining tasks and shutdown hooks.
  for (;;) {
    if (confirmShutdown()) {
      break;
    }
  }
} finally {
  try {
    cleanup();
  } finally {
    STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
    threadLock.release();
    if (!taskQueue.isEmpty()) {
      logger.warn(
        "An event executor terminated with " +
        "non-empty task queue (" + taskQueue.size() + ')');
    }

    terminationFuture.setSuccess(null);
  }
}
```

实际执行NioEventLoop的cleanup关闭selector

```java
@Override
protected void cleanup() {
  try {
    selector.close();
  } catch (IOException e) {
    logger.warn("Failed to close a selector.", e);
  }
}
```

至此，NioEventLoop关闭完成

#### 2.EventLoop

​	EventLoop是一个单例的线程池，里面一直重复三个任务：监听端口、处理端口时间、处理队列事件。

```java
protected void run() {
  for (;;) {
    	switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
        case SelectStrategy.CONTINUE:
          continue;
        case SelectStrategy.SELECT:
          //第一个任务
          select(wakenUp.getAndSet(false));
          if (wakenUp.get()) {
            selector.wakeup();
          }
        default:
      }
      cancelledKeys = 0;
      needsToSelectAgain = false;
      final int ioRatio = this.ioRatio;
      if (ioRatio == 100) {
        try {
          //第二个任务
          processSelectedKeys();
        } finally {
          //第三个任务
          runAllTasks();
        }
      } else {
        final long ioStartTime = System.nanoTime();
        try {
          processSelectedKeys();
        } finally {
          // Ensure we always run tasks.
          final long ioTime = System.nanoTime() - ioStartTime;
          runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
        }
 }
```

select():

```java
private void select(boolean oldWakenUp) throws IOException {
  Selector selector = this.selector;
  try {
    int selectCnt = 0;
    long currentTimeNanos = System.nanoTime();
    long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);
    for (;;) {
      //默认阻塞一秒钟，如有定时任务，则加上5s进行阻塞
      long timeoutMillis = (selectDeadLineNanos - currentTimeNanos + 500000L) / 1000000L;
      if (timeoutMillis <= 0) {
        if (selectCnt == 0) {
          selector.selectNow();
          selectCnt = 1;
        }
        break;
      }
      if (hasTasks() && wakenUp.compareAndSet(false, true)) {
        selector.selectNow();
        selectCnt = 1;
        break;
      }

      int selectedKeys = selector.select(timeoutMillis);
      selectCnt ++;

      if (selectedKeys != 0 || oldWakenUp || wakenUp.get() || hasTasks() || hasScheduledTasks()) {
      	break;
      }
      if (Thread.interrupted()) {
        if (logger.isDebugEnabled()) {
          logger.debug("Selector.select() returned prematurely because " +
                       "Thread.currentThread().interrupt() was called. Use " +
                       "NioEventLoop.shutdownGracefully() to shutdown the NioEventLoop.");
        }
        selectCnt = 1;
        break;
      }

      long time = System.nanoTime();
      if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
        selectCnt = 1;
      } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                 selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
        logger.warn(
          "Selector.select() returned prematurely {} times in a row; rebuilding Selector {}.",selectCnt, selector);

        rebuildSelector();
        selector = this.selector;
        selector.selectNow();
        selectCnt = 1;
        break;
      }

      currentTimeNanos = time;
    }

    if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS) {
      if (logger.isDebugEnabled()) {
        logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                     selectCnt - 1, selector);
      }
    }
  } catch (CancelledKeyException e) {
    if (logger.isDebugEnabled()) {
      logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?", selector, e);
    }
  }
}

```

#### 3.ServerBootstrap

ServerBootstrap的使用：

```java
ServerBootstrap serverBootstrap = new ServerBootstrap();

serverBootstrap.group(bossGroup, workerGroup)//传入线程组
.channel(NioServerSocketChannel.class);//传入NioServerSocketChannel的类对象
.option(ChannelOption.SO_BACKLOG,100);//传入TCP参数，放在linkedHashMap中
.handler(new LoggingHandler(LogLevel.INFO));//传入一个专属于ServerSocketChannel的Handler
.childHandler(new ChannelInitializer<SocketChannel>() {//传入一个为每个客户端连接使用时调用的handler，供SocketChannel使用
  @Override
  public void initChannel(SocketChannel sc) throws Exception{
    ChannelPipeline p=sc.pipeline();
    p.addLast(new MyHandler());
  }
}
```

绑定端口分析：

```java
//服务器在bind()中启用
public ChannelFuture bind() {
  validate();
  SocketAddress localAddress = this.localAddress;
  if (localAddress == null) {
    throw new IllegalStateException("localAddress not set");
  }
  return doBind(localAddress);
}
```

doBind():两个方法initAndRegister和doBind0

```java
private ChannelFuture doBind(final SocketAddress localAddress) {
  //initAndRegister初始化NioServerSocketChannel并注册handler
  final ChannelFuture regFuture = initAndRegister();
  final Channel channel = regFuture.channel();
  if (regFuture.cause() != null) {
    return regFuture;
  }

  if (regFuture.isDone()) {
    ChannelPromise promise = channel.newPromise();
    //完成端口绑定
    doBind0(regFuture, channel, localAddress, promise);
    return promise;
  } else {
    final PendingRegistrationPromise promise = new PendingRegistrationPromise(channel);
    regFuture.addListener(new ChannelFutureListener() {
      @Override
      public void operationComplete(ChannelFuture future) throws Exception {
        Throwable cause = future.cause();
        if (cause != null) {
          promise.setFailure(cause);
        } else {
          promise.registered();
          doBind0(regFuture, channel, localAddress, promise);
        }
      }
    });
    return promise;
  }
}
```

initAndRegister():

```JAVA
final ChannelFuture initAndRegister() {
  Channel channel = null;
  try {
    //反射创建一个NioServerSocketChannel
    channel = channelFactory.newChannel();
    //在此方法中设置了NioServerSocketChannel的TCP属性
    init(channel);
  } catch (Throwable t) {
    if (channel != null) {
      channel.unsafe().closeForcibly();
      return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
    }
    return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
  }

  ChannelFuture regFuture = config().group().register(channel);
  if (regFuture.cause() != null) {
    if (channel.isRegistered()) {
      channel.close();
    } else {
      channel.unsafe().closeForcibly();
    }
  }
  return regFuture;
}
```

doBind0():

```java
private static void doBind0(
  final ChannelFuture regFuture, final Channel channel,
  final SocketAddress localAddress, final ChannelPromise promise) {
  channel.eventLoop().execute(new Runnable() {
    @Override
    public void run() {
      if (regFuture.isSuccess()) {
        //绑定端口
        channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
      } else {
        promise.setFailure(regFuture.cause());
      }
    }
  });
}
```

#### 4.Pipeline、Handler

##### 1.Pipelin、Handler、Context的创建

1. 每当创建 ChannelSocket 的时候都会创建一个绑定的 pipeline，一对一的关系，创建 pipeline 的时候也会创建 tail 节点和 head 节点，形成**最初的链表**。
2. 在调用 pipeline 的 addLast 方法的时候，会根据给定的 handler 创建一个 Context，然后，将这个 Context 插入到链表的尾端（tail 前面）。
3. Context 包装 handler，多个 Context 在 pipeline 中形成了双向链表
4. 入站方向叫 inbound，由 head 节点开始，出站方法叫 outbound ，由 tail 节点开始

##### 2.ChannelPipeline调度Handler

以read为例(AbstractChannelHandlerContext)

```java
//DefaultChannelPipeline
public final ChannelPipeline fireChannelRead(Object msg) {
  AbstractChannelHandlerContext.invokeChannelRead(head, msg);
  return this;
}
```



```java
//AbstractChannelHandlerContext)
public ChannelHandlerContext fireChannelRead(final Object msg) {
  invokeChannelRead(findContextInbound(), msg);
  return this;
}

static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
  final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
  EventExecutor executor = next.executor();
  if (executor.inEventLoop()) {
    next.invokeChannelRead(m);
  } else {
    executor.execute(new Runnable() {
      @Override
      public void run() {
        next.invokeChannelRead(m);
      }
    });
  }
}

private void invokeChannelRead(Object msg) {
  if (invokeHandler()) {
    try {
      //handler调用
      ((ChannelInboundHandler) handler()).channelRead(this, msg);
    } catch (Throwable t) {
      notifyHandlerException(t);
    }
  } else {
    fireChannelRead(msg);
  }
}
```

handler

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
  boolean release = true;
  try {
    if (acceptInboundMessage(msg)) {
      @SuppressWarnings("unchecked")
      I imsg = (I) msg;
      channelRead0(ctx, imsg);
    } else {
      release = false;
      //会再次调用context的下一节点
      ctx.fireChannelRead(msg);
    }
  } finally {
    if (autoRelease && release) {
      ReferenceCountUtil.release(msg);
    }
  }
}
```

#### 5.心跳检测

​	Netty 提供了 IdleStateHandler ，ReadTimeoutHandler，WriteTimeoutHandler 三个**Handler** 检测连接的有效性，重点分析 **IdleStateHandler** .

![image-20200522171111142](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200522171111142.png)

1) IdleStateHandler 可以实现心跳功能，当服务器和客户端没有任何读写交互时，并超过了给定的时间，则会触发用户handler 的userEventTriggered 方法。用户可以在这个方法中尝试向对方发送信息，如果发送失败，则关闭连接。
2) IdleStateHandler 的实现基于EventLoop 的定时任务，每次读写都会记录-一个值，在定时任务运行的时候，通过计算当前时问和设置时间和上次事件发生时间的结果，来判断足否空闲。
3)内部有3个定时任务，分别对应读事件，写事件，读写事件。通常用户监听读写事件就足够了。
4)同时，IdleStateHandler内部也考虑了一些极端情况:客户端接收缓慢，一次接收数据的速度超过了设置的空闲时间。Netty通过构造方法中的observeOutput属性来决定是否对出站缓冲区的情况进行判断。
5)如果出站缓慢，Netty 不认为这是空闲，也就不触发空闲事件。但第- - 次无论如何也是要触发的。因为第一次无法判断是出站缓慢还是空闲。当然，出站缓慢的话，可能造成0OM , 0OM比空闲的问题更大。
6)所以，当你的应用出现了内存溢出，0OM之类，并且写空闲极少发生(使用observeOutput 为true) ，那么就需要注意是不是数据出站速度过慢。

7)还有一个注意的地方:就是ReadTimeoutHandler ，它继承自IdleStateHandler， 当触发读空闲事件的时候，就触发ctx.fireExceptionCaught 方法，并传入一个ReadTimeoutException, 然后关闭Socket。
8)而WriteTimeoutHandler 的实现不是基于IdleStateHandler 的，他的原理是，当调用write 方法的时候，会创建一个定时任务，任务内容是根据传入的promise 的完成情况来判断是否超出了写的时间。当定时任务根据指定时间开始运行，发现promise 的isDone 方法返回false, 表明还没有写完，说明超时了，则抛出异常。当write方法完成后，会打断定时任务。



以读事件为例：（IdleStateHandler的内部类）

```java
private final class ReaderIdleTimeoutTask extends AbstractIdleTask {

  ReaderIdleTimeoutTask(ChannelHandlerContext ctx) {
    super(ctx);
  }

  @Override
  protected void run(ChannelHandlerContext ctx) {
		//得到用户设置的超时时间
    long nextDelay = readerIdleTimeNanos;
    if (!reading) {
      nextDelay -= ticksInNanos() - lastReadTime;
    }

    if (nextDelay <= 0) {
      // Reader is idle - set a new timeout and notify the callback.
      readerIdleTimeout = schedule(ctx, this, readerIdleTimeNanos, TimeUnit.NANOSECONDS);

      boolean first = firstReaderIdleEvent;
      firstReaderIdleEvent = false;

      try {
        IdleStateEvent event = newIdleStateEvent(IdleState.READER_IDLE, first);
        channelIdle(ctx, event);
      } catch (Throwable t) {
        ctx.fireExceptionCaught(t);
      }
    } else {
      // Read occurred before the timeout - set a new timeout with shorter delay.
      readerIdleTimeout = schedule(ctx, this, nextDelay, TimeUnit.NANOSECONDS);
    }
  }
}
```



#### 6.异步线程池

1. 在 Netty 中做耗时的，不可预料的操作，比如数据库，网络请求，会严重影响 Netty 对 Socket 的处理速度。

2. 而解决方法就是将耗时任务添加到异步线程池中。但就添加线程池这步操作来讲，可以有2种方式

   - handler 中加入线程池

   - Context 中添加线程池

##### 1.handler 中加入线程池

###### 1.示例

```java
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

  // group 就是充当业务线程池，可以将任务提交到该线程池
  // 这里我们创建了16个线程
  static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

    //按照原来的方法处理耗时任务
    /*
        //解决方案1 用户程序自定义的普通任务

        ctx.channel().eventLoop().execute(new Runnable() {
            @Override
            public void run() {

                try {
                    Thread.sleep(5 * 1000);
                    //输出线程名
                    System.out.println("EchoServerHandler execute 线程是=" + Thread.currentThread().getName());
                    ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));

                } catch (Exception ex) {
                    System.out.println("发生异常" + ex.getMessage());
                }
            }
        });

        ctx.channel().eventLoop().execute(new Runnable() {
            @Override
            public void run() {

                try {
                    Thread.sleep(5 * 1000);
                    //输出线程名
                    System.out.println("EchoServerHandler execute 线程2是=" + Thread.currentThread().getName());
                    ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));

                } catch (Exception ex) {
                    System.out.println("发生异常" + ex.getMessage());
                }
            }
        });*/

    /*
        //将任务提交到 group线程池
        group.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {

                //接收客户端信息
                ByteBuf buf = (ByteBuf) msg;
                byte[] bytes = new byte[buf.readableBytes()];
                buf.readBytes(bytes);
                String body = new String(bytes, "UTF-8");
                //休眠10秒
                Thread.sleep(10 * 1000);
                System.out.println("group.submit 的  call 线程是=" + Thread.currentThread().getName());
                ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));
                return null;

            }
        });

        //将任务提交到 group线程池
        group.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {

                //接收客户端信息
                ByteBuf buf = (ByteBuf) msg;
                byte[] bytes = new byte[buf.readableBytes()];
                buf.readBytes(bytes);
                String body = new String(bytes, "UTF-8");
                //休眠10秒
                Thread.sleep(10 * 1000);
                System.out.println("group.submit 的  call 线程是=" + Thread.currentThread().getName());
                ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));
                return null;

            }
        });


        //将任务提交到 group线程池
        group.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {

                //接收客户端信息
                ByteBuf buf = (ByteBuf) msg;
                byte[] bytes = new byte[buf.readableBytes()];
                buf.readBytes(bytes);
                String body = new String(bytes, "UTF-8");
                //休眠10秒
                Thread.sleep(10 * 1000);
                System.out.println("group.submit 的  call 线程是=" + Thread.currentThread().getName());
                ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));
                return null;

            }
        });*/


    //普通方式
    //接收客户端信息
    ByteBuf buf = (ByteBuf) msg;
    byte[] bytes = new byte[buf.readableBytes()];
    buf.readBytes(bytes);
    String body = new String(bytes, "UTF-8");
    //休眠10秒
    Thread.sleep(10 * 1000);
    System.out.println("普通调用方式的 线程是=" + Thread.currentThread().getName());
    ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));

    System.out.println("go on ");

  }
}

```

###### 2.源码

AbstractChannelHandlerContext:

```java
private void write(Object msg, boolean flush, ChannelPromise promise) {
  AbstractChannelHandlerContext next = findContextOutbound();
  final Object m = pipeline.touch(msg, next);
  EventExecutor executor = next.executor();
  if (executor.inEventLoop()) {
    if (flush) {
      next.invokeWriteAndFlush(m, promise);
    } else {
      next.invokeWrite(m, promise);
    }
  } else {
    //若不是executor线程不是当前线程时，将当前工作封装成task，然后放到mpsc队列，等IO任务执行完毕后执行队列中任务
    AbstractWriteTask task;
    if (flush) {
      task = WriteAndFlushTask.newInstance(next, m, promise);
    }  else {
      task = WriteTask.newInstance(next, m, promise);
    }
    safeExecute(executor, task, promise, m);
  }
}
```

##### 2.Context 中添加线程池

###### 1.示例

```java
//创建业务线程池
//这里我们就创建2个子线程
static final EventExecutorGroup group = new DefaultEventExecutorGroup(2);

public static void main(String[] args) throws Exception {
  // Configure SSL.
  final SslContext sslCtx;
  if (SSL) {
    SelfSignedCertificate ssc = new SelfSignedCertificate();
    sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey()).build();
  } else {
    sslCtx = null;
  }

  // Configure the server.
  EventLoopGroup bossGroup = new NioEventLoopGroup(1);
  EventLoopGroup workerGroup = new NioEventLoopGroup();
  try {
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
      .channel(NioServerSocketChannel.class)
      .option(ChannelOption.SO_BACKLOG, 100)
      .handler(new LoggingHandler(LogLevel.INFO))
      .childHandler(new ChannelInitializer<SocketChannel>() {
        @Override
        public void initChannel(SocketChannel ch) throws Exception {
          ChannelPipeline p = ch.pipeline();
          if (sslCtx != null) {
            p.addLast(sslCtx.newHandler(ch.alloc()));
          }
          //p.addLast(new LoggingHandler(LogLevel.INFO));
          p.addLast(new EchoServerHandler());
          //说明: 如果我们在addLast 添加handler ，前面有指定
          //EventExecutorGroup, 那么该handler 会优先加入到该线程池中
          //p.addLast(group, new EchoServerHandler());
        }
      });

    // Start the server.
    ChannelFuture f = b.bind(PORT).sync();

    // Wait until the server socket is closed.
    f.channel().closeFuture().sync();
  } finally {
    // Shut down all event loops to terminate all threads.
    bossGroup.shutdownGracefully();
    workerGroup.shutdownGracefully();
  }
}

```

###### 2.源码

AbstractChannelHandlerContext:

```java
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
  final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
  EventExecutor executor = next.executor();
  //判断是不是IO线程，不是则使用自创线程池异步执行
  if (executor.inEventLoop()) {
    next.invokeChannelRead(m);
  } else {
    executor.execute(new Runnable() {
      @Override
      public void run() {
        next.invokeChannelRead(m);
      }
    });
  }
}
```

##### 3.对比

1)第一种方式在handler 中添加异步，可能更加的自由，比如如果需要访问数据库，那我就异步，如果不需要，就不异步，异步会拖长接口响应时间。因为需要将任务放进mpscTask 中。如果I0时间很短，task 很多，可能一个循环下来，都没时间执行整个task， 导致响应时间达不到指标。
2)第二种方式足Netty 标准方式(即加入到队列)，但是，这么做会将整个handler 都交给业务线程池。不论耗时不耗时，都加入到队列里，不够灵活。
3)各有优劣，从灵活性考虑，第一种较好



### 7.RPC

#### 1.介绍

1. RPC（Remote Procedure Call）— 远程过程调用，是一个计算机通信协议。该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程
2. 两个或多个应用程序都分布在不同的服务器上，它们之间的调用都像是本地方法调用一样
3. 常见的 RPC 框架有: 比较知名的如阿里的Dubbo、google的gRPC、Go语言的rpcx、Apache的thrift， Spring 旗下的 Spring Cloud

![截屏2020-05-22 下午5.58.29](/Users/yinwenbo/Documents/img/截屏2020-05-22 下午5.58.29.png)

#### 2.RPC调用流程图

![image-20200522180012283](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200522180012283.png)

PRC调用流程说明：

1. *服务消费方*(client)以本地调用方式调用服务
2. client stub 接收到调用后负责将方法、参数等封装成能够进行网络传输的消息体
3. client stub 将消息进行编码并发送到服务端
4. server stub 收到消息后进行解码
5. server stub 根据解码结果调用本地的服务
6. 本地服务执行并将结果返回给 server stub
7. server stub 将返回导入结果进行编码并发送至消费方
8. client stub 接收到消息并进行解码
9. *服务消费方*(client)得到结果



小结：RPC 的目标就是将 2-8 这些步骤都封装起来，用户无需关心这些细节，可以像调用本地方法一样即可完成远程服务调用。