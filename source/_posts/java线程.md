---
title: java线程学习笔记
date: 2020-11-16
categories: Java核心    #分类
tags: java线程          #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

## 1. 线程基本概念

### 1.1 线程与进程

当一个程序进入内存运行时，即变成一个进程。进程是系统进行资源分配和调度的一个独立单元。

线程是进程的组成部分，一个进程可以拥有多个线程，一个线程必须有一个父进程。

>   归纳起来说就是：操作系统可以同时执行多个任务，每个任务就是进程；而进程也可以同时执行多个任务，每个任务就是线程。

### 1.2 并发与并行

并行：在同一时刻，有多条指令在多个处理器上同时执行

并发：在同一时刻，只能有一条指令执行，但多个指令在一个处理器上被快速轮流切换执行。



## 2. 线程的创建与启动

**线程创建有4种方式，但是线程的启动只能调用线程对象的start() 方法**

创建线程有以下4种方式：

1.  继承 Thread 类
2.  实现 Runnable 接口
3.  实现 Callable 接口
4.  线程池


### 2.1 继承Thread抽象类

~~~java
public class Thread1 extends Thread {
    private int i;

    @Override
    public void run() {
        for (; i< 10; i++){
            System.out.println(getName()+ " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+ " " + i);
            if (i == 10){
                new Thread1().start();
                new Thread1().start();
            }
        }
    }
}
~~~

运行结果：

![](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201208112902783-377538696.png)

### 2.2 实现Runnable接口

~~~java
public class Demo{
    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+ " " + i);
            if (i == 10){
                //Lambda表达式实现runnable接口
                new Thread(()->{
                        for (int j= 0; j< 10; j++){
                            System.out.println(Thread.currentThread().getName()+ " " + j);
                        }
                }, "线程2").start();

                //Lambda表达式实现runnable接口
                new Thread(()-> {
                        for (int j = 0; j< 10; j++){
                            System.out.println(Thread.currentThread().getName()+ " " + j);
                        }
                }, "线程1").start();
            }
        }
    }
}
~~~

运行结果：

![](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201208113221333-2130705942.png)

### 2.3 实现Callable接口

~~~java
public class CallableDemo {

    public static void main(String[] args) {
		//使用FutureTask对象老包装 Callable 对象
      	//Callable 对象含有返回值
        FutureTask<Integer> task = new FutureTask<>(()-> {
                int i = 0;
                for( ; i< 10; i++){
                    System.out.println(Thread.currentThread().getName()+ " " + i);
                }
                return i;
        });
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+ " " + i);
            if (i == 10){
              	//start() 方法启动线程
                new Thread(task, "有返回值的线程").start();
            }
        }
        try {
          	//通过FutureTask 对象的get() 方法获取返回值
            System.out.println("子线程返回值：" + task.get());
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}
~~~

运行结果：

![](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201208113417698-2089956495.png)

>   先介绍以上三种，线程池后面细讲

### 2.4 三种方式对比

从实现方式上对比，实现 Runnable 接口与实现 Callable 接口属于同一种线程启动的方式。

采用继承Thread类创建线程的方式有以下优缺点：

+   **优势：编写简单，要获取当前线程不需要使用 Thread.currentThread() 方法，只需要使用 this 即可；**
+   **劣势：java只能单继承，因为线程类继承了 Thread 类就不能再继承其他父类。**

采用实现 Runnable、Callable 接口方式有以下优缺点：

+   **优势：实现接口后还能继承其他父类；**
+   **优势：在这种方式下，多个线程可以共享同一个目标对象，所以非常适合多个线程来处理同一份资源的情况（也就是生产者消费者模式）；**
+   **劣势：变成稍稍复杂，要使用 Thread.currentThread() 方法获取当前线程。**

---



## 3. 线程控制

### 3.1 线程生命周期

在线程的生命周期中，要经过新建（New）、就绪（Runnable）、运行（Run）、阻塞（Blocked）、和死亡（Dead）5种状态。尤其是当线程启动后，它不能一直霸占着CPU独自运行，所以CPU需要在多条线程之间切换，于是线程状态也会在多次运行、阻塞之间切换。

![](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201208140238901-2035332028.png)

### 3.2 join 线程

>    => join(): 等待被join的线程执行完成；

>   => join(long millis): 等待被join的线程时间最长为millis毫秒；

当在某个程序执行过程中调用其他线程的 join() 方法，调用线程将被阻塞，直到被 join() 方法加入的 join 线程执行完为止。

【例】在一道程序中有A线程和B线程两个，在B线程中调用 A.join(); 则程序会在A线程执行完毕后才会开始执行B线程。

~~~java
public class Thread1 extends Thread {

    private int i;

    @Override
    public void run() {
        for (; i< 10; i++){
            System.out.println(getName()+ " " + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+ " " + i);
            if (i == 5){
                Thread1 t1 = new Thread1();
                t1.start();
                //main 线程调用了t1 线程的join() 方法
                // 必须等t1 执行结束才会向下执行
                t1.join();
            }
        }
    }
}
~~~

运行结果：

![](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201208145557326-2071910976.png)

### 3.3 线程睡眠：sleep 

static void sleep(long millis)：让当前正在执行的线程暂停 millis 毫秒，并进入阻塞状态；

static vold sleep(long millis, int nanos)：让当前正在执行的线程暂停 millis 毫秒加 nanos 毫微秒，并进入阻塞状态。

>   【注】调用sleep() 方法不会释放所占有的资源

### 3.4 线程等待：wait

wait() 是Object 基类的方法，作用与sleep 类似，使得线程进入等待队列。**不同的是，当线程调用wait()方法后会进入等待队列（进入这个状态会释放所占有的所有资源，与阻塞状态不同）**

线程调用 wait() 方法后，要调用notify / notifyAll() 方法才能将其唤醒，进入就绪状态。

>   **调用notify() 方法导致解除阻塞的线程是从因调用该对象的 wait()方法而阻塞的线程中随机选取的**，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。
>
>   **调用 notifyAll()方法将把因调用该对象的 wait()方法而阻塞的所有线程一次性全部解除阻塞** 。当然，只有获得锁的那一个线程才能进入可执行状态。

### 3.5 线程让步：yield

yield() 方法是一个和sleep() 方法有点相似的方法，它也可是 Thread 类提供的一个静态方法，它可以让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程转入就绪状态。

>   yield() 只是让当前线程暂停一下，让系统的线程调度器重新调用一次，完全可能的情况是：当某个线程调用了yield() 方法暂停后，线程调度器又将其重新调度出来执行。

### 3.6 线程优先级：setPriority

每个线程执行时都具有一定的优先级，优先级高的线程会获得较多的执行机会。

每个线程默认的优先级都与创建它的父线程的优先级相同。（main 线程除外，默认情况下 main 线程创建的子线程和 main 线程都具有普通优先级）

>   Thread 类提供了 setPriority(int new Priority)、getPriority() 方法来设置和返回指定线程的优先级。范围的1~10之间。也可以使用 Thread 类的以下三个静态常量：
>
>   MAX_PRIORITY：其值为10。
>
>   MIN_PRIORITY：其值为1。
>
>   NORM_PRIORITY：其值为5。

---

## 4. 线程同步

解决线程安全问题的方式有：

1.  同步代码块
2.  同步方法
3.  同步锁（Lock）

### 4.1 同步代码块

线程开始执行同步代码块之前，必须先获得同步监视器的锁定。语法格式：

~~~java
synchronized(obj){
  ...
  //此处的代码为同步代码块
}
~~~

>   【注意】任何时刻只能有一个线程可以获得对同步监视器的锁定，当同步代码块执行完成后，该线程会释放对同步监视器的锁定。

>   【提示】java 程序允许使用任意对象作为同步监视器，**通常推荐使用可能被并发访问的共享资源充当同步监视器**。

### 4.2 同步方法

同步方法就是使用 synchronized 关键字来修饰某个方法。对于 synchronized 修饰的实例方法（非 static 方法）而言，同步方法的同步监视器就是 this，也就是调用该方法的对象。

>   JDK 所提供的 StringBuilder、StringBuffer 就是为了照顾单线程环境和多线程环境所提供的类。两个类代码几乎相似，只是 StringBuffer 的方法使用了synchronized 关键字修饰。
>
>   所以在单线程环境下应该使用 StringBuilder 来保证较好的性能；多线程环境应使用 StringBuffer  来保证线程安全。

### 4.3 同步锁（Lock）

同步锁是比 synchronized 方法和 synchronized 代码块更细粒度的同步机制，由 Lock 对象充当。

在线程安全控制中，比较常用的是 ReentrantLock (可重入锁)。使用该Lock 对象可显示地加锁、释放锁，通常使用 ReentrantLock 的代码格式如下：

~~~java
class xxx {
  //定义锁对象
  private final ReentrantLock lock = new ReentrantLock();
  //...
  //定义需要保证线程安全的方法
  public void method(){
    //加锁
    lock.lock();
    try{
      //需保证线程安全的代码
      //... method body
    }finally{	//使用 finally 块来保证释放锁
      //释放锁
      lock.unlock;
    }
  }
}
~~~

## 5. 线程通信

### 5.1 传统的线程通信

传统的线程通信是借助于 Object 类提供的 wait()、notify()、notifyAll() 三个方法，结合 synchronized 修饰的同步方法或同步代码块实现的。在这里不做介绍。

### 5.2 Condition 控制线程

Java 提供了一个Condition 类，使用 Condition 可以让那些已经得到 Lock 对象却无法继续执行的线程释放 Lock对象，也可以唤醒	其他处于等待状态的线程。

Condition 将 Object 监视器方法（wait、notify 和 notifyAll）分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用。

要获得特定 Lock 实例的Condition 实例，调用该 Lock 对象的 newCondition() 方法即可。

>Condition 类提供了如下三个方法：
>
>await()：让当前线程进入等待状态，直到其他线程调用该 Condition 的 signal() 方法或 signalAll() 方法来唤醒该线程。
>
>signal()：唤醒在此 Condition 对象上等待的单个线程。如果有多个，则会随机唤醒一个。
>
>signalAll()：唤醒在此 Condition 对象上等待的所有线程。



例：经典问题：三个线程依次打印10次ABC，代码示例如下：

~~~java
class Print {
    private Lock lock = new ReentrantLock();
    private Condition conditionA = lock.newCondition();
    private Condition conditionB = lock.newCondition();
    private Condition conditionC = lock.newCondition();
    private String type = "A"; //内部状态

    /*
     * 方法的基本要求为：
     * 1、该方法必须为原子的。
     * 2、当前状态必须满足条件。若不满足，则等待；满足，则执行业务代码。
     * 3、业务执行完毕后，修改状态，并唤醒指定条件下的线程。
     */
    public void printA() {
        lock.lock(); //锁，保证了线程安全。
        try {
            while (type != "A") { //type不为A，
                try {
                    conditionA.await(); //将当前线程阻塞于conditionA对象上，将被阻塞。
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            //type为A，则执行。
            System.out.println(Thread.currentThread().getName() + " 正在打印A");
            type = "B"; //将type设置为B。
            conditionB.signal(); //唤醒在等待conditionB对象上的一个线程。将信号传递出去。
        } finally {
            lock.unlock(); //解锁
        }
    }

    public void printB() {
        lock.lock(); //锁
        try {
            while (type != "B") { //type不为B，
                try {
                    conditionB.await(); //将当前线程阻塞于conditionB对象上，将被阻塞。
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            //type为B，则执行。
            System.out.println(Thread.currentThread().getName() + " 正在打印B");
            type = "C"; //将type设置为C。
            conditionC.signal(); //唤醒B
        } finally {
            lock.unlock(); //解锁
        }
    }

    public void printC() {
        lock.lock(); //锁
        try {
            while (type != "C") {
                try {
                    conditionC.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            System.out.println(Thread.currentThread().getName() + " 正在打印C");
            type = "A";
            conditionA.signal();    //唤醒A
        } finally {
            lock.unlock(); //解锁
        }
    }
}


public class Test{

    public static void main(String[] args) {
        final Business business = new Business();//业务对象。

        //线程1号，打印10次A。
        Thread ta = new Thread(()-> {

                for(int i=0;i<10;i++){
                    business.printA();
                }
        });

        //线程2号，打印10次B。
        Thread tb = new Thread(()-> {

                for(int i=0;i<10;i++){
                    business.printB();
                }
        });

        //线程3号，打印10次C。
        Thread tc = new Thread(()-> {

                for(int i=0;i<10;i++){
                    business.printC();
                }
        });

        //执行3条线程。
        ta.start();
        tb.start();
        tc.start();
    }
}
~~~

### 5.3 阻塞队列 BlockingQueue

Java 5 提供了一个 BlockingQueue 接口，作为线程同步的工具。

BlockingQueue 具有一个特征：**当生产者线程试图向 BlockingQueue 中放入元素时，如果该队列已满，则该线程被阻塞；当消费者线程试图从 BlockingQueue 中取出元素时，如果该队列为空，则该线程被阻塞。**

程序的两个线程通过交替向 BlockingQueue 中放入元素、取出元素，即可很好地控制线程的通信。

>   BlockingQueue 提供了如下两个支持阻塞的方法：
>
>   put(E e)：尝试把 E 元素插入 BlockingQueue 尾部，如果该队列已满，则阻塞该线程。
>
>   take()：尝试从 BlockingQueue 的头部取出元素，如果该队列的元素已空，则阻塞该线程。

代码例子：

~~~java
public class BlockingQueueTest{
  public static void main(String[] args) throws Excption{
    // 定义一个长度为 2 的阻塞队列
    BlockingQueue<String> bq = new ArrayBlockingQueue<>(2);
    bq.put("Java");	// 将 "Java" 放入 bq
    bq.put("Spring"); // 将 "Spring" 放入 bq
    bq.put("JDBC");	// 阻塞线程（且 "JDBC" 不会被放入）
    Stirng str = bq.take(); // "Java" 被取出
  }
}
~~~

## 6. 线程池

系统启动一个新线程的成本是比较高的，因为涉及与操作系统交互。使用线程池可以很好地提高性能，尤其是当程序需要创建大量生存期很短的线程时，更应该考虑线程池。

与数据库连接池相类似，**线程池在系统启动时即创建大量空闲的线程，程序将一个 Runnable 对象或 Callable 对象传给线程池，线程池就会启动一个线程来执行他们的 run() 或 call() 方法，当 run() 或 call() 方法执行结束后，该线程不会死亡，而是再次返回线程池中成为空闲状态。**

### 6.1 四种常见的线程池

1.  Executors.newCacheThreadPool()：**可缓存线程池，先查看池中有没有以前建立的线程，如果有，就直接使用。如果没有，就建一个新的线程加入池中，缓存型池子通常用于执行一些生存期很短的异步型任务；**
2.  **Executors.newFixedThreadPool(int n)：**创建一个可重用的、固定线程数的线程池；
3.   **Executors.newSingleThreadExecutor()：**创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
4.   **Executors.newScheduledThreadPool(int corePoolSize)：**创建一个指定线程数的线程池，它可以在指定延迟时间间隔后再执行线程任务；

>   上面的**前三个方法返回一个 ExecutorService 对象**，该对象代表一个线程池，可以执行 Runnable 对象或 Callable 对象所代表的线程；而**第 4 个方法返回一个 ScheduledExecutorService 对象**，它是 ExecutorService 对象的子类，可以在指定延迟后执行线程任务。

前三个方法的代码例子：

~~~java
public class ThreadPoolTest {
    public static void main(String[] args) {
        //创建一个可重用的、数量为6的线程池
        ExecutorService pool = Executors.newFixedThreadPool(6);
        // 创建一个可缓存线程池
//        ExecutorService pool = Executors.newCachedThreadPool();
        //创建一个只有单线程的线程池
//        ExecutorService pool = Executors.newSingleThreadExecutor();

        // Runnable 对象
        Runnable target = () -> {
            for (int i = 0; i < 21; i++) {
                System.out.println(Thread.currentThread().getName()+" " + i);
            }
        };

        // ExecutorService对象的 submit() 方法接收一个 Runnable 对象交给线程池，执行其 run() 方法；
        // 也可以接收 Callable 对象，执行其 call() 方法
        pool.submit(target);
        pool.submit(target);

        // 关闭线程池。会在全部线程任务执行完成后再关闭
        pool.shutdown();
    }
}
~~~

newScheduledThreadPool(int corePoolSize) 方法的代码例子：

~~~java
public class ThreadPoolTest {
    public static void main(String[] args) {
        //创建一个数量为3的 ScheduledExecutorService 线程池
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(3);

      	// Runnable 对象
        Runnable target = () -> {
            for (int i = 0; i < 21; i++) {
                System.out.println(Thread.currentThread().getName()+" " + i);
            }
        };

        // ScheduledExecutorService 对象的 schedule() 方法可接收 Runnable 或 Callable 对象
        // 并指定延迟时间。以下代码会在 2 秒后再执行所以线程
        pool.schedule(target,1, TimeUnit.SECONDS);
      	pool.schedule(target,1, TimeUnit.SECONDS);

        // 关闭线程池。会在全部线程任务执行完成后再关闭
        pool.shutdown();
    }
}
~~~

