## 					JVM

![image-20191113191402628](/Users/yinwenbo/Documents/img/image-20191113191402628.png)

####一：jvm内存结构

##### 1.程序计数器（pc）

​	1.1作用：当前线程所执行的字节码的行号指示器

​	1.2注：  1>每个线程都有一条独立的pc

​					2>此区域时唯一一个在虚拟机规范中没有规定任何OutOfMemoryError的区域

##### 2.虚拟机栈

虚拟机栈也是线程私有的，每个方法在执行同时都会创建一个栈帧，每一个方法从调用到执行的过程就对应着一个栈帧在虚拟机栈中从出栈入栈的过程

![image-20191113192604242](/Users/yinwenbo/Documents/img/image-20191113192604242.png)

#####3..本地方法栈

<img src="/Users/yinwenbo/Documents/img/image-20191113192814607.png" alt="image-20191113192814607" />

​	虚拟机使用到的Native方法服务

##### 4.堆

​	4.1堆被所有线程共享，用于存放对象实例

​	4.2会抛出OOMError异常

##### 5.方法区

​	**元空间是方法区的实现**

![image-20191113193309783](/Users/yinwenbo/Documents/img/image-20191113193309783.png)

​	内存溢出案例：

![屏幕快照 2019-11-16 下午4.15.47](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午4.15.47.png)

​	spring mybatis动态加载类，使用不当就会导致OOM

##### 6.运行时常量池

![image-20191113193428534](/Users/yinwenbo/Documents/img/image-20191113193428534.png)

![image-20191113193537347](/Users/yinwenbo/Documents/img/image-20191113193537347.png)

##### 7.直接内存

##### 

#### 二：GC

​	（守护线程最典型的应用）

#####1.分代回收算法

​	1.1图示：

![屏幕快照 2019-11-14 09.18.02 PM](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-14 09.18.02 PM.png)

​	1.2VM参数

![屏幕快照 2019-11-14 09.19.40 PM](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-14 09.19.40 PM.png)

##### 2.垃圾回收器

###### 2.1CMS垃圾回收器

![屏幕快照 2019-11-15 12.54.24 PM](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-15 12.54.24 PM.png)

​		2.1.1缺点			

​			1>总吞吐量降低

​			2>无法处理浮动垃圾

​			3>产生内部碎片（标记清除算法）

​			4>当碎片太多时会并发失败，退化为串行SerialOld

![屏幕快照 2019-11-15 下午12.57.07](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-15 下午12.57.07.png)

![屏幕快照 2019-11-15 下午12.58.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-15 下午12.58.33.png)

​	

######2.2G1收集器

​	![屏幕快照 2019-11-15 下午2.19.00](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-15 下午2.19.00.png)

​		

​	2.2.1使用的算法

​		整体上使用标记整理算法，两个Region之间使用复制算法

​	2.2.2回收阶段

​		young collection

​		young collection+cm

​		mixed collection

#### 三：类加载与字节码技术

##### 1.类文件结构

![屏幕快照 2019-11-16 下午4.59.38](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午4.59.38.png)

```
access_flag：修饰信息（public   abstract）

attribute_count：附加的属性信息
```

##### 2.字节码指令

​	2.1运行流程：

![屏幕快照 2019-11-16 下午7.23.50](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.23.50.png)

​	b=a++ + ++a + a--的执行流程（a++和++a的区别就是先iload还是先iinc）

![屏幕快照 2019-11-16 下午7.31.26](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.31.26.png)

​	2.2 if判断指令的执行流程

![屏幕快照 2019-11-16 下午7.38.09](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.38.09.png)

###### 2.3 循环控制指令

​		2.3.1while

![屏幕快照 2019-11-16 下午7.39.31](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.39.31.png)

​		2.2.3 do-while

![屏幕快照 2019-11-16 下午7.44.25](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.44.25.png)

​		**先做的iinc在做的比较**

​		2.2.4 for

![屏幕快照 2019-11-16 下午7.49.31](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午7.49.31.png)

###### 	2.4构造方法

​		2.4.1 <cinit>()V:在类加载的初始化阶段被调用

![屏幕快照 2019-11-16 下午8.07.19](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午8.07.19.png)

​		2.4.2<init>()V:在实例化对象时被调用

![屏幕快照 2019-11-16 下午8.12.21](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午8.12.21.png)

![屏幕快照 2019-11-16 下午8.20.46](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午8.20.46.png)

###### 2.5方法调用

![屏幕快照 2019-11-16 下午8.25.07](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午8.25.07.png)

![屏幕快照 2019-11-16 下午8.25.17](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午8.25.17.png)

**对象.静态方法**会产生下面两条没必要的虚拟机指令(20,21)：

​		aload 1

​		pop

###### 2.6异常处理

​	2.6.1：一个catch

```java
public static void main(String []args){
  	int i=0;
  	try{
      	i=10;
    }catch(Exception e){
      	i=20;
    }
}
```

![屏幕快照 2019-11-18 下午6.58.37](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-18 下午6.58.37.png)

​	2.6.2：多个catch

![屏幕快照 2019-11-18 下午7.05.34](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-18 下午7.05.34.png)

​	2.6.3 finally

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
```

![屏幕快照 2019-11-18 下午7.16.00](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-18 下午7.16.00.png)
​	finally中的代码复制到try-catch和 catch中不包括的异常类型中，保证其执行。
​	finally 中的return会吞掉athrow指令！

###### 2.7finally面试题

```java
public static void main(String []args){
  	int i=0;
  	try{
      	i=10;
      	return i;
    }finally{
      	i=20;
    }
}
//此时输出返回值为10
```



![屏幕快照 2019-11-18 下午7.32.57](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-18 下午7.32.57.png)

**try中的代码执行完会弹出栈顶的原因是固定返回值，执行完finally后再加载到栈顶并返回**

###### 2.8synchronized

```java
public static void main(String []args){
  	Object obj=new Object();
  	synchronized(obj){
      	System.out.print("OK");
    }
}
```

![屏幕快照 2019-11-18 下午7.48.55](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-18 下午7.48.55.png)

##### 3.语法糖

#####4.类加载

![屏幕快照 2019-11-19 下午6.57.23](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 下午6.57.23.png)



###### 4.1加载

![屏幕快照 2019-11-19 上午8.47.44](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 上午8.47.44.png)

###### 4.3初始化

![屏幕快照 2019-11-19 上午9.10.04](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 上午9.10.04.png)

##### 5.类加载器

###### 5.1层级关系

![屏幕快照 2019-11-19 上午9.23.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 上午9.23.33.png)

###### 5.4双亲委派

```java
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 检查此类是否已被加载过
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
						 //若启动类加载器仍不加载此类
             if (c == null) {
                      
                      long t1 = System.nanoTime();
                      //在classpath下找到此类进行加载
                      c = findClass(name);

                      // this is the defining class loader; record the stats
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

###### 5.5自定义类加载器

​	**包名，类名，类加载器都一致的类才是同一个类**

![屏幕快照 2019-11-19 下午6.30.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 下午6.30.49.png)

<img src="/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 下午6.49.26.png" alt="屏幕快照 2019-11-19 下午6.49.26" style="zoom:50%;" />

##### 6.运行期优化

###### 6.1即时编译

​	6.1.1分层编译

![屏幕快照 2019-11-19 下午8.05.17](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 下午8.05.17.png style=)

​	6.1.2方法内联

<img src="/Users/yinwenbo/Documents/img/屏幕快照 2019-11-19 下午8.07.39.png" alt="屏幕快照 2019-11-19 下午8.07.39" style="zoom:50%;" />

​	6.2.3字段优化

###### 6.2反射优化

**control+f4查看实现类**

```java

    public Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException {
        if (++this.numInvocations > ReflectionFactory.inflationThreshold() &&	!ReflectUtil.isVMAnonymousClass(this.method.getDeclaringClass()))
        {
            MethodAccessorImpl var3 = (MethodAccessorImpl)(new 	MethodAccessorGenerator()).generateMethod(this.method.getDeclaringClass(), this.method.getName(), this.method.getParameterTypes(), this.method.getReturnType(), this.method.getExceptionTypes(), this.method.getModifiers());
            this.parent.setDelegate(var3);
        }

        return invoke0(this.method, var1, var2);
    }
```



​	当反射生成相同类超过阈值（默认15次）时，会动态生成MethodAccessorGenerator类优化生成速度

#### 四：Java内存模型

![屏幕快照 2019-11-20 下午4.05.40](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午4.05.40.png)

![屏幕快照 2019-11-20 下午4.05.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午4.05.49.png)

##### 1.原子性 可见性 有序性

```java
	原子性：由JMM来直接保证的原子性变量操作包括read、load、assign、use、store、和write，大致可以认为基本数据类型的访问读写是具备原子性的
	可见性：当一个线程修改了共享变量的值，其他线程可以立即得知这个修改
	有序性：在本线程内观察，所有线程都是有序的，从一个线程观察另一个线程，所有线程都是无序的。（前半句指'线程内表现为串行的语义'，后半句指'指令重排和主内存和工作内存同步延迟'）
```

#####2.先行发生原则(happen-before)

![屏幕快照 2019-11-20 下午5.04.03](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.04.03.png)

![屏幕快照 2019-11-20 下午5.05.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.05.49.png)

![屏幕快照 2019-11-20 下午5.05.58](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.05.58.png)

##### 3.线程状态

![屏幕快照 2019-11-20 下午5.14.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.14.35.png)

![屏幕快照 2019-11-20 下午5.14.17](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.14.17.png)

![屏幕快照 2019-11-20 下午5.13.25](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.13.25.png)

**图：**

![屏幕快照 2019-11-20 下午5.15.23](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.15.23.png)

##### 4.乐观锁与悲观锁

![屏幕快照 2019-11-20 下午5.31.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午5.31.33.png)

![屏幕快照 2019-11-20 下午6.14.11](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-20 下午6.14.11.png)

![屏幕快照 2019-11-23 下午5.12.31](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-23 下午5.12.31.png)

###### 4.1unSafe

![屏幕快照 2019-11-24 上午10.07.44](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-24 上午10.07.44.png)

![屏幕快照 2019-11-24 上午10.08.29](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-24 上午10.08.29.png)

##### 5.锁优化

###### 1.锁消除

对于被检测出不可能存在竞争的共享数据的锁进行消除

###### 2.锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

对同一个对象只加一次锁

###### 3.自旋锁

线程访问已经被锁定的资源时，先自旋一段时间，若成功则获取资源，不成功则阻塞

###### 4.偏向锁

线程偏向于让已经获得过的资源的线程得到锁

###### 5.轻量级锁

四个状态

- 无锁状态
- 偏向锁状态
- 轻量级锁状态
- 重量级锁状态

#### 附1：juc面试

##### 1.公平锁与非公平锁

![屏幕快照 2019-11-25 上午9.09.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-25 上午9.09.49.png)

###### 1.1二者区别

![屏幕快照 2019-11-25 上午9.12.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-25 上午9.12.49.png)

**非公平锁的优点在于吞吐量比公平锁大**

ReentrantLock 默认非公平锁 	synchronized也是非公平锁

##### 2.可重入锁（递归锁）

![屏幕快照 2019-11-25 上午9.22.36](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-25 上午9.22.36.png)

**最大的作用：避免死锁**

ReentrantLock和synchronized都是可重入锁



##### 3.自旋锁

![屏幕快照 2019-11-25 上午10.15.25](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-25 上午10.15.25.png)

###### 3.1手写自旋锁

```java
class SpinLock{
    private AtomicReference<Thread> ar=new AtomicReference<>();
    public void myLock(){
        System.out.println("come in lock");
        Thread thread=Thread.currentThread();
        while(!ar.compareAndSet(null,thread)){

        }
        System.out.println(thread.getName()+"获得锁");
    }

    public void myUnlock(){
        Thread thread=Thread.currentThread();
        ar.compareAndSet(thread,null);
        System.out.println(thread.getName()+"解锁");
    }
}
```

##### 4.读写锁

​	独占锁，共享锁，互斥锁

![屏幕快照 2019-11-25 上午10.41.56](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-25 上午10.41.56.png)

##### 5.线程池

###### 5.1线程池的优势，为什么使用

![屏幕快照 2019-11-27 下午4.00.23](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午4.00.23.png)

###### 5.2线程池的工作流程

![屏幕快照 2019-11-27 下午4.30.55](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午4.30.55.png)

![屏幕快照 2019-11-27 下午4.33.07](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午4.33.07.png)

```java
int c = ctl.get();
//如果线程数小于核心线程数
if (workerCountOf(c) < corePoolSize) {
  //运行线程
  if (addWorker(command, true))
    	return;
  c = ctl.get();
}
//大于核心线程数，加入队列
if (isRunning(c) && workQueue.offer(command)) {
  int recheck = ctl.get();
  if (! isRunning(recheck) && remove(command))
    //执行拒绝策略
    reject(command);
  else if (workerCountOf(recheck) == 0)
    addWorker(null, false);
}
else if (!addWorker(command, false))
  reject(command);
```

###### 5.3四大拒绝策略

![屏幕快照 2019-11-27 下午4.35.19](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午4.35.19.png)

###### 5.4使用何种方式创建线程池，为什么

![屏幕快照 2019-11-27 下午4.39.17](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午4.39.17.png)

###### 5.5线程池关闭

```java
我们知道，使用shutdownNow方法，可能会引起报错，使用shutdown方法可能会导致线程关闭不了。

所以当我们使用shutdownNow方法关闭线程池时，一定要对任务里进行异常捕获。

当我们使用shutdown方法关闭线程池时，一定要确保任务里不会有永久阻塞等待的逻辑，否则线程池就关闭不了

interrupt():如果有阻塞，则抛出异常，无阻塞，则标志一个中断位
isInterrupted():判断标志位，若标记过则为true
interrupted():判断标志位，若标记过则为true,并清除标记位，连续判断下次则为false
```

##### 6.手写死锁及其排查

###### 6.1手写死锁

```java
class DeadLock implements Runnable{

    String lock1;
    String lock2;

    public DeadLock(String lock1, String lock2) {
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    @Override
    public void run() {
        synchronized (lock1){
            System.out.println("已持有"+lock1+",想获取"+lock2);
            try {
                TimeUnit.SECONDS.sleep(2);}catch(InterruptedException e){ e.printStackTrace();}
            synchronized (lock2){
                System.out.println("已持有"+lock2+",想获取"+lock1);
            }
        }
    }
}

public class TestDeadLock {
    public static void main(String[] args) {
        String s1="lock1";
        String s2="lock2";
        DeadLock deadLock=new DeadLock(s1,s2);
        DeadLock deadLock2=new DeadLock(s2,s1);
        new Thread(deadLock).start();
        new Thread(deadLock2).start();
    }
}
```

###### 6.2排查

6.2.1：jps找到进程id

![屏幕快照 2019-11-27 下午5.09.13](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午5.09.13.png)

6.2.2：jstack排查问题

![屏幕快照 2019-11-27 下午5.09.22](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-27 下午5.09.22.png)

##### 7.线程安全的单例模式

###### double check

```java
//volatile 禁止指令重排
/*
      由于singletonTest = new SingletonTest()操作并不是一个原子性指令，会被分为多个指令：

        memory = allocate(); //1：分配对象的内存空间
        ctorInstance(memory); //2：初始化对象
        instance = memory; //3：设置instance指向刚分配的内存地址
        但是经过重排序后如下：

        memory = allocate(); //1：分配对象的内存空间
        instance = memory; //3：设置instance指向刚分配的内存地址，此时对象还没被初始化
        ctorInstance(memory); //2：初始化对象
      若有A线程进行完重排后的第二步，且未执行初始化对象。此时B线程此时B线程来取singletonTest时，
      发现singletonTest不为空，于是便返回该值，但由于没有初始化完该对象，此时返回的对象是有问题的。
        上述代码的改进方法：将singletonTest声明为volatile类型即可（volatile有内存屏障的功能）。
     */
public class SingleTon{
  	private static volatile SingleTon singleton;
  	private SingleTon(){}
  	public static SingleTon getInstance(){
      if(singleton==null){
        synchronized(SingleTon.class){
					if(singleton==null){
						singleton=new SingleTon();
          }
        }
      }
      return singleton;
    }
  
}
```

###### 饿汉式

```java
public class SingleTon{
  	private static  SingleTon INSTANCE=new SingleTon();
  	private SingleTon(){}
  	public static SingleTon getInstance(){
      return INSTANCE;
    }
  
}
```

##### 8.线程交替打印

###### 1.wait、notify

```JAVA
public class AlterPrint implements Runnable{
  private int i=1;
  public void run(){
    while(true){
      synchronized(this){
        notify();
        if(i<=100){
          System.out.println(Thread.currentThread().getName() + ":" + i);
          i++;
          try{
            wait();
          }catch(Exception e){
            e.printStackTrace();
          }
        }else break;
      }
    }
  }
}
```

###### 2.await、signal

```java
class TestNewComm{
    private int i=1;
    private Lock lock=new ReentrantLock();
    Condition c1=lock.newCondition();
    Condition c2=lock.newCondition();
    public void print1() {
        lock.lock();
        try {
            while ((i & 1) == 0) {
                try {
                    c1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("1:" + i);
            i++;
            c2.signal();
        }finally {
            lock.unlock();
        }
    }
    public void print2() {
        lock.lock();
        try {
            while ((i & 1) == 1) {
                try {
                    c2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("2:" + i);
            i++;
            c1.signal();
        }finally {
            lock.unlock();
        }
    }
}

		public static void main(String[] args) {
     
        TestNewComm t=new TestNewComm();
        new Thread(()->{
            for (int i = 0; i <5; i++)
                t.print1();
        },"1").start();
        new Thread(()->{
            for (int i = 0; i <5; i++)
                t.print2();
        },"2").start();
		}
```





#### 附2：jvm面试

##### 1.类加载器

###### 1.1什么是类加载器

Java类加载器是Java运行时环境的一部分，负责动态加载Java类到Java虚拟机的内存空间中。在经过 Java 编译器编译之后就被转换成 Java 的.class 文件。类加载器负责读取 Java 字节代码，并转换成 `java.lang.Class`类的一个实例。

###### 1.2类加载器种类

###### 1.3双亲委派

![屏幕快照 2019-11-30 上午11.18.57](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-30 上午11.18.57.png)

###### 1.4沙箱安全模型

![沙箱安全机制](/Users/yinwenbo/Documents/img/沙箱安全机制.jpeg)

##### 2.GC Root

######2.1哪些对象可以回收

在GC Root Set中以gc root为起点进行链路访问，如果对象可达，则不被回收，如果引用不可达，则可以回收

###### 2.2哪些对象可以作为GC Root

1.虚拟机栈（局部变量表）中引用的对象

2.方法区中的类静态属性引用的对象

3.方法区中常量引用的对象

4.本地方法栈中引用的对象

##### 3.jvm参数

###### 3.1.参数类型

3.1.1 标配参数

​	-verison

​	-help

3.2.2 X参数（了解）

​	-Xint （解释执行）

​	-Xcomp（编译执行）

​	-Xmixed（混合执行）

**3.2.3 XX参数**

​	1.boolean类型

​		-XX：+或者-  某个属性值

​		+表示开启-表示关闭

​		*例：*

​		-XX：+PrintGCDetails（是否打印GC收集细节）

​		-XX：-UseSerialGC（是否打印GC收集细节）

​	2.KV设值类型

​		-XX：属性key=属性值value

​		*例：*

​		-XX:MetaspaceSize=128m（设置元空间大小为128m）

​	3.jinfo

​		jinfo -flag 配置项 进程编号（查看当前运行程序的配置）

​	4.-Xms和-Xmx

​		-Xms等价于 -XX:InitialHeapSize

​		-Xms等价于 -XX:MaxHeapSize

###### 3.2查看JVM默认值

1.java -XX:+PrintFlagsInitial（查看初始默认值）

2.java -XX:+PirntFlagsFinal（主要查看修改更新）

3.java -XX:+PrintCommandLineFlags

##### 4.常用参数

###### 1.-Xms

初始大小内存，默认为物理内存1/64

等价于-XX:InitialHeapSize

######2.-Xmx

最大分配内存，默认为物理内存1/4

等价于-XX:MaxHeapSize

###### 3.-Xss

设置单个线程的大小，一般默认为512K~1024K

等价于-XX:ThreadStackSize

###### 4.-Xmn

设置年轻代大小

###### 5.-XX:MetaspaceSize

设置元空间大小

-Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX:+PrintFlagsFinal

###### 6.-XX:+PrintGCDetails

输出详细GC收集日志信息

MINIOR GC:

![屏幕快照 2019-12-01 上午11.47.13](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-01 上午11.47.13.png)

FULL GC:

![屏幕快照 2019-12-01 上午11.50.37](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-01 上午11.50.37.png)

###### 7.-XX:SurvivoRatio

![image-20191201121438272](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20191201121438272.png)

###### 8.-XX:NewRatio

![image-20191201121513048](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20191201121513048.png)

###### 9.-XX:MaxTenuringThreshold

设置垃圾最大年龄（必须为0-15）

![屏幕快照 2019-12-01 下午12.07.38](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-01 下午12.07.38.png)

##### 5.java.lang.ref

###### 1.强引用

直接new的对象，只要有引用，就不会被GC

###### 2.软引用

SoftReference，GC && 空间不足时就会被回收

###### 3.弱引用

WeakReference，只要GC就会被回收

```java
/*WeakHashMap的使用（用于缓存）
 当已经put的键值对的key被置为null时，
 下次gc会回收次键值对
*/
public static void main(String[] args) {

        WeakHashMap<Integer,String> map=new WeakHashMap<>();

        Integer i=new Integer(1);
        String name="weakhashmap";

        map.put(i,name);
        System.out.println(map);//{1=weakhashmap}

        i=null;
        System.out.println(map);//{1=weakhashmap}

        System.gc();


        System.out.println(map+"   "+i);//{}   null


    }
```

###### 4.软弱的应用场景

![屏幕快照 2019-12-03 上午8.54.56](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 上午8.54.56.png)

###### 5.虚引用

PhantomReference，随时会被回收，必须配合引用队列使用，被回收时会被添加到引用队列中，允许被回收的引用做一些后续的通知

![屏幕快照 2019-12-03 上午9.26.15](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 上午9.26.15.png)

```java
/*
     * java.lang.Object@60e53b93
     * null
     * null
     * =========================
     * null
     * null
     * java.lang.ref.PhantomReference@5e2de80c
     */
    public static void main(String[] args) {

        Object o=new Object();
        ReferenceQueue queue=new ReferenceQueue();
        PhantomReference pr=new PhantomReference(o,queue);
        System.out.println(o);
        System.out.println(pr.get());
        System.out.println(queue.poll());

        System.out.println("=========================");

        o=null;
        System.gc();
        System.out.println(o);
        System.out.println(pr.get());
        System.out.println(queue.poll());

    }
```

##### 6.OOM

###### 1.StackOverflowError

```java
public static void main(String[] args) {
        stackOverflowError();
    }

    private static void stackOverflowError() {
        stackOverflowError();
    }
```

###### 2.Java heap space

```java
public static void main(String[] args) {
        String s="1";
        while (true){
            s+="1"+ new Random().nextInt(10000000);
        }
    }
```

###### 3.GC overhead limit exceeded

超过98%的时间去GC但回收了不到2%的堆内存

![屏幕快照 2019-12-03 下午6.50.02](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 下午6.50.02.png)

```java
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
```

###### 4.Direct buffer Memory

NIO的allocateDirect()分配过大导致的直接内存溢出

![屏幕快照 2019-12-03 下午7.04.47](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 下午7.04.47.png)

```java
		public static void main(String[] args) {
				//-Xms10m -Xmx10m -XX:MetaspaceSize=5m
        ByteBuffer bf=ByteBuffer.allocateDirect(6*1024*1024);

    }
```

###### 5.unable to create new native thread

![屏幕快照 2019-12-03 下午7.19.38](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 下午7.19.38.png)

```java
public static void main(String[] args) {
        int i=0;
        while (true){
            i++;
            new Thread(()->{
                try {
                    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);}catch(InterruptedException e){ e.printStackTrace();}
            }).start();
            System.out.println(i);
        }
    }
```

###### 6.Metaspace

![屏幕快照 2019-12-03 下午7.46.24](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-03 下午7.46.24.png)

![屏幕快照 2019-11-16 下午4.15.47](/Users/yinwenbo/Documents/img/屏幕快照 2019-11-16 下午4.15.47.png)

##### 7.GC算法与收集器的关系

​      ***GC垃圾回收算法和垃圾收集器的关系?分别是什么请你谈谈***

```java
关系：GC算法（标清,标整,复制,分代回收）是内存回收的理论，垃圾收集器就是算法落地实现

四种垃圾收集器：
1.串行垃圾回收器（Serial）：它为单线程环境设计并且只使用一个线程进行垃圾回收，会暂停所有的用户线程。（一个服务员清理多个桌子）
2.并行垃圾回收器（Parallel）：多个垃圾回收线程并行工作，此时用户线程是暂停的。（多个服务员清理多个桌子）
3.并发垃圾回收器（CMS）：用户线程和垃圾收集线程同时执行（不一定是并行，可能交替执行），不需要停顿用户线程互联网公司多用它，适用于对响应时间有要求的场景
4.G1垃圾回收器：G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收
```

#####8.垃圾收集器的配置和理解

​		***怎么查看服务器默认的垃圾收集器是那个?生产上如何配置垃圾收集器的?谈谈你对垃圾收集器的理解?***

######1.查看默认的垃圾收集器：

![屏幕快照 2019-12-04 下午3.38.15](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午3.38.15.png)

######2.生产上如何配置垃圾收集器:

![屏幕快照 2019-12-04 下午3.48.05](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午3.48.05.png)

######3.对垃圾收集器的理解

串行：

![屏幕快照 2019-12-04 下午3.54.31](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午3.54.31.png)

CMS:

![屏幕快照 2019-12-04 下午4.15.27](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午4.15.27.png)

1.初始标记：只是标记一下GC roots能直接关联的对象，**会STW**，速度很快

2.并发标记：进行GC ROOTS跟踪的过程，和用户线程一起工作，**不会STW**。主要标记过程，标记全部对象

3.重新标记：（二次确认过程）修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，**会STW**。

4.并发清除：清除GC ROOT不可达对象，和用户线程一起工作，**不会STW**。

**优缺点：**

优点：并发收集低停顿

缺点：

​		1>并发执行，对CPU资源压力大。(CMS在收集与应用程序会同时增加对堆内存的占用，也就是说，CMS必须在老年代堆内存用尽之前完成立即回收，否则CMS回收失败时，将会触发担保机制，退化为串行收集器，进行较长时间的STW)

​		2>标记清除产生碎片

######4.各个垃圾收集器算法

![屏幕快照 2019-12-04 下午4.49.52](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午4.49.52.png)

##### 9.G1

###### 1.G1是什么

![屏幕快照 2019-12-04 下午5.02.00](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.02.00.png)

###### 2.底层原理

1.Region区域化垃圾收集器

​	最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可

2.回收过程

![屏幕快照 2019-12-04 下午5.56.37](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.56.37.png)

3.回收步骤

![屏幕快照 2019-12-04 下午5.56.12](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.56.12.png)

![屏幕快照 2019-12-04 下午5.55.21](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.55.21.png)

![屏幕快照 2019-12-04 下午5.55.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.55.33.png)

######3.较CMS的优势

![屏幕快照 2019-12-04 下午5.59.01](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-04 下午5.59.01.png)

##### 10.java对象的生命周期

![image-20200803220448675](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200803220448675.png)

[文献](https://web.archive.org/web/20120626144027/http://java.sun.com/docs/books/performance/1st_edition/html/JPAppGC.fm.html)：https://web.archive.org/web/20120626144027/http://java.sun.com/docs/books/performance/1st_edition/html/JPAppGC.fm.html