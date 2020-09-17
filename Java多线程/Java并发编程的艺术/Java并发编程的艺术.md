---
typora-root-url: images
---

# Java并发编程的艺术

## 第一章 并发编程的挑战

### 1.1 上下文切换

即使是单核CPU也支持多线程执行代码，CPU通过给每个线程分配CPU时间片来实现这个机制，时间片式CPU分配给各个线程的时间，因为时间片非常短，所以CPU不停的切换线程执行,让我们感觉同时是多个线程时同时执行，

#### 	1.1.1 多线程一定快吗？

下面代码演示串行和并发执行并累加操作时间，下面代码并发执行一定比串行快吗？

```java
public class ConcurrencyTest {
    private static final long count = 100001;

    public static void main(String[] args) throws InterruptedException {
        concurrency();
        serial();
    }
    /**
    并发
    */
    private static void concurrency() throws InterruptedException {
        long start = System.currentTimeMillis();
         Thread t1 = new Thread(() ->{
             int a = 0;
             for(long i = 0; i < count; i++) {
                 a += 5;
             }
         },"t1");
         t1.start();
         int b = 0;
         for(long i = 0; i < count; i++) {
             b--;
         }
         t1.join();
         long time = System.currentTimeMillis() - start;
         System.out.println("concurrency :" + time + "ms,b=" + b);
    }
    /**
    串行
    */
    private static void serial() {
        long start = System.currentTimeMillis();
        int a = 0;
        for(long i = 0; i < count; i++) {
            a += 5;
        }
        int b = 0;
        for(long i= 0; i < count; i++) {
            b --;
        }
        long time = System.currentTimeMillis() - start;
        System.out.println("serial :" + time + "ms,b=" + b );
    }
}
```

##### 1-1测试结果

| 循环次数 | 串行执行耗时/ms | 并发执行耗时 | 并发比串行快多少 |
| -------- | --------------- | ------------ | ---------------- |
| 1亿      | 130             | 77           | 约一倍           |
| 1千万    | 18              | 9            | 约一倍           |
| 1百万    | 5               | 5            | 差不多           |
| 10万     | 4               | 3            | 差不多           |
| 1万      | 0               | 1            | 慢               |

#### 	1.1.2 测试上下文切换次数和时长

使用Lmbench3可以测试上下文切换市场

使用vmstat可以测量上下文切换次数

#### 	1.1.3 如何减少上下文切换

> 减少上下文切换的方法有无锁并发编程，CAS算法，使用最少线程和使用线程

- 无锁并发编程，多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以用一些方法来避免使用锁，如果将数据ID按照Hash算法取模分段，不同的线程处理不同段的数据
- CAS 算法 Java的 Atomic 包 使用CAS算法来更新数据，而不需要加锁
- 使用最少线程。避免创建不需要的线程，比如线程很少，创建了很多任务来处理，这样会造成大量线程都处于等待状态
- 协程 在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

#### 	1.1.4 减少上下文切换实战

### 1.2 死锁

锁是个非常有用的工具，运用场景就非常多，因为它使用起来非常简单，易于理解，同时他也带来一些困扰，那就是可能引起一些死锁，一旦产生死锁，就会造成系统功能不可用

```java
public class DeadLockDemo {
    private static String A = "A";
    private static String B = "B";

    public static void main(String[] args) {
        new DeadLockDemo().deadLock();
    }
    private void deadLock() {
         Thread t1 = new Thread(() ->{
             synchronized (A) {
                 try {
                     Thread.sleep(2000);
                 } catch (Exception e) {
                     e.printStackTrace();
                 }
                 synchronized (B) {
                     System.out.println("1");
                 }
             }
         },"t1");


        Thread t2 = new Thread(() ->{
            synchronized (B) {
                try {
                    Thread.sleep(2000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                synchronized (A) {
                    System.out.println("2");
                }
            }
        },"t2");
        t1.start();
        t2.start();
    }
}
```

> t1拿到锁后，因为一些异常情况没有释放锁(死循环)，又或者是t1拿到一个数据库锁，锁释放的时候抛出了异常，没释放掉

避免死锁的几个常见方法

- 避免一个线程同时获取多个锁
- 避免一个线程在锁的同时占用多个资源，尽量保证每个锁只占用一个资源
- 尝试使用定时锁，使用lock.tryLock(timeout) 来替代使用内部锁机制
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况

### 1.3 资源限制挑战

1)什么是资源限制

资源限制是指在进行并发编程时，程序的执行速度受限于计算机硬件资源或软件资源，

2）资源限制引发的问题

并发编程中，将代码执行速度加快的原则是代码中串行执行的部分变成并发执行，但是如果某段串行的代码执行，因为受限于资源，任然在串行中执行，这时候程序不仅会加快，反而会更慢，因为增加了上下文切换和资源调度的时间

3）如果解决资源限制的问题

对于硬件资源的限制，可以考虑使用集群并行执行程序，单机有资源限制，那就多机器运行

4）在资源限制的情况下进行并发编程

根据不同的资源限制调整程序的并发度，比如下载文件依赖两个资源，宽带和读写速度，有数据库的操作时，涉及到数据库连接数，如果SQL执行快，线程数量比数据库连接数大很多，则某系线程会阻塞，等待数据库连接

### 1.4本章小结

并发和串行

synchronized 









# 第四章 Java并发编程的基础

Java从诞生就明智的选择了内置对多线程的支持，这使得Java语言相比同一时期的其他语言具有明显的优势，线程作为操作系统中调度最小的单元，多个线程能够同时执行，这将显著的提升程序的性能，

## 4.1 线程简介

### 	4.1.1 什么是线程

现代操作系统在运行一个程序的时候，会为其创建一个线程，例如启动一个Java程序，操作系统就会创建一个Java进程，现代操作系统中调度最小单元是线程，也叫轻量级线程

实际上Java天生就是多线程程序

下列通过JMX来查看一个普通的 Java 程序 包含那些线程

```java
public class MultiThread {
    public static void main(String[] args) {
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println("[" + threadInfo.getThreadId() + "] " + 		          threadInfo.getThreadName() );
        }
    }
}
```

结果

```java
[6] Monitor Ctrl-Break
[5] Attach Listener
[4] Signal Dispatcher
[3] Finalizer
[2] Reference Handler
[1] main
```

一个Java 程序不仅仅是 main 方法的运行，而是main线程和多个其他的线程同时运行

### 	4.1.2 为什么要使用多线程

执行一个简单的Helloworld 却启动这么多无关的线程  是不是问题复杂化？

使用多线程主要以下几点

1、更多的处理器核心

2、更快的响应时间

例如 一笔订单的创建，它包含很多操作..

3、更好的编程模型

Java 为多线程提供良好、考究并且一致的编程模型

### 	4.1.3 线程优先级

现代操作系统是以时分片的形式来调度线程

在 Java 中使用一个整型成员变量 Priority 来控制优先级，优先级范围从1~10，默认5，优先级越高获得执行机会越多

```java
public class Priority {
    private static volatile boolean notStart = true;
    private static volatile boolean notEnd   = true;

    public static void main(String[] args) throws Exception {
        List<Job> jobs = new ArrayList<Job>();
        for (int i = 0; i < 10; i++) {
            int priority = i < 5 ? Thread.MIN_PRIORITY : Thread.MAX_PRIORITY;
            Job job = new Job(priority);
            jobs.add(job);
            Thread thread = new Thread(job, "Thread:" + i);
            thread.setPriority(priority);
            thread.start();
        }
        notStart = false;
        Thread.currentThread().setPriority(8);
        System.out.println("done.");
        TimeUnit.SECONDS.sleep(10);
        notEnd = false;

        for (Job job : jobs) {
            System.out.println("Job Priority : " + job.priority + ", Count : " + job.jobCount);
        }

    }

    static class Job implements Runnable {
        private int  priority;
        private long jobCount;

        public Job(int priority) {
            this.priority = priority;
        }

        public void run() {
            while (notStart) {
                Thread.yield();
            }
            while (notEnd) {
                Thread.yield();
                jobCount++;
            }
        }
    }
}
```

执行结果

```java
done.
Job Priority : 1, Count : 4591616
Job Priority : 1, Count : 4694474
Job Priority : 1, Count : 4528424
Job Priority : 1, Count : 4613296
Job Priority : 1, Count : 4682638
Job Priority : 10, Count : 4985583
Job Priority : 10, Count : 4983315
Job Priority : 10, Count : 5036855
Job Priority : 10, Count : 5092943
Job Priority : 10, Count : 5036526
```

线程优先级没有生效，优先级1和优先级10的Job计数 结果非常相近，没有明显差距，程序正确性不能依赖线程的优先级高低

### 	4.1.4  线程的状态

Java 线程在运行生命周期中有6中不同状态

| 状态名称     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| NEW          | 初始状态，线程被构建，但是没有调用start()方法                |
| RUNNABLE     | 运行状态，Java 线程将操作系统中就绪和运行两种状态统称为 “运行状态” |
| BLOCKED      | 阻塞状态，表示线程阻塞于锁                                   |
| WAITING      | 等待状态，表示线程进入等待状态，进入该状态表示，当前线程需要等到其他线程做出一些动作（打断/通知） |
| TIME_WAITING | 超时等待状态，该状态不同于 WAITING ，表示可以在指定时间内自行返回 |
| TERMINATED   | 终止状态，表示当前线程以及执行完毕                           |

使用 jstack ID

```java
"DestroyJavaVM" #18 prio=5 os_prio=0 tid=0x0000000003604000 nid=0x8df0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"SyncThread-2" #17 prio=5 os_prio=0 tid=0x000000001f094000 nid=0x93e8 waiting on condition [0x000000002034e000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076bb750b0> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
        at cn.gcq.chapter4.ThreadState$Sync.run(ThreadState.java:71)
        at java.lang.Thread.run(Thread.java:748)

"SyncThread-1" #16 prio=5 os_prio=0 tid=0x000000001f093800 nid=0x4ebc waiting on condition [0x000000002024e000]
   //超时等待
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at cn.gcq.chapter4.SleepUtils.second(SleepUtils.java:12)
        at cn.gcq.chapter4.ThreadState$Sync.run(ThreadState.java:73)
        at java.lang.Thread.run(Thread.java:748)

"BlockedThread-2" #15 prio=5 os_prio=0 tid=0x000000001f080800 nid=0x8990 waiting for monitor entry [0x000000002014f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at cn.gcq.chapter4.ThreadState$Blocked.run(ThreadState.java:61)
        - waiting to lock <0x000000076bb7ce40> (a java.lang.Class for cn.gcq.chapter4.ThreadState$Blocked)
        at java.lang.Thread.run(Thread.java:748)

"BlockedThread-1" #14 prio=5 os_prio=0 tid=0x000000001f080000 nid=0x8020 waiting on condition [0x000000002004e000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at cn.gcq.chapter4.SleepUtils.second(SleepUtils.java:12)
        at cn.gcq.chapter4.ThreadState$Blocked.run(ThreadState.java:61)
        - locked <0x000000076bb7ce40> (a java.lang.Class for cn.gcq.chapter4.ThreadState$Blocked)
        at java.lang.Thread.run(Thread.java:748)

"WaitingThread" #13 prio=5 os_prio=0 tid=0x000000001f07f000 nid=0x8308 in Object.wait() [0x000000001ff4e000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x000000076bb7a360> (a java.lang.Class for cn.gcq.chapter4.ThreadState$Waiting)
        at java.lang.Object.wait(Object.java:502)
        at cn.gcq.chapter4.ThreadState$Waiting.run(ThreadState.java:45)
        - locked <0x000000076bb7a360> (a java.lang.Class for cn.gcq.chapter4.ThreadState$Waiting)
        at java.lang.Thread.run(Thread.java:748)

"TimeWaitingThread" #12 prio=5 os_prio=0 tid=0x000000001f075000 nid=0x96e4 waiting on condition [0x000000001fe4f000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at cn.gcq.chapter4.SleepUtils.second(SleepUtils.java:12)
        at cn.gcq.chapter4.ThreadState$TimeWaiting.run(ThreadState.java:31)
        at java.lang.Thread.run(Thread.java:748)

```

![](/Snipaste_2020-09-15_14-21-21.png)

线程创建后，调用start()方法开始运行，当调用wait()方法后，线程进入等待状态，进入到等待状态的线程就需要依靠其他线程的通知或唤醒才能返回到运行状态，超时状态在等待状态上加入了时间的限制，，超时间后会自动进入到运行状态，当调用方法没有拿到锁后，线程就会进入到阻塞状态,线程执行run()方法完后，就会进入到终止状态

### 	4.1.5 Daemon 线程

Daemon线程是一个支持型的线程，主要被用作程序中后台调度以及支持性工作，当一个 Java 虚拟机不存在非Daemon线程，Java虚拟机就会自动退出，

```java
public class Daemon {
    public static void main(String[] args) {
        Thread thread = new Thread(new DaemonRunner(),"DaemonRunner");
        thread.setDaemon(true);
        thread.start();
    }
    static class DaemonRunner implements Runnable {

        @Override
        public void run() {
           try { TimeUnit.SECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); } finally {
               System.out.println("DaemonThread finally run.");
           }
        }
    }
}
```

Java虚拟机以及没有非Daemon线程，虚拟机就要退出，Java 虚拟机中所有Daemon线程都会终止，因此程序立即终止

## 4.2 启动和终止线程

### 	4.2.1 构造线程

在运行线程前，需要构造一个线程对象，线程对象在构造时候需要提供线程所属的线程组，线程优先级，是否是Daemon线程等信息

Java.lang.Thread 中对线程初始化部分

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;
		// 当前线程就是该线程的父线程
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
    	// 给daemon、priority 属性设置为父线程对应的属性
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
    	// 将父线程的 InheritableThradPool 赋值过来
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;
		// 分配一个线程 ID
        /* Set thread ID */
        tid = nextThreadID();
    }
```



### 	4.2.2 启动线程

> 调用 start() 方法启动线程，start() 含义是 当前线程(即parent线程) 同步告知 Java虚拟机，只要有线程规划 ,应立即启动调用 start() 线程

### 	4.2.3 理解中断

中断可以理解为线程的一个标识位属性，他表示一个运行中的线程是否被其他线程进行了中断操作，中断好比其他线程对该线程打了个招呼，调用线程的interrupt()对其中断操作

isInterrupted()  判断线程是否被中断

```java
public class Interupted {
    public static void main(String[] args) {
        Thread sleepThread = new Thread(new SleepRunner(),"SleepThread");
        sleepThread.setDaemon(true);
        Thread busyThread = new Thread(new BusyRunner(),"BusyThread");
        busyThread.setDaemon(true);
        sleepThread.start();
        busyThread.start();
        // 休眠5秒
        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
        sleepThread.interrupt();
        busyThread.interrupt();
        System.out.println("SleepThread interrupted is " + sleepThread.isInterrupted());
        System.out.println("BusyThread interrupted is " + busyThread.isInterrupted());
        //防止sleep和busy一同退出
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
    }
    static class SleepRunner implements Runnable {

        @Override
        public void run() {
            while (true) {
                try { TimeUnit.SECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
            }
        }
    }
    static class BusyRunner implements Runnable {

        @Override
        public void run() {
            while (true) {

            }
        }
    }
}
```

结果

```java
SleepThread interrupted is false
BusyThread interrupted is true
java.lang.InterruptedEntxception: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at cn.gcq.chapter4.Interupted$SleepRunner.run(Interupted.java:31)
	at java.lang.Thread.run(Thread.java:748)
```

抛出InterruptedEntxception被清除，忙碌工作的BusyThread 没有被清除

### 	4.2.4 过期的suspend、resume、stop

```java
public class Deprecated {
    @SuppressWarnings("deprecation")
    public static void main(String[] args) throws Exception {
        DateFormat format = new SimpleDateFormat("HH:mm:ss");
        Thread printThread = new Thread(new Runner(), "PrintThread");
        printThread.setDaemon(true);
        printThread.start();
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行暂停，输出内容工作停止
        printThread.suspend();
        System.out.println("main suspend PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(20);
        // 将PrintThread进行恢复，输出内容继续
        printThread.resume();
        System.out.println("main resume PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行终止，输出内容停止
        printThread.stop();
        System.out.println("main stop PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
    }

    static class Runner implements Runnable {
        @Override
        public void run() {
            DateFormat format = new SimpleDateFormat("HH:mm:ss");
            while (true) {
                System.out.println(Thread.currentThread().getName() + " Run at " + format.format(new Date()));
                SleepUtils.second(1);
            }
        }
    }
}

```

结果

```java
PrintThread Run at 13:37:25
PrintThread Run at 13:37:26
PrintThread Run at 13:37:27
main suspend PrintThread at 13:37:28
main resume PrintThread at 13:37:48
PrintThread Run at 13:37:48
PrintThread Run at 13:37:49
PrintThread Run at 13:37:50
main stop PrintThread at 13:37:51
```

stop() 终止工作

suspend() 暂停

resume() 恢复

均已过时 已有等待/通知取代

### 4.2.5 安全的终止线程

```java
public class Shutdown {
    public static void main(String[] args) throws Exception {
        Runner one = new Runner();
        Thread countThread = new Thread(one, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();
        Runner two = new Runner();
        countThread = new Thread(two, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        two.cancel();
    }

    private static class Runner implements Runnable {
        private long             i;

        private volatile boolean on = true;

        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()) {
                i++;
            }
            System.out.println("Count i = " + i);
        }

        public void cancel() {
            on = false;
        }
    }
}

```

结果

```java
Count i = 614240524
Count i = 661158663
```

通过标识符中断操作的方式，能够使线程在终止前有机会去清理资源，而不是武断的停止线程，显得更加优雅

## 4.3 线程间的通信

### 	4.3.1 voliatile 和 synchronized 关键字

Java 支持多个线程同时访问一个对象或者对象的成员变量，关键字 volatitle 用来修饰字段，告知程序任何对变量的访问均需要从共享内存中获取，而且对他的改变必须同步刷新回共享内存,他能保证所有线程对变量的访问可见性

关键字synchronized 可以修饰方法或以同步代码块的形式来使用，主要确保多个线程在同一时刻，只能有一个线程处于方法或同步代码块中，保证了线程对变量访问的可见性和排他性

```java
public class SynchronizedTest {
    public static void main(String[] args) {
        // 对synchronized进行加锁
        synchronized (SynchronizedTest.class) {
            
        }
        //静态同步方法 对synchronized class对象 加锁
        m(); 
    }
    public static synchronized void m () {
        
    }
}
```

同步代码块实现使用了 monitorenter 和 monitorexit 指令，同步方法则是依靠方法修饰符上的 ACC_SYNCHRONIZED 来完成

![](/Snipaste_2020-09-16_13-51-16.png)

任意线程对Object（Object由synchronized保护）的访问，首先要获得 

Object的监视器。如果获取失败，线程进入同步队列，线程状态变为BLOCKED。当访问Object 

的前驱（获得了锁的线程）释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新 

尝试对监视器的获取

### 	4.3.2 等待/通知机制

一个线程修改一个对象的值，而另一个线程感知到了变化，然后进行相应的操作，整个过程中开始于一个线程，而最终又是执行另一个线程，前者生产者，后者消费者

| 方法名称       | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| notify()       | 通知一个在对象上等待的线程，使其从 wait() 方法中返回，而返回的线程前提是获得了这个对象的锁 |
| notifyAll()    | 通知所有在等待的线程                                         |
| wait()         | 调用该方法进入WAITING 状态，只有等待另外线程的通知或被中断才会返回，需要注意，调用wait()方法后，会释放对象的锁 |
| wait（long）   | 超时等待一段时间，参数毫秒，等待n长毫秒，没有通知就超时返回  |
| wait(long,int) | 对于超时时间更细粒度的控制，可以达到纳秒                     |

```java
public class WaitNotify {
    static boolean flag = true;
    static Object  lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);

        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait @ "
                                + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }
}
```

结果

```java
Thread[WaitThread,5,main] flag is true. wait @ 13:30:18
Thread[NotifyThread,5,main] hold lock. notify @ 13:30:19
Thread[NotifyThread,5,main] hold lock again. sleep @ 13:30:24
Thread[WaitThread,5,main] flag is false. running @ 13:30:29
```

![](/Snipaste_2020-09-17_13-32-00.png)

WaitThread首先获取了对象的锁，然后调用对象的wait()方法，从而放弃了锁 

并进入了对象的等待队列WaitQueue中，进入等待状态。由于WaitThread释放了对象的锁，NotifyThread随后获取了对象的锁，并调用对象的notify()方法，将WaitThread从WaitQueue移到 SynchronizedQueue中，此时WaitThread的状态变为阻塞状态。NotifyThread释放了锁之后， WaitThread再次获取到锁并从wait()方法返回继续执行

### 	4.3.3 等待/通知经典范式

等待方(消费者) 通知方(生产者)

等待规则如下

1）获取对象的锁

2）如果条件不满足，那么调用对象的 wait() 方法，被通知后仍需要检查条件

3）条件满足则执行对应逻辑

```java
synchronized(对象) {
    while(条件不满足){
        对象.wait();
    }
}
```

通知方遵循如下原则

```java
synchronized(对象){
    改变条件
    对象.notifyAll();
}
```



### 	4.3.4 管道输入/输出流

主要用于线程之间数据传递

具体四种实现类 PipedOutputStream,PipedInputStream,PipedReader,PipedWriter

```java
public class Piped {

    public static void main(String[] args) throws Exception {
        PipedWriter out = new PipedWriter();
        PipedReader in = new PipedReader();
        // 将输出流和输入流进行连接，否则在使用时会抛出IOException
        out.connect(in);

        Thread printThread = new Thread(new Print(in), "PrintThread");
        printThread.start();
        int receive = 0;
        try {
            while ((receive = System.in.read()) != -1) {
                out.write(receive);
            }
        } finally {
            out.close();
        }
    }

    static class Print implements Runnable {
        private PipedReader in;

        public Print(PipedReader in) {
            this.in = in;
        }

        public void run() {
            int receive = 0;
            try {
                while ((receive = in.read()) != -1) {
                    System.out.print((char) receive);
                }
            } catch (IOException ex){
            }
        }
    }
}
```

```java
123
123
123
123
```



### 	4.3.5 Thread.join

当一个线程 A 执行了 thread.join() 含义是 当前线程等待 thread线程终止之后才从thread.join()中返回，

```java
public class Join {
    public static void main(String[] args) {
        Thread previous = Thread.currentThread();
        for(int i = 0; i<10; i++) {
            // 每一个线程拥有前一个线程的引用，需要等待前一个线程的终止 才能从等待中返回
            Thread thread = new Thread(new Domino(previous),String.valueOf(i));
            thread.start();
            previous = thread;
        }
        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println(Thread.currentThread().getName() + " terminate");
    }
    static class Domino implements Runnable {
        private Thread thread;
        public Domino(Thread thread) {
            this.thread = thread;
        }
        @Override
        public void run() {
            try {
                thread.join();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " terminate");
        }
    }
}
```

每个线程终止的前提是前驱线程的终止，每个线程等待前驱线程 终止后，才从join()方法返回，这里涉及了等待/通知机制（等待前驱线程结束，接收前驱线程结束通知）。 



### 	4.3.6 ThreadLocal使用

```java
public class Profiler {
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>() {
        @Override
        protected Long initialValue() {
            return System.currentTimeMillis();
        }
    };
    public static final void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }
    public static final long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }
    public static void main(String[] args) throws InterruptedException {
        Profiler.begin();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("cost: " + Profiler.end() + "mills");
    }
}
```

方法入口前执行begin 方法调用后end

## 4.4 线程应用实例

```java
public synchronized Object get(long mills) throws InterruptedException {
        long future = System.currentTimeMillis() + mills;
        long remaining = mills; // 当超时大于0并且result返回值不满足要求 
        while ((result == null) && remaining > 0) {
            wait(remaining);
            remaining = future - System.currentTimeMillis();
        }
        return result;
    }
```



### 	4.4.1 等待超时模式

### 	4.4.2 一个简单的数据库连接实例

### 	4.4.3 线程池技术以及实例

### 	4.4.4 一个基于线程池技术的简单Web服务器

## 4.5 本章小结