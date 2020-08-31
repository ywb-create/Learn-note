# Java并发编程

## 多线程基础

### 1.线程创建

#### 1.继承Thread类

```java
class MyThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i <100 ; i++) {
            if(i % 2 ==0){
                System.out.println(Thread.currentThread().getName()+"  "+i);
            }
        }
    }
}

public class TestThreadCreate {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();//1.启动当前线程 2.调用此线程的run()方法
        thread.run();//执行的是main线程
//        thread.start();//不能执行两次start(),否则会抛出IllegalThreadStateException
        for (int i = 0; i <100 ; i++) {
            if(i % 2 ==0){
                System.out.println(Thread.currentThread().getName()+"  "+i+"*******");
            }
        }
    }
}
```

#### 2.实现Runnable接口

```java
/**
 * 实现Runnable接口（推荐使用）
 *  好处：1.可以避免类单继承的局限性
 *       2.更适合处理多个线程共享数据的情况
 *          注：一个对象传入多个线程类中
 *  底层：
 *      class Thread implements Runnable{}
 *      Thread 类也是实现的Runnable接口，重写了runnable的run()
 *
 */
class TestThread02 implements Runnable{

    private int target=100;
    @Override
    public void run() {
        while (true) {
            if (target>0) {
                System.out.println(Thread.currentThread().getName() + ":" + target);
                target--;
            }else {
                break;
            }

        }
    }
}

public class TestRunnable {
    public static void main(String[] args) {
        TestThread02 thread02=new TestThread02();
        Thread thread1=new Thread(thread02);
        Thread thread2=new Thread(thread02);
        Thread thread3=new Thread(thread02);
        thread1.start();
        thread2.start();
        thread3.start();

    }
}
```

#### 3.实现Callable接口

```java
//如何理解callable比runnable强大

/**
 * 1.callable可以有返回值
 * 2.callable可以抛出异常
 * 3.callable支持泛型
 */

//1.创建一个类实现Callable接口
class TestThread04 implements Callable<Integer>{

    //2.实现call()
    @Override
    public Integer call() throws Exception {
        int sum=0;
        for (int i = 1; i <= 100; i++) {
            if (i%2==0){
                System.out.println(Thread.currentThread().getName()+":"+i);
                sum+=i;
            }
        }

        return sum;
    }
}


public class TestCreateThreadCallable {
    public static void main(String[] args) {
        //3.创建实现接口的类
        TestThread04 thread=new TestThread04();
        //4.创建futureTask实例，将实现类传入
        FutureTask<Integer> futureTask=new FutureTask(thread);
        //5.将future实例传入线程中，调用start()
        new Thread(futureTask).start();

        try {
            //6.调用FutureTask的get()接受返回值
            Integer num=futureTask.get();
            System.out.println(num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

#### 4.线程池创建

```java
class TestThread05 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 != 0){
                System.out.println(Thread.currentThread().getName()+":"+i);
            }
        }
    }
}
//使用线程池创建线程（开发中常用）

/**
 * 使用线程池的好处：
 *  1.提高响应速度
 *  2.降低资源消耗
 *  3.便于线程管理
 */
public class TestCreateThreadThreadPool {
    public static void main(String[] args) {
        //1.提供指定数量的线程池
        ExecutorService service = Executors.newFixedThreadPool(5);//线程池工具类Executors，用于创建线程池

        //1.1 设置线程池属性。(ThreadPoolExecutor)是ExecutorService的实现类，
        ThreadPoolExecutor service1= (ThreadPoolExecutor) service;
        service1.setCorePoolSize(10);//设置核心池的大小
        //service1.setKeepAliveTime(1);//线程没有任务时最多保持多长时间会终止
        service1.setMaximumPoolSize(10);//设置最大线程数

        //2.执行指定的线程操作
        service.execute(new TestThread05());//用于执行实现runnable接口的实现类的方法

        // 适用于callable 有返回值
        //service.submit();

        //3.关闭线程池
        service.shutdown();//关闭线程池
    }
}
```

### 2.常用方法

#### 1.设置守护线程

```java
/*
注意：
(1) thread.setDaemon(true)必须在thread.start()之前设置，
    否则会抛出一个IllegalThreadStateException异常。
    不能把正在运行的常规线程设置为守护线程。
(2) 在Daemon线程中产生的新线程也是Daemon的。
(3) 不要认为所有的应用都可以分配给Daemon来进行服务，比如读写操作或者计算逻辑。
*/
class MyDaemon implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+"第"+i+"次运行");
            try { TimeUnit.SECONDS.sleep(1);}catch(InterruptedException e){ e.printStackTrace();}
        }
    }
}

public class TestDaemonThread {
    public static void main(String[] args) {

        MyDaemon myDaemon=new MyDaemon();
        Thread thread=new Thread(myDaemon,"b");
        thread.setDaemon(true);//将线程设置为守护线程
        thread.start();

        //结果：b线程走了5次，a线程执行完就退出了程序
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName()+"第"+i+"次运行");
                try {TimeUnit.MILLISECONDS.sleep(500);}catch(InterruptedException e){ e.printStackTrace();}
            }
        },"a").start();
    }
}
```

#### 2.中断方法

- interrupt()：如果线程中有阻塞，则抛出异常，无阻塞，则标志一个中断位
- isInterrupted()：判断标志位，若标记过则为true
- interrupted()：判断标志位，若标记过则为true,并清除标记位，连续判断下次则为false

```java
public class TestInterrupt extends Thread {

    volatile boolean stop=false;

    public static void main(String[] args) {
        TestInterrupt t=new TestInterrupt();
        t.start();
        try {TimeUnit.SECONDS.sleep(2);}catch(InterruptedException e){ e.printStackTrace();}
        //通知t线程退出循环，但是t线程sleep 100秒阻塞，无法退出
        t.stop=true;
        //打断t线程，使得t线程抛出InterruptedException退出循环
        t.interrupt();

        try {TimeUnit.SECONDS.sleep(1);}catch(InterruptedException e){ e.printStackTrace();}
        System.out.println("stop");
    }

    @Override
    public void run() {
        while(!stop){
            System.out.println("run");
            try {TimeUnit.SECONDS.sleep(100);}catch(InterruptedException e){ e.printStackTrace();}
        }
        System.out.println("exit");
    }
}

/*
结果：
run
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at java11.TestInterrupt.run(TestInterrupt.java:27)
exit
stop
*/
```

#### 3.其他方法

```java
/*测试
     Thread currentThread() 返回当前线程
     join()  final方法，在线程a中调用线程b的join方法，会使线程a阻塞，直到线程b执行完线程a才会结束阻塞状态
     sleep() 本地静态方法，线程休眠阻塞，不释放锁
     yield() 本地静态方法，释放cpu执行权，不会释放锁
     boolean isAlive()
     set/getPriority()
        1>子类线程会继承父类线程的优先级
        2>优先级要在start()之前设置
*/
class TestThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName()+":"+i);
            if(i%20 ==0){
                //public static native void yield();
                yield();//释放当前cup执行权(让优先级高的先执行)，如果是同优先级，则竞争
            }

            if(i!=0 && i%80 ==0){
                try {
                    //public static native void sleep(long millis) throws InterruptedException;
                    sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

public class TestThreadMethod {
    public static void main(String[] args) {
        TestThread  = new TestThread();
        //优先级高代表执行的概率高，不代表一定先执行
        t1.setPriority(Thread.MAX_PRIORITY);//设置优先级要在start()之前
        t1.start();

        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);
            if (i!=0&&i%50 ==0){
                try {
                    //public final void join() throws InterruptedException {
                    t1.join();//在线程a中调用线程b的join方法，会使a阻塞，直到b执行完a才结束阻塞
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

### 3.线程安全

#### 1.同步代码块

```java
synchronized (同步监视器){
  //操作共享数据的代码
}

共享数据：多个线程共同操作的代码
同步监视器：任何一个对象都可充当
线程同步时多个对象要使用同一把锁

'共享数据为继承Thread类时，锁可以使用 类.class'
'共享数据为实现Runnabel接口时,锁可以使用this'
```

##### Thread

```java
class TestThread extends Thread{

    private static int target=100;//将票数静态，使得三个对象共享target
    static Object obj=new Object();//静态化锁，使三个线程只有同一把锁
    @Override
    public void run() {
        while (true) {
            synchronized(obj){//可以简化为TestThread.class

                if (target > 0) {
                    System.out.println(Thread.currentThread().getName() + ":" + target);
                    target--;
                } else {
                    break;
                }
            }
        }
    }
}

public class TestSynchronizedBlockExtends {
    public static void main(String[] args) {
        Thread t1=new TestThread();
        Thread t2=new TestThread();
        Thread t3=new TestThread();

        t1.start();
        t2.start();
        t3.start();
    }
}
```

##### Runnable

```java
//同步代码块解决线程安全问题
class TestThread02 implements Runnable{
    private int target=100;
    Object obj=new Object();
    @Override
    public void run() {
        while (true) {
            synchronized(obj){//为简化，可以把obj换成this，但是只是在对象唯一的前提下可以换

                if (target > 0) {
                    System.out.println(Thread.currentThread().getName() + ":" + target);
                    target--;
                } else {
                    break;
                }
            }
        }
    }
}

public class TestSynchronizedBlockRunnable {
    public static void main(String[] args) {
        TestThread02 thread02=new TestThread02();
        Thread thread1=new Thread(thread02);
        Thread thread2=new Thread(thread02);
        Thread thread3=new Thread(thread02);
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

#### 2.同步方法

​	如果操作共享数据的代码完整的声明在一个方法中，此时可以将方法声明成同步方法

##### Thread

```java
class TestThread03 extends Thread{

    private static int target=100;//将票数静态，使得三个对象共享target
    @Override
    public void run() {
        while (true) {
            show();
        }

    }

    //方法静态，保证同步监视器唯一
    public static synchronized void show(){//此时的锁是TestThread03.class
        if (target > 0) {
            System.out.println(Thread.currentThread().getName() + ":" + target);
            target--;
        }
    }
}

public class TestSynchronizedMethodExtends {
    public static void main(String[] args) {
        Thread t1=new TestThread03();
        Thread t2=new TestThread03();
        Thread t3=new TestThread03();

        t1.start();
        t2.start();
        t3.start();
    }
}
```

##### Runnable

```java
//同步方法
class TestThread01 implements Runnable{

    private int target=100;
    @Override
    public void run() {
            while (true) {
               show();
            }
    }
    private synchronized void show(){//同步监视器(锁)：this
        if (target > 0) {
            System.out.println(Thread.currentThread().getName() + ":" + target);
            target--;
        }
    }
}
public class TestSynchronizedMethodRunnable {
    public static void main(String[] args) {
        TestThread01 thread=new TestThread01();
        Thread thread1=new Thread(thread);
        Thread thread2=new Thread(thread);
        Thread thread3=new Thread(thread);
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

#### 3.ReetrantLock

```java
class Window implements Runnable{

    private int target=100;
    //1.创建ReentrantLock对象
    private ReentrantLock lock=new ReentrantLock();

    @Override
    public void run() {

        lock.lock();//2.锁
        try {
            while(true) {
                if (target > 0) {
                    System.out.println(Thread.currentThread().getName() + ":" + target);
                    target--;
                } else {
                    break;
                }
            }

        }finally {
            lock.unlock();//3.finally中解锁
        }

    }
}

public class TestReentrantLock {

    public static void main(String[] args) {
        Window window=new Window();
        Thread t1=new Thread(window);
        Thread t2=new Thread(window);
        Thread t3=new Thread(window);

        t2.start();
        t1.start();
        t3.start();
    }
}

```

##### ReetrantLock和synchronized的异同

​	相同点：二者都为解决线程安全问题

​	不同点：

- lock是显示锁，需要手动开启关闭，synchronized是隐式锁。
- lock能使jvm用较少的时间调度线程，性能更好，并且拥有更好的扩展性。

### 4.死锁

​	死锁是多个线程因资源竞争而产生的一种僵局，若无外力帮助，则永远无法向前推进

```java
public class DeadLock {
    public static void main(String[] args) {
        StringBuilder s1=new StringBuilder();
        StringBuilder s2=new StringBuilder();

        new Thread(() -> {
            synchronized (s1){
                s1.append(1);
                s2.append(2);

                try {
                    Thread.sleep(100);//增大产生死锁概率
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (s2){
                    s1.append(3);
                    s2.append(4);
                    System.out.println(s1);
                    System.out.println(s2);
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (s2){
                    s1.append(5);
                    s2.append(6);

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (s1){
                        s1.append(7);
                        s2.append(8);
                        System.out.println(s1);
                        System.out.println(s2);

                    }
                }
            }
        }).start();

    }
}
```

##### 排查死锁

```shell
[yinwenbo]~$ jps
6802 DeadLock							#找到死锁的pid
6803 Launcher
6804 Jps
843 
[yinwenbo]~$ jstack 6802	#jstack pid进行分析
"Thread-1":
        at interview.deadlock.DeadLock.run(TestDeadLock.java:24)
        - waiting to lock <0x0000000795789090> (a java.lang.String)
        - locked <0x00000007957890c8> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:745)
"Thread-0":
        at interview.deadlock.DeadLock.run(TestDeadLock.java:24)
        - waiting to lock <0x00000007957890c8> (a java.lang.String)
        - locked <0x0000000795789090> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.					#找到死锁
```

### 5.线程通信

```java
//wiat和notify是Object类的方法，用于阻塞和唤醒线程
/*
		 Object wait()：使当前线程阻塞，且释放锁（sleep也会使线程阻塞，但是不会释放锁）
     Object notify()/notifyAll()：唤醒一个/全部线程，优先级高的线程会先被唤醒
*/
public class TestNotify {

    private static Object lock=new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread a=new Thread(()->{
            synchronized (lock){
                try {
                    System.out.println("1 wait");
                    lock.wait();
                    System.out.println("1 exit");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"1");

        Thread b=new Thread(()->{
            synchronized (lock){
                try {
                    System.out.println("2 wait");
                    lock.wait();
                    System.out.println("2 exit");
                } catch (InterruptedException e) {
                }
            }
        },"2");
        a.start();
        b.start();

        try { TimeUnit.SECONDS.sleep(2);}catch(InterruptedException e){ e.printStackTrace();}

        Thread c=new Thread(()->{
            synchronized (lock){
                System.out.println("3 notifyAll");
                //唤醒阻塞线程中的随机一个(可在上面阻塞的线程wait()下加入notify()解决不能唤醒问题)
                lock.notify();/*1 wait  2 wait 3 notify 1 exit 程序不能退出，2不能被唤醒*/
                //唤醒所有阻塞线程
                lock.notifyAll();/*1 wait  2 wait 3 notifyAll 1 exit 2 exit main exit程序退出*/
            }
        },"3");
        c.start();
        //使主线程等待abc三个线程执行完
        a.join();
        b.join();
        c.join();
        System.out.println("main exit");
    }
}
```

#### 两个线程交替打印1-100

```java
//测试线程通信：两个线程交替打印1-100,两种方式


方式1：
//wait(),notify(),notifyAll()必须在同步代码块中使用
class TestComm implements Runnable {

    private int i = 1;

    //    private Object object=new Object();
    @Override
    public void run() {


        while (true) {
            synchronized (this) {//同步监视器的对象必须要和调用wait和notify的对象相同,
                // 否则会报IllegalMonitorStateException
                notify();
                if (i <= 100) {
                    System.out.println(Thread.currentThread().getName() + ":" + i);
                    i++;

                    try {
                        wait();//使线程阻塞并释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }else {
                    break;
                }
            }
        }
    }
}

方式二：
//使用await()  signal() signalAll()
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

public class TestThreadCommunication {
    public static void main(String[] args) {
        TestComm t = new TestComm();
        new Thread(t).start();
        new Thread(t).start();
//        TestNewComm t=new TestNewComm();
//        new Thread(()->{
//            for (int i = 0; i <5; i++)
//                t.print1();
//        },"1").start();
//        new Thread(()->{
//            for (int i = 0; i <5; i++)
//                t.print2();
//        },"2").start();

    }
}
```

#### 生产者消费者

```java
//产品池
class ProductPool{

    private int product=0;
    //生产产品
    public synchronized void product() {

        if(product<20){//如果产品少于20，生产

            product++;
            System.out.println(Thread.currentThread().getName()+"生产第"+product+"个产品");

            this.notify();//生产完通知消费者消费
        }else {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    //消费产品
    public synchronized void customer() {
        if(product>0){//如果产品大于0消费
            System.out.println(Thread.currentThread().getName()+"消费第"+product+"个产品");
            product--;
            notify();//消费完通知生产者生产
        }else {
            try {
                this.wait();//如果无产品，消费者阻塞，释放锁等待生产者生产
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

//生产者
class Producer extends Thread{
    private ProductPool pool;
    public Producer(ProductPool productPool){//传入产品库是为了保证产品池对象唯一
        this.pool=productPool;
    }

    @Override
    public void run() {
        while (true){


            pool.product();
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

//消费者
class Customer extends Thread{
    private ProductPool pool;
    public Customer(ProductPool productPool){
        this.pool=productPool;
    }

    @Override
    public void run() {
        while (true){
            pool.customer();
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class ProducerAndCustomer {
    public static void main(String[] args) {
        ProductPool pool=new ProductPool();
        Producer p=new Producer(pool);
        p.setName("生产者");
        p.start();
        Customer customer=new Customer(pool);
        customer.setName("消费者");
        customer.start();
        Customer c=new Customer(pool);
        c.setName("消费者2");
        c.start();
    }

}
```

## 多线程高级

### 1.volatile

#### 1.可见性

##### 问题：

​	可见性指一个线程内的共享变量发生改变，另一个线程可立马得知这个改变。由于JMM规定，共享变量必须读到线程的工作内存内才可以写入，且另一个线程无法的访问其他线程工作内存中的变量，所以可能会出现可见性的问题。问题代码如下

```java
public class TestVisibility {

    public static void main(String[] args) throws InterruptedException {
        Thread01 t=new Thread01();
        t.start();
        while (true){
            if (t.getA()==0){	//1s后无法读取t线程工作内存中的值，无法退出循环
                break;
            }
        }
        System.out.println("exit");
    }
}

class Thread01 extends Thread{
    private  int a=1;					//内存中初始为1

    @Override
    public void run() {
      	//等待1秒，便于main线程读取a=1
        try { TimeUnit.SECONDS.sleep(1);}catch(InterruptedException e){ e.printStackTrace();}
      	//刷新本线程工作内存
        a=0;
    }

    public int getA() {
        return a;
    }
}
```

##### 解决：

```java
//共享变量上加入volatile关键字
private volatile int a=1;

//volatile的作用
1.volatile修饰的值改变后会立即写回到主存  	2.使其他线程工作内存中的值无效，从主内存中读取共享变量的值。
```

#### 2.有序性

​	JVM为提高效率，可能会更改程序的执行顺序，如果JVM认为程序的执行顺序不会影响最终结果，则会发生指令重排。

##### 问题：

```java
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
     */
public class SingleTon{
  	private static  SingleTon singleton;
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

##### 解决：

```java
//将singletonTest声明为volatile类型即可（volatile有内存屏障的功能）。
private static volatile SingleTon singleton;

volatile可以禁止指令重排
```

#### 3.原子性

​	原子是一个不可再分的操作，在多线程操作时，可能会将单线程不会分割的原子操作进行分割，导致数据出错

##### 问题:

```java
//volatile不具有原子性
class MyThread02 implements Runnable{
    private volatile int a=0;

    public int getA() {
        return a;
    }

    public void setA(int a) {
        this.a = a;
    }


    @Override
    public void run() {

        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        a++;
        System.out.println(getA());
    }
}

public class TestAtomic {

    @Test
    public  void testVolatile() throws InterruptedException {
        MyThread02 thread=new MyThread02();
        for (int i = 0; i < 10; i++) {
            new Thread(thread).start();
        }
    }
}

//运行结果：764
```

##### 解决：

```java
//使用juc.atomic 底层用volatile+cas实现原子性
class MyThread03 implements Runnable{

    AtomicInteger a=new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(a.getAndIncrement());
        }
    }
}

public class TestAtomic {

    @Test
    public void testAtomic() {

        MyThread03 thread=new MyThread03();
        for (int i = 0; i < 10; i++) {
            new Thread(thread).start();

        }
    }
}

//运行结果：999
```

#### 4.volatile和syn的区别

- volatile不能保证原子性，synchronized可以
- volatile只是变量修饰符，synchronized作用于方法、代码块
- volatile不会使线程阻塞，synchronized会阻塞线程
- volatile只是在线程的工作内存和主内存间同步某个变量的值，而synchronized通过加锁解锁对资源进行同步，synchronized比volatile更耗资源

### 2.CAS

​	CAS（compare and swap），是一条并发原语。它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值。

​	CAS指令需要三个操作数,分别为内存位置(在Java中可简单理解为变量的内存地址V)，旧的预期值A和新值B，CAS指令执行时，当且仅当V符合旧预期值A时，持利器用新值B更新V的值，否则重新计算和获取V值（自旋），此阶段不会释放CPU执行权，效率较悲观锁高。

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

​	CAS并发原语体现在Java语言就是`sun.misc.Unsafe`类的方法，调用Unsafe类的CAS方法，JVM会帮我们实现出CAS汇编指令。Unsafe类中都是native方法，且Unsafe类也不能直接获得，想使用CAS时可以借助原子类和原子引用类。

​	CAS虽然可以解决变量的原子性问题，但是也有三个明显的缺点

- 循环开销大。方法执行时，如果CAS失败，会一直进行尝试。如果CAS长时间一直不成功，可能会给CPU带来很大的开销。
- ABA问题

- 只能保证一个共享变量的原子操作

#### 解决单个共享变量问题

​	CAS只能保证一个共享变量的原子操作，如想要同时对int 和 String的变量进行操作时，无法保证两个变量的原子性，此时可以对变量**加锁**或者使用JUC包下的**原子引用类**AtomicReference来解决此问题

```java
class User{
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
/*
    原子引用：
        将bean包装成原子类
 */
public class TestAtomicReference {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch=new CountDownLatch(2);
        User zs=new User("zs",1);
        User ls=new User("ls",2);

        AtomicReference<User> ar=new AtomicReference<>();
        ar.set(zs);

        new Thread(()->{
            while(true){
                //如果当前值是ls，换为zs
                if(ar.compareAndSet(ls,zs)) {
                    System.out.println(ar.get().toString() + Thread.currentThread().getName());
                    //计数器减1
                    latch.countDown();
                    break;
                }else {//否则线程休眠3秒，让其他线程执行
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();


        new Thread(()->{
            //如果当前值是zs，换为ls
            ar.compareAndSet(zs,ls);
            System.out.println(ar.get().toString()+ Thread.currentThread().getName());
            latch.countDown();
        }).start();

        latch.await();

        System.out.println("交换结果："+ar.compareAndSet(zs,ls)+"    "+ar.get().toString());
        System.out.println("交换结果："+ar.compareAndSet(zs,ls)+"    "+ar.get().toString());

    }
}

//运行结果
/*
User{name='ls', age=2}Thread-1
User{name='zs', age=1}Thread-0
交换结果：true    User{name='ls', age=2}
交换结果：false    User{name='ls', age=2}
*/
```

#### 解决ABA问题

​	CAS会产生ABA问题，即A线程正在进行CAS期间，B线程更改了V但又改回了原值，此时A获得CPU后CAS 仍可成功。可已添加一个时间戳或者版本戳来解决此问题。

```java
//解决ABA问题（AtomicStampedReference）
public class TestSolveABA {

    static AtomicReference<Integer> ar=new AtomicReference<>();
    // public AtomicStampedReference(V 初始值, int 初始版本号)
    static AtomicStampedReference<Integer> asr=new AtomicStampedReference<>(100,1);

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch=new CountDownLatch(2);
        ar.set(100);
        System.out.println("==========ABA产生===========");
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
                ar.compareAndSet(100,101);
                System.out.println(Thread.currentThread().getName()+"第一次更改"+ar.get());
                ar.compareAndSet(101,100);//完成一次ABA
                System.out.println(Thread.currentThread().getName()+"第二次更改"+ar.get());
                latch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                ar.compareAndSet(100,2000);
                System.out.println(Thread.currentThread().getName()+"进行比较更新,更新的值为"+ar.get());
                latch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t2").start();
        latch.await();


        System.out.println("==========ABA解决===========");
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName()+"第一次更改的版本号"+asr.getStamp());
                asr.compareAndSet(100,101,asr.getStamp(),asr.getStamp()+1);
                System.out.println(Thread.currentThread().getName()+"第二次更改的版本号"+asr.getStamp());
                asr.compareAndSet(101,100,asr.getStamp(),asr.getStamp()+1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t3").start();

        new Thread(()->{
            try {
                int stamp = asr.getStamp();
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"更新时的版本号"+asr.getStamp());
                boolean result=asr.compareAndSet(100,2019,stamp,stamp+1);
                System.out.println(Thread.currentThread().getName()+"更新的结果为："+result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t4").start();

    }

}

//运行结果
/*
==========ABA产生===========
t1第一次更改101
t1第二次更改100
t2进行比较更新,更新的值为2000
==========ABA解决===========
t3第一次更改的版本号1
t3第二次更改的版本号2
t4更新时的版本号3
t4更新的结果为：false
*/

```

### 3.虚假唤醒和Lock

#### wait()的虚假唤醒

```java
class Air{
    private int i=0;

    public synchronized void increase(){
        //如果i不是0，等待
//        if (i!=0){//虚假唤醒
        while (i!=0){//把等待的线程拉回来再判断一次
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //i是0，++
        i++;
        System.out.println(Thread.currentThread().getName()+i);
        notifyAll();
    }

    public synchronized void decrease(){
//        if(i==0){//虚假唤醒
        while (i==0){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        i--;
        System.out.println(Thread.currentThread().getName()+i);

        notifyAll();
    }

}

//测试虚假唤醒

//唤醒等待在循环
public class TestFakeWake {
    public static void main(String[] args) {
        Air air=new Air();

        //多个存在通信的线程启动后，如果在if中判断，会出现虚假唤醒，此时要把if改为while
        new Thread(()->{ for (int i = 0; i < 10; i++) air.increase(); },"A").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) air.decrease(); },"B").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) air.increase(); },"C").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) air.decrease(); },"D").start();


    }

}
/*
	当判断条件为if时，可能会出现多次加操作 出现A1 C2 A3的结果，理由是if会导致虚假唤醒，比如在C进行判断后还未进行+操作时，CPU执行改到了下一个线程，然后下个线程执行后 i变为1 ，此时cpu给到了C线程，C线程就继续往下执行，进行+1操作，此时出现了并发错误，使得i变为了2。这时就需要把if改为while ， 将等待的C线程拉回来再判断一次。
*/
```

#### 线程按顺序调用

​	Condition的线程通信方法await()和signal()可以使多个线程按照顺序进行调用。

```java
//测试condition精确打击（解决线程调用顺序问题）
/*
三个线程
		A线程:打印1次A
    B线程:打印2次B
    C线程:打印3次C
 */

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

//定义三把钥匙的原因是：如果一个线程等待，那么线程堵塞，底下的signal()不会执行，无法唤醒其他线程，导致所有线程堵塞

class A {
    Lock lock=new ReentrantLock();
    //定义三把钥匙
    Condition condition1=lock.newCondition();
    Condition condition2=lock.newCondition();
    Condition condition3=lock.newCondition();

    int num=1;

    public void printA(){
        lock.lock();
        try {
            while (num!=1){
                try {
                    condition1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("A");

            num=2;
            condition2.signal();
        }finally {
            lock.unlock();
        }
    }
    public void printB(){
        lock.lock();
        try {
            while (num!=2){
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("B");
            System.out.println("B");

            num=3;
            condition3.signal();
        }finally {
            lock.unlock();
        }
    }
    public void printC(){
        lock.lock();
        try {
            while (num!=3){
                try {
                    condition3.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("C");
            System.out.println("C");
            System.out.println("C");
            num=1;
            condition1.signal();
        }finally {
            lock.unlock();
        }
    }
}




public class TestConditionAdvantage {
    public static void main(String[] args) {
        A a=new A();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                a.printA();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                a.printB();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                a.printC();
            }
        },"C").start();

    }
}
```

### 4.并发常用类

#### 1.集合类

​	多线程操作同一个集合时，多个线程可能会对集合中同一个对象进行修改，或者对同一个集合添加数据导致扩容......此时传统的集合类就可能会出导致集合内发生数据丢失等情况。因此JUC提供了几个在并发情形下数据安全的集合类。

```java
public class TestCollectionOfThread {

    public static void main(String[] args) {
//       testList();
//       testSet();
        testMap();
    }

    //适合进行读操作，添加时要复制，效率低
    public static void testList(){
        //List ist=new ArrayList();//ConcurrentModificationException
        //原理：写线程拿到锁之后修改集合，写完创建一个新集合（老集合size()+1）返回一个修改好的新集合 然后释放锁
        List<String> list=new CopyOnWriteArrayList();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            }).start();
        }
    }

    public static void testSet(){
        Set<String> set=new CopyOnWriteArraySet();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(set);
            }).start();
        }
    }

    /*
    	transient volatile Node<K,V>[] table;	ConcurrentHashMap的Node用volatile修饰，保证可见性
    
    	put()：利用cas写入，失败自旋，成功加锁写入
     */
    public static void testMap(){
//        Map map=new HashMap();
        Map map=new ConcurrentHashMap();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0,8));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }

}
```

#### 2.CountDownLatch

​	CountDownLatch（闭锁），在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。**一个线程等待获取到其他线程的执行结果**

```java
/*
    当一个或多个线程调用await()时，线程阻塞
    其他线程调用countDown()时会将计数器减一
    当计数器变为0时，被阻塞的线程被唤醒

    守护线程！！！！
 */

//7 - 0 await()的线程等待其他线程countDown()，countdown()的线程在操作后扔继续运行
//主线程阻塞等待其他线程，其他线程不阻塞
public class TestCountDownLatch {

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch latch=new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                System.out.println("第"+Thread.currentThread().getName()+"个同学离开");
                latch.countDown();//--操作
                //在执行countDown()后，线程还可继续执行
                try {
                    Thread.sleep(20000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();

        }
        latch.await();
        System.out.println("main线程结束");
    }

}
```



#### 3.CyclicBarrier

​	CyclicBarrier（栅栏），所有线程必须全部到达栅栏位置才可继续执行。**所有线程互相等待，然后再同时开始做各自的事情**

```java
// 0-7
//栅栏，用于等待其他线程   当所有线程全部到达屏障后，栅栏将会重置（reset()）  闭锁的计数器只可以用一次
//主线程等待其他线程await()，其他线程await()后阻塞，等待所有线程一起运行
public class TestCyclicBarrier {
    public static void main(String[] args) {
        //public CyclicBarrier(int parties, Runnable barrierAction)
//        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()-> System.out.println("召唤神龙"));
//        for (int i = 1; i <= 7; i++) {
//            final int t=i;
//            new Thread(()->{
//                System.out.println("得到第"+t+"颗龙珠");//线程运行完通知栅栏到达屏障
//                try {
//                    //执行await后等待，count++，使得该线程堵塞；直到count==parties时等待线程全部释放
//                    cyclicBarrier.await();
//                    System.out.println("测试一下能不能输出，测试完是可以输出的");
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                } catch (BrokenBarrierException e) {
//                    e.printStackTrace();
//                }finally {
//                    System.out.println(Thread.currentThread().getName()+"完成");
//                }
//            },String.valueOf(i)).start();
//        }
//
//        //栅栏将会重置，可以继续使用
//
//        for (int i = 1; i <= 7; i++) {
//            final int t=i;
//            new Thread(()->{
//                System.out.println("第二次得到第"+t+"颗龙珠");//线程运行完通知栅栏到达屏障
//                try {
//                    cyclicBarrier.await();//运行完等待，parties--，使得该线程堵塞；直到到达指定个数等待线程全部释放
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                } catch (BrokenBarrierException e) {
//                    e.printStackTrace();
//                }
//            },String.valueOf(i)).start();
//        }
        CyclicBarrier cyclicBarrier1=new CyclicBarrier(7,()-> {
            System.out.println("召唤神龙");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("此线程运行完后其他线程才运行await()后的代码");
        });
        MyThread05 myThread05=new MyThread05(cyclicBarrier1);

        for (int i=0;i<7;i++){
            Thread thread = new Thread(myThread05);
            thread.start();
        }
    }
}

class MyThread05 implements Runnable{

    private CyclicBarrier cyclicBarrier;

    public MyThread05(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        System.out.println("111");
        try {
            cyclicBarrier.await();
            System.out.println(Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}
```

#### 4.Semaphore

```java
//用于多个共享资源的互斥使用和并发线程数的控制
public class TestSemaphore {
    public static void main(String[] args) {

        Semaphore semaphore=new Semaphore(2);//资源数
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println("第"+Thread.currentThread().getName()+"抢到了资源");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println("第"+Thread.currentThread().getName()+"释放了资源");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            }).start();
        }

    }
}
```

#### 5.ForkJoin

```java
/*
	多线程并发处理框架，利用了递归的思想
	将复杂任务分给多个线程处理在将每个线程的结果合并
*/
public class TestForkJoin {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        MyTask myTask=new MyTask(1,100);

        ForkJoinPool pool=new ForkJoinPool();

        ForkJoinTask<Integer> task=pool.submit(myTask);

        System.out.println(task.get());//获得最终值

        //关闭池
        pool.shutdown();

    }

}

class MyTask extends RecursiveTask<Integer>{

    private static final int MIN=10;
    private int start;
    private int end;
    private int result;

    public MyTask(int start, int end) {
        this.start = start;
        this.end = end;
    }



    @Override
    protected Integer compute() {

        if (end - start <= MIN){
            for(int i=start;i<=end;i++){
                System.out.println(Thread.currentThread().getName()+"正在计算"+i);
                result+=i;
            }
        }else {
            int mid=(start+end)/2;
            MyTask myTask = new MyTask(start, mid);//fork()进行递归
            MyTask myTask1 = new MyTask(mid + 1, end);
            myTask.fork();
            myTask1.fork();
            result= myTask.join()+myTask1.join();//join() 返回结果


        }

        return result;
    }
}
```

#### 6.CompletableFuture

```java
//异步回调  主线程不会等待异步函数执行完再去执行下面的代码，而是给分线程去执行，等到执行完后再回到结果
public class TestCompletableFuture {

    public static void main(String[] args) throws Exception {
        //runAsync():没有返回值

        /*
        CompletableFuture<Void> completableFuture=CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("update Mysql success");
        });

        new Thread(()->{
            int result = 0;
            for (int i = 0; i < 100; i++) {
                result+=i;
            }
            System.out.println("result:"+result);
        }).start();

        completableFuture.get();*/

        CompletableFuture<Integer> com=CompletableFuture.supplyAsync(()->{

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
//            int i=1/0;
            return 1024;
        });

        //上个线程异步处理，main处理下个线程
        new Thread(()->{
            int result = 0;
            for (int i = 0; i < 100; i++) {
                result+=i;
            }
            System.out.println("result:"+result);
        }).start();

        System.out.println(com.whenComplete((u, t) -> {
            System.out.println("成功调用" + u);
            System.out.println("失败调用" + t);
        }).exceptionally(t -> {
            System.out.println(t.getMessage());
            return 4444;
        }).get());//get()回调，获取函数结果

    }

}
```

### 5.AQS

#### 1.介绍

​	AQS:AbstractQueuedSynchronizer(抽象队列同步器)。是由**双向链表加锁状态**实现的同步队列，底层**应用CAS实现**，使用了**模板方法模式**，JUC是基于AQS 实现的，用一个volatile int修饰的变量state表示同步状态

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200830232028308.png" alt="image-20200830232028308" style="zoom:50%;" />



​		以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

　　再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

　　一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。

#### 2.具体实现

​	AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。
  不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false

##### 1.acquire(int)

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

说明：

- tryAcquire()尝试直接去获取资源，如果成功则直接返回；

- addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；

- acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。

- 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

流程图：

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200830232140081.png" alt="image-20200830232140081" style="zoom:50%;" />

方法实现：

###### 1.tryAcquire(int)

```java
		protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
```

​	tryAcquire尝试以独占的方式获取资源，如果获取成功，则直接返回true，否则直接返回false。该方法可以用于实现Lock中的tryLock()方法。该方法的默认实现是抛出`UnsupportedOperationException`，具体实现由自定义的扩展了AQS的同步类来实现。AQS在这里只负责定义了一个公共的方法框架。



###### 2.addWirter()

```java
		private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        Node pred = tail;
      	//如果队列不为空，将此节点加入等待队列
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
      	//如果队列为空，初始化队列并将当前节点插入等待队列
        enq(node);
        return node;
    }
```

​	该方法用于将当前线程根据不同的模式（`Node.EXCLUSIVE`互斥模式、`Node.SHARED`共享模式）**加入到等待队列的队尾，并返回当前线程所在的结点**。如果队列不为空，则以通过`compareAndSetTail`方法以CAS的方式将当前线程节点加入到等待队列的末尾。否则，通过enq(node)方法初始化一个等待队列，并返回当前节点。

​	**enq():**

  `enq(node)`用于***将当前节点插入等待队列，如果队列为空，则初始化当前队列***。整个过程以CAS自旋的方式进行，直到成功加入队尾为止。源码如下：

```java
		private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

###### 3.acquireQueued(Node, int)

  `acquireQueued()`用于队列中的线程自旋地以独占且不可中断的方式获取同步状态（acquire），直到拿到锁之后再返回。该方法的实现分成两部分：如果当前节点已经成为头结点，尝试获取锁（tryAcquire）成功，然后返回；否则检查当前节点是否应该被park，然后将该线程park并且检查当前线程是否被可以被中断。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;//标记等待过程中是否被中断过
        
        //“自旋”！
        for (;;) {
            final Node p = node.predecessor();//拿到前驱
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                failed = false; // 成功获取资源
                return interrupted;//返回等待过程中是否被中断过
            }
            
            //如果自己可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
        }
    } finally {
        if (failed) // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
            cancelAcquire(node);
    }
}
```

###### 4.整体流程:

- 调用自定义同步器的tryAcquire()尝试直接去获取资源，如果成功则直接返回；

- 没成功，则addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；

- acquireQueued()使线程在等待队列中休息，有机会时（轮到自己，会被unpark()）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。

- 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

##### 2.release(int)

说明：此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。这也正是unlock()的语义，当然不仅仅只限于unlock()


```java
 		public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

​	tryRelease(int) ：

​		tryRelease()方法也是需要独占模式的自定义同步器去实现的。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以**自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。**

```java
		protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
```

​	unparkSuccessor()

​		**用于唤醒等待队列中下一个线程**。这里要注意的是，下一个线程并不一定是当前节点的next节点，而是下一个可以用来唤醒的线程

```java
private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
  			//找出可唤醒的等待队列中的线程
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
          	//唤醒
            LockSupport.unpark(s.thread);
    }
```



##### 3.acquireShared(int)

```java
		public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

这里tryAcquireShared()依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：**负值代表获取失败；0代表获取成功**，**但没有剩余资源；正数表示获取成功，还有剩余资源**，其他线程还可以去获取。所以这里acquireShared()的流程就是：

	tryAcquireShared()尝试获取资源，成功则直接返回；
	失败则通过doAcquireShared()进入等待队列，直到获取到资源为止才返回。

其实跟acquire()的流程大同小异，只不过多了个**自己拿到资源后，还会去唤醒后继队友的操作**

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);//加入队列尾部
    boolean failed = true;//是否成功标志
    try {
        boolean interrupted = false;//等待过程中是否被中断过的标志
        for (;;) {
            final Node p = node.predecessor();//前驱
            if (p == head) {//如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
                int r = tryAcquireShared(arg);//尝试获取资源
                if (r >= 0) {//成功
                    setHeadAndPropagate(node, r);//将head指向自己，还有剩余资源可以再唤醒之后的线程
                    p.next = null; // help GC
                    if (interrupted)//如果等待过程中被打断过，此时将中断补上。
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```



##### 4.releaseShared()

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放资源
        doReleaseShared();//唤醒后继结点
        return true;
    }
    return false;
}
```

​		此方法的流程也比较简单，一句话：**释放掉资源后，唤醒后继**。跟独占模式下的release()相似，但有一点稍微需要注意：独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后tryReleaseShared(2)返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，tryReleaseShared(2)返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定tryReleaseShared()的返回值。

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);//唤醒后继
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)// head发生变化
            break;
    }
}
```

#### 3.应用

​	Mutex是一个不可重入的互斥锁实现。锁资源（AQS里的state）只有两种状态：0表示未锁定，1表示锁定。

互斥锁的实现：

```java
class Mutex implements Lock, java.io.Serializable {
    // 自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {
        // 判断是否锁定状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 尝试获取资源，立即返回。成功则返回true，否则false。
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // 这里限定只能为1个量
            if (compareAndSetState(0, 1)) {//state为0才设置为1，不可重入！
                setExclusiveOwnerThread(Thread.currentThread());//设置为当前线程独占资源
                return true;
            }
            return false;
        }

        // 尝试释放资源，立即返回。成功则为true，否则false。
        protected boolean tryRelease(int releases) {
            assert releases == 1; // 限定为1个量
            if (getState() == 0)//既然来释放，那肯定就是已占有状态了。只是为了保险，多层判断！
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);//释放资源，放弃占有状态
            return true;
        }
    }

    // 真正同步类的实现都依赖继承于AQS的自定义同步器！
    private final Sync sync = new Sync();

    //lock<-->acquire。两者语义一样：获取资源，即便等待，直到成功才返回。
    public void lock() {
        sync.acquire(1);
    }

    //tryLock<-->tryAcquire。两者语义一样：尝试获取资源，要求立即返回。成功则为true，失败则为false。
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    //unlock<-->release。两者语文一样：释放资源。
    public void unlock() {
        sync.release(1);
    }

    //锁是否占有状态
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }
}
```

### 6.锁

#### 1.公平锁与非公平锁

​	公平锁：多个线程按照锁的申请顺序来获取锁。FIFO

​	非公平锁：多个线程获取锁的顺序不固定，可能会造成线程优先级反转或者饥饿现象。

​	**非公平锁的优点在于吞吐量比公平锁大**，ReentrantLock 默认非公平锁 。synchronized也是非公平锁

#### 2.可重入锁（递归锁）

​	可重入锁也叫递归锁，指的是同一线程外层函数获得锁后，内层递归函数仍然能获得该锁的代码，在同一个线程的外层方法获得锁的时候，在进入内层方法会自动获取锁。即线程可以进入任何一个它已经拥有的锁所同步着的代码块。可重入锁可**避免死锁**。ReentrantLock和synchronized都是可重入锁

```java
class Phone{
    public synchronized void sendMes(){
        System.out.println(Thread.currentThread().getName()+"sendMes");
        sendEmail();
    }

    public synchronized void sendEmail(){
        System.out.println(Thread.currentThread().getName()+"sendEmail");
    }
}
//测试ReentrantLock
class Phone02 implements Runnable{
    Lock lock=new ReentrantLock();

    @Override
    public void run() {
        get();
    }

    public void get(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"get");
            set();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }

    private void set() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"set");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

}

public class EntrantLock {
    public static void main(String[] args) {
        Phone phone=new Phone();
        new Thread(()->{
            phone.sendMes();
        },"t1").start();

        new Thread(()->{
            phone.sendMes();
        },"t2").start();

        Phone02 p2=new Phone02();
        new Thread(p2,"t3").start();
        new Thread(p2,"t4").start();
    }

}
```

#### 3.自旋锁

​	自旋锁是指尝试获取的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环消耗可能会消耗CPU资源

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
public class TestSpinLock {
    public static void main(String[] args) {
        SpinLock lock=new SpinLock();
        new Thread(()->{
            lock.myLock();
            try {TimeUnit.SECONDS.sleep(5);}catch(InterruptedException e){ e.printStackTrace();}
            lock.myUnlock();
        },"t1").start();

        try {TimeUnit.SECONDS.sleep(1);}catch(InterruptedException e){ e.printStackTrace();}

        new Thread(()->{
            lock.myLock();
            lock.myUnlock();
        },"t2").start();
    }
}

```

#### 4.读写锁

​	独占锁：指该锁只能被一个线程持有。

​	共享锁：可被多个线程持有。

​	ReentrantReadWriteLock的读锁为共享锁，写锁为独占锁。该锁可保证高效的并发读

```java
//读写锁(写锁--独占锁，读锁--共享锁)
class ReadAndWrite {

    private volatile Map<String,Object> map=new HashMap<>();
    private ReadWriteLock lock=new ReentrantReadWriteLock();

    public void put(String key,Object value) throws InterruptedException {
        lock.writeLock().lock();
        try {
            System.out.println("写入数据"+key+":"+value);
            map.put(key,value);
            TimeUnit.SECONDS.sleep(1);
            System.out.println("写入完成");
        }finally {
            lock.writeLock().unlock();
        }
    }

    public void get(String key) throws InterruptedException{
        lock.readLock().lock();
        try {
            Object o = map.get(key);
            System.out.println("读出数据"+key+":"+o);
            TimeUnit.SECONDS.sleep(1);
            System.out.println("读出完成");
        }finally {
            lock.readLock().unlock();
        }
    }

}

public class TestReadWriteLock {
    public static void main(String[] args) {
        ReadAndWrite rw=new ReadAndWrite();
        for (int i = 0; i < 10; i++) {
            final int tmp=i;
            new Thread(()->{
                try {
                    rw.put(Thread.currentThread().getName(),tmp);
                    rw.get(Thread.currentThread().getName());

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### 6.线程池

#### 1.线程池的创建

​	线程池的三个实现类：

- Executors.newCachedThreadPool()：一池N线程，可扩容，根据需要创建新县城，当先前线程可用时会复用。
- Executors.newFixedThreadPool(int i)：一池i线程，在执行长期任务时性能较高
- Executors.newSingleThreadExecutor()：一池一线程

##### 创建底层：

```java
//newSingleThreadExecutor
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

//newCachedThreadPool
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
}

//newFixedThreadPool
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
}
```

​	这三个线程池的实现了底层都创建了一个`ThreadPoolExecutor`，创建的参数依次为：

- **corePoolSize**：常驻核心线程数
- **maximumPoolSize**：可容纳同时执行的最大线程数
- **keepAliveTime**：多余线程的存活时间
- unit：时间单位
- **workQueue**：阻塞队列
  - ArrayBlockingQueue：底层1个数组+1把锁+两个条件，出入队时用一把锁，在仅出队或入队的时候性能高，必须指定容量
  - LinkedBlockingQueue：底层1个单向链表+2把锁+两个条件，2把锁一个出队一个入队。最大容量为整数最大值
  - SynchronousQueue：单个元素的队列

- ThreadFactory：线程工厂
- **RejectedExecutionHandler**：拒绝策略

##### 创建要求：

​	在创建线程池时最好手动new	ThreadPoolExecutor，这样可以避免资源耗尽的风险。通过Executors返回线程池的弊端如下：

- FixedThreadPool和SingleThreadExecutor：允许的队列长度（LinkedBlockingQueue）为Integer.Max_VALUE，可能会堆积大量请求，从而OOM。
- CachedThreadPool和ScheduledThreadPOOL：允许创建的线程数量（maximumPoolSize）为Integer.Max_VALUE，可能会创建大量的线程，从而OOM

```java
public class TestThreadPool {

    public static void main(String[] args) {

			 	//获取cpu核数
        int cpu = Runtime.getRuntime().availableProcessors();
        System.out.println("cpu核数："+cpu);
        ExecutorService pool=new ThreadPoolExecutor(
                2,
                cpu+1,//cpu密集型，最大的线程池的线程数量设置为cpu核心数+1
                2l,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<Runnable>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
                );
        try {
            for (int i = 0; i < 15; i++) {
                final int j=i;

                pool.execute(() -> {

                    try {
                        if(j==5)
                            TimeUnit.SECONDS.sleep(10);
                        else
                            TimeUnit.SECONDS.sleep(2);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "线程在执行");
                });
            }
        }finally {
            pool.shutdown();//关闭线程池
        }
    }
}
```

#### 2.线程池的优势

​	线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。

​	他的主要特点为：线程复用；控制最大并发数；管理线程。具体的优势如下：

第一：**降低资源消耗**。通过重复利用已经创建的线程降低线程创建和销毁造成的消耗。

第二：**提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。

第三：**提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

#### 3.线程池的工作流程

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200831223441946.png" alt="image-20200831223441946" style="zoom:33%;" />

1. 在创建了线程池后，等待提交过来的任务请求

2. 当调用 execute()方法添加一个请求任务时，线程池会做如下判断：

   2.1 如果正在运行的线程数量小于 corPoolSize，那么马上创建线程运行这个任务

   2.2 如果正在运行的线程数量大于或等于 corPoolSize，那么将这个任务放入队列

   2.3 如果这时候队列满了且正在运行的线程数量还小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务

   2.4 如果队列满了且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会启动饱和拒绝策略来执行

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行

4. 当一个线程无事可做超过一定的时间（keepalivetime）时，线程池将线程数恢复为核心线程数量。

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

#### 4.设置最大线程数的值

##### CPU密集型

​	设置为**cpu核数+1**。原因是在CPU密集型的线程恰好在某时因为发生一个页错误或者因为其他原因而暂停，刚好有一个额外的线程可以确保在这种情况下CPU周期不会中断工作

##### IO密集型

```java
线程数=cpu核心数*(1/cpu利用率)=cpu核心数*(1+IO耗时/cpu耗时)
  
如：
  计算操作需要5ms，DB操作需要100ms，对于一个8cpu的服务器怎么设置线程数？如果DB的qps上限为1000，线程数应该为多大
  
  线程数=8*（1+100/5）=168。
  
  1s=1000ms
  每个线程每秒处理的任务数： 1000/105
  168个线程每秒处理的任务数： 168*1000/105=1600qps
  1600/1000=168/x
  x=105
  
```

#### 5.拒绝策略

- AbortPolicy（默认）：直接抛出RejectExecutionException阻止系统正常运行
- CallerRunsPolicy：改策略既不会抛弃任务，也不会抛出异常，而是将多出的线程返回给调用者
- DiscardOldestPolicy：抛弃任务重等待最久的任务，然后将新到的任务加入队列再次提交当前任务
- DiscardPolicy：直接丢弃任务，不予任何处理也不抛出异常。允许任务丢失时最好的方案

#### 6.线程池的优雅关闭

```java
//如何优雅的关闭线程池
/*
使用shutdownNow方法，可能会引起报错，
使用shutdown方法可能会导致线程关闭不了。

所以使用shutdownNow方法关闭线程池时，一定要对任务里进行异常捕获。
使用shutdown方法关闭线程池时，一定要确保任务里不会有永久阻塞等待的逻辑，否则线程池就关闭不了。
 */
public class TestThreadPoolInterrupt {

    public static void main(String[] args) {

        ThreadPoolExecutor pool=new ThreadPoolExecutor(
                2,2,2l,TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(15),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());

        for (int i = 0; i < 15; i++) {
            final int j=i;
            pool.execute(()->{
                try {
                    System.out.println(j);
//                    Thread.sleep(2000l);
                    //此时pool.shutdown()无法退出;pool.shutdownNow()可以退出
                    while (!Thread.currentThread().isInterrupted()); //此方法不清除标志位。判断标志位，若标记过则为true
//                    while (!Thread.interrupted());                 //此方法会清除标志位
                } catch (Exception e) {
                    System.out.println("中断后抛出了异常");
                }
            });
        }

//     shutdownNow方法：线程池拒接收新提交的任务，同时立马关闭线程池，线程池里的任务不再执行。
//      此方法会遍历线程池里的所有工作线程，然后调用所有线程的interrupt方法
//      若线程中存在sleep方法，则会抛出异常后直接退出，若不存在sleep还有循环，要判断标志位后才能退出
//      如果阻塞队列中有任务,不管直接关闭
        pool.shutdownNow();

//     shutdown方法：线程池拒接收新提交的任务，同时等待线程池里的任务执行完毕后关闭线程池。
//      此方法不管怎样，都会继续运行完池中的任务，任务不完毕，则不会释放资源。
//      如果阻塞队列中有任务，则唤醒执行，无任务则退出
//        pool.shutdown();

        System.out.println("main end");
    }
}

```

### 7.ThreadLocal

#### 1.概述

​	ThreadLocal是一个线程内部的存储类。ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题。此外虽然在方法中创建的变量在栈帧中存在也是线程私有的，但是在数据库连接或者Session中频繁的获取释放会增大服务器的压力，严重影响程序执行性能

```java
//l2.set(1)-->Thread.threadLocalMap-->map.set(l2,1)
//一个线程的threadLocalMap可以存储多个local对象
public static void main(String[] args) {
        ThreadLocal l1=new ThreadLocal();
        ThreadLocal l2=new ThreadLocal();
        new Thread(()->{
            l1.set(1);
            l2.set(2);
            System.out.println(l2.get());//2
            l2.set(1);
            System.out.println(l2.get());//1
        },"1").start();
        new Thread(()->{
            Object o = l1.get();
            Object o1 = l2.get();
            System.out.println(o1+" "+o);//null null
        },"1").start();
}
```

#### 2.原理

类中提供的主要方法

- public T get()
- public void set(T value)
- public void remove() 
- proteced T initialValue()

##### 1.get()

```java
public T get() {
  	//获取当前线程	
    Thread t = Thread.currentThread();
  	//获取当前线程内部的map  
  	ThreadLocalMap map = getMap(t);
  	//如或map不为空
    if (map != null) {
      	//将this（当前threadlocal变量）获取map的键值对
        ThreadLocalMap.Entry e = map.getEntry(this);
      	//如果键值对不为空
        if (e != null) {
           	//获取值并返回
            T result = (T)e.value;
            return result;
        }
    }
  	//如果为空，则初始化
    return setInitialValue();
}
```



getMap()

```java
ThreadLocalMap getMap(Thread t) {
  	//thread类中的成员变量 ThreadLocal.ThreadLocalMap threadLocals = null;
    return t.threadLocals;
}
```



setInitialValue()：

```java
private T setInitialValue() {
    T value = initialValue();// initialValue() 如未重写直接return null;
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);//t.threadLocals = new ThreadLocalMap(this, firstValue);
    return value;
}
```



##### 2.set()

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```



##### 3.initialValue()

```java
protected T initialValue() {
    return null;
}
```

创建threadLocal变量时重写此方法，则get()前未set()不报空指针异常

##### 4.整体流程

​		首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

　　初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

　　然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200830233823038.png" alt="image-20200830233823038" style="zoom:50%;" />

#### 3.总结

- 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals(map)中的；
- 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，**因为每个线程中可有多个threadLocal变量。**
- 在进行get之前，必须先set，否则会报空指针异常
-  如果重写了initialValue方法，则可以不set()先get()



#### 4.应用场景

##### 1.并发下的数据库连接

```java
private static ThreadLocal<Connection> connectionHolder= newThreadLocal<Connection>() {
	publicConnection initialValue() {
    return DriverManager.getConnection(DB_URL);
	}
};
public static Connection getConnection() {
	return connectionHolder.get();
}
```

##### 2.Session

```java
private static final ThreadLocal threadSession = new ThreadLocal();
public static Session getSession() throws InfrastructureException {
    Session s = (Session) threadSession.get();
    try {
        if(s == null) {
            s = getSessionFactory().openSession();
            threadSession.set(s);
        }
    	}catch(HibernateException ex) {
        throw new InfrastructureException(ex);
    	}
    	return s;
}
```

#### 4.说明

##### 1.这种存储结构的好处：

​	1、线程死去的时候，线程共享变量ThreadLocalMap则销毁。

​	2、ThreadLocalMap<ThreadLocal,Object>键值对数量为ThreadLocal的数量，一般来说ThreadLocal数量很少，相比在ThreadLocal中用Map<Thread, Object>键值对存储线程共享变量（Thread数量一般来说比ThreadLocal数量多），性能提高很多。



##### 2.关于弱引用问题：

​	ThreadLocalMap<ThreadLocal, Object>的弱引用:**ThreadLocal中的内部类**：	`static class Entry extends WeakReference<ThreadLocal<?>>` 。当线程没有结束，但是ThreadLocal已经被回收，则可能导致线程中存在ThreadLocalMap<null, Object>的键值对，造成内存泄露。（ThreadLocal被回收，ThreadLocal关联的线程共享变量还存在）。

​	虽然ThreadLocal的get，set方法可以清除ThreadLocalMap中key为null的value，但是get，set方法在内存泄露后并不会必然调用，所以为了防止此类情况的出现，有两种方法可避免：

​	1、使用完线程共享变量后，显示调用ThreadLocalMap.remove方法清除线程共享变量；

​	2、JDK建议ThreadLocal定义为private static，这样ThreadLocal的弱引用问题则不存在了。

