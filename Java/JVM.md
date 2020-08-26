# 					JVM

##1.JVM内存模型

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200819113646807.png" alt="image-20200819113646807" style="zoom:50%;" />

### 1.程序计数器

​	程序计数器（Program Counter Register）是一块较小的内存空间，由寄存器构成。它的作用可以看做是当前线程所执行的字节码的行号指示器，用来**记住下一条jvm指令的执行地址**。在虚拟机的概念模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

**特点**：

- 每个线程都有一条独立的pc，用来确保在多线程环境下，线程切换后能恢复到正确的执行位置。
- 此区域时唯一一个在虚拟机规范中没有规定任何OutOfMemoryError的区域

### 2.虚拟机栈

​	与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，**它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型**：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。**每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。**

​	在Java虚拟机规范中，都虚拟机栈规定了两种异常情况：如果线程请求的栈深度大于虚拟机所允许的深度，会抛出StackOverflowError异常；如果虚拟机栈可以动态扩展，并且扩展时无法申请到足够的内存，就会抛出OutOfMemoeyError异常。

```java
//StackOverflowError	无限递归可触发此异常。两个线程互相引用时也可触发此异常
public class TestStackOverflowError {
    public static void main(String[] args) {       
        //情况一
        //stackOverflowError();
        
        //情况二
        TestStackOverflowError t=new TestStackOverflowError();
        t.a();
    }
    private static void stackOverflowError() {
        stackOverflowError();
    }
    public void a(){
        b();
    }
    public void b(){
        a();
    }
}


//OutOfMemoeyError 		在多线程环境下，无限制创建新线程，且让线程一直处于死循环中可以触发此异常。
//别试!!!!!!!!!!!!!!!!，差点a把我mbp干报废喽
//Java进程内存=Java最大堆内存(Xmx)+Java方法区内存+Java每个线程分配的栈内存(Xss)*线程数量，每个线程分配的栈内存越多，可创建的线程数量越少
public class TestStackOOM {

    public  void loop(){
        while (true){

        }
    }

    public void create(){
        while (true){
            new Thread(()->{
                loop();
            }).start();
        }
    }

    public static void main(String[] args) {
        TestStackOOM t=new TestStackOOM();
        t.create();
    }
}

```

#### 线程诊断

##### 1.cpu占用过高

```shell
#1.top 定位占用cpu过高的进程id
top
#2.ps命令定位线程id
ps H -eo pid,tid,%cpu |grep 进程id
#3.jstack定位源码行号
jstack 进程id

#输出信息
java.lang.Thread.State: RUNNABLE
	at intetview.OOM.TestStackOOM.loop(TestStackOOM.java:6)
	at intetview.OOM.TestStackOOM.main(TestStackOOM.java:22)
```

##### 2.程序运行长时间无结果

```shell
#可能出现死锁
jstack 线程id
```

###3.本地方法栈

​	本地方法栈（Native MethodStacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而**本地方法栈则是为虚拟机使用到的Native 方法服务**。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（比如Sun HotSpot 虚拟机）直接就把本地方法栈和虚拟机栈合二为一。

​	与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

### 4.堆

​	对于大多数应用来说，Java堆（Java Heap）是Java虚拟机所管理的内存中**最大**的一块。Java堆是被所有**线程共享**的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，**几乎所有的对象实例都在这里分配内存**。

​	Java堆是**垃圾收集**器管理的主要区域，当没有空间再new对象时，会出现OOM异常。

```java
public class TestHeapOOM {
    public static void main(String[] args) {
        String s="1";
        while (true){
            s+="1"+ new Random().nextInt(10000000);
        }
    }
}

//jvm参数设置   idea RUN->VM options:-Xms1m -Xmx1m
/*
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at intetview.OOM.TestHeapOOM.main(TestHeapOOM.java:9)
*/
```

#### 堆内存诊断

##### 1.jmap

```shell
#jmap可查看某一时刻的堆内存占用情况
[yinwenbo]~ $ jps			#查看java进程
13050 Launcher
13051 TestHeapOOM
13052 Jps

[yinwenbo]~ $ jmap -heap 13051	#jmap打印的当前java的内存信息
Attaching to process ID 13051, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:						#堆的配置信息
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2097152 (2.0MB)
   NewSize                  = 1572864 (1.5MB)
   MaxNewSize               = 1572864 (1.5MB)
   OldSize                  = 524288 (0.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:									#堆中各个部分的使用情况
PS Young Generation
Eden Space:
   capacity = 524288 (0.5MB)
   used     = 503000 (0.47969818115234375MB)
   free     = 21288 (0.02030181884765625MB)
   95.93963623046875% used
From Space:
   capacity = 524288 (0.5MB)
   used     = 0 (0.0MB)
   free     = 524288 (0.5MB)
   0.0% used
To Space:
   capacity = 524288 (0.5MB)
   used     = 0 (0.0MB)
   free     = 524288 (0.5MB)
   0.0% used
PS Old Generation
   capacity = 524288 (0.5MB)
   used     = 436640 (0.416412353515625MB)
   free     = 87648 (0.083587646484375MB)
   83.282470703125% used

2233 interned Strings occupying 157976 bytes.
```

##### 2.jconsole

​	jconsole 是java监视和管理控制台，具有图形界面可以**连续查看堆内存使用情况**

```shell
[yinwenbo]~ $ jconsole
```

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200821160954806.png" alt="image-20200821160954806" style="zoom:33%;" />



##### 3.jvisualvm

```shell
[yinwenbo]~ $ jvisualvm
```

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200821161612518.png" alt="image-20200821161612518" style="zoom:25%;" />

### 5.方法区

​	方法区和堆一样，是各个**线程共享**的内存区域。用于**存放已被加载的类信息、常量、静态变量**、**即时编译器编译后的代码**等数据。对这块区域进行垃圾回收的主要目标是对**常量池的回收和对类的卸载**，也会出现OOM异常。

​	Java7之前方法区在 Hotspot 中的具体实现是永久代，Java8 及之后的版本使用 Metaspace 来取代永久代。Metaspace与永久代最大的区别是**Metaspace并不在虚拟机内存中而是使用本地内存**。

​	当spring 、mybatis等框架动态加载类时，使用不当就可能会导致元空间OOM

```java
//不断生成类往元空间灌，类占据的空间超过MaxMetaspaceSize就会产生OOM
//-XX:MaxMetaspaceSize=1m
public class TestMetaspaceOOM extends ClassLoader{
    public static void main(String[] args) {
        int j=0;
        try{
            TestMetaspaceOOM t=new TestMetaspaceOOM();
            for (int i = 0; i < 1000000; i++,j++) {
              	//ClassWriter生成类的二进制字节码
                ClassWriter cw=new ClassWriter(0);
              	//版本号，public,类名，包名，父类，接口
                cw.visit(Opcodes.V1_8,Opcodes.ACC_PUBLIC,"Class"+i,null,"java/lang/Object",null);
                byte[] array = cw.toByteArray();
              	//执行类的加载
                t.defineClass("Class"+i,array,0,array.length);
            }
        }finally {
            System.out.println(j);
        }
    }
}
//结果
/*
731
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at intetview.OOM.TestMetaspaceOOM.main(TestMetaspaceOOM.java:18)
*/
```

### 6.运行时常量池

java中的常量池分3种：

（1）常量池（class文件常量池）：**存储在堆中**，编译时产生对应的class文件，主要包含**字面量和符号引用**

（2）运行时常量池：**存储在与本地内存的元空间中**，JVM运行时，**在类加载完成后，每个class常量池中信息就会放入运行时常量池**，并把常量池的符号引用转换为直接引用。**程序运行期间生成的常量也会存储于此**。

（3）字符串常量池 ：**存储于堆中**。字符串常量池（StringTable）是运行时常量池的组成部分，但存储于堆中，StringTable是一个哈希表，本质上是`HashSet<String>`，不能扩容，里面存储的是字符串引用，具体的实例对象存储在堆中，这个StringTable表只有一份，被**所有类共享**。串池1.6在方法区中，full gc时回收，1.8在堆中，**年轻代时回收**。**若分配内存失败，会回收无引用的字符串常量，在内存紧张时触发。**

```java
//字符串变量stringBuilder   字符串常量编译期优化
//使用intern会把串池中没有的字符串放入串池
public class TestString {
    public static void main(String[] args) {
        //最开始"a"是在常量池中，只是常量池中的一个符号，还不是java的字符串对象。
        String s1="a";//类加载 第一次用到"a"符号时，ldc指令会把a符号变成字符串a对象放入串池，此时串池：["a"]
        String s2="b";//["a","b"]
        String s3="ab";//["a","b","ab"]

        //1.new StringBuilder().append("a").append("b").toString();
        //2.s4=toString(): return new String(value, 0, count);
        String s4=s1+s2;//此时s4对象在堆（new String）中

        System.out.println(s3==s4);     //false

        String s5="a"+"b";//直接在串池中找到"ab"赋给s5，在编译期确定，属于编译期优化，s4则是在运行时执行的
        System.out.println(s3==s5);     //true



        //intern()会将堆中的对象放到串池，
        //若串池中没有此字符串，则会放入并返回串池的对象   -->对应true
        //若串池中有此字符串，则不会放入串池。           -->对应false
//        String x="xy";                //false
        String y=new String("x")+new String("y");
        y.intern();
        String x="xy";                  //true
        System.out.println(x==y);       
        
    }
}
```

#### 串池调优

```java
//由于StringTable底层是哈希表实现的。桶的数量决定性能，桶的个数太小会导致碰撞概率越高，可以适当调整hash桶的大小
-XX:StringTableSize=20000
  
  
//若项目中存在大量的字符串，且字符串有相同值，可以使用intern()将堆中的字符串添加到串池中，减少堆内存的使用。
```

### 7.直接内存

​	直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现。**直接内存常见于NIO操作**，用作数据缓冲区。由于直接在物理内存中读写，所以**读写性能高，但分配回收的成本也较高**，并且不受**JVM内存回收的管理**。

```java
//NIO的allocateDirect()分配过大导致的直接内存溢出
//-XX:MaxDirectMemorySize=10m
public class TestDirectbuffermemory {
     public static void main(String[] args) {
        //使用的是直接内存，不收GC控制
        ByteBuffer bf=ByteBuffer.allocateDirect(1024*1024*1024);

        //使用的堆内存，受GC控制
        ByteBuffer.allocate(1024*1024);
    }
}

//输出信息
/*
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
	at java.nio.Bits.reserveMemory(Bits.java:693)
	at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
	at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:311)
	at intetview.OOM.TestDirectbuffermemory.main(TestDirectbuffermemory.java:8)
*/
```

#### 分配释放原理

```java
//在分配的时候直接在DirectByteBuffer的构造器中执行unsafe的allocateMemory方法
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
//unsafe类
public native long allocateMemory(long var1);


//释放
//Cleaner extends PhantomReference<Object>，Cleaner是一个虚引用对象，会监测DirectByteBuffer，当DirectByteBuffer被回收时，会由RefrenceHandler守护线程调用Cleaner对象的clean方法，此方法就会执行下面的run方法shifangneicun。
private static class Deallocator implements Runnable{

  private static Unsafe unsafe = Unsafe.getUnsafe();

  private long address;
  private long size;
  private int capacity;

  private Deallocator(long address, long size, int capacity) {
    assert (address != 0);
    this.address = address;
    this.size = size;
    this.capacity = capacity;
  }

  public void run() {
    if (address == 0) {
      // Paranoia
      return;
    }
    unsafe.freeMemory(address);
    address = 0;
    Bits.unreserveMemory(size, capacity);
  }

}
//Unsafe中释放内存的方法
public native void freeMemory(long var1);
```

#### 禁用显示回收的影响

​	在项目中，JVM优化时常常会禁止显示回收（System.gc() ），因为System.gc()会触发full gc，耗费性能，而在内存足够时，未使用的直接内存不会被回收，占用大量的内存，此时需要显示的调用unsafe.freeMemory()来回收直接内存。

###其他OOM的情况

#### GC overhead limit exceeded

```java
//超过98%的时间去GC但回收了不到2%的堆内存
//-Xms5m -Xmx5m
public class TestGCoverheadlimitexceeded {
    public static void main(String[] args) {
        int i=0;
        List list=new ArrayList();
        try{
            while (true){
                list.add(String.valueOf(i++).intern());
            }
        }catch(Throwable e){
            System.out.println("i="+i);
            e.printStackTrace();
            throw e;
        }

    }
}

/*
i=78189
java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.Integer.toString(Integer.java:403)
	at java.lang.String.valueOf(String.java:3099)
	at intetview.OOM.TestGCoverheadlimitexceeded.main(TestGCoverheadlimitexceeded.java:13)
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.Integer.toString(Integer.java:403)
	at java.lang.String.valueOf(String.java:3099)
	at intetview.OOM.TestGCoverheadlimitexceeded.main(TestGCoverheadlimitexceeded.java:13)

*/
```

#### unable to create new native thread

```java
//线程创建过多时引发的异常
public class TestUnableToCreateNewNativeThread {
    public static void main(String[] args) {
        int i=0;
        while (true){
            i++;
            new Thread(()->{
                try {
                    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
                }catch(InterruptedException e){
                  e.printStackTrace();
                }
            }).start();
            System.out.println(i);
        }
    }
}
/*
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
	at java.lang.Thread.start0(Native Method)
	at java.lang.Thread.start(Thread.java:714)
	at intetview.OOM.TestUnableToCreateNewNativeThread.main(TestUnableToCreateNewNativeThread.java:13)
*/
```

## 2.GC

​	GC关注Java堆和方法区的内存该如何管理。因为Java堆和方法区这两个区域有着很显著的不确定性：一个接口的多个实现类需要的内存可能会不一样，一个方法所执行的不同条件分支所需要的内存也可能不一样，只有处于运行期间，我们才能知道程序究竟会创建哪些对象，创建多少个对象，这部分内存的分配和回收是动态的。

​	GC是**守护线程最典型的应用**。垃圾回收**主要发生在堆**中，少部分发生在方法区，**在方法区主要回收常量池中废弃的常量和卸载不再使用的类**。

### 1.方法区的GC

方法区的垃圾收集主要回收两个部分内容：废弃的常量和不再使用的类。

- 回收废弃常量：一个字符串"java"曾经进入常量池，但是当前系统有没有任何一个字符串对象的值是"java"，换句话说，已经没有任何字符串对象引用常量池中的"java"常量，且虚拟机中也没有其他地方引用这个字面量。如果再这个时候发生内存回收就会被系统清理出常量池。常量池中其他类（接口）、方法、字段的符号引用也与此类似。
- 判断一个类型是否属于"不再被使用的类"需要满足下面三个条件：
  - 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
  - 加载该类的类加载器已经被回收。
  - 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

### 2.回收判断算法

#### 1.引用计数法

​	每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。

```java
//循环引用示例
public class ReferenceCountingGC {
    public Object obj = null;
		//占用内存 方便观察
    private byte[] occupy= new byte[1024*1024];

    public static void testGC() {
        ReferenceCountingGC r1 = new ReferenceCountingGC();
        ReferenceCountingGC r2 = new ReferenceCountingGC();
        r1.obj = r2;
        r2.obj = r1;

        r1 = null;
        r2 = null;

        //显示回收
        System.gc();
    }

    public static void main(String[] args) {
     		testGC();
    }
}
```

#### 2.可达性分析法

​	可达性分析法是通过一系列成为"GC Roots"的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为"引用链"（Reference Chain），如果**某个对象到GC Roots间没有任何引用链相连**，或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。

##### 可作为GC Root的对象

- 在虚拟机栈（栈帧中的本地变量表）中引用的对象，譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
- 本地方法栈中引用的对象
- 在方法区中类静态属性引用的对象
- 在方法区中常量引用的对象，如字符串常量（String Table）里的引用。

#### 3.四种引用(java.lang.ref)

​	引用分为强引用（Strongly Reference）、软引用（Soft Reference）、弱引用（Weak Reference）和虚引用（Phantom Reference）4种，这4种引用强度依次逐渐减弱。

- 强引用是最传统的"引用"的定义，是指在程序代码之中普遍存在的引用赋值，所有**new出来的对象**均为强引用。**只要强引用对象还可达，就不会被垃圾回收。**
- 软引用是用来描述一些还有用、但非必须的对象。**当发生垃圾回收并且内存空间不够时，就会回收软引用对象**。
- 弱引用也是用来描述那些非必须对象，但是他的强度比软引用更弱一些，**只要发生GC，就会被回收**。

```java
public class TestWeakHashMap {
    public static void main(String[] args) {
        //WeakHashMap的使用（用于缓存）当已经put的键值对的key被置为null时，下次gc会回收次键值对
        WeakHashMap<Integer,String> map=new WeakHashMap<>();

        Integer i=new Integer(1);
        String name="weakhashmap";

        map.put(i,name);
        System.out.println(map);					//{1=weakhashmap}

        i=null;
        System.out.println(map);					//{1=weakhashmap}

        System.gc();
        
        System.out.println(map+"   "+i);	//{}   null 
    }
}
```

- 虚引用也称为"幽灵引用"或者"幻影引用"，它是最弱的一种引用关系，**随时会被回收，必须配合引用队列使用**，虚引用对象被回收时会被添加到引用队列中，允许被回收的引用做一些后续的通知。如监测直接内存回收的cleaner对象就是一个虚引用对象。

```java
//当GC释放对象内存时，会将虚引用对象加入引用队列中，方便对虚引用监测的对象进行通知
//当关联的引用队列（queue）有数据时，意味着虚引用指向堆内存的对象（obj）被回收。
public class TestPhantomReference {
    public static void main(String[] args) {

        Object obj=new Object();
        ReferenceQueue queue=new ReferenceQueue();
        PhantomReference pr=new PhantomReference(obj,queue);
        System.out.println(obj);															//java.lang.Object@60e53b93
        System.out.println(pr.get());													//null
        System.out.println(queue.poll());											//null

        System.out.println("=========================");

        obj=null;
        System.gc();
        System.out.println(obj);															//null
        System.out.println(pr.get());													//null
        System.out.println(queue.poll());											//java.lang.ref.PhantomReference@5e2de80c

    }
}
```

##### 软弱引用的应用场景

```shell
假如有一个应用需要读取大量的本地图片:
	如果每次读取图片都从硬盘读取则会严重影响性能，
	如果一次性全部加载到内存中又可能造成内存溢出

此时使用软引用可以解决这个问题:
	设计思路是：用一个 Hashmap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，VM 会自动回收这些缓存图片对象所占用的空间，从而有效地避免了 OOM 的问题。
	Map<String,SoftReference<Bitmap>> imageCache=new Hashmap<String, SoftReference<Bitmap>>)
```

#### 4.对象的真正死亡

- 在可达性算法中被判定为不可达对象后，至少要经历两次标记过程对象才会真正死亡：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它会被第一次标记，随后进行一次筛选，筛选的条件是此对象是否有必要执行finalize()方法。加入对象没有**重写finalize()方法**，或者**finalize()方法已经被虚拟机调用过**，那么虚拟机将这两种情况都视为"没有必要执行"。
- 如果这个对象被判定为确有必要执行finalize()方法，那么该对象将会被放置在一个名为F-Queue的队列之中，并在稍后由一条虚拟机自动建立的、低调度优先级的Finalizer线程去执行它们的finalize()方法。finalize()方法是对象逃脱死亡命运的最后一次机会，如果对象在finalize()中重新与引用链上的一个对象建立关联即可避免被回收，譬如把自己（this关键字）赋值给某个类变量或者对象的成员变量。

```java
public class TestFinalize {

    //回收后与此GC Root关联
    private static TestFinalize SAVE_HOOK = null;

    public void isAlive() {
        System.out.println("Yes, I am still alive!");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method is exceted!");
        // 重新建立关联，避免被回收
        SAVE_HOOK = this;
    }

    public static void main(String[] args) throws InterruptedException {
        SAVE_HOOK=new TestFinalize();
        //第一次置null
        SAVE_HOOK=null;
        System.gc();
        //Finalizer方法优先级很低，暂停0.5秒以等待它
        Thread.sleep(500);

        if(SAVE_HOOK!=null){
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("dead................");
        }

        //第二次置null，测试finalize是否还会执行
        SAVE_HOOK=null;
        System.gc();
        //Finalizer方法优先级很低，暂停0.5秒以等待它
        Thread.sleep(500);

        if(SAVE_HOOK!=null){
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("dead................");
        }
    }
}

/*结果
finalize method is exceted!
Yes, I am still alive!
dead................
*/
```

### 3.垃圾回收算法

#### 1.标记清除算法

​	标记-清除算法是最基础的垃圾收集算法，该算法分为"标记"和"清除"两个阶段：首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象

缺点：

- **执行效率不稳定**，如果Java堆中包含大量对象，而且其中大部分是需要回收的，这时必须进行大量标记和清除的动作，导致标记和清除两个过程的执行效率都随对象数量增长而降低。
-  内存空间的碎片化问题，**标记、清除之后会产生大量不连续的内存碎片**，空间碎片太多可能会导致当以后再程序运行过程中需要分配较大对象时**无法找到足够的连续内存而不得不提前触发另一次垃圾收集**。

#### 2.标记整理算法

​	首先标记出所有需要回收的对象，之后让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。

缺点：

- 虽不产生内存碎片，但是移动对象时需要产生大量的开销

#### 3.复制算法

​	复制算法将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉。

缺点：

- 复制算法意味着将内存的**缩小为实际内存的一半**

#### 4.分代回收算法

​	分代回收算法是将堆中的内存分为新生代和老年代，新生代的大多数对象都是朝生夕死，所以采用复制算法，而老年代的对象存活时间长，采用标记清理或标记整理算法。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200823170014144.png" alt="image-20200823170014144" style="zoom:33%;" />

##### 回收流程

- 对象首先分配到Eden区
- Eden区满后，会触发Minor GC，Eden区存活的对象**寿命+1**放入From区
- 之后的Minro Gc会回收Eden区和From区，将存活的对象寿命+1放入To区，之后From和To区交换
- 重复以上流程，当对象的寿命到达15（ 默认）时，将对象加入老年代中
- 当老年代空间不足时会先触发Minor GC，回收后空间仍不足则触发Full GC

##### 解释

- Minor GC会引发STW(Stop the world)，就是会暂停用户线程，等垃圾回收线程结束后才会恢复用户线程。（因为GC时对象的地址会发生改变）
- Full GC引发STW的时间会更长（System.gc() 就会引发Full GC）
- 如果Full GC后内存仍不足，则会导致OOM
- 若添加的对象占用内存大于新生代的剩余内存，则会将此对象直接放入老年代中

##### JVM参数

| 含义                                           | 参数                               |
| ---------------------------------------------- | ---------------------------------- |
| 堆初始大小                                     | -Xms(-XX:InitialHeapSize)          |
| 堆最大大小                                     | -Xmx(-XX:MaxHeapSize)              |
| 新生代大小                                     | -Xmn                               |
| 幸存区比例（如果为4，则Eden:From:To=4:1:1）    | -XX:SurvivorRatio=ratio            |
| 晋升阈值（必须为0-15，0则直接进入老年代）      | -XX:MaxTenuringThreshold=threshold |
| 打印晋升详情                                   | -XX:+PrintTenuringDistribution     |
| 开启Full GC前先Minor  GC                       | -XX:+ScavengeBeforeFullGC          |
| 配置老年代占比（如果为4，则老年代:新生代=4:1） | -XX:NewRatio=ratio                 |

### 4.垃圾收集器

#### 1.垃圾收集器与回收算法

​	GC**算法**是内存回收的**理论**，垃圾**收集器**就是按照这些算法的具体**实现**。每个垃圾收集器的回收算法也有差异。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-202008231700141.png" alt="image-202008231700141" />

#### 2.Serial收集器

​	Serial（串行）收集器是一个**单线程**工作的收集器，**进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束**。适用于**单线程环境下且内存较小的环境**。（一个服务员清理多个桌子）

#### 3.Serial Old收集器

​	Serial Old是Serial收集器的老年代版本，它同样是一个**单线程收集器，使用标记整理算法**。这个收集器的主要意义也是供客户端模式下的HotSpot虚拟机使用。如果在服务端模式下，它也有可能有两种用途： 一种是在JDK5以及之前的版本中与Parallel Scavenge收集器搭配使用，另一种就是作为CMS收集器发生失败时的后备预案，在并发收集发生Concurrent Mode Failure时使用。

#### 4.Parallel Scavenge收集器

​	Parallel Scavenge（并行）收集器的**多个垃圾回收线程并行工作，此时用户线程是暂停的**（多个服务员清理多个桌子）。**主要适合在多线程环境下且堆内存较大的环境，可用于后台运算而不需要太多交互的分析任务。**

​	Parallel Scavenge收集器在**单位时间内STW的时间是最短**的，它的目标是达到一个可控制的**吞吐量**（吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 运行垃圾收集时间)）。比如用户代码加上垃圾收集总共耗费了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%

#### 5.Parallel Old收集器

​	Parallel Old是Parallel Scavenge收集器的老年代版本，支持多线程并发收集，基于**标记整理**算法实现。该收集器是JDK6时才开始提供的。

#### 6.ParNew收集器

​	ParNew收集器实质上是Serial收集器的多线程并行版本，除了同时使用多条线程进行垃圾收集之外，其余的行为包括Serial收集器可用的所有控制参数、收集算法Stop The World、 对象分配规则、回收策略等都与Serial收集器完全一致。

#### 7.CMS收集器

​	CMS（Concurrent Mark Sweep）收集器是一种以获取**最短回收停顿时间**为目标的收集器。CMS收集器是基于**标记清除**算法实现的，它的运作过程分为四个阶段

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200823204932375.png" alt="image-20200823204932375" style="zoom:50%;" />

- 初始标记：只是标记一下GC roots能直接关联的对象，**会STW**，速度很快

- 并发标记：进行GC ROOTS跟踪的过程，和用户线程一起工作，**不会STW**。主要标记过程，标记全部对象

- 重新标记：（二次确认过程）修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，**会STW**。

- 并发清除：清除GC ROOT不可达对象，因为是标记清理不用移动存活对象，所以可以和用户线程一起工作，**不会STW**。

##### CMS的优缺点：

​	CMS的优点是回收阶段是并发执行的，低停顿低延迟，适合互联网等对响应时间要求较高的项目。

​	CMS的缺点主要体现在以下几个方面：

- CMS收集器会**降低总吞吐量**，因为在并发阶段，它虽然不会导致用户线程停顿，但却会因为占用了一部分线程（或者说处理器的计算器的计算能力）而导致应用程序变慢。
- CMS收集器**无法处理浮动垃圾**。在CMS的并发标记和并发清理阶段， 用户线程是还在继续运行的，程序在运行自然就还会伴随着有新的垃圾对象不断产生，但这一部分垃圾对象是出现在标记过程结束以后，CMS无法在当次收集中处理掉它们，只好留待下一次垃圾收集时再清理掉。这一部分垃圾就称为"浮动垃圾"。
- CMS收集器**会产生内存碎片**。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很多剩余空间， 但就是无法找到足够大的连续空间来分配当前对象，而不得不**提前触发一次Full GC**的情况。

#### 8.Grabage First收集器

​	Garbage First（简称G1）收集器是垃圾收集器技术发展历史上的里程碑式的成果，它开创了收集器面向**局部收集的设计思路和基于Region的内存布局形式**。G1将堆内存分割成不同的区域然后并发的对其进行垃圾回收，是一款主要面向服务端应用的垃圾收集器。**Garbage First的意思是优先处理那些垃圾多的内存块的意思**。

​	**G1把连续的Java堆划分为多个大小相等的独立区域（Region），每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor空间或者老年代空间**。收集器能够对扮演不同角色的Region采用不同的策略去处理， 这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。

​	**Region中还有一类特殊的Humongous区域，专门用来存储大对象。G1认为只要大小超过一个Region容量一半的对象即可判定为大对象**。每个Region的大小可以通过参数-XX:G1HeapRegionSize设定， 取值范围为1MB~32MB，且应为2的N次幂。而对于那些超过了整个Region容量的超级大对象，将会被存放在N个连续的Humongous Region之中，G1的大多数行为都把Humongous Region作为老年代的一部分进行看待。

##### 1.实现算法

​	G1从**整体来看是基于"标记-整理"算法**实现的收集器，但从**局部（两个Region之间）上看是基于"标记-复制"算法实现**，无论如何，这两种算法都意味着G1运作期间**不会产生内存空间碎片**， 垃圾收集完成之后能提供规整的可用内存。这种特性有利于程序长时间运行，在程序为大对象分配内存时不容易因无法找到连续内存空间而提前触发下一次收集。

##### 2.回收过程

​	G1垃圾回收分为两个阶段：
​	（1）全局并发标记阶段(Global Concurrent marking)
​	（2）拷贝存活对象阶段(evacuation)

###### 全局并发标记阶段

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200823225403959.png" alt="image-20200823225403959" style="zoom:50%;" />

- 初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，在Young GC时标记，**会STW**
- 并发标记：从GC Root开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。
- 最终标记：修正并发标记期间因程序运行导致标记发生变化的一小部分对象，JVM会将引用发生变化的对象加入写屏障指令,把此对象加入一个队列,然后将队列中的对象取出做最终标记。**会STW**
- 筛选回收：对各个Region的回收价值和成本进行排序， 根据用户所期望的停顿时间来制定回收计划，**G1每次并不会回收整代内存，到底回收多少内存就看用户配置的暂停时间，配置的时间短就少回收点，配置的时间长就多回收点，伸缩自如（`-XX:MaxGCPauseMillis=200`设置停顿时间）。**G1可以自由选择任意多个Region构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧Region的全部空间。 这里的操作涉及存活的对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的。**会STW**

###### 拷贝存活对象阶段

​	Evacuation阶段是全暂停的。该阶段把一部分Region里的活对象拷贝到另一部分Region中，从而实现垃圾的回收清理。此阶段的整个过程是完全暂停的。

##### 3.回收步骤

​	G1的垃圾回收分为Young GC、YoungGC+并发标记和Mixed GC。如果实在赶不上回收速度的情况下还会使用Full GC(Serial GC)。Young GC、YoungGC+并发标记和Mixed GC三个模式会不断地切换进行

###### Young GC

Young GC主要是针对 Eden 区进行收集，Eden 区耗尽后会被触发，主要是小区域收集+形成连续的内存块，避免内存碎片

（1）Eden 区的数据移动到 Survivor 区，假如出现 Survivor 区空间不够，Eden 区数据会部会晋升到 Old 区

（2）Survivor 区的数据移动到新的 Survivor 区，部分数据晋升到 Old 区 

（3）最后 Eden 区收拾干浄了，GC 结束，用户的应用程序继续执行。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200824000443979.png" alt="image-20200824000443979" style="zoom:50%;" />

###### YoungGC+并发标记

​	当老年代占用堆空间达到阈值时，会进行并发标记。由`-XX:InitiatingHeapOccupancyPercent=45`决定，默认为45%

###### Mixed GC

​	对E、S、O进行全面的回收，会收集年轻代的所有Region+全局并发标记阶段选出的收益高的Region。

##### 4.优势

- 不会产生内存碎片
- 可根据用户所期望的停顿时间来制定回收计划

#### 9.查看默认的垃圾收集器

```shell
#在VM参数中输入：
-XX:+PrintCommandLineFlags -version

#显示结果	：	-XX:+UseParallelGC 
-XX:InitialHeapSize=134217728 -XX:MaxHeapSize=2147483648 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC 
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```

## 3.类加载与字节码

### 1.字节码指令

#### 1.常见字节码指令

```shell
bipush 10  			#将一个byte压入操作数栈（长度补齐四个字节）
sipush 10  			#将一个short压入操作数栈（长度补齐四个字节）
ldc				 			#将一个int压入操作数栈
ldc 2_w		 			#将一个long压入操作数栈（分两次压入）

istore 1   			#将操作数栈顶数据弹出，存入局部变量表
iload  1   			#将局部变量表slot 1中的数据读到操作数栈中
getStatic  			#将运行时常量池中符号常量对应的堆中对象的引用放入操作数栈中
invokervirtual	#找到常量池中的数据，定位到运行时常量池，然后定位到方法区的方法，生成新的栈帧，传参执行栈帧中的字节码

iinc 1,1			  #直接在局部变量表上进行 1+1的运算(i++ 和 ++i 的区别只是先执行iload还是iinc，i++是先iload)

goto            #转到对应的指令号（循环，判断）
```

##### iinc

```java
public class Iinc {
    public static void main(String[] args) {
        int i=1;
        i=i++;                  //i=1
        int j=i++;              //j=1 i=2
        int k= i+ ++i * i++;    //k=2+3*3  i=4
        System.out.println("i="+i+" j="+j+" k="+k);
    }
}
```

#### 2.控制语句的字节码

##### 判断

```java
public class TestByteCode {
    public static void main(String[] args) {
        int a=0;
        if(a==0){
            a=10;
        }else {
            a=20;
        }
    }
}
/*
stack=1, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iload_1
         3: ifne          12
         6: bipush        10
         8: istore_1
         9: goto          15
        12: bipush        20
        14: istore_1
        15: return
*/
```

##### while

```java
public class TestByteCode {
    public static void main(String[] args) {
        int a=0;
        while(a<10){
            a++;
        }
    }
}
/*
 stack=2, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iload_1
         3: bipush        10
         5: if_icmpge     14
         8: iinc          1, 1
        11: goto          2
        14: return
*/
```

##### do while

```java
public class TestByteCode {
    public static void main(String[] args) {
        int a=0;
        do {
            a++;
        }while(a<10);
    }
}

/*
stack=2, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iinc          1, 1					//自增后才和条件里做比较，do while循环体内的内容至少执行一次
         5: iload_1
         6: bipush        10
         8: if_icmplt     2
        11: return
*/
```

##### for循环

```java
public static void main(String[] args) {
  for (int i = 0; i < 10; i++) {

  }
}
/* 
stack=2, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iload_1
         3: bipush        10
         5: if_icmpge     14
         8: iinc          1, 1
        11: goto          2
        14: return
*/
```

#### 3.构造方法	

##### `<cinit>`

​	`<cinit>`在类加载的初始化阶段调用，编译器会**从上至下**收集所有静态代码块和静态成员赋值的代码，合并为特殊的方法`<cinit>()V`

```java
public class TestCinit {
    static int a=10;

    static {
        a=20;
    }

}

/*
 stack=1, locals=0, args_size=0
         0: bipush        10
         2: putstatic     #2                  // Field a:I
         5: bipush        20
         7: putstatic     #2                  // Field a:I
        10: return
*/
```

##### `<init>`

​	`<init>()V`在**实例化对象**时被调用，编译器会根据从上至下的顺序收集所有{}代码块和成员变量赋值的代码，形成新的构造方法，但是原始的构造方法总是放到最后。

```java
public class TestInit {

    private String a="a";


    {
        a="b";
    }

    {
        b=20;
    }
    private int b=10;

    public TestInit(String a, int b) {
        this.a = a;
        this.b = b;
    }

    public static void main(String[] args) {
        TestInit t=new TestInit("c",30);
        System.out.println(t.a);				//c
        System.out.println(t.b);				//30
    }
}

/*
 stack=2, locals=3, args_size=3
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String a
         7: putfield      #3                  // Field a:Ljava/lang/String;
        10: aload_0
        11: ldc           #4                  // String b
        13: putfield      #3                  // Field a:Ljava/lang/String;
        16: aload_0
        17: bipush        20
        19: putfield      #5                  // Field b:I
        22: aload_0
        23: bipush        10
        25: putfield      #5                  // Field b:I
        28: aload_0
        29: aload_1
        30: putfield      #3                  // Field a:Ljava/lang/String;
        33: aload_0
        34: iload_2
        35: putfield      #5                  // Field b:I
        38: return
*/
```

#### 4.方法调用

```java
public class TestMethod {

    private void test1(){}									  //私有方法invokespecial，指令在编译器确定
    public void test2(){}											//普通方法invokevirtual，可能存在重写，指令在运行期确定

    public static void test3(){}							//静态方法invokestatic，指令在编译器确定

    private final void test4(){}							//私有方法invokespecial，指令在编译器确定

    public TestMethod() {}										//构造方法invokespecial，指令在编译器确定

    public static void main(String[] args) {
        TestMethod t=new TestMethod();

        t.test1();
        t.test2();
        t.test3();														//对象.静态方法底层会产生 aload_1和pop两条没用的虚拟机指令

        t.test4();
    }
}

/*
 stack=2, locals=2, args_size=1
         0: new           #2                  // class JVM03/TestMethod
         3: dup
         4: invokespecial #3                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: invokespecial #4                  // Method test1:()V
        12: aload_1
        13: invokevirtual #5                  // Method test2:()V
        16: aload_1
        17: pop
        18: invokestatic  #6                  // Method test3:()V
        21: aload_1
        22: invokespecial #7                  // Method test4:()V
        25: return
*/
```

#### 5.多态原理

```java
public class TestPolym {

    public void polym(A a){
        a.say();
    }

    public static void main(String[] args) {
      TestPolym t=new TestPolym();
      t.polym(new B());
    }
}

abstract class A {
    public abstract void say();
}

class B extends A{

    @Override
    public void say() {
        System.out.println("B");
    }
}

class C extends A{

    @Override
    public void say() {
        System.out.println("C");
    }
}

/*
 stack=3, locals=2, args_size=1
         0: new           #3                  // class JVM03/TestPolym
         3: dup
         4: invokespecial #4                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: new           #5                  // class JVM03/B
        12: dup
        13: invokespecial #6                  // Method JVM03/B."<init>":()V
        16: invokevirtual #7                  // Method polym:(LJVM03/A;)V
        19: return
*/

	在执行多态的指令时，调用的是invokevirtual。此指令在多态环境下会做出如下操作：
    1.先通过栈帧中的对象引用找到对象
    2.分析对象头，找到对象的实际Class
    3.Class结构中有vtabke，在类加载的连接阶段就已经根据方法的重写规则生成了
    4.查表得到方法的具体地址
    5.执行方法的字节码
```

#### 6.异常处理

##### catch

```java
public static void main(String []args){
  	int i=0;
  	try{
      i=10;
    }catch(ArithmeticException e){
      i=20;
    }catch (Exception e){
      i=30;
    }
}
/*
stack=1, locals=3, args_size=1
         0: iconst_0
         1: istore_1
         2: bipush        10
         4: istore_1
         5: goto          19
         8: astore_2
         9: bipush        20
        11: istore_1
        12: goto          19
        15: astore_2
        16: bipush        30
        18: istore_1
        19: return

*/
```

##### finally

​	finally中的代码会复制到每个catch和try中，保证其执行。

```java
public static void main(String []args){
  	int i=0;
  	try{
      	i=10;
    }catch(Exception e){
      	i=20;
    }finally{
      	i=30;
    }
}

/*
stack=1, locals=4, args_size=1
         0: iconst_0
         1: istore_1
         2: bipush        10
         4: istore_1
         5: bipush        30				//finally 中的代码
         7: istore_1
         8: goto          27
        11: astore_2
        12: bipush        20
        14: istore_1
        15: bipush        30				//finally 中的代码
        17: istore_1
        18: goto          27
        21: astore_3
        22: bipush        30				//finally 中的代码
        24: istore_1
        25: aload_3
        26: athrow
        27: return
      Exception table:							//异常信息表
         from    to  target type		//from 和 to是检测范围，若匹配到type异常，则跳到target中的指令行
             2     5    11   Class java/lang/ArithmeticException
             2     5    21   any		//剩余的异常类型，比如Error
            11    15    21   any		//剩余的异常类型，比如Error
*/
```

##### 异常处理中的return

```java
public class TestTryCatchFinally {
    public int a(){
        int i=0;
        try{
            i=10;
            return i;	//try中的代码执行完会弹出栈顶的原因是固定返回值，执行完finally后再加载到栈顶并返回
        }finally{
            i=20;
        }
    }

    public static void main(String[] args) {
        TestTryCatchFinally t=new TestTryCatchFinally();
        int a = t.a();
        System.out.println(a);      //10
    }
}
/*
stack=1, locals=4, args_size=1
         0: iconst_0
         1: istore_1
         2: bipush        10
         4: istore_1
         5: iload_1
         6: istore_2							//将10放入slot 1，暂存至slot 1，为了固定返回值
         7: bipush        20
         9: istore_1							//给i赋值为20
        10: iload_2								//slot 1(10)载入slot 1暂存的值
        11: ireturn								//返回栈顶的int(10)
        12: astore_3
        13: bipush        20
        15: istore_1
        16: aload_3
        17: athrow

*/
```

finally 中的return会吞掉athrow指令，在开发中，**建议不在finally中return**

```java
//finally 无return
public int a(){
  int i=0;
  try{
    i=10/0;
    return i;
  }finally{
    i=20;
  }
}

public static void main(String[] args) {
  TestTryCatchFinally t=new TestTryCatchFinally();
  int a = t.a();
  System.out.println(a);      
}
//结果
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at JVM03.TestTryCatchFinally.a(TestTryCatchFinally.java:21)
	at JVM03.TestTryCatchFinally.main(TestTryCatchFinally.java:31)

//finally 有return
public int a(){
  int i=0;
  try{
    i=10/0;
    return i;
  }finally{
    i=20;
    return i;
  }
}

public static void main(String[] args) {
  TestTryCatchFinally t=new TestTryCatchFinally();
  int a = t.a();
  System.out.println(a);      
}
//结果
20
```

#### 7.synchronized

```java
public class TestSynchronized {
    public static void main(String[] args) {
        Object obj=new Object();
        synchronized (obj){
            System.out.println(obj);
        }
    }
}
/*
 stack=2, locals=4, args_size=1
         0: new           #2                  // class java/lang/Object
         3: dup
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: astore_1
         8: aload_1
         9: dup											//复制一份引用，两个引用分别用于加锁monitorenter和解锁monitorexit
        10: astore_2
        11: monitorenter						//monitorenter(lock)
        12: getstatic     #3        // Field java/lang/System.out:Ljava/io/PrintStream;
        15: aload_1
        16: invokevirtual #4        // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
        19: aload_2
        20: monitorexit							//monitorexit(lock)
        21: goto          29
        24: astore_3								//如果12-20期间出现了异常，则执行下面的解锁操作，类似于finally
        25: aload_2
        26: monitorexit
        27: aload_3
        28: athrow
        29: return
*/
```

### 2.类加载

#### 1.类加载阶段

​	类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading)7 个阶段。其中验证、准备、解析 3 个部分统称为连接（Linking）。

##### 1.加载

​	将类的字节码加载到方法区中，如果这个类还有父类未加载则先加载父类，加载和连接可能是交替进行的。

##### 2.验证

​	验证上阶段加载的类是否符合JVM规范

##### 3.准备

​	为**类变量分配内存**并**设置类变量初始值**，这些变量所使用的内存都将在**方法区**中进行分配。类的初始值如public static int value = 1; 在这里准备阶段过后的value值为0，而不是1。赋值为1的动作在初始化阶段。

##### 4.解析

​	将符号引用解析为直接引用

- 符号引用：以一组符号来描述所引用的目标，可以是任何形式的字面量，只要是能无歧义的定位到目标就好，就好比在班级中，老师可以用张三来代表你，也可以用你的学号来代表你，但无论任何方式这些都只是一个代号（符号），这个代号指向你（符号引用）
- 直接引用：直接引用是可以指向目标的指针、相对偏移量或者是一个能直接或间接定位到目标的句柄。和虚拟机实现的内存有关，不同的虚拟机直接引用一般不同。

##### 5.初始化

​	初始化即调用类的`<cinit>`。JVM会保证类的初始化过程是线程安全的。

```java
类的初始化时懒惰的。
//以下情形会初始化类
  main方法所在的类会被首先初始化
  首次访问这各类的静态方法或静态变量时
  子类初始化会导致未初始化的父类先初始化
  子类访问父类的静态变量，只会触发父类的初始化
  Class.forName
  new会导致初始化
//以下情形不会初始化类
  访问类的static final静态常量（基本类型和字符串）不会触发初始化
  类对象.class不会触发
  创建该类的数组不会初始化
  类加载器的loadClass方法
  Class.forName的参数2为false时
```

#### 2.类加载顺序

```java
//<cinit>： 父静子静（块和变量按照代码从上到下执行） 
//<init> ： 父变量（块和变量按照代码从上到下执行）父构造 子变量（块和变量按照代码从上到下执行）子构造
public class Father {
    private int i=test();
    private static int j=method();

    static {
        System.out.print("(1) ");
    }

    public Father(){
        System.out.print("(2) ");
    }

    {
        System.out.print("(3) ");
    }

    public static int method() {
        System.out.print("(4) ");
        return 0;
    }

    public int test() {
        System.out.print("(5) ");
        return 0;
    }
}

public class Son extends Father {
    private int i=test();
    private static int j=method();

    static {
        System.out.print("(6) ");
    }

    public Son(){
        System.out.print("(7) ");
    }

    {
        System.out.print("(8) ");
    }

    public static int method() {
        System.out.print("(9) ");
        return 0;
    }

    @Override
    public int test() {
        System.out.print("(10) ");
        return 0;
    }
  
  	public static void main(String[] args) {
        Son son=new Son();
        System.out.println();
        Son son1=new Son();
    }
}

/*
    父类的<cinit>  4 1
    子类的<cinit>  9 6
    父类的<init>   10 3 2
    子类的<init>   10 8 7

    没有（5）的原因：子类重写了父类的test()方法，父类的<init>中初始化的就是子类的test()方法
*/
```

#### 3.对象的生命周期

[参考文献](https://web.archive.org/web/20120626144027/http://java.sun.com/docs/books/performance/1st_edition/html/JPAppGC.fm.html)

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200803220448675.png" alt="image-20200803220448675.png"/>

### 3.类加载器

​	Java类加载器是Java运行时环境的一部分，负责动态加载Java类到Java虚拟机的内存空间中。在经过 Java 编译器编译之后就被转换成 Java 的.class 文件。类加载器负责读取 Java 字节代码，并转换成 `java.lang.Class`类的一个实例。

#### 1.类加载器种类

| 名称                      | 加载哪的类            | 说明                           |
| ------------------------- | --------------------- | ------------------------------ |
| 启动类(Bootstrqp)加载器   | JAVA_HOME/jre/lib     | 无法直接访问，由c++编写        |
| 扩展类(Extension)加载器   | JAVA_HOME/jre/lib/ext | 上级为启动类加载器，显示为null |
| 应用类(Application)加载器 | classpath             | 上级为扩展类加载器             |
| 自定义类加载器            | 自定义                | 上级为应用类加载器             |

#### 2.双亲委派机制

​	当一个类收了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载器。只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的 Class，子类加载器才会尝试自己去加载。

​	采用双亲委派的一个好处是比如加载位于 `rt.jar` 包中的类 `java.lang.Object`，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个 Object 对象。

```java
//类加载器加载类时的源码 
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{
  synchronized (getClassLoadingLock(name)) {
    //检查此类是否已被加载过
    Class<?> c = findLoadedClass(name);
    //如果没被加载
    if (c == null) {
      long t0 = System.nanoTime();
      try {
        //如果有上级类加载器
        if (parent != null) {
          //看上级类加载器是否加载
          c = parent.loadClass(name, false);
        } else {
          //如果没有，看启动类加载器是否加载
          c = findBootstrapClassOrNull(name);
        }
      } catch (ClassNotFoundException e) {
        //捕捉异常，但是不做处理
      }
      //若启动类加载器不加载此类
      if (c == null) {
        long t1 = System.nanoTime();
        //在classpath下找到此类进行加载
        c = findClass(name);
        sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
        sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
        sun.misc.PerfCounter.getFindClasses().increment();
      }
    }
    if (resolve) {
      resolveClass(c);
    }
    //加载完成后返回Class
    return c;
  }
}
```

#### 3.自定义类加载器

​	当加载的类的**包名、类名、类加载器都一致时才是同一个类**

##### 自定义的情况：

（1）想加载classpath随意路径下的类文件

（2）都是通过接口来使用实现，希望解耦时，常用于框架设计

（3）这些类希望隔离，不同应用的同名类逗了一家在，不冲突。常见于tomcat

##### 步骤：

（1）继承ClassLoader类

（2）遵从双亲委派机制，重写findClass方法

（3）读取类文件的字节码

（4）调用父类的defineClass方法来加载类

（5）使用者调用该类加载器的loadClass方法

```java
class MyClassLoader extends ClassLoader{
  @Override
  protected Class<?> findClass(String name) throws ClassNotFoundException{
    String path="e:\\myclasspath\\"+name+".class";
    try{
      ByteArrayOutputStream os=new ByteArrayOutputStream();
      Files.copy(Path.get(path),os);
      //得到字节数组
      byte[] bytes=os.toByteArray();
      return defineClass(name,bytes,0,byte.length);
    }catch(IOException e ){
      e.printStackTrace();
      throw new ClassNotFoundException("类文件未找到",e);
    }
  }
}
```

## 4.高效并发

### 1.Java内存模型

​	Java内存模型（Java Memory Model，JMM）是用来屏蔽掉各种硬件和操作系统的内存访问差异，已实现Java程序在各个平台下都能达到一致的内存访问效果。

​	JMM规定了所有的变量都存储在主内存（Main Memory）中。每条线程还有自己的工作内存（Working Memory）， 线程的工作内存中保存了被该线程使用的变量的主内存副本，**线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行， 而不能直接读写内存中的数据**。**不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成**。

#### 1.原子性、可见性和有序性

```java
	原子性：由JMM来直接保证的原子性变量操作包括read、load、assign、use、store、和write，大致可以认为基本数据类型的访问读写是具备原子性的
	可见性：当一个线程修改了共享变量的值，其他线程可以立即得知这个修改
	有序性：在本线程内观察，所有线程都是有序的，从一个线程观察另一个线程，所有线程都是无序的。（前半句指线程内表现为串行的语义，后半句是指 指令重排 现象和 工作内存与主内存同步延迟现象）
```

有序性的问题

```java
//double check 单例
public class Singleton{
  private static Singleton instance;
  private Singleton(){}
  public static Singleton getInstance(){
		if(instance==null){
      synchronized(Singleton.class){
				if(instance==null){
          instance=new Singleton();
        }
      }
    }
    return instance;
  }
}
//上面的单例模式在多线程下可能会产生指令重排
/*	
  17: new           #3                  // class JVM03/Singleton
  20: dup
  21: invokespecial #4                  // Method "<init>":()V
  24: putstatic     #2                  // Field instance:LJVM03/Singleton;
*/
//21 和 24 JVM的执行顺序不确定，引发并发安全问题
/*
假设t1 和 t2两个线程
	t1执行17，为Singleton对象生成了引用地址
	t1执行24，将引用地址赋值给instance，此时instance！=null
	
	t2执行到了第一个if发现instance！=null，将此时的instance返回。
	
	t1执行21，执行instance的构造方法
	
	
	如果这两个线程这样执行，则会让t2返回错误的对象，产生并发问题。
	
	可以加volatile关键字解决
*/
```

####2.先行发生原则(happen-before)

​	先行发生是Java内存模型中定义的两项操作之间的偏序关系，比如说操作A先行发生于操作B，其实就是说在发生操作B之前， 操作A产生的影响能被操作B观察到，”影响“包括修改了内存中共享变量的值、发送了消息、调用了方法等。

​	下面是Java内存模型下一些”天然的“先行发生关系，这些先行发生关系无须任何同步协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列， 并且无法从下列规则推导出来，则它们就没有顺序性保障，虚拟机可以对它们随意地进行重排序。

- 程序次序规则（Program Order Rule）：在一个线程内，按照控制流顺序，书写在前面的操作线性发生于书写在后面的操作。 注意，这里说的是控制流顺序而不是程序代码顺序，因为要考虑分支、循环等结构。
- 管程锁定规则（Monitor Lock Rule）：一个unlock操作先行发生于后面对同一个锁的lock操作。 这里必须强调的是”同一个锁“，而”后面“是指时间上的先后。
- volatile变量规则（Volatile Variable Rule）：对一个volatile变量的写操作先行发生于后面对这个变量的读操作， 这里的”后面“同样是指时间上的先后。
- 线程启动规则（Thread Start Rule）：Thread对象的start()方法先行发生于此线程的每一个动作。
- 线程终止规则（Thread Termination Rule）：线程中所有操作都先行发生于对此线程的终止检测，我们可以通过Thread::join()方法是否结束、 Thread::isAlive()的返回值等手段检测线程是否已经终止执行。
- 线程中断规则(Thread Interruption Rule)：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生， 可以通过Thread::interrupted()方法检测到是否有中断发生。
- 对象终结规则（Finalizer Rule）：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
- 传递性（Transitivity）：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

#### 3.volatile

##### 保证可见性：

​	当一个变量被定义成volatile之后，它将具备两项特性：第一项是保证此变量对所有线程的可见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的。 而普通变量并不能做到这一点，普通变量的值在线程间传递时均需要通过主内存来完成。比如，线程A修改了一个普通变量的值，然后向主内存进行回写，另外一条线程B在线程A回写完成了之后再对主内存进行读取操作，新变量值才会对线程B可见。

​	volatile修饰的变量不在和CPU高速缓存通信，而是直接与主存通信。通俗的说volatile会做两件事：**1.volatile修饰的值改变后会立即写回到主存  	2.使其他线程工作内存中的值无效，从主内存中读取共享变量的值。**

##### 保证有序性

​	使用volatile变量的第二个语义是**禁止指令重排序优化**，普通的变量仅会保证在该方法的执行过程中所有依赖赋值结果的地方都能够获取到正确的结果， 而不能保证变量赋值操作的顺序与程序代码中的执行顺序一致。

### 2.线程与锁

#### 1.线程状态

​	Java语言定义了6种线程状态，在任意一个时间点中，一个线程只能有且只有其中一种状态，并且可以通过特定的方法在不同状态之间转换。 这6种状态分别是：

- 新建（New）：创建后尚未启动的线程处于这种状态。
- 运行（Runnable）：包括操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行， 也有可能正在等待着操作系统为它分配执行时间。
- 无限期等待（Waiting）：处于这种状态的线程不会被分配处理器执行时间，它们要**等待被其他线程显式唤醒**。以下方法会让线程陷入无限期的等待状态：
  - 没有设置Timeout参数的Object的wait()方法；
  - 没有设置Timeout参数的Thread的join()方法；
  - LockSupport park()方法
- 限期等待（Timed Waiting）：处于这种状态的线程也不会被分配处理器执行时间，不过**无需等待被其他线程显式唤醒**， 在一定时间之后它们会由系统自动唤醒。一下方式会让线程进入限期等待状态：
  - Thread的sleep() 方法；
  - 设置了Timeout参数的Object的wait()方法；
  - 设置了Timeout参数的Thread的join()方法；
  - LockSupport的parkNanos()方法；
  - LockSupport的parkUntil()方法；
- 阻塞（Blocked）线程被阻塞了，"阻塞状态"与"等待状态"的区别是"阻塞状态"在等待着获取到一个排它锁，这个事件将在另外一个线程放弃这个锁的时候发生； 而"等待状态"则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
- 结束（Terminated）：已终止线程的线程状态，线程已经结束执行。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200826202422625.png" alt="image-20200826202422625" style="zoom:33%;" />



#### 2.乐观锁与悲观锁

- CAS是基于乐观的思想：最乐观的估计，不怕别的线程来修改共享变量，改了就重试
- synchronized基于悲观锁，防其他线程修改共享变量，上锁之后其他线程不可访问

##### CAS的实现

​	CAS（compare and swap），是一条并发原语。它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值。

​	CAS指令需要三个操作数,分别为内存位置(在Java中可简单理解为变量的内存地址V)，旧的预期值A和新值B，CAS指令执行时，当且仅当V符合旧预期值A时，持利器用新值B更新V的值，否则他就不执行更新，但是无论是否更新了V的值，都会返回V的旧值这个过程是原子的。

```java
class CAS{
    private int v;
    public void set(int a,int s){
        if (v==a){
            v=s;
            System.out.println(v);
        }else{
            System.out.println("未成功修改");
        }
    }
    public int get(){
        return v;
    }
}
```

##### CAS底层

​	CAS并发原语体现在Java语言就是`sun.misc.Unsafe`类的方法，调用Unsafe类的CAS方法，JVM会帮我们实现出CAS汇编指令。Unsafe类中都是native方法，且Unsafe类也不能直接获得，想使用CAS时可以借助原子类和原子引用类。

#### 3.synchronized优化

##### 1.轻量级锁

​	当无其他线程竞争时，synchronized会优化为轻量级锁，被锁的对象的Mark Word的标志位为00（轻量级锁状态，01为无锁状态）。

##### 2.锁膨胀

​	当CAS不成功时，轻量级锁会升级为重量级锁。竞争失败后，被锁的对象的Mark Word的标志位为10

##### 3.自旋锁

​	当对象是重量级锁状态，其他线程t2访问这个对象，t2不会直接阻塞，而是先自旋一段时间，若成功则获取资源，不成功则阻塞。防止阻塞和唤醒的操作时间长

##### 4.偏向锁

​	轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。Java6 中引入了偏向锁来做进一步优化，只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竟争，不用重新 CAS。

- 当出现竞争时，偏向锁会升级为轻量级锁，此时要撤销偏向锁，这个过程中所有线程需要暂停（STW）
- 访问对象的 hashcode 也会撤销偏向锁
- 如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T 的对象仍有机会重新偏向 T2, 重偏向会重置对象的 Thread ID
- 撤销偏向和重偏向都是批量进行的，以类为单位
- 如果撤销偏向到达某个阈值，整个类的所有对象都会变为不可偏向的
- 可以主动使用-XX:-UseBiasedLocking 禁用偏向锁

##### 5.锁粗化

​	如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。所以JVM会将同一个对象的多次加锁合并为一次加锁操作。

##### 6.锁消除

​	JVM会进行代码的逃逸分析，如某个加锁对象是方法内部的局部变量，不会被其他线程访问到，这时候就会被即时编译器忽略掉所有同步操作。

## 5.常用jvm参数汇总

### 1.XX参数

#### 1.boolean类型

​		-XX：+或者-  某个属性值

​		+表示开启-表示关闭

​		*例：*

​		-XX：+PrintGCDetails（是否打印GC收集细节）

​		-XX：-UseSerialGC（是否打印GC收集细节）

#### 2.KV设值类型

​		-XX：属性key=属性值value

​		*例：*

​		-XX:MetaspaceSize=128m（设置元空间大小为128m）

#### 3.jinfo

​		jinfo -flag 配置项 进程编号（查看当前运行程序的配置）

#### 4.-Xms和-Xmx

​		-Xms等价于 -XX:InitialHeapSize

​		-Xms等价于 -XX:MaxHeapSize

| 含义                                           | 参数                               |
| ---------------------------------------------- | ---------------------------------- |
| 堆初始大小                                     | -Xms(-XX:InitialHeapSize)          |
| 堆最大大小                                     | -Xmx(-XX:MaxHeapSize)              |
| 新生代大小                                     | -Xmn                               |
| 幸存区比例（如果为4，则Eden:From:To=4:1:1）    | -XX:SurvivorRatio=ratio            |
| 晋升阈值（必须为0-15，0则直接进入老年代）      | -XX:MaxTenuringThreshold=threshold |
| 打印晋升详情                                   | -XX:+PrintTenuringDistribution     |
| 开启Full GC前先Minor  GC                       | -XX:+ScavengeBeforeFullGC          |
| 配置老年代占比（如果为4，则老年代:新生代=4:1） | -XX:NewRatio=ratio                 |
| 输出详细GC收集日志信息                         | -XX:+PrintGCDetails                |

MINIOR GC:

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/younggc.png" alt="younggc" />

FULL GC:

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/fullgc.png" alt="fullgc"  />

#### 5.查看JVM默认值

1.java -XX:+PrintFlagsInitial（查看初始默认值）

2.java -XX:+PirntFlagsFinal（主要查看修改更新）

3.java -XX:+PrintCommandLineFlags

### 2.标配参数

​	-verison

​	-help

### 3.X参数

​	-Xint （解释执行）

​	-Xcomp（编译执行）

​	-Xmixed（混合执行）

