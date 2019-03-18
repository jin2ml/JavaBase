
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

## 6. 守护线程

## 7. 线程与进程的区别和关系

# 三、 线程池

## 1. 自己设计线程池

## 2. submit()和execute()

## 3. 线程池原理

## 4. 为什么不允许使用Executors创建线程池

# 四、 线程安全

## 1. 死锁

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

## 6. 写代码解决生产者消费者问题
