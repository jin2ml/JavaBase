
# 一、 并发与并行

## 1. 什么是并发？

同一个时间间隔（某个时间段内）有多个事件发生

## 2. 什么是并行？

同一个时刻有多个事件发生

## 3. 并发与并行的区别

+ 举个栗子：

> 你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。</br></br>
你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。  （不一定是同时的）</br></br>
你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。

+ 并发不一定是并行，并行一定是并发；
+ 并发编程的目标是充分利用处理器的每一个核，以达到最高的处理性能

# 二、 线程

## 1. 线程的实现

**1. 继承`Thread`类创建线程**

`Thread`类本质上是实现了`Runnable`接口的一个实例，代表一个线程的实例。启动线程的唯一方法就是通过`Thread`类的`start()`实例方法。`start()`方法是一个`native`方法，它将启动一个新线程，并执行`run()`方法。这种方式实现多线程很简单，通过自己的类直接继承`Thread`，并复写`run()`方法，就可以启动新线程并执行自己定义的`run()`方法。例如：

```java
public class MyThread extends Thread{

    @Override
    public void run(){
        System.out.println("MyThread.run()");
    }
}

Mythread thread1 = new MyThread();
Mythread thread2 = new MyThread();
thread1.start();
thread2.start();
```

**2. 实现`Runnable`接口创建线程**

如果自己的类已经继承了其他类，就无法继承`Thread`类了，这时可以实现一个Runnable接口，例如：

```java
public class MyThread extends OtherClass implements Runnable {

    @Override
    public void run(){
        System.out.println("Mythread.run()");
    }
}
```

为了启动MyThread，首先要实例化一个Thread，并传入自己的实现了Runnable接口的Thread实例：

```java
MyThread myThread = new MyThread();
Thread thread = new Thread(myThread);
thread.start();
```

事实上，当传入一个Runnable target参数给Thread后，Thread的run()方法就会调用target.run()，参考JDK源码：

```java
public void run(){
    if(target != null){
        target.run();
    }
}
```

**3. 实现`Callable`接口通过`FutureTask`包装器来创建`Thread`线程**

Callable接口只有一个方法，定义如下：

```java
public interface Callable<V> {
    V call() throws Exception;
}

public class SomeCallable<V> extends OtherClass implements Callable<V> {
    @Override
    public V call() throws Exception {
        return null;
    }
}

Callable<Integer> callable = new SomeCallable<Integer>();
//由Callable<Integer>创建一个FutureTask<Integer>对象：
FutureTask<Integer> oneTask = new FutureTask<Integer>(callable);
//FutureTask<Integer>是一个包装器，它通过接受Callable<Integer>来创建，它同时实现了Future和Runnable接口。
//由FutureTask<Integer>创建一个Thread对象：
Thread thread = new Thread(oneTask);
thread.start();
```

**4.使用`ExecutorService`、`Callable`、`Future`实现有返回结果的进程**

`ExecutorService`、`Callable`、`Future`三个接口实际上都是属于`Executor`框架。返回结果的线程是在`JDK1.5`中引入的新特征，有了这种特征就不需要再为了得到返回值而大飞周折了。而且自己实现了也有可能漏洞百出。

可返回值的任务必须实现`Callable`接口。同样，无返回值的任务必须实现`Runnable`接口。

执行`Callable`任务后，可以获取一个Future的对象，在该对象上调用`get`接可以获取到`Callable`任务返回的`Object`了。

>注意，`get`方法是阻塞的，即：线程无返回结果，`get`方法会一直等待。

再结合线程池接口`ExecutorServic`e就可以实现传说中有返回结果的多线程了。

```java
import java.util.concurrent.*;
import java.util.Date;
import java.util.List;
import java.util.ArrayList;

/**
* 有返回值的线程
*/  
@SuppressWarnings("unchecked")
public class Test {  
    public static void main(String[] args) throws ExecutionException,  InterruptedException {

    System.out.println("----程序开始运行----");  
    Date date1 = new Date();  

    int taskSize = 5;  
    // 创建一个线程池  
    ExecutorService pool = Executors.newFixedThreadPool(taskSize);  
    // 创建多个有返回值的任务  
    List<Future> list = new ArrayList<Future>();  
    for (int i = 0; i < taskSize; i++) {  
        Callable c = new MyCallable(i + " ");  
        // 执行任务并获取Future对象  
        Future f = pool.submit(c);  
        // System.out.println(">>>" + f.get().toString());  
        list.add(f);  
    }  
    // 关闭线程池  
    pool.shutdown();  

    // 获取所有并发任务的运行结果  
    for (Future f : list) {  
        // 从Future对象上获取任务的返回值，并输出到控制台  
        System.out.println(">>>" + f.get().toString());  
    }  

    Date date2 = new Date();  
    System.out.println("----程序结束运行----，程序运行时间【"  
        + (date2.getTime() - date1.getTime()) + "毫秒】");  
    }  
}  
  
class MyCallable implements Callable<Object> {  
    private String taskNum;  
  
    MyCallable(String taskNum) {  
    this.taskNum = taskNum;  
    }  
  
    public Object call() throws Exception {  
        System.out.println(">>>" + taskNum + "任务启动");  
        Date dateTmp1 = new Date();  
        Thread.sleep(1000);  
        Date dateTmp2 = new Date();  
        long time = dateTmp2.getTime() - dateTmp1.getTime();  
        System.out.println(">>>" + taskNum + "任务终止");  
        return taskNum + "任务返回运行结果,当前任务时间【" + time + "毫秒】";  
    }
}
```

## 2. 线程的状态

![线程状态示意图](<https://images2015.cnblogs.com/blog/716271/201703/716271-20170320112245721-1831918220.jpg> '线程状态示意图')

说明：线程共包括以下5种状态。

1. 新建状态(New)         : 线程对象被创建后，就进入了新建状态。例如，`Thread thread = new Thread()`。
2. 就绪状态(Runnable): 也被称为“可执行状态”。线程对象被创建后，其它线程调用了该对象的start()方法，从而来启动该线程。例如，`thread.start()`。处于就绪状态的线程，随时可能被CPU调度执行。
3. 运行状态(Running) : 线程获取CPU权限进行执行。需要注意的是，线程只能从就绪状态进入到运行状态。
4. 阻塞状态(Blocked)  : 阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：

    (01) 等待阻塞 -- 通过调用线程的`wait()`方法，让线程等待某工作的完成。

    (02) 同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。

    (03) 其他阻塞 -- 通过调用线程的`sleep()`或`join()`或发出了I/O请求时，线程会进入到阻塞状态。当`sleep()`状态超时、`join()`等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。
5. 死亡状态(Dead)    : 线程执行完了或者因异常退出了`run()`方法，该线程结束生命周期。

## 3. 优先级

每个线程都有优先级，优先级的高低只和线程获得执行机会的次数多少有关。

并非线程优先级越高的就一定先执行，哪个线程的先运行取决于CPU的调度;

默认情况下`main`线程具有普通的优先级，而它创建的线程也具有普通优先级。

`Thread`对象的`setPriority(int x)`和`getPriority()`来设置和获得优先级。

`MAX_PRIORITY` :值是10

`MIN_PRIORITY` :值是1

`NORM_PRIORITY` :值是5(主方法默认优先级)

注意：每个线程默认的优先级都与创建他的父线程的优先级相同，在在默认的情况下，`main`线程具有普通优先级，由`main`线程创建的子线程也具有普通优先级。另外如果该线程已经属于一个线程组`ThreadGroup`，该线程的优先级不能超过该线程组的优先级。

## 4. 线程调度

线程调度一般指的是系统为线程分配处理器使用权的过程，这个过程会产生上下文切换，即操作系统的CPU利用时间片轮转的方式给每个任务分配一定的执行时间，然后把当前任务状态保存下来，接着加载下一任务的状态并执行，它是一个状态保存与加载的过程。

一般线程调度模式分为两种——抢占式调度和协同式调度。

抢占式调度指的是每条线程执行的时间、线程的切换都由系统控制，线程的切换不由线程本身决定，系统控制指的是在系统某种运行机制下，可能每条线程都分同样的执行时间片，也可能是某些线程执行的时间片较长，甚至某些线程得不到执行的时间片。在这种机制下，一个线程的堵塞不会导致整个进程堵塞。

协同式调度指某一线程执行完后主动通知系统切换到另一线程上执行，线程的执行时间由线程本身控制，这种模式就像接力赛一样，一个人跑完自己的路程就把接力棒交接给下一个人，下个人继续往下跑。线程的执行时间由线程本身控制，线程切换可以预知，不存在多线程同步问题，但它有一个致命弱点：如果一个线程编写有问题，运行到一半就一直堵塞，那么可能导致整个系统崩溃。

Java使用的是哪种线程调度模式？此问题涉及到JVM的实现，JVM规范中规定每个线程都有优先级，且优先级越高越优先执行，但优先级高并不代表能独自占用执行时间片，可能是优先级高得到越多的执行时间片，反之，优先级低的分到的执行时间少但不会分配不到执行时间。JVM的规范没有严格地给调度策略定义，一般Java使用的线程调度是抢占式调度，在JVM中体现为让可运行池中优先级高的线程拥有CPU使用权，如果可运行池中线程优先级一样则随机选择线程，但要注意的是实际上一个绝对时间点只有一个线程在运行（这里是相对于一个CPU来说），直到此线程进入非可运行状态或另一个具有更高优先级的线程进入可运行线程池，才会使之让出CPU的使用权。

## 5. 创建线程的多种方式

见1

## 6. 守护线程

所谓守护线程是指在程序运行的时候在后台提供一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者，并且这种线程并不属于程序中不可或缺的部分。因 此，当所有的非守护线程结束时，程序也就终止了，同时会杀死进程中的所有守护线程。反过来说，只要任何非守护线程还在运行，程序就不会终止。

守护线程和用户线程的没啥本质的区别：唯一的不同之处就在于虚拟机的离开：如果用户线程已经全部退出运行了，只剩下守护线程存在了，虚拟机也就退出了。 因为没有了被守护者，守护线程也就没有工作可做了，也就没有继续运行程序的必要了。

将线程转换为守护线程可以通过调用`Thread`对象的`setDaemon(true)`方法来实现。在使用守护线程时需要注意一下几点：

(1) `thread.setDaemon(true)`必须在`thread.start()`之前设置，否则会跑出一个`IllegalThreadStateException`异常。你不能把正在运行的常规线程设置为守护线程。

(2) 在`Daemon`线程中产生的新线程也是`Daemon`的。

(3) 守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候甚至在一个操作的中间发生中断。

## 7. 线程与进程的区别和关系

一个程序至少有一个进程,一个进程至少有一个线程。

进程就是一个应用程序在处理机上的一次执行过程，它是一个动态的概念，而线程是进程中的一部分，进程包含多个线程在运行。

1. 一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程。线程是操作系统可识别的最小执行和调度单位。

2. 资源分配给进程，同一进程的所有线程共享该进程的所有资源。 同一进程中的多个线程共享代码段(代码和常量)，数据段(全局变量和静态变量)，扩展段(堆存储)。但是每个线程拥有自己的栈段，栈段又叫运行时段，用来存放所有局部变量和临时变量。

3. 处理机分给线程，即真正在处理机上运行的是线程。

4. 线程在执行过程中，需要协作同步。不同进程的线程间要利用消息通信的办法实现同步。

# 三、 线程池

## 1. 自己设计线程池

自定义线程池的整个工作过程：

1. 初始化线程池，指定线程池的大小。

2. 向线程池中放入任务执行。

3. 如果线程池中创建的线程数目未到指定大小，则创建我们自定义的线程类放入线程池集合，并执行任务。执行完了后该线程会一直监听队列

4. 如果线程池中创建的线程数目已满，则将任务放入缓冲任务队列

5. 线程池中所有创建的线程，都会一直从缓存任务队列中取任务，取到任务马上执行

```java
public class MyThreadPool{
    /**
    * 初始化线程数
    */
    private int initThreadNumbers;
    /**
    * 工作线程数
    */
    private int workThreadNumbers;
    /**
    * 工作中线程
    */
    private ArrayList<Runnable> workThreads;
    /**
    * 暂存线程
    */
    private BlockingQueue<Runnable> tempStoredThreads;

    private Lock mainLock = new ReentrantLock();

    public MyThreadPool(int threads){
        this.initThreadNumbers = threads;
        this.workThreads = new ArrayList(threads)<>;
        this.tempStoredThreads = new ArrayBlockingQueue(threads * 4)<>;
        this.workThreadNumbers = 0;
    }

    public void execute(Runnable work){
        try{
            mainLock.lock();
            if (workThreadNumbers < initThreadNumbers{
                MyThread thread = new MyThread(work);
                thread.start();
                workThreads.add(thread);
                workThreadNumbers++;
            } else {
                if(!tempStoredThreads.offer(work)){
                    refuseWork();
                }
            }
        } finally {
            mainLock.unlock();
        }
    }

    private void refuseWork(){
        System.out.println("线程数量已经超出允许值！");
    }

    class MyThread extends Thread {
        private Runnable task;

        public MyThread(Runnable runnable) {
            this.task = runnable;
        }

        @Override
        public void run(){
            for(;;){
                if(task != null){
                    task.run();
                    task = null;
                } else {
                    Runnable runnable = tempStoredThreads.poll();
                    if(runnable != null){
                        runnable.run();
                    }
                }
            }
        }
    }
}
```

## 2. `submit()`和`execute()`

1. 接收参数不一样，`submit()`有返回值

```java
void execute(Runnable runnable);
Future<?> submit(Callable<T> task);
Future<?> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

2. 线程池`submit()`方法本质上还是调用`execute()`方法，所以在不需要返回值的情况下最好使用`execute()`方法。

3. 线程池`submit()`方便`Exception`处理

如果你再一个线程当中有异常，并且希望外部调用者感知到，那么可以使用`submit()`方法，通过捕获Future.get()抛出异常。

```java
public class RunnableTestMain {

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        /**
         * execute(Runnable x) 没有返回值。可以执行任务，但无法判断任务是否成功完成。
         */
        pool.execute(new RunnableTest("Task1")); 

        /**
         * submit(Runnable x) 返回一个future。可以用这个future来判断任务是否成功完成。请看下面：
         */
        Future future = pool.submit(new RunnableTest("Task2"));

        try {
            if(future.get()==null){//如果Future's get返回null，任务完成
                System.out.println("任务完成");
            }
        } catch (InterruptedException e) {
        } catch (ExecutionException e) {
            //否则我们可以看看任务失败的原因是什么
            System.out.println(e.getCause().getMessage());
        }

    }

}

public class RunnableTest implements Runnable {

    private String taskName;

    public RunnableTest(final String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println("Inside "+taskName);
        throw new RuntimeException("RuntimeException from inside " + taskName);
    }

}
```

## 3. 线程池原理

1. 线程池的优点

+ 线程是稀缺资源，使用线程池可以减少创建和销毁线程的次数，每个工作线程都可以重复使用。

+ 可以根据系统的承受能力，调整线程池中工作线程的数量，防止因为消耗过多内存导致服务器崩溃。

2. 线程池的创建

```java
public ThreadPoolExecutor(int corePoolSize,
                               int maximumPoolSize,
                               long keepAliveTime,
                               TimeUnit unit,
                               BlockingQueue<Runnable> workQueue,
                               RejectedExecutionHandler handler)

```

`corePoolSize`：线程池核心线程数量

`maximumPoolSize`:线程池最大线程数量

`keepAliverTime`：当活跃线程数大于核心线程数时，空闲的多余线程最大存活时间

`unit`：存活时间的单位

`workQueue`：存放任务的队列

`handler`：超出线程范围和队列容量的任务的处理程序

3. 线程池实现原理

+ 判断线程池里的`核心线程`是否都在执行任务，如果不是（核心线程空闲或者还有核心线程没有创建）则创建一个新的工作线程来执行任务，否则进入下一个流程。

+ 判断线程池工作队列是否已满，如果不是则将新提交的任务放入工作队列，否则进入下一个流程。

+ 判断线程池里的线程数量是否达到最大线程数，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

4. RejectedExecutionHandler：饱和策略

当队列和线程池都满了，说明线程池处于饱和状态，那么必须对新提交的任务采用一种特殊的策略来进行处理。这个策略默认配置是AbortPolicy，表示无法处理新的任务而抛出异常。JAVA提供了4种策略：

+ AbortPolicy：直接抛出异常

+ CallerRunsPolicy：只用调用所在的线程运行任务

+ DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。

+ DiscardPolicy：不处理，丢弃掉。

## 4. 为什么不允许使用Executors创建线程池

原因如下：

1. 阿里巴巴Java开发手册中明确指出，不允许使用Executors创建线程池：
![不允许使用Executors创建线程池](<https://www.hollischuang.com/wp-content/uploads/2018/10/15406254121131.jpg> '不允许使用Executors创建线程池')

原因如下：

+ java中`BlockingQueue`主要有两种实现，分别是`ArrayBlockingQueue`和`LinkedBlockingQueue`。`ArrayBlockingQueue`是用数组实现的有界阻塞队列，必须设置容量。而`LinkedBlockingQueue`是一个用链表实现的有界阻塞队列，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为`Integer.MAX_VALUE`。

+ 查看new SingleExecutor时的源码可以发现，在创建LinkedBlockingQueue时，并未指定容量。此时，LinkedBlockingQueue就是一个无边界队列，对于一个无边界队列来说，是可以不断的向队列中加入任务的，这种情况下就有可能因为任务过多而导致内存溢出的问题。

2. 创建线程池的正确方法

避免使用`Executors`创建线程池，主要是避免使用其中的默认实现，那么我们可以自己直接调用`ThreadPoolExecutor`的构造函数自己创建线程池。在创建的同时，给`BlockQueue`指定容量就可以了。

```java
private static ExecutorService executor = new ThreadPoolExecutor(10, 10,
    60L, TimeUnit.SECONDS,
    new ArrayBlockingQueue(10));
```

# 四、 线程安全

## 1. 死锁

死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，`若无外力作用，它们都将无法推进下去。`此时称系统处于死锁状态或系统产生了死锁，这些永远在`互相等待`的进程称为死锁进程。

进程在运行过程中，请求和释放资源的顺序不当，也同样会导致死锁。例如，并发进程 P1、P2分别保持了资源R1、R2，而进程P1申请资源R2，进程P2申请资源R1时，两者都会因为所需资源被占用而阻塞。

Java中死锁最简单的情况是，一个线程T1持有锁L1并且申请获得锁L2，而另一个线程T2持有锁L2并且申请获得锁L1，因为默认的锁申请操作都是阻塞的，所以线程T1和T2永远被阻塞了。导致了死锁。这是最容易理解也是最简单的死锁的形式。但是实际环境中的死锁往往比这个复杂的多。可能会有多个线程形成了一个死锁的环路，比如：线程T1持有锁L1并且申请获得锁L2，而线程T2持有锁L2并且申请获得锁L3，而线程T3持有锁L3并且申请获得锁L1，这样导致了一个锁依赖的环路：T1依赖T2的锁L2，T2依赖T3的锁L3，而T3依赖T1的锁L1。从而导致了死锁。

从上面两个例子中，我们可以得出结论，产生死锁可能性的最根本原因是：线程在获得一个锁L1的情况下再去申请另外一个锁L2，也就是锁L1想要包含了锁L2，也就是说在获得了锁L1，并且没有释放锁L1的情况下，又去申请获得锁L2，这个是产生死锁的最根本原因。另一个原因是默认的锁申请操作是阻塞的。

死锁产生的必要条件

1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。

2. 不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放（只能是主动释放)。

3. 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。

4. 循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有，如图1所示。

![不允许使用Executors创建线程池](<https://images2017.cnblogs.com/blog/249993/201801/249993-20180119114001287-1936540249.jpg> '不允许使用Executors创建线程池')

下面再来通俗的解释一下死锁发生时的条件：

1. 互斥条件：一个资源每次只能被一个进程使用。独木桥每次只能通过一个人。

2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。乙不退出桥面，甲也不退出桥面。

3. 不剥夺条件: 进程已获得的资源，在未使用完之前，不能强行剥夺。甲不能强制乙退出桥面，乙也不能强制甲退出桥面。

4. 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。如果乙不退出桥面，甲不能通过，甲不退出桥面，乙不能通过。

死锁例子：

```java
package com.demo.test;

/**
 * 一个简单的死锁类
 * t1先运行，这个时候flag==true,先锁定obj1,然后睡眠1秒钟
 * 而t1在睡眠的时候，另一个线程t2启动，flag==false,先锁定obj2,然后也睡眠1秒钟
 * t1睡眠结束后需要锁定obj2才能继续执行，而此时obj2已被t2锁定
 * t2睡眠结束后需要锁定obj1才能继续执行，而此时obj1已被t1锁定
 * t1、t2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。 
 */
public class DeadLock implements Runnable{

    private static Object obj1 = new Object();
    private static Object obj2 = new Object();
    private boolean flag;

    public DeadLock(boolean flag){
        this.flag = flag;
    }

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName() + "运行");

        if(flag){
            synchronized(obj1){
                System.out.println(Thread.currentThread().getName() + "已经锁住obj1");
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }
                synchronized(obj2){
                    // 执行不到这里
                    System.out.println("1秒钟后，"+Thread.currentThread().getName() + "锁住obj2");
                }
            }
        }else{
            synchronized(obj2){
                System.out.println(Thread.currentThread().getName() + "已经锁住obj2");
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }
                synchronized(obj1){
                    // 执行不到这里
                    System.out.println("1秒钟后，"+Thread.currentThread().getName() + "锁住obj1");
                }
            }
        }
    }
}
```

```java
package com.demo.test;

public class DeadLockTest {

     public static void main(String[] args) {
         Thread t1 = new Thread(new DeadLock(true), "线程1");
         Thread t2 = new Thread(new DeadLock(false), "线程2");

         t1.start();
         t2.start();
    }
}
```

运行结果：

>线程1运行

>线程1已经锁住obj1

>线程2运行

>线程2已经锁住obj2

线程1锁住了obj1（甲占有桥的一部分资源），线程2锁住了obj2（乙占有桥的一部分资源），线程1企图锁住obj2（甲让乙退出桥面，乙不从），进入阻塞，线程2企图锁住obj1（乙让甲退出桥面，甲不从），进入阻塞，死锁了。

从这个例子也可以反映出，死锁是因为多线程访问共享资源，由于访问的顺序不当所造成的，通常是一个线程锁定了一个资源A，而又想去锁定资源B；在另一个线程中，锁定了资源B，而又想去锁定资源A以完成自身的操作，两个线程都想得到对方的资源，而不愿释放自己的资源，造成两个线程都在等待，而无法执行的情况。

如何避免死锁：

+ 加锁顺序（线程按照一定的顺序加锁）

避免嵌套封锁：这是死锁最主要的原因的，如果你已经有一个资源了就要避免封锁另一个资源。如果你运行时只有一个对象封锁，那是几乎不可能出现一个死锁局面的。

再举个生活中的例子，比如银行转账的场景下，我们必须同时获得两个账户上的锁，才能进行操作，两个锁的申请必须发生交叉。这时我们也可以打破死锁的那个闭环，在涉及到要同时申请两个锁的方法中，总是以相同的顺序来申请锁，比如总是先申请 id 大的账户上的锁 ，然后再申请 id 小的账户上的锁，这样就无法形成导致死锁的那个闭环。

```java
public class Account {
    private int id;    // 主键
    private String name;
    private double balance;

    public void transfer(Account from, Account to, double money){
        if(from.getId() > to.getId()){
            synchronized(from){
                synchronized(to){
                    // transfer
                }
            }
        }else{
            synchronized(to){
                synchronized(from){
                    // transfer
                }
            }
        }
    }

    public int getId() {
        return id;
    }
}
```

这样的话，即使发生了两个账户比如 id=1的和id=100的两个账户相互转账，因为不管是哪个线程先获得了id=100上的锁，另外一个线程都不会去获得id=1上的锁(因为他没有获得id=100上的锁)，只能是哪个线程先获得id=100上的锁，哪个线程就先进行转账。这里除了使用id之外，如果没有类似id这样的属性可以比较，那么也可以使用对象的hashCode()的值来进行比较。

此例中只有先获取到大的id的锁才能继续去获取小的锁，所以不会出现未获取到大id的情况下就获取到小id的锁的情况。即获取锁的顺序是先大id后小id。

+ 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）

另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行(加锁超时后可以先继续运行干点其它事情，再回头来重复之前加锁的逻辑)。

需要注意的是，由于存在锁的超时，所以我们不能认为这种场景就一定是出现了死锁。也可能是因为获得了锁的线程（导致其它线程超时）需要很长的时间去完成它的任务。此外，如果有非常多的线程同一时间去竞争同一批资源，就算有超时和回退机制，还是可能会导致这些线程重复地尝试但却始终得不到锁。如果只有两个线程，并且重试的超时时间设定为0到500毫秒之间，这种现象可能不会发生，但是如果是10个或20个线程情况就不同了。因为这些线程等待相等的重试时间的概率就高的多（或者非常接近以至于会出现问题）。(超时和重试机制是为了避免在同一时间出现的竞争，但是当线程很多时，其中两个或多个线程的超时时间一样或者接近的可能性就会很大，因此就算出现竞争而导致超时后，由于超时时间一样，它们又会同时开始重试，导致新一轮的竞争，带来了新的问题。)

+ 死锁检测

死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

每当一个线程获得了锁，会在线程和锁相关的数据结构中（map、graph等等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。当一个线程请求锁失败时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程A请求锁7，但是锁7这个时候被线程B持有，这时线程A就可以检查一下线程B是否已经请求了线程A当前所持有的锁。如果线程B确实有这样的请求，那么就是发生了死锁（线程A拥有锁1，请求锁7；线程B拥有锁7，请求锁1）。

当然，死锁一般要比两个线程互相持有对方的锁这种情况要复杂的多。线程A等待线程B，线程B等待线程C，线程C等待线程D，线程D又在等待线程A。线程A为了检测死锁，它需要递进地检测所有被B请求的锁。从线程B所请求的锁开始，线程A找到了线程C，然后又找到了线程D，发现线程D请求的锁被线程A自己持有着。这是它就知道发生了死锁。

下面是一幅关于四个线程（A,B,C和D）之间锁占有和请求的关系图。像这样的数据结构就可以被用来检测死锁。

![不允许使用Executors创建线程池](<https://images2017.cnblogs.com/blog/249993/201801/249993-20180119161940131-89848320.png> '不允许使用Executors创建线程池')

那么当检测出死锁时，这些线程该做些什么呢？

一个可行的做法是释放所有锁，回退，并且等待一段随机的时间后重试。这个和简单的加锁超时类似，不一样的是只有死锁已经发生了才回退，而不会是因为加锁的请求超时了。虽然有回退和等待，但是如果有大量的线程竞争同一批锁，它们还是会重复地死锁（原因同超时类似，不能从根本上减轻竞争）。

一个更好的方案是给这些线程设置优先级，让一个（或几个）线程回退，剩下的线程就像没发生死锁一样继续保持着它们需要的锁。如果赋予这些线程的优先级是固定不变的，同一批线程总是会拥有更高的优先级。为避免这个问题，可以在死锁发生的时候设置随机的优先级。

总结：避免死锁的方式

1. 让程序每次至多只能获得一个锁。当然，在多线程环境下，这种情况通常并不现实。

2. 设计时考虑清楚锁的顺序，尽量减少嵌在的加锁交互数量。

3. 既然死锁的产生是两个线程无限等待对方持有的锁，那么只要等待时间有个上限不就好了。当然synchronized不具备这个功能，但是我们可以使用Lock类中的tryLock方法去尝试获取锁，这个方法可以指定一个超时时限，在等待超过该时限之后便会返回一个失败信息。

我们可以使用ReentrantLock.tryLock()方法，在一个循环中，如果tryLock()返回失败，那么就释放以及获得的锁，并睡眠一小段时间。这样就打破了死锁的闭环。比如：线程T1持有锁L1并且申请获得锁L2，而线程T2持有锁L2并且申请获得锁L3，而线程T3持有锁L3并且申请获得锁L1。此时如果T3申请锁L1失败，那么T3释放锁L3，并进行睡眠，那么T2就可以获得L3了，然后T2执行完之后释放L2, L3，所以T1也可以获得L2了执行完然后释放锁L1, L2，然后T3睡眠醒来，也可以获得L1, L3了。打破了死锁的闭环。

## 2. 死锁如何排查

## 3. 线程安全和内存模型的关系

# 五、 锁

## 1. CAS

## 2. 乐观锁与悲观锁

## 3. 数据库相关锁机制

## 4. 分布式锁

## 5. 偏向锁

## 6. 轻量级锁

## 7. 重量级锁

## 8. monitor

## 9. 锁优化

### 9.1. 锁消除

### 9.2. 锁粗化

### 9.3. 自旋锁

### 9.4. 可重入锁

## 10. 阻塞锁

# 六、 死锁

## 1. 死锁的原因

## 2. 死锁的解决办法

# 七、 synchronized

## 1. synchronized是如何实现的

## 2. synchronized和lock之间的关系

## 3. 不用synchronized如何实现一个线程安全的单例

## 4. synchronized和原子性，可见性，和有序性之间的关系

# 八、 volatile

## 1. happens-before

## 2. 内存屏障

## 3. 编译器指令重排和CPU指令重排

## 4. volatile的实现原理

## 5. volatile和原子性，可见性，和有序性之间的关系

## 6. 有了synchronized为什么还需要volatile

# 九、 并发包

## 1. Thread

## 2. Runnable

## 3. callable

## 4. ReentrantLock

## 5. ReentrantReadWriteLock

## 6. Atom*

## 7. Semaphore

## 8. CountDownLatch

## 9. ConcurrentHashMap

## 10. Executors

# 十、 其他

## 1. sleep和wait

## 2. wait和notify

## 3. notify和notifyAll

## 4. ThreadLocal

## 5. 写一个死锁的程序

```java
package com.demo.test;

/**
 * 一个简单的死锁类
 * t1先运行，这个时候flag==true,先锁定obj1,然后睡眠1秒钟
 * 而t1在睡眠的时候，另一个线程t2启动，flag==false,先锁定obj2,然后也睡眠1秒钟
 * t1睡眠结束后需要锁定obj2才能继续执行，而此时obj2已被t2锁定
 * t2睡眠结束后需要锁定obj1才能继续执行，而此时obj1已被t1锁定
 * t1、t2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。 
 */
public class DeadLock implements Runnable{

    private static Object obj1 = new Object();
    private static Object obj2 = new Object();
    private boolean flag;

    public DeadLock(boolean flag){
        this.flag = flag;
    }

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName() + "运行");

        if(flag){
            synchronized(obj1){
                System.out.println(Thread.currentThread().getName() + "已经锁住obj1");
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }
                synchronized(obj2){
                    // 执行不到这里
                    System.out.println("1秒钟后，"+Thread.currentThread().getName() + "锁住obj2");
                }
            }
        }else{
            synchronized(obj2){
                System.out.println(Thread.currentThread().getName() + "已经锁住obj2");
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }
                synchronized(obj1){
                    // 执行不到这里
                    System.out.println("1秒钟后，"+Thread.currentThread().getName() + "锁住obj1");
                }
            }
        }
    }
}
```

```java
package com.demo.test;

public class DeadLockTest {

     public static void main(String[] args) {
         Thread t1 = new Thread(new DeadLock(true), "线程1");
         Thread t2 = new Thread(new DeadLock(false), "线程2");

         t1.start();
         t2.start();
    }
}
```

运行结果：

线程1运行

线程1已经锁住obj1

线程2运行

线程2已经锁住obj2


## 6. 写代码解决生产者消费者问题
