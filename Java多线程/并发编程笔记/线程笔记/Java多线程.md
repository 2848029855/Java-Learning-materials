---
 typora-root-url: images


---

# 2.1进程与线程

### 进程

> - 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至CPU,数据加载至内存，在指令运行过程中还需要用到磁盘，网络等设备，进程就是用来记载指令，管理内存，管理IO的
> - 当一个程序运行，从磁盘加载这个程序的代码到内存，这时候就开启了一个进程
> - 进程就可以视为程序的一个实例，大部分程序可以同时运行多个实例(列如 记事本，画图)也有的程序只能启动一个实例进程(网易云音乐)



### 线程

> - 一个进程之内可以分为一到多个线程
> - 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给CPU执行
> - Java中，线程作为最小调度单位，进程作为资源分配的最小单位，在windows中进程是不活动的，只是作为线程的容器



### 二者对比

> - 进程基本上是相互独立的，而线程存于进程内，是进程的一个子集
> - 进程拥有共享的资源，如内存空间等，供其内部的线程共享
> - 进程的通信比较复杂
>   - ​	同一台计算机的进程通信称之为IPC（Inter process communication）
>   - ​	不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，列入HTTP
> - 线程通信相对简单，因为他们共享进程内的内存，一个列子是多个线程可以访问同一个共享变量
> - 线程更轻量，线程上下文切换成本比一般要比进程上下文切换低



# 2.2 并行和并发

单核 CPU 下，线程实际上还是串行执行的 操作系统中有一个叫任务调度器，将 CPU 的时间片(windows下时间最小为15毫秒) 分给不同的线程使用，只是由于 CPU 在线程间 （时间很短） 的切换非常快，人类感觉是同时运行的，总结一句话是 微观是串行，宏观并行

一般会将这种线程轮流使用 CPU 的做法叫做并发 concurrent

![](/image-20200822141709585.png)

多核 cpu 下 每个核心 core 都可以调度运行线程，这时候线程是可以并行的

![](/Snipaste_2020-08-22_14-22-24.png)

引用 Rob Pike 的一段描述:

并发 (current) 是同一时间应对 (dealing with) 多件事情的能力

并行 (parallel) 是同一时间动手做 (doing) 多件事情的能力



##### 例子

- 家庭主妇做饭、打扫卫生、给孩子喂奶、她一个人轮流交替做多见事情，这时就是并发
- 家庭主妇雇了个保姆、她们一起做这些事、这时有并发也有并行(这时会产生竞争，列如锅只有一口，一个人用锅，另一个人就要等待)
- 雇了三个保姆，一个专做饭，一个专打扫卫生，一个专喂奶，互不干扰，这时是并行



# 2.3 应用

应用值提高效率

从方法调用角度来讲，如果

需要等待结果的返回，才能继续运行就是同步

不需要等待结果返回，就能继续运行的就是异步
注意：同步在多线程还有另外一层意思，是让多个线程步调一致

### 1）设计

多线程可以让方法变为异步（即不要巴巴干等着） 比如说读取磁盘文件时，假设读取花费了5秒钟，如果没有线程调度机制，这5秒钟调用者什么都不做不了，其代码都得暂停

### 2）结论

比如在项目中，视频文件需要转换格式等操作时，这时候开一个新线程处理视频转换，避免阻塞线程

tomcat 的异步 servlet 也是类似目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作运行

ui程序中，开线程进行其他操作，避免阻塞 ui 线程



应用之提高效率

充分利用多核 cpu 的优势，提高运行效率，想象下面的场景，执行三个计算，最后将计算结果汇总

```
计算 1 花费 10ms

计算 2 花费 11ms

计算 3 花费 9ms

汇总需要1ms
```

如果是串行执行，那么总共需要的的时间是 10+ 11 + 9 + 1 = 31ms

但是如果是四核 cpu，各个核心分别使用线程1执行计算1，线程2执行计算2 线程3执行计算3，那么三个线程是并行的，花费时间取决与最长的那个线程运行的时间，即11ms最后加上汇总时间只会花费12ms

注意

需要在多核 cpu 下才能提高效率，单核任然是轮流执行

1）设计

2）结论







# 3.Java线程

### 3.1创建和运行线程

#### 方法一：直接使用Thread

```java
   new Thread(() -> {
          log.debug("--running");
      },"input thread name").start();
```

#### 方法二:使用Runnable

```java
Runnable runnable = new Runnable() {
          @Override
          public void run() {
      
          }
 };
```

#### 方法三： FutureTask 配置Thread

get:返回当前线程执行的结果

```java
FutureTask<Integer> integerFutureTask = new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                log.debug("running...");
                Thread.sleep(2000);
                return 100;
            }
        });
        new Thread(integerFutureTask,"t2").start();
        //get 返回结果
        log.debug("{}",integerFutureTask.get());
```

### 3.2查看进程线程方法

#### windows

> 任务管理器可以查看进程和线程数，也可以用来杀死进程
>
> `tasklist` 查看进程
>
> `taskkill`杀死进程



#### linux

> - `ps -fe` 查看所有线程
> - `ps -fT -p` <PID> 查看某个进程(PID) 的所有线程
> - `kill` 杀死线程
> - `top` 按大写H切换是否线程线程
> - `top -H -p <PID>` 查看某个进程(PID) 的所有线程



#### Java

`jps` 查看所有Java线程

`jstack <PID> `查看某个Java进程(PID)的所有线程状态

`jconsole` 来查看某个Java进程中线程的运行情况(图形界面)

`jconsole` 远程监控配置

需要如下方式运行Java类

```java
java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote -
Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 -
Dcom.sun.management.jmxremote.authenticate=是否认证 java类
```

> - 修改 /etc/hosts 文件将 127.0.0.1 映射至主机名
> - 如果要认证访问，还需要做如下步骤
> - 复制 jmxremote.password 文件
> - 修改 jmxremote.password 和 jmxremote.access 文件的权限为 600 即文件所有者可读写
> - 连接时填入 controlRole（用户名），R&D（密码）



### 3.4原理之线程运行

#### 栈与栈帧

Java Virtual Machine Stacks (Java虚拟机中栈)

我们都知道JVM由堆、栈、方法区组成，其中栈内存是给谁用的，其实就是线程，每个线程启动后，虚拟机就会为它分配一块占内存

- 每个栈由多个栈帧(Frame)组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一次活动栈帧，对应着当前正在执行的那个方法



#### 线程上下文切换 (Thread Context Switch)

因为以下一些原因导致 cpu 不再执行当前线程，转而执行另一个线程的代码

- 线程的 cpu 时间片用完
- 垃圾回收
- 有更高优先级线程需要运行
- 线程自己调用了sleep，yield、wait、join、park、synchronized、lock 等方法

当Context Switch 发生时，需要由操作系统，保持当前线程状态，并恢复另一个线程的状态，Java中对应的概念就是程序计数器，(Program Conunter Register) 他的作用时记住下一条jvm指令的执行地址， 是线程私有的

- 状态包括程序计数器 虚拟机栈中每个栈帧的信息，如局部变量，操作数栈，返回地址等
- Context Switch 频繁发生会影响性能

### 3.5常见方法



| 方法名           | static | 功能说明                                      |                             注意                             |
| ---------------- | ------ | --------------------------------------------- | :----------------------------------------------------------: |
| start()          |        | 启动一个新线程，在新的线程运行run方法中的代码 | start方法只是让线程进入就绪，里面的代码不一定立刻运行(CPU时间片还没有分给他)。每个线程对象的start方法只能调用一次，多次调用会出现IllegalThreadStateExceoption |
| run()            |        | 新线程启动后会调用的方法                      | 如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会调用 Runnable 中的 run 方法，否则默 认不执行任何操作。但可以创建 Thread 的子类对象，来覆盖默认行为 |
| join()           |        | 等待线程结束                                  |                                                              |
| join(long n)     |        | 等待线程运行结束，最多等待n毫秒               |                                                              |
| getId()          |        | 获取线程长整型id                              |                            id唯一                            |
| getName()        |        | 获取线程名字                                  |                                                              |
| setName(String)  |        | 设置线程名字                                  |                                                              |
| getPriority()    |        | 获取线程优先级                                |                             1-10                             |
| setPriority(int) |        | 修改线程优先级                                | java中规定线程优先级是1~10 的整数，较大的优先级能提高该线程被 CPU 调度的机率 |
| getState()       |        | 获取线程状态                                  | Java 中线程状态是用 6 个 enum 表示，分别为：NEW,RUNNABLE,   BLOCKED,WAITING, TIMED_WAITING,TERMINATED |
| isInterrupted()  |        | 判断是否被打断                                |                      不会清楚 打断标记                       |
| isAlive()        |        | 线程是否存货(还没有运行完毕)                  |                                                              |
| interrupt()      |        | 打断线程                                      | 如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，并清除 打断标记 ；如果打断的正在运行的线程，则会设置 打断标记 ；park 的线程被打断，也会设置 打断标记 |
| interrupted()    | static | 判断当前线程是否被打断                        |                        会清楚打断标记                        |
| currentThread()  | static | 获取当前正在执行的线程                        |                                                              |
| sleep(long n)    | static | 让当前线程休眠n毫秒，休眠时让出cpu给其他线程  |                                                              |
| yield()          | static | 提示线程调度器，让出当前线程堆CPU的调用       |                     主要是为了测试和调试                     |



### 3.6 sleep和yield

#### **Sleep**

> - 调用 sleep 会让当前线程从 *Running* 进入 *Timed Waiting* 状态（阻塞）
> - 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
> - 睡眠结束后的线程未必会立刻得到执行
> - 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性

#### **yield**

> - 调用 yield 会让当前线程从 *Running* 进入 *Runnable* 就绪状态，然后调度执行其它线程
>
> - 具体的实现依赖于操作系统的任务调度器



线程优先级

线程优先级会提示(hint) 调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它，

如果cpu比较忙，那么优先级高的线程会获得更多的时间片，但cpu闲的时候，优先级几乎没用



3.7join方法详解

为什么需要join

```java
public class Test2 {
    static int r = 0;
    public static void main(String[] args) throws InterruptedException {
        log.debug("开始");
        Thread t1 = new Thread(() -> {
            log.debug("线程开始");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.debug("结束");
            r = 10;
        },"A");
        t1.start();
        //等待t1线程结束
        t1.join();

        log.debug("{}",r);
        log.debug("结束");
    }
}
```

分析

- 因为主线程和线程t1是并行执行的，t1线程需要1秒之后才能算出r=10
- 而主线程一开始就要打印r的结果，所以只能打印初r = 0

解决方法

- 用sleep行不行 ? 为什么
- 用join() 加在t1.start() 之后执行



从调用方角度来讲

需要等待结果返回，才能继续运行就是同步

不需要等待返回结果返回，就能继续运行就是异步

![](/Snipaste_2020-08-23_11-16-01.png)

等待多个结果

问，下列代码cost大约需要多少秒

```java
   static int r1 = 0;
    static int r2 = 0;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1); //休眠1秒
                r1 = 10;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1");
        
        Thread t2 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2); //休眠两秒
                r1 = 20;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t2");
        
        t1.start();
        t2.start();
        
        long start = System.currentTimeMillis();
        log.debug("join begin");
        t1.join();//等待t1线程结束
        
        log.debug("t1 join end");
        t2.join();
        
        log.debug("t2 join end");
        long end = System.currentTimeMillis();
        log.debug("r1:{},r2:{},cost:{}",r1,r2,end-start);
    }
```

分析如下

第一个join 等待t1时，t2并没有停止，而在运行

第二个join 1s后执行到此 t2也运行了1s 因此也只需要等待1s

颠倒两个join呢？

![](/Snipaste_2020-08-23_11-21-01.png)



### 3.7 inteerrupt方法详解

打断sleep、wait 、join 的线程

这几个方法都会让线程进入阻塞状态

打断sleep的线程，会清空打断状态，以sleep为例子

```java
private static void test1() throws InnterruptedException {
    Thread t1 = new Thread(() ->{
        TimeUnit.SENCONDS.sleep(1);
    },"t1");
    t1.start();
    TimeUnit.SENCONDS.sleep(0.5);
    t1.interrupt();
    log.debug("打断状态:{}",t1.isInterrupted())
}
```

输出

```java
java.lang.InterruptedException: sleep interrupted
 at java.lang.Thread.sleep(Native Method)
 at java.lang.Thread.sleep(Thread.java:340)
 at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
 at cn.itcast.n2.util.Sleeper.sleep(Sleeper.java:8)
 at cn.itcast.n4.TestInterrupt.lambda$test1$3(TestInterrupt.java:59)
 at java.lang.Thread.run(Thread.java:745)
21:18:10.374 [main] c.TestInterrupt - 打断状态: false
```

打断正常运行的线程

```java
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while(true) {
                boolean interrupted = Thread.currentThread().isInterrupted();
                if (interrupted) {
                    log.debug("被打断，退出循环");
                    break;
                }
            }
        },"t1");
        t1.start();
        Thread.sleep(1000);
        log.debug("interrupt");
        t1.interrupt();
    }
```



不推荐使用的方法

| 方法名    | static | 功能说明           |
| --------- | ------ | ------------------ |
| stop()    |        | 停止线程运行       |
| suspend() |        | 挂起(暂停)线程运行 |
| resume()  |        | 恢复线程运行       |

### 3.8 主线程和守护线程

默认情况下，Java线程需要等待所有线程都运行结束，才会结束，有一种叫特殊线程叫守护线程，只要其他非守护线程运行结束，即使守护线程的代码没有执行完，也会强制结束

```java
  public static void main(String[] args) {
        Thread a = new Thread(() -> {
            while (true) {
                if (Thread.currentThread().isInterrupted()) {

                }
            }
        },"A");
        a.setDaemon(true);
        a.start();
        log.debug("end");
    }
```

```
11:39:44:722 [main] cn.itcast.test.Test7 - end
```

注意

垃圾回收线程就是一种守护线程

Tomcat中的Acceptor 和 Poller 线程就是守护线程，所以Tomcat接收到shutdown命令后，不会等待他们处理完请求

3.五种状态

从操作系统层面描述

![](/Snipaste_2020-08-23_11-45-51.png)

- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由 CPU 调度执行
- 【运行状态】指获取了 CPU 时间片运行中的状态
  - 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
  - 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入【阻塞状态】
  - 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑
  - 调度它们
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态



### 3.9六种状态

根据Java API 层面描述的

根据Thread.State枚举 分为六种状态

![](/Snipaste_2020-08-23_11-59-17.png)

- `NEW` 线程刚被创建，但是还没有调用 start()方法

- `RUNNABLE` 当调用了start()方法 注意，Java API 层面的`RUNNABLE` 状态覆盖 操作系统 层面的 可以运行状态，运行状态和阻塞状态， (由于BIO 导致线程阻塞，在Java里无法区分，任然认为可以运)

- `BOLCKED`、`WAITING`、`TIMED_WAITING`  都是Java API 层面堆[阻塞状态]的细分 后面会在状态转换一节详述

- `TERMINATED` 当线程代码运行结束

- 新建 、可运行（就绪）、阻塞、等待、死亡、定时等待

  代码：

```java
public class TestState {
    public static void main(String[] args) {
        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                log.debug("running..."); //new 新建
            }
        };

        Thread t2 = new Thread(() -> {
            while(true) { //runnable 运行状态

            }
        },"t2");
        t2.start();

        Thread t3 = new Thread("t3") {
            @Override
            public void run() {
                log.debug("running");//运行完后 terminated 结束
            }
        };
        t3.start();

        Thread t4 = new Thread("t4") {
            @Override
            public void run() {
                synchronized (TestState.class) {
                    try {
                        Thread.sleep(10000000); //time_waiting 定时等待
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t4.start();

        Thread t5 = new Thread("t5") {
            @Override
            public void run() {
                try {
                    t2.join();//Waiting 等待 一直等待
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        t5.start();

        Thread t6 =  new Thread() {
            @Override
            public void run() {
                synchronized (TestState.class) {
                    try {
                        Thread.sleep(1000000); //blocked 拿不到锁 阻塞状态
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t6.start();

        try {
            Thread.sleep(500);
        } catch (Exception e) {
            e.printStackTrace();
        }
        log.debug("t1 state {}",t1.getState());
        log.debug("t2 state {}",t2.getState());
        log.debug("t3 state {}",t3.getState());
        log.debug("t4 state {}",t4.getState());
        log.debug("t5 state {}",t5.getState());
        log.debug("t6 state {}",t6.getState());


    }
}
```

输入结果：

```java
12:12:03:882 [t3] cn.itcast.test.TestState - running
12:12:04:382 [main] cn.itcast.test.TestState - t1 state NEW
12:12:04:383 [main] cn.itcast.test.TestState - t2 state RUNNABLE
12:12:04:383 [main] cn.itcast.test.TestState - t3 state TERMINATED
12:12:04:383 [main] cn.itcast.test.TestState - t4 state TIMED_WAITING
12:12:04:383 [main] cn.itcast.test.TestState - t5 state WAITING
12:12:04:383 [main] cn.itcast.test.TestState - t6 state BLOCKED
```



### 3.10 习题

阅读华罗庚《统筹方法》，给出烧水泡茶的多线程解决方案，提示

- 参考图二，用两个线程（两个人协作）模拟烧水泡茶过程
  - 文中办法乙、丙都相当于任务串行
  - 而图一相当于启动了 4 个线程，有点浪费
- 用 sleep(n) 模拟洗茶壶、洗水壶等耗费的时间

附：华罗庚《统筹方法》



> 统筹方法，是一种安排工作进程的数学方法。它的实用范围极广泛，在企业管理和基本建设中，以及关系复
>
> 杂的科研项目的组织与管理中，都可以应用。
>
> 怎样应用呢？主要是把工序安排好
>
> 比如，想泡壶茶喝。当时的情况是：开水没有；水壶要洗，茶壶、茶杯要洗；火已生了，茶叶也有了。怎么
>
> 办？
>
> - 办法甲：洗好水壶，灌上凉水，放在火上；在等待水开的时间里，洗茶壶、洗茶杯、拿茶叶；等水开了，泡茶喝。
> - 办法乙：先做好一些准备工作，洗水壶，洗茶壶茶杯，拿茶叶；一切就绪，灌水烧水；坐待水开了，泡茶喝。
> - 办法丙：洗净水壶，灌上凉水，放在火上，坐待水开；水开了之后，急急忙忙找茶叶，洗茶壶茶杯，泡茶喝。
>
> 哪一种办法省时间？我们能一眼看出，第一种办法好，后两种办法都窝了工。
>
> 这是小事，但这是引子，可以引出生产管理等方面有用的方法来。
>
> 水壶不洗，不能烧开水，因而洗水壶是烧开水的前提。没开水、没茶叶、不洗茶壶茶杯，就不能泡茶，因而
>
> 这些又是泡茶的前提。它们的相互关系，可以用下边的箭头图来表示：

![](/Snipaste_2020-08-23_12-08-53.png)

> 从这个图上可以一眼看出，办法甲总共要16分钟（而办法乙、丙需要20分钟）。如果要缩短工时、提高工作
>
> 效率，应当主要抓烧开水这个环节，而不是抓拿茶叶等环节。同时，洗茶壶茶杯、拿茶叶总共不过4分钟，大
>
> 可利用“等水开”的时间来做。
>
> 
>
> 是的，这好像是废话，卑之无甚高论。有如走路要用两条腿走，吃饭要一口一口吃，这些道理谁都懂得。但
>
> 稍有变化，临事而迷的情况，常常是存在的。在近代工业的错综复杂的工艺过程中，往往就不是像泡茶喝这
>
> 么简单了。任务多了，几百几千，甚至有好几万个任务。关系多了，错综复杂，千头万绪，往往出现“万事俱
>
> 备，只欠东风”的情况。由于一两个零件没完成，耽误了一台复杂机器的出厂时间。或往往因为抓的不是关
>
> 键，连夜三班，急急忙忙，完成这一环节之后，还得等待旁的环节才能装配。
>
> 洗茶壶，洗茶杯，拿茶叶，或先或后，关系不大，而且同是一个人的活儿，因而可以合并成为

![](/Snipaste_2020-08-23_12-09-17.png)

> 看来这是“小题大做”，但在工作环节太多的时候，这样做就非常必要了。
>
> 这里讲的主要是时间方面的事，但在具体生产实践中，还有其他方面的许多事。这种方法虽然不一定能直接
>
> 解决所有问题，但是，我们利用这种方法来考虑问题，也是不无裨益的。

完整代码

```java
public class Test16 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() ->{
            log.debug("洗水壶");
            try {
                TimeUnit.SECONDS.sleep(1);
                log.debug("烧开水");
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"老王");

        Thread t2 = new Thread(() -> {
            log.debug("洗茶壶");
            try {
                TimeUnit.SECONDS.sleep(1);
                log.debug("洗茶杯");
                TimeUnit.SECONDS.sleep(2);
                log.debug("拿茶叶");
                TimeUnit.SECONDS.sleep(1);
                t1.join();//烧开水结束后
                log.debug("泡茶");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"小王");
        t1.start();
        t2.start();

    }
}
```



### 本章小结

重点在于掌握

- 线程创建
- 线程重要api 如 start run sleep join interrupt
- 线程状态
- 应用方面
  - 异步调用：主线程执行期间，其他线程异步执行耗时操作
  - 提高效率：并行计算，缩短运算时间
  - 同步等待：join
  - 统筹规划： 合理使用他线程 得到最优效果
- 原理方面：
  - 线程运行流程：栈、栈帧、上下文切换、程序计数器
  - Threa d 两种创建方式的源码
- 模式方面 
  - 终止模式之两阶段终止





# 4.共享模型之管程

#### 本章内容

- 共享问题
- synchronized
- 线程安全分析
- Monitor
- wait/notify
- 线程状态切换
- 活跃性
- Lock



### 4.1共享带来的问题

#### 小故事

- 老王（操作系统）有一个功能强大的算盘（CPU），现在想把它租出去，赚一点外快
- ![](/Snipaste_2020-08-24_08-18-05.png)
- 小南、小女（线程）来使用这个算盘来进行一些计算，并按照时间给老王支付费用
- 但小南不能一天24小时使用算盘，他经常要小憩一会（sleep），又或是去吃饭上厕所（阻塞 io 操作），有
- 时还需要一根烟，没烟时思路全无（wait）这些情况统称为（阻塞）
- ![](/Snipaste_2020-08-24_08-19-23.png)
- 在这些时候，算盘没利用起来（不能收钱了），老王觉得有点不划算
- 另外，小女也想用用算盘，如果总是小南占着算盘，让小女觉得不公平
- 于是，老王灵机一动，想了个办法 [ 让他们每人用一会，轮流使用算盘 ]
- 这样，当小南阻塞的时候，算盘可以分给小女使用，不会浪费，反之亦然
- 最近执行的计算比较复杂，需要存储一些中间结果，而学生们的脑容量（工作内存）不够，所以老王申请了
- 一个笔记本（主存），把一些中间结果先记在本上
- 计算流程是这样的
- ![](/Snipaste_2020-08-24_08-20-13.png)
- 但是由于分时系统，有一天还是发生了事故
- 小南刚读取了初始值 0 做了个 +1 运算，还没来得及写回结果
- 老王说 [ 小南，你的时间到了，该别人了，记住结果走吧 ]，于是小南念叨着 [ 结果是1，结果是1...] 不甘心地
- 到一边待着去了（上下文切换）
- 老王说 [ 小女，该你了 ]，小女看到了笔记本上还写着 0 做了一个 -1 运算，将结果 -1 写入笔记本
- 这时小女的时间也用完了，老王又叫醒了小南：[小南，把你上次的题目算完吧]，小南将他脑海中的结果 1 写
- 入了笔记本
- ![](/Snipaste_2020-08-24_08-20-22.png)
- 小南和小女都觉得自己没做错，但笔记本里的结果是 1 而不是 0



#### Java的体现

两个线程堆初始值为0的静态变量一个做自增，一个做自减，各做5000次，结果还是0吗？

```java
public class Test1 {

    static int counter = 0;
    static Object lock = new Object();
    
    @Test
    public void test1() throws InterruptedException {
        
        Room room = new Room();
        
        long begin = System.currentTimeMillis();
        Thread t1 = new Thread(() -> {
                for(int i = 0; i< 5000; i++) {
                    room.increment();
                }
        },"t1");
        Thread t2 = new Thread(() -> {
            for(int i = 0; i< 5000; i++) {
                room.decrement();
            }
        },"t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        long end = System.currentTimeMillis();
        log.debug("{},time={}",room.getCounter(),(end-begin));
    }

}
class Room {
    private int counter = 0;

    public void increment() {
        synchronized (this) {
            counter++;
        }
    }

    public void decrement() {
        synchronized (this) {
            counter--;
        }
    }

    public int getCounter() {
        synchronized (this) {
            return counter;
        }
    }
}
```

以上的结果可能是正数，负数 零 为什呢？ 因为对Java中的静态变量自增，自减并不是原子操作，要彻底理解，必须从字节码来进行分析

类如对于 `i++` 而言 (i 为静态变量)  实际会产生如下JVM字节码指令

```java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
iadd // 自增
putstatic i // 将修改后的值存入静态变量i
```

而对应 `i--` 

```java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
isub // 自减
putstatic i // 将修改后的值存入静态变量i
```

而Java的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换

![](/Snipaste_2020-08-24_08-28-47.png)



如果是单线程以上8行代码是顺序执行(不会交错) 没有问题

![](/Snipaste_2020-08-24_08-29-56.png)

但是多线程下这8行代码可能交错执行

出现负数的情况

![](/Snipaste_2020-08-24_08-30-42.png)

出现正数情况

![](/Snipaste_2020-08-24_08-32-29.png)



#### 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的
- 问题出在多个线程访问共享资源
  - 多个线程读共享资源其实也是没有问题的
  - 在多个线程对共享资源读写操作时发生指令交错，就会出现问题
- 一段代码内如果存在对共享资源的多线程读写操作，称这段代码为临界区

```java
static int counter = 0;

static void increment() 
// 临界区
{ 
 counter++; 
}

static void decrement() 
// 临界区
{ 
 counter--; 
}
```



#### 竞态条件 Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件

### 4.2 synchronized 解决方案

为了避免临界区的竞态条件的发生，有多种条件可以达到目的

- ​	阻塞式的解决方案：synchronized Lock
- ​	非阻塞式的解决方案 原子变量

 使用阻塞式的解决方案: synchronized 来解决上述问题 即俗称[对象锁] ，它采用互斥的方式让同一时刻至多只有一个线程能持有[对象锁],其它线程在想获取这个对象锁 就会阻塞住，这样就能保证拥有锁的线程可以安全的执行临界区的代码，不用担心线程上下文切换

> - 注意
> - 虽然Java中互斥和同步都可以采用 synchronized 关键字来完成 ，但是他们还是有区别的
> - 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区的代码
> - 同步时由于线程执行的先后，顺序不同，需要一个线程等待其他线程运行到某个点



synchronized

```
synchronized(对象) {
	临界区
}
```



![](/Snipaste_2020-08-24_08-47-36.png)



你可以这样做对比

- synchronized(对象) 中的对象，可以想象为一个房间（room），有唯一入口（门）房间只能一次进入一人
- 进行计算，线程 t1，t2 想象成两个人
- 当线程 t1 执行到 synchronized(room) 时就好比 t1 进入了这个房间，并锁住了门拿走了钥匙，在门内执行
- count++ 代码
- 这时候如果 t2 也运行到了 synchronized(room) 时，它发现门被锁住了，只能在门外等待，发生了上下文切
- 换，阻塞住了
- 这中间即使 t1 的 cpu 时间片不幸用完，被踢出了门外（不要错误理解为锁住了对象就能一直执行下去哦），
- 这时门还是锁住的，t1 仍拿着钥匙，t2 线程还在阻塞状态进不来，只有下次轮到 t1 自己再次获得时间片时才
- 能开门进入
- 当 t1 执行完 synchronized{} 块内的代码，这时候才会从 obj 房间出来并解开门上的锁，唤醒 t2 线程把钥
- 匙给他。t2 线程这时才可以进入 obj 房间，锁住了门拿上钥匙，执行它的 count-- 代码



用图来表示

![](/Snipaste_2020-08-24_08-50-02.png)

![](/Snipaste_2020-08-24_08-50-21.png)



#### 思考

synchronized 实际是用对象锁保证了临界区的代码的原子性，临界区的代码对外不可分割，不会被线程切换所打断

为了加深理解，思考下面问题

如果把 synchronized（obj) 放在for外面 如何理解 原子性

如果t1 synchronized (obj) 而t2 synchronized (obj2) 会怎样运作 锁对象

如果t1 synchronized (obj) 而t2没有加会怎么样 如何理解 锁对象



#### 面向对象改进

把需要保护的共享变量放入一个类

```java
class Room {
    private int counter = 0;

    public void increment() {
        synchronized (this) {
            counter++;
        }
    }

    public void decrement() {
        synchronized (this) {
            counter--;
        }
    }

    public int getCounter() {
        synchronized (this) {
            return counter;
        }
    }
}
```



### 4.3方法上的 synchronized

```java
class Test{
 	public synchronized void test() {
 
  	}
}
等价于
class Test{
 public void test() {
 	synchronized(this) {
 
 	}
 }
}
```

```java
class Test{
	public synchronized static void test() {
 
	}
}
等价于
class Test{
 public static void test() {
 	synchronized(Test.class) {
 
 	}
 }
}
```

不加 synchronized 的方法

不加 synchronized的方法就好比 不遵守规则的人，不老实去排队(好比翻窗户进去的)



#### 线程8锁

```java
class Phone
{
    public static  synchronized void senEmail()throws  Exception{
         try { TimeUnit.SECONDS.sleep(4);} catch (InterruptedException e){e.printStackTrace();}
        System.out.println("------sendEmail");
    }
    public synchronized void sendSMS()throws  Exception{
        System.out.println("------sendSMS");
    }
    public void hello(){
        System.out.println("-----hello");
    }
}

/**
 * @author Guo
 * @Create 2020/8/20 10:08
 * 题目 多线程8锁
 * 1、标准访问 请问先打印邮件还是短信? EM
 * 2、邮件方法暂停4秒，请问先打印邮件还是短信? EM
 * 3、新增一个普通方法 hello () 请先打印邮件还是hello? hello
 * 4、两部手机 请问先打印邮件还是短信? EM
 * 5、两个静态同步方法 同一部手机 请问先打印邮件还是短信? EM
 * 6、两个静态同步方法，2部手机 请问先打印邮件还是短信?
 * 7、一个普通同步方法，一个静态同步方法 1部手机 请问先打印邮件还是短信? sms
 * 8、一个普通同步方法，一个静态同步方法 2部手机 请问先打印邮件还是短信? sms
 *
 *
 * 1、2
 *  一个对象里面如果有多个synchronized方法，某一时刻内，只要有有一个线程去调用其中的某一个synchronized方法，
 *  其他的线程只能等待，换句话说 某一时刻内，只能有唯一的一个线程去访问这些synchronized方法
 *  锁的只是当前对象this，被锁定后 其他线程都不能进入到当前类对象的其他synchronized方法
 *
 * 3
 * 加个普通方法后发现和同步锁无关
 * 换成两个对象后，不是同一把锁，情况立马改变
 *
 * 都换成静态同步方法后，情况又变化
 * 所有的非静态头同步方法用的都是同一把锁 实例对象本身
 * new this 具体的一部手机
 * 静态 class唯一的一个模板
 *
 * synchronized 实现同步的基础： Java中每一个对象都可以作为锁
 * 具体表现为以下三种形式
 * 对于普通同步方法 锁是当前实例对象
 * 对于静态同步方法 锁是当前类的Class对象
 * 对于同步方法块，锁是synchronized括号里配置的对象

 *
 * 当一个线程视图访问同步代码块时，它首先得到锁，退出或抛出异常时，必须释放锁
 *
 * 也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放后才能获取锁，
 * 可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁
 * 所以无须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们的锁
 *
 * 所有的静态同步方法用的也是同一把锁-类对象本身
 * 这两把锁（this/class）是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞争条件的
 * 但是一旦一个静态同步方法获取到锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁
 * 而不管是同一个实例对象的静态同步方法之间
 * 还是不同的实例对象的静态同步方法之间，只要他们同一个类的实例对象
 *
 *
 */
public class Lock8 {
    public static void main(String[] args) throws InterruptedException {

        Phone phone = new Phone();
        Phone phone2 = new Phone();

        new Thread(() -> {
         try {
             phone.senEmail();
         } catch (Exception e) {
             e.printStackTrace();
         }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            try {
//                Phone.sendSMS();
                phone2.sendSMS();
//                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"B").start();
    }
}
```



### 4.4 变量的线程安全分析

#### 成员变量和静态变量是否线程安全？

- 如果他们没有共享，则线程安全
- 如果他们被共享了，根据他们的状态是否能改变，又分两种情况
  - 如果只有读操作，则线程安全
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全



#### 局部变量是否线程安全？

- 局部变量是线程安全的
- 但局部变量引用的对象未必
- 如果该对象没有逃离方法的作用访问，他是线程安全的
- 如果该对象逃离方法的作用范围，需要考虑线程安全

```java
public static void test1() {
	int i = 10;
    i++;
}
```

每个线程调用test1()方法时，局部变量i 会在每个线程的栈帧内存中被创建多份，因此不存在共享

```java
public static void test1();
	descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
 Code:
 stack=1, locals=1, args_size=0
 0: bipush 10
 2: istore_0
 3: iinc 0, 1
 6: return
 LineNumberTable:
 line 10: 0
 line 11: 3
 line 12: 6
 LocalVariableTable:
 Start Length Slot Name Signature
 3 4 0 i I
```

![](/Snipaste_2020-08-24_09-09-33.png)



成员变量例子

```java
class ThreadUnsafe {
 ArrayList<String> list = new ArrayList<>();
 public void method1(int loopNumber) {
 	for (int i = 0; i < loopNumber; i++) {
 	// { 临界区, 会产生竞态条件
 	method2();
 	method3();
        // } 临界区
 	}
 }
     private void method2() {
     	list.add("1");
     }
     private void method3() {
     	list.remove(0);
     }
    //执行
    static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;
    public static void main(String[] args) {
     	ThreadUnsafe test = new ThreadUnsafe();
     	for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                test.method1(LOOP_NUMBER);
            }, "Thread" + i).start();
     }
    }
}
```

其中一种情况是，如果线程2还未add 线程1 remove就会报错

```java
Exception in thread "Thread1" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0 
     at java.util.ArrayList.rangeCheck(ArrayList.java:657) 
     at java.util.ArrayList.remove(ArrayList.java:496) 
     at cn.itcast.n6.ThreadUnsafe.method3(TestThreadSafe.java:35) 
     at cn.itcast.n6.ThreadUnsafe.method1(TestThreadSafe.java:26) 
     at cn.itcast.n6.TestThreadSafe.lambda$main$0(TestThreadSafe.java:14) 
     at java.lang.Thread.run(Thread.java:748)
```

分析

- 无论那个线程中的method2 引用的都是同一个对象中的list成员变量
- method3与 method2分析相同

![](/Snipaste_2020-08-24_09-15-23.png)

将list修改为局部变量

```java
 public final void method1(int loopNumber) {
 	ArrayList<String> list = new ArrayList<>();
     for (int i = 0; i < loopNumber; i++) {
         method2(list);
         method3(list);
     }
 }
     private void method2(ArrayList<String> list) {
     list.add("1");
     }java
     private void method3(ArrayList<String> list) {
     list.remove(0);
 }
}
```



分析

- list是局部变量，每个线程调用时会创建其不同的实力，没有共享
- 而method2 的参数 时从method1 中传递过来 与method1引用同一对象
- method3的参数分析与method2相同

![](/Snipaste_2020-08-24_09-21-31.png)



方法修饰符带来的思考 如果把method2和method3的方法修改为public 会不会代理线程安全问题？

- 情况1：有其他线程调用method2 和method3
- 情况2：在情况一的基础上，为ThreadSate类添加子类，子类覆盖method2和method3方法即

```java
class ThreadSafe {
 public final void method1(int loopNumber) {
     ArrayList<String> list = new ArrayList<>();
     for (int i = 0; i < loopNumber; i++) {
         method2(list);
         method3(list);
     }
 }
 private void method2(ArrayList<String> list) {
 	list.add("1")
  }
 private void method3(ArrayList<String> list) {
 	list.remove(0);
 }
}
class ThreadSafeSubClass extends ThreadSafe{
     @Override
     public void method3(ArrayList<String> list) {
         new Thread(() -> {
         	list.remove(0);
         }).start();
     }
}
```



> 从这个例子可以看书 private 或 final 提供 安全 的意思所在 请体会开闭原则中的闭



常见线程安全类

String

Integer

StringBuffer

Random

Vector

Hashtable

Java.util.concurrent 包下类

这里说的线程安全是指，多个线程调用他们同一个实例或某个方法时，是线程安全，也可以理解为

```java
Hashtable table = new Hashtable();
    new Thread(()->{
        table.put("key", "value1");
    }).start();

    new Thread(()->{
        table.put("key", "value2");
    }).start();
```

他们每个方法是原子的

但注意他们多个方法的组合不是原子的，见后面分析



#### 线程安全类方法组合

分析下面代码是否线程安全

```java
Hashtable table = new Hashtable();
// 线程1，线程2
if( table.get("key") == null) {
 table.put("key", value);
}
```

![](/Snipaste_2020-08-24_09-29-47.png)

不可变类线程安全性

String ,Integer 等都是不可变类，因为其内部的状态不可以改变，因此他们的方法都是线程安全

有疑问 String有 replace,substring 等发给发都可以改变值 ，那么这些方法又是如何保证线程安全的



#### 案例分析

例1:

```java
public class MyServlet extends HttpServlet {
     // 是否安全？
     Map<String,Object> map = new HashMap<>();
     // 是否安全？
     String S1 = "...";
     // 是否安全？
     final String S2 = "...";
     // 是否安全？
     Date D1 = new Date();
     // 是否安全？
     final Date D2 = new Date();
 
 public void doGet(HttpServletRequest request, HttpServletResponse response) {
      // 使用上述变量
 }
}
```

例2:

```java
public class MyServlet extends HttpServlet {
     // 是否安全？
     private UserService userService = new UserServiceImpl();

     public void doGet(HttpServletRequest request, 	HttpServletResponse response) {
     userService.update(...);
     }
    }
    public class UserServiceImpl implements UserService {
     // 记录调用次数
     private int count = 0;

     public void update() {
     // ...
     count++;
     }
}
```

例3:

```java
@Aspect
@Component
public class MyAspect {
 // 是否安全？
 private long start = 0L;
 
 @Before("execution(* *(..))")
 public void before() {
 	start = System.nanoTime();
 }
 
 @After("execution(* *(..))")
 public void after() {
 	long end = System.nanoTime();
 	System.out.println("cost time:" + (end-start));
 }
}
```

例4:

```java
public class MyServlet extends HttpServlet {
 	// 是否安全
 	private UserService userService = new UserServiceImpl();
 
     public void doGet(HttpServletRequest request, HttpServletResponse response) {
        userService.update(...);
     }
}
    public class UserServiceImpl implements UserService {
     // 是否安全
     private UserDao userDao = new UserDaoImpl();
 
     public void update() {
        userDao.update();
     }
}
public class UserDaoImpl implements UserDao { 
 public void update() {
     String sql = "update user set password = ? where username = ?";
     // 是否安全
     try (Connection conn = DriverManager.getConnection("","","")){
     // ...
     } catch (Exception e) {
     // ...
     }
 }
}
```

例5:

```
public class MyServlet extends HttpServlet {
 // 是否安全
 	private UserService userService = new UserServiceImpl();
 
 	public void doGet(HttpServletRequest request, HttpServletResponse response) {
 	userService.update(...);
 }
}
public class UserServiceImpl implements UserService {
 // 是否安全
     private UserDao userDao = new UserDaoImpl();

     public void update() {
     	userDao.update();
     }
}
public class UserDaoImpl implements UserDao {
     // 是否安全
     private Connection conn = null;
     public void update() throws SQLException {
     String sql = "update user set password = ? where username = ?";
     conn = DriverManager.getConnection("","","");
     // ...
     conn.close();
     }
}
```

例6:

```java
public class MyServlet extends HttpServlet {
 // 是否安全
 private UserService userService = new UserServiceImpl();
 
     public void doGet(HttpServletRequest request, HttpServletResponse response) {
     userService.update(...);
     }
}
public class UserServiceImpl implements UserService { 
     public void update() {
         UserDao userDao = new UserDaoImpl();
         userDao.update();
     }
}
public class UserDaoImpl implements UserDao {
     // 是否安全
     private Connection = null;
     public void update() throws SQLException {
     String sql = "update user set password = ? where username = ?";
     conn = DriverManager.getConnection("","","");
     // ...
     conn.close();
     }
}
```

例7：

```java
public abstract class Test {
 
 public void bar() {
 	// 是否安全
 	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
 	foo(sdf);
  }
 
 public abstract foo(SimpleDateFormat sdf);
 
 
 public static void main(String[] args) {
 	new Test().bar();
 }
}
```

其中 foo 的行为是不确定的，可能导致不安全的发生，被称之为**外星方法**

```java
public void foo(SimpleDateFormat sdf) {
     String dateStr = "1999-10-11 00:00:00";
     for (int i = 0; i < 20; i++) {
     new Thread(() -> {
     try {
     sdf.parse(dateStr);
     } catch (ParseException e) {
     e.printStackTrace();
     }
     }).start();
     }
}
```

请比较JDK中String类的实现

```java
public String replace(char oldChar, char newChar) {
        if (oldChar != newChar) {
            int len = value.length;
            int i = -1;
            char[] val = value; /* avoid getfield opcode */

            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            if (i < len) {
                char buf[] = new char[len];
                for (int j = 0; j < i; j++) {
                    buf[j] = val[j];
                }
                while (i < len) {
                    char c = val[i];
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                //创建个新的String 赋值 并返回
                return new String(buf, true);
            }
        }
        return this;
    }
```



### 4.5习题

买票练习

```java
public class Test2 {
    public static void main(String[] args) throws InterruptedException {
        TicketWindow window = new TicketWindow(1000);
        //所有线程的集合
        List<Thread> threadList = new ArrayList<>();
        //卖出的票数统计
        List<Integer> amountList = new Vector<>();
        for(int i = 0; i< 2000; i++) {
            Thread thread = new Thread(() -> {
                //买票
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int amount = window.sell(randomAmout());
                //保存转账金额
                amountList.add(amount);
            });
            //保存线程组
            threadList.add(thread);
            thread.start();
        }
        for (Thread thread : threadList) {
            thread.join();
        }

        log.debug("余票:{}",window.getCount());
        log.debug("卖出的票数:{}",amountList.stream().mapToInt(i->i).sum());
    }
    //线程安全类
    static Random random = new Random();

    //随机1-5
    public static int randomAmout() {
        return random.nextInt(5) + 1;
    }
}
//售票窗口
class TicketWindow {
    private int count;

    public TicketWindow(int count) {
        this.count = count;
    }

    public int getCount() {
        return count;
    }

    //转账操作
    public synchronized int sell(int amount) {
        if (this.count >= amount) {
            this.count -= amount;
            return amount;
        } else {
            return 0;
        }
    }

}
```

### 4.6 Monitor概念

### 4.7 synchronized原理进阶

#### 1.轻量级锁

**轻量级锁使用场景：如果有一个对象虽然有多个线程要加锁，但加锁时间是错开的(也就是没有竞争者) 那么可以使用轻量锁来优化**

轻量级对使用是透明的 语法任然是`synchronized`

假设有两个方法同步代码块，利用同一个对象加锁

```java
static final Object obj = new Object();

    public static void method1() {
         synchronized( obj ) {
         	// 同步块 A
         	method2();
         }
    }

    public static void method2() {
         synchronized( obj ) {
         	// 同步块 B
         }
}
```

创建锁记录(Lock Record)对象，每个线程的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word(标记字)



![](/Snipaste_2020-08-24_11-32-56.png)

让锁记录中 Object reference 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word 将Mark Wod的值存入锁记录

![](/Snipaste_2020-08-24_11-38-41.png)

如果 cas 替换成功 ，对象头中存储了 锁记录地址和状态00 表示由该线程给对象加锁 这时图如下

![](/Snipaste_2020-08-24_11-39-44.png)

- 如果 cas 失败 有两种情况
  - 如果是其他线程已经持有了该 Object 的轻量锁，这时表明有竞争，进入锁膨胀过程
  - 如果是自己执行了 synchronized 锁重入 那么在添加一条Lock Record 作为重入的记录

![](/Snipaste_2020-08-24_11-41-31.png)

当退出 synchronized 代码块 解锁时 如果有取值为null的锁记录，表示重入，这时重置锁记录，表示重入计数减一

![](/Snipaste_2020-08-24_11-43-13.png)

- 当退出 synchronized 代码块(解锁时) 锁记录不为null,这时候使用 cas 将 Mark Word 的值恢复给对象头
  - 成功，则解锁成功
  - 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程





### 4.8 wait notify 原理

![](/Snipaste_2020-08-25_09-54-24.png)

- **Owner 拥有者所有者 EntryList 入口列表 WaitSet 等待列表 Monitor 监控**
- Owner 线程发现条件不满足，调用wait方法，即可进入 WaitSet 变为 WAITING 状态
- BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片
- BLOCKED 线程会在 Owner 线程释放锁时唤醒
- WAITING 线程会在 Owner 线程调用 notify 或 notifyAll,但唤醒后并不意味着立刻获取锁，仍需要进入EntiryList重新竞争



### 4.9 Park & Unpark

基本使用

他们是 LockSupport 类中的方法

```java
//暂停当前线程
LockSupport.park()
   
//恢复某个线程的运行
LockSupport.unpark(暂停线程对象)
```

先 park 再unpark

```java
public static void main(String[] args) throws InterruptedException {
         Thread t1 = new Thread(() ->{
             log.debug("start...");
             try {
                 TimeUnit.SECONDS.sleep(1);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             } 
             log.debug("park...");
             LockSupport.park();
             log.debug("resume");
         },"t1");
         t1.start();
         Thread.sleep(2000);
         log.debug("unpark.....");
         LockSupport.unpark(t1);
    }
```

输出

```java
14:58:57:775 [t1] cn.itcast.test3.TestMultiLock - start...
14:58:58:779 [t1] cn.itcast.test3.TestMultiLock - park...
14:58:59:774 [main] cn.itcast.test3.TestMultiLock - unpark.....
14:58:59:774 [t1] cn.itcast.test3.TestMultiLock - resume
```

先 unpark 再 parK

```
Thread t1 = new Thread(() ->{
             log.debug("start...");
             try {
                 TimeUnit.SECONDS.sleep(2);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             log.debug("park...");
             LockSupport.park();
             log.debug("resume");
         },"t1");
         t1.start();

         Thread.sleep(1);
         log.debug("unpark.....");
         LockSupport.unpark(t1);
```

输出

```java
15:12:45:287 [main] cn.itcast.test3.TestMultiLock - unpark.....
15:12:45:287 [t1] cn.itcast.test3.TestMultiLock - start...
15:12:47:291 [t1] cn.itcast.test3.TestMultiLock - park...
15:12:47:291 [t1] cn.itcast.test3.TestMultiLock - resume
```

#### 特点

- 与 Object 的 wait & notify 相比
- wait、notify 和 notifyall 必须配合 Object Monitor 一起使用，而 park unpark 不必
- park & unpark 是以线程为单位来【阻塞】 和【唤醒】线程，而notify只能随机唤醒一个等待线程，notifyAll 是唤醒所有的等待线程，就不那么【精确】
- park & unpark 可以先 unpark 而 wait & notify 不能先 notify



<h1 style="color:blue">**具体参考 原理之park & unpark**</h1>

### 4.10 重新理解线程状态切换

![](/Snipaste_2020-08-25_16-26-46.png)

#### 情况1 NEW -->RUNNABLE

- 当调用 t.start() 方法时，由 NEW --> RUNNABLE



#### 情况2 RUNNABLE <--> WAITING

- t 线程用 synchronized(obj) 获取对象锁之后
- 调用obj.wait() 方法时，t线程从 RUNNABLE --> WAITING
- 调用`obj.notify() ,obj.notifyAll() t.interrupt()`时
  - 竞争锁成功，t线程 `WAITING --> RUNNABLE`
  - 竞争锁失败，t线程从 `WAITING --> BLOCKED`

```java
final static Object obj = new Object();
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("执行...");
                try {
                    obj.wait();//让线程在obj一直等待下去
                } catch (Exception e) {
                    e.printStackTrace();
                }
                log.debug("其他代码...");
            }

        },"t1").start();

        new Thread(() -> {
            synchronized (obj) {
                log.debug("执行...");
                try {
                    obj.wait();//让线程在obj一直等待下去
                } catch (Exception e) {
                    e.printStackTrace();
                }
                log.debug("其他代码...");
            }
        },"t2").start();


        Thread.sleep(500);
        log.debug("唤醒obj上其他代码");
        synchronized (obj) {
            obj.notifyAll();//唤醒obj上所有等待线程
        }
    }
```



#### 情况3 RUNNABLE <--> WAITING

- 当前线程调用 t.join() 方法时，当前线程从 RUNNABLE -- > WAITING
  - 注意是当前线程在t线程对象的监视器上等待
- t线程运行结束，或调用了当前线程的interrupt()时，当前线程从 WAITINHG --> RUNNABLE



#### 情况4 RUNNABLE <--> WAITING 

- 当前线程调用 LockSupport.park() 方法会让当前线程从 RUNNABLE --> WAITING
- 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，会让目标线程从 WAITING --> RUNNABLE



#### **情况** **5** RUNNABLE <--> TIMED_WAITING

- **t** **线程**用 synchronized(obj) 获取了对象锁后
- 调用 obj.wait(long n) 方法时，**t** **线程**从 RUNNABLE --> TIMED_WAITING
- **t** **线程**等待时间超过了 n 毫秒，或调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时
- 竞争锁成功，**t** **线程**从 TIMED_WAITING --> RUNNABLE
- 竞争锁失败，**t** **线程**从 TIMED_WAITING --> BLOCKED



#### 情况6 RUNNABLE <--> TIMED_WAITING

**当前线程**调用 t.join(long n) 方法时，**当前线程**从 RUNNABLE --> TIMED_WAITING

- 注意是**当前线程**在**t** **线程对象**的监视器上等待
- **当前线程**等待时间超过了 n 毫秒，或**t** **线程**运行结束，或调用了**当前线程**的 interrupt() 时，**当前线程**从
- TIMED_WAITING --> RUNNABLE



#### **情况** **7** **RUNNABLE <--**> TIMED_WAITING

- 当前线程调用 Thread.sleep(long n) ，当前线程从 RUNNABLE --> TIMED_WAITING
- **当前线程**等待时间超过了 n 毫秒，**当前线程**从 TIMED_WAITING --> RUNNABLE



#### **情况** **8** **RUNNABLE <**--**> TIMED_WAITING**

- 当前线程调用 LockSupport.parkNanos(long nanos) 或 LockSupport.parkUntil(long millis) 时，**当前线**
- **程**从 RUNNABLE --> TIMED_WAITING
- 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，或是等待超时，会让目标线程从
- TIMED_WAITING--> RUNNABLE



#### **情况** **9** **RUNNABLE <**--**> BLOCKED**

- **t** **线程**用 synchronized(obj) 获取了对象锁时如果竞争失败，从 RUNNABLE --> BLOCKED
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED 的线程重新竞争，如果其中 **t** **线程**竞争
- 成功，从 BLOCKED --> RUNNABLE ，其它失败的线程仍然 BLOCKED



#### **情况** **10** **RUNNABLE <**--**> TERMINATED**

- 当前线程所有代码运行完毕，进入 TERMINATED



### 4.11 多把锁

##### 多把不想干的锁

一间大屋子有两个功能，睡觉 学习 互不相干

现在小南要学习，小女要睡觉，但如果只用一间屋子(一个对象锁) 的话，那么并发度很低

解决的方法是准备多个房间(多个对象锁)

```java
class BigRoom {
    public void sleep() {
        synchronized(this) {
            log.debug("sleeping 2小时");
            Thread.sleep(2);
        }
    }
    
    public void study() {
        synchronized(this){
            log.debug("study 1小时");
            Thread.sleep(1);
        }
    }
}
```

执行

```java
BigRoom bigRoom = new BigRoom();
new Thread(() -> {
 	bigRoom.compute();
},"小南").start();
new Thread(() -> {
 	bigRoom.sleep();
},"小女").start();
```

结果

```java
12:13:54.471 [小南] c.BigRoom - study 1 小时
12:13:55.476 [小女] c.BigRoom - sleeping 2 小时
```



改进

```java
class BigRoom{
    private final Object studyRoom = new Object();
    private final Object bedRoom = new Object();
    
    public void sleep() {
        synchronized(bedRoom) {
            log.debug("sleep2小时");
            Thread.sleep(2);
        }
    }
    public void study() {
        synchronized(this) {
            log.debug("study1小时");
            Thread.sleep(1);
        }
    }
}
```

某次执行结果

```java
12:15:35.069 [小南] c.BigRoom - study 1 小时
12:15:35.069 [小女] c.BigRoom - sleeping 2 小时
```

将锁的粒度细分

- 好处，是可以增强并发度
- 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁





### 4.12 活跃性

#### 死锁

有这样一个情况：一个线程需要同时获得多把锁，这时就容易发生死锁

```java
Object A = new Object();
Object B = new Object();
Thread t1 = new Thread(() -> {
     synchronized (A) {
     	log.debug("lock A");
     	sleep(1);
     	synchronized (B) {
             log.debug("lock B");
             log.debug("操作...");
     }
   }
}, "t1");

Thread t2 = new Thread(() -> {
     synchronized (B) {
     	log.debug("lock B");
     	sleep(0.5);
     	synchronized (A) {
            log.debug("lock A");
             log.debug("操作...");
     }
    }
}, "t2");
t1.start();
t2.start();
```

结果

```java
12:22:06.962 [t2] c.TestDeadLock - lock B 
12:22:06.962 [t1] c.TestDeadLock - lock A
```



#### 定位死锁

检测死锁就可以用jconsole工具，或者使用jps定位进程id，再用jstack定位锁

```java
F:\Multithreading\concurrent>jps
4992 Launcher
1892 TestDeadLock
4292 Jps
17304 KotlinCompileDaemon
18184
2236 RemoteMavenServer36

```

```java
F:\Multithreading\concurrent>jstack 1892
2020-08-27 08:42:37
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.241-b07 mixed mode):

"DestroyJavaVM" #14 prio=5 os_prio=0 tid=0x0000000003084800 nid=0x2900 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"t2" #13 prio=5 os_prio=0 tid=0x000000001fe01000 nid=0x4088 waiting for monitor entry [0x000000002028f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at cn.itcast.test3.TestDeadLock.lambda$test1$1(TestDeadLock.java:41)
        - waiting to lock <0x000000076c713010> (a java.lang.Object)
        - locked <0x000000076c713020> (a java.lang.Object)
        at cn.itcast.test3.TestDeadLock$$Lambda$2/1792393294.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

"t1" #12 prio=5 os_prio=0 tid=0x000000001fe0a000 nid=0x79c waiting for monitor entry [0x000000002018f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at cn.itcast.test3.TestDeadLock.lambda$test1$0(TestDeadLock.java:26)
        - waiting to lock <0x000000076c713020> (a java.lang.Object)
        - locked <0x000000076c713010> (a java.lang.Object)
        at cn.itcast.test3.TestDeadLock$$Lambda$1/679890578.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

"Service Thread" #11 daemon prio=9 os_prio=0 tid=0x000000001eaf8800 nid=0x3f3c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #10 daemon prio=9 os_prio=2 tid=0x000000001ea59000 nid=0x4234 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x000000001ea4e000 nid=0x3460 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x000000001ea47800 nid=0x13b8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x000000001ea46800 nid=0x17bc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x000000001ea45000 nid=0x3614 runnable [0x000000001f1de000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:171)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
        - locked <0x000000076bccb550> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(InputStreamReader.java:184)
        at java.io.BufferedReader.fill(BufferedReader.java:161)
        at java.io.BufferedReader.readLine(BufferedReader.java:324)
        - locked <0x000000076bccb550> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(BufferedReader.java:389)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:61)

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001e91f000 nid=0x43e0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001e91e000 nid=0x3398 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000001e900800 nid=0xbb8 in Object.wait() [0x000000001eede000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x000000076ba08ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x000000076ba08ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000001caed800 nid=0x368 in Object.wait() [0x000000001edde000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x000000076ba06c00> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x000000076ba06c00> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x000000001cae9000 nid=0x834 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x000000000309a000 nid=0x451c runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x000000000309b800 nid=0x414 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x000000000309d000 nid=0x1e90 runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x000000000309e800 nid=0x73c runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x00000000030a0800 nid=0x2fe4 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x00000000030a3000 nid=0x1c50 runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x00000000030a6000 nid=0x3190 runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x00000000030a7000 nid=0x46d4 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x000000001eafe800 nid=0x4360 waiting on condition

JNI global references: 316


Found one Java-level deadlock:
=============================
"t2":
  waiting to lock monitor 0x000000001caf1168 (object 0x000000076c713010, a java.lang.Object),
  which is held by "t1"
"t1":
  waiting to lock monitor 0x000000001caf2fa8 (object 0x000000076c713020, a java.lang.Object),
  which is held by "t2"

Java stack information for the threads listed above:
===================================================
"t2":
        at cn.itcast.test3.TestDeadLock.lambda$test1$1(TestDeadLock.java:41)
        - waiting to lock <0x000000076c713010> (a java.lang.Object)
        - locked <0x000000076c713020> (a java.lang.Object)
        at cn.itcast.test3.TestDeadLock$$Lambda$2/1792393294.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"t1":
        at cn.itcast.test3.TestDeadLock.lambda$test1$0(TestDeadLock.java:26)
        - waiting to lock <0x000000076c713020> (a java.lang.Object)
        - locked <0x000000076c713010> (a java.lang.Object)
        at cn.itcast.test3.TestDeadLock$$Lambda$1/679890578.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

- 避免死锁要注意加锁顺序
- 另外如果由于某个线程进入死循环，导致其他线程一直等待，对于这种情况 linux 下可以通过 top 先定位到 CPU 占用高的 Java线程 再利用 top-Hp 进程id 来定位是那个线程，最后再用jstack排查



#### 哲学家就餐问题

![](/Snipaste_2020-08-27_08-45-23.png)

有五位哲学家，围坐在圆桌旁

他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭接着思考

吃饭时要用两根筷子吃，桌上共有5根筷子，每位哲学家左手右手各有一根筷子

如果筷子被身边的人拿着，自己就得等待

筷子类

```java
class Chopstick extends ReentrantLock {
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

哲学家类

```java
@Slf4j
class Philosopher extends Thread {
    Chopstick left;
    Chopstick right;

    public Philosopher(String name,Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }

    @SneakyThrows
    @Override
    public void run() {
        while (true) {
            //尝试获得左手筷子
           synchronized (right) {
                //尝试右手获得筷子
                synchronized (right) {
                    eat();
               }
            }
    }
    public void eat() throws InterruptedException {
        log.debug("eating...");
        Thread.sleep(1000);
    }
}
```

就餐

```java
08:50:28:288 [亚里士多德] cn.itcast.test3.v1.Philosopher - eating...
08:50:28:288 [苏格拉底] cn.itcast.test3.v1.Philosopher - eating...
08:50:29:292 [赫拉克利特] cn.itcast.test3.v1.Philosopher - eating...
08:50:29:292 [柏拉图] cn.itcast.test3.v1.Philosopher - eating...
08:50:30:292 [苏格拉底] cn.itcast.test3.v1.Philosopher - eating...
08:50:30:292 [亚里士多德] cn.itcast.test3.v1.Philosopher - eating...
08:50:31:293 [亚里士多德] cn.itcast.test3.v1.Philosopher - eating...
```

执行不多会，就执行不下去了

```java
12:33:15.575 [苏格拉底] c.Philosopher - eating... 
12:33:15.575 [亚里士多德] c.Philosopher - eating... 
12:33:16.580 [阿基米德] c.Philosopher - eating... 
12:33:17.580 [阿基米德] c.Philosopher - eating... 
// 卡在这里, 不向下运行
```

使用jconsole 检测死锁 发现

```java
F:\Multithreading\concurrent>jstack 16116
2020-08-27 08:56:23
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.241-b07 mixed mode):

"DestroyJavaVM" #17 prio=5 os_prio=0 tid=0x00000000029a4800 nid=0x2950 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"阿基??德" #16 prio=5 os_prio=0 tid=0x000000001f78d000 nid=0x1ff8 waiting on condition [0x000000001ff6f000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.itcast.test3.Philosopher.eat(TestDeadLock.java:70)
        at cn.itcast.test3.Philosopher.run(TestDeadLock.java:49)
        - locked <0x000000076c7150d8> (a cn.itcast.test3.Chopstick)
        - locked <0x000000076c7150d8> (a cn.itcast.test3.Chopstick)

"赫拉克利特" #15 prio=5 os_prio=0 tid=0x000000001f78c800 nid=0x42dc waiting on condition [0x000000001fe6e000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.itcast.test3.Philosopher.eat(TestDeadLock.java:70)
        at cn.itcast.test3.Philosopher.run(TestDeadLock.java:49)
        - locked <0x000000076c7151b8> (a cn.itcast.test3.Chopstick)
        - locked <0x000000076c7151b8> (a cn.itcast.test3.Chopstick)

"亚里士??德" #14 prio=5 os_prio=0 tid=0x000000001f799800 nid=0x758 waiting on condition [0x000000001fd6e000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.itcast.test3.Philosopher.eat(TestDeadLock.java:70)
        at cn.itcast.test3.Philosopher.run(TestDeadLock.java:49)
        - locked <0x000000076c715180> (a cn.itcast.test3.Chopstick)
        - locked <0x000000076c715180> (a cn.itcast.test3.Chopstick)

"柏拉图" #13 prio=5 os_prio=0 tid=0x000000001f798800 nid=0x1b28 waiting on condition [0x000000001fc6f000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.itcast.test3.Philosopher.eat(TestDeadLock.java:70)
        at cn.itcast.test3.Philosopher.run(TestDeadLock.java:49)
        - locked <0x000000076c715148> (a cn.itcast.test3.Chopstick)
        - locked <0x000000076c715148> (a cn.itcast.test3.Chopstick)

"苏格拉底" #12 prio=5 os_prio=0 tid=0x000000001f797000 nid=0x370 waiting on condition [0x000000001fb6f000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.itcast.test3.Philosopher.eat(TestDeadLock.java:70)
        at cn.itcast.test3.Philosopher.run(TestDeadLock.java:49)
        - locked <0x000000076c715110> (a cn.itcast.test3.Chopstick)
        - locked <0x000000076c715110> (a cn.itcast.test3.Chopstick)

```

这种线程没有按预期结束，执行不下去的情况，归类为活跃性问题，除了死锁以外，还有活锁和 饥饿者两种情况



#### 活锁

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束，列如

```java
@Slf4j
public class TestLiveLock {
    static volatile int count = 10;
    static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            //期望大于0退出循环
            while (count > 0) {
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                count--;
                log.debug("count:{}",count);
            }
        },"t1").start();
        new Thread(() -> {
            //期望超过20退出循环
            while (count < 20) {
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                count++;
                log.debug("count:{}",count);
            }
        },"t2").start();
    }
}
```

很多教程 把**饥饿** 定义为 一个线程由于优先级太低 始终得不到 CPU 调度执行，也不能够结束，**饥饿**的情况不以言是，读写锁时会涉及到**饥饿**问题

下面一个线程**饥饿**的例子 先看看使用顺序加锁的方式解决之前的死锁问题

![](/Snipaste_2020-08-27_09-23-53.png)

顺序加锁的解决方案

![](/Snipaste_2020-08-27_09-24-22.png)



### 4.13 ReentrantLock

相对于 synchronized 它具备如下特点

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量



与synchronized一样 都支持可重入

基本语法

```java
//获取锁
reentrantLock.lock();
try{
    //临界区
} catch{
    释放锁
        reentrantlock.unlock();
}
```



可重入

可重入是指同一线程如果首次获得了这把锁，那么因为他时这把锁的拥有者，因此他有权利再次获取这把锁，如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

```java
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        lock.lock();
        try {
            log.debug("enter main");
            m1();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
    public static void m1(){
        lock.lock();
        try {
            log.debug("enter m1");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void m2() {
        lock.lock();
        try {
            log.debug("enter m2");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
```

输出

```java
09:32:11:8 [main] cn.itcast.test3.reentrantlock.Test - enter main
09:32:11:11 [main] cn.itcast.test3.reentrantlock.Test - enter m1
```

可打断

```java
private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(() -> {
            try {
                log.debug("尝试获取锁");
                //如果没有竞争锁，此方法就会获取到lock锁
                //如果有竞争就进入阻塞队列，可以被其它线程用 interrupt 方法打断
                lock.lockInterruptibly();
            } catch (Exception e) {
                log.debug("没有获得锁 返回");
                e.printStackTrace();
            }
            try {
             log.debug("获取到锁");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        },"t1");


        lock.lock();
        t1.start();

        log.debug("打断");
        Thread.sleep(1000);
        t1.interrupt();

    }
```

输出

```java
09:35:43:970 [t1] cn.itcast.test3.reentrantlock.Test2 - 尝试获取锁
09:35:43:970 [main] cn.itcast.test3.reentrantlock.Test2 - 打断
09:35:44:973 [t1] cn.itcast.test3.reentrantlock.Test2 - 没有获得锁 返回
09:35:44:974 [t1] cn.itcast.test3.reentrantlock.Test2 - 获取到锁
```

注意如果是不可中断模式，那么使用了interrupt也不会让等待中断



#### tryLock 解决

```java
    public static void main(String[] args) {
        Chopstick c1 = new Chopstick("1");
        Chopstick c2 = new Chopstick("2");
        Chopstick c3 = new Chopstick("3");
        Chopstick c4 = new Chopstick("4");
        Chopstick c5 = new Chopstick("5");
        new Philosopher("苏格拉底", c1, c2).start();
        new Philosopher("柏拉图", c2, c3).start();
        new Philosopher("亚里士多德", c3, c4).start();
        new Philosopher("赫拉克利特", c4, c5).start();
        new Philosopher("阿基米德", c5, c1).start();


    }
}
@Slf4j
class Philosopher extends Thread {
    Chopstick left;
    Chopstick right;

    public Philosopher(String name,Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }

    @SneakyThrows
    @Override
    public void run() {
        while (true) {
            //尝试获得左手筷子
//            synchronized (right) {
//                //尝试右手获得筷子
//                synchronized (right) {
//                    eat();
//                }
//            }
            if (left.tryLock()) {
                try {
                    //尝试获取右手筷子
                    if (right.tryLock()) {
                      try {
                          eat();
                      } finally {
                          right.unlock();
                      }
                    }
                } finally {
                    left.unlock();//释放自己手里的筷子
                }
            }
        }
    }
    public void eat() throws InterruptedException {
        log.debug("eating...");
        Thread.sleep(1000);
    }
}

/**
 * 筷子
 */
class Chopstick extends ReentrantLock {
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
```



#### 公平锁

ReentrantLock默认是不公平的

```java
   public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock(false);
        lock.lock();
        for (int i = 0; i < 500; i++) {
            new Thread(() -> {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " running...");
                } finally {
                    lock.unlock();
                }
            }, "t" + i).start();
        }
        // 1s 之后去争抢锁
        Thread.sleep(1000);
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " start...");
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " running...");
            } finally {
                lock.unlock();
            }
        }, "强行插入").start();
        lock.unlock();
    }
```

强行插入，有机会在中间输出

> 该实现不一定能复现

```java
t0 running...
强行插入 start...
t1 running...
```

改为公平锁后，总在最后输出

公平锁一般是没有必要，会降低并发度，后面分析原理会讲解



#### 条件变量

synchronized 中也有条件变量，就是我们讲原理时 那个 waitSet 休息室，当条件不满足时进入 waitSet等待

ReentranLock 的条件变量比 synchronized 强大之处在于 他是支持多个条件变量的，这就好比

- synchronized 是那些不满足条件的线程都在一间休息室等消息
- 而ReentrantLock支持多间休息室，有专门的等烟休息室，专门等早餐的休息室，唤醒时也是按休息室来唤醒

使用要点

await 前需要获得锁

await 执行后，会释放锁，进入 conditionObject等待

await 的线程被唤醒(或打断，或超时) 重新竞争lock锁

竞争lock锁成功后，从await后继续执行

例子

```java
    static final Object room = new Object();
    static boolean hasCigarette = false; //有没有烟
    static boolean hasTakeout = false;

    private static ReentrantLock ROOM = new ReentrantLock();
    //等待烟的休息室
    static Condition waitCigaretteSet = ROOM.newCondition();
    //等外卖的休息室
    static Condition waitTakeoutSet = ROOM.newCondition();


    public static void main(String[] args)throws InterruptedException {
        new Thread(() -> {
            ROOM.lock();
             try {
                 log.debug("有烟没？[{}]",hasCigarette);
                     while (!hasCigarette) {
                         log.debug("没烟，先歇会！");
                         try {
                             waitCigaretteSet.await();
                         } catch (Exception e) {
                             e.printStackTrace();
                         }
                     }
                     log.debug("可以开始干活了");
             } finally {
                ROOM.unlock();
             }

        },"小南").start();


            new Thread(() -> {
                ROOM.lock();
                try {
                    log.debug("外卖送到没？[{}]",hasTakeout);
                    while (!hasTakeout) {
                        log.debug("没外卖，先歇会！");
                        try {
                            waitTakeoutSet.await();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    log.debug("可以开始干活");
                } finally {
                    ROOM.unlock();
                }

            },"小女").start();


        Thread.sleep(1000);

        //唤醒外卖
        new Thread(() -> {
            ROOM.lock();
            try {
                //等外卖
                hasTakeout = true;
                waitTakeoutSet.signal();

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                ROOM.unlock();
            }
        },"送外卖的").start();

        Thread.sleep(1000);

        //烟
        new Thread(() -> {
            ROOM.lock();
            try {

                hasCigarette = true;
                waitCigaretteSet.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                ROOM.unlock();
            }
        },"送烟的").start();
    }
```

输出

```java
09:57:15:180 [小南] cn.itcast.test3.reentrantlock.Test5 - 有烟没？[false]
09:57:15:185 [小南] cn.itcast.test3.reentrantlock.Test5 - 没烟，先歇会！
09:57:15:185 [小女] cn.itcast.test3.reentrantlock.Test5 - 外卖送到没？[false]
09:57:15:185 [小女] cn.itcast.test3.reentrantlock.Test5 - 没外卖，先歇会！
09:57:16:166 [小女] cn.itcast.test3.reentrantlock.Test5 - 可以开始干活
09:57:17:166 [小南] cn.itcast.test3.reentrantlock.Test5 - 可以开始干活了
```

### 本章小结

- 分析多线程访问共享资源时，那些代码属于临界区
- 分析多线程访问共享资源时，哪些代码片段属于临界区
- 使用 synchronized 互斥解决临界区的线程安全问题
  - 掌握 synchronized 锁对象语法
  - 掌握 synchronzied 加载成员方法和静态方法语法
  - 掌握 wait/notify 同步方法
- 使用 lock 互斥解决临界区的线程安全问题
  -  掌握 lock 的使用细节：可打断、锁超时、公平锁、条件变量
- 学会分析变量的线程安全性、掌握常见线程安全类的使用
- 了解线程活跃性问题：死锁、活锁、饥饿
- 应用方面
  - 互斥：使用 synchronized 或 Lock 达到共享资源互斥效果
  - 同步：使用 wait/notify 或 Lock 的条件变量来达到线程间通信效果
- <font style='color:blue'>原理方面</font>
  - monitor、synchronized 、wait/notify 原理
  - synchronized 进阶原理
  - park & unpark 原理
- <font style='color:red'>模式方面</font>
  - 同步模式之保护性暂停
  - 异步模式之生产者消费者
  - 同步模式之顺序控制





# 5. 共享模型之内存

上一章讲解的 Monitor 主要关注的是访问共享变量时，保证临界区代码的原子性

进一步深入学习共享变量在多线程间的【可见性】 问题与多条指令执行时的 【有序性】问题

### 5.1 Java 内存模型

JMM 即 Java Memory Model ，它定义了主存，工作内存抽象概念，底层对应着 CPU 寄存器，缓存，硬件内存， CPU指令优化等

JMM 体现在以下几个方面

- 原子性 - 保证指令不会受到线程上下文切换的影响
- 可见性 - 保证指令不会受 cpu 缓存的影响
- 有序性 - 保证指令不会受 cpu 指令并行优化的影响

### 5.2 可见性

退不出的循环

先来看一个现象 main线程对 run 变量的修改对于 t 线程不可见，导致 t线程 无法停止

```java
 static boolean run = true;
    public static void main(String[] args) throws InterruptedException {
         Thread t1 = new Thread(() ->{
             while(run) {
                 //....

             }
         },"t1");
         t1.start();

         Thread.sleep(1000);
         log.debug("停止");
         run =false;

    }
```

为什么呢？分析一下：

- 初始状态， t 线程刚开始从主内存读取了 run 的值到工作内存

![](/Snipaste_2020-08-27_19-27-56.png)

- 因为 t 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中，减少对主存中 run 的访问，提高效率

![](/Snipaste_2020-08-27_19-28-39.png)

-  1 秒之后，main 线程修改了 run 的值，并同步至主存，而 t 是从自己工作内存中的高速缓存中读取这个变量的值，结果永远是旧值

![](/Snipaste_2020-08-27_19-29-21.png)

**解决方法**

volatile（易变关键字）

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取

它的值，线程操作 volatile 变量都是直接操作主存



#### **可见性** **vs** **原子性**

前面例子体现的实际就是可见性，它保证的是在多个线程之间，一个线程对 volatile 变量的修改对另一个线程可

见， 不能保证原子性，仅用在一个写线程，多个读线程的情况： 上例从字节码理解是这样的：

```
getstatic run // 线程 t 获取 run true 
getstatic run // 线程 t 获取 run true 
getstatic run // 线程 t 获取 run true 
getstatic run // 线程 t 获取 run true 
putstatic run // 线程 main 修改 run 为 false， 仅此一次
getstatic run // 线程 t 获取 run false
```

比较一下之前我们将线程安全时举的例子：两个线程一个 i++ 一个 i-- ，只能保证看到最新值，不能解决指令交错

```
// 假设i的初始值为0 
getstatic i 	// 线程2-获取静态变量i的值 线程内i=0 
getstatic i 	// 线程1-获取静态变量i的值 线程内i=0 
iconst_1 	// 线程1-准备常量1 
iadd 	// 线程1-自增 线程内i=1 
putstatic i 	// 线程1-将修改后的值存入静态变量i 静态变量i=1 
iconst_1 	// 线程2-准备常量1 
isub 	// 线程2-自减 线程内i=-1 
putstatic i 	// 线程2-将修改后的值存入静态变量i 静态变量i=-1
```

> **注意** synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是synchronized 是属于重量级操作，性能相对更低
>
> 如果在前面示例的死循环中加入 System.out.println() 会发现即使不加 volatile 修饰符，线程 t 也能正确看到对 run 变量的修改了，想一想为什么？



### 5.3 有序性

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序，思考下面一段代码

```java
static int i;
static int j;
// 在某个线程内执行如下赋值操作
i = ...; 
j = ...;
```

可以看到，至于是先执行 i 还是 先执行 j ，对最终的结果不会产生影响。所以，上面代码真正执行时，既可以是

```
i = ...; 
j = ...;
```

也可以是

```
j = ...; 
i = ...;
```

这种特性称之为『指令重排』，多线程下『指令重排』会影响正确性。为什么要有重排指令这项优化呢？从 CPU

执行指令的原理来理解一下吧

诡异的结果

```java
  int num = 0;
    volatile boolean ready = false;

    @Actor
    public void actor1(I_Result r) {
        if(ready) {
            r.r1 = num + num;
        } else {
            r.r1 = 1;
        }
    }
    @Actor
    public void actor2(I_Result r) {
        num = 2;
        ready = true;
    }
```

I_Result 是一个对象，有一个属性 r1 用来保存结果，问，可能的结果有几种？

有同学这么分析

情况1：线程1 先执行，这时 ready = false，所以进入 else 分支结果为 1

情况2：线程2 先执行 num = 2，但没来得及执行 ready = true，线程1 执行，还是进入 else 分支，结果为1

情况3：线程2 执行到 ready = true，线程1 执行，这回进入 if 分支，结果为 4（因为 num 已经执行过了）

但我告诉你，结果还有可能是 0 😁😁😁，信不信吧！

这种情况下是：线程2 执行 ready = true，切换到线程1，进入 if 分支，相加为 0，再切回线程2 执行 num = 2

相信很多人已经晕了 😵😵😵

这种现象叫做指令重排，是 JIT 编译器在运行时的一些优化，这个现象需要通过大量测试才能复现：

借助 java 并发压测工具 jcstress https://wiki.openjdk.java.net/display/CodeTools/jcstress

```
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.openjdk.jcstress -
DarchetypeArtifactId=jcstress-java-test-archetype -DarchetypeVersion=0.5 -DgroupId=cn.itcast -
DartifactId=ordering -Dversion=1.0
```

执行

mvn clean install

java -jar jcstress.jar

```java
*** INTERESTING tests
  Some interesting behaviors observed. This is for the plain curiosity.

  2 matching test results.
      [OK] cn.itcast.ConcurrencyTest
    (JVM args: [-XX:-TieredCompilation])
  Observed state   Occurrences              Expectation  Interpretation

               0           982   ACCEPTABLE_INTERESTING  !!!!

               1    42,637,266               ACCEPTABLE  ok

               4    50,505,603               ACCEPTABLE  ok


      [OK] cn.itcast.ConcurrencyTest
    (JVM args: [])
  Observed state   Occurrences              Expectation  Interpretation

               0         1,528   ACCEPTABLE_INTERESTING  !!!!

               1    32,862,258               ACCEPTABLE  ok

               4    41,551,105               ACCEPTABLE  ok



```

可以看到，出现结果为 0 的情况有 638 次，虽然次数相对很少，但毕竟是出现了。

**解决方法**

volatile 修饰的变量，可以禁用指令重排

```java
volatile boolean ready = false;
```

结果为

```
*** INTERESTING tests 
 Some interesting behaviors observed. This is for the plain curiosity. 
 0 matching test results
```



### happens-before

happens-before 规定了对共享变量的读写操作对其他线程的读操作可见，他是可见性与有序性的一套规则总结，抛开以下 happends-before 规则,JVM 并不能保证一个线程对共享表变量的写，对于其他线程对该共享变量的读写可见

线程解锁 m 之前对变量的写， 对于接下来对 m 加锁的其他线程对该变量的读可见

```java
static int x;
static Object m = new Object();

new Thread(()->{
     synchronized(m) {
     	x = 10;
 	}
},"t1").start();

new Thread(()->{
     synchronized(m) {
     	System.out.println(x);
 	}
},"t2").start();
```

线程对 volatile 变量的写，对接下来其它线程对该变量的读可见

```java
volatile static int x;
new Thread(()->{
     x = 10;
},"t1").start();

new Thread(()->{
     System.out.println(x);
},"t2").start();
```

线程 start 前对变量的写，对该线程开始后对该变量的读可见

```java
static int x; 
//静态变量在类加载的时候加载
x = 10;
new Thread(()->{
     System.out.println(x);
},"t2").start();
```

线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）

```java
static int x;
Thread t1 = new Thread(()->{
 	x = 10;
},"t1");
t1.start();
t1.join();
System.out.println(x);
```

线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过t2.interrupted 或 t2.isInterrupted）

```java
static int x;
public static void main(String[] args) {
     Thread t2 = new Thread(()->{
     while(true) {
         if(Thread.currentThread().isInterrupted()) {
             System.out.println(x);
         break;
            }
         }
     },"t2");
 t2.start();
    
 new Thread(()->{
     sleep(1);
     x = 10;
     t2.interrupt();
     },"t1").start();
    
     while(!t2.isInterrupted()) {
     	Thread.yield();
     }
     System.out.println(x);
}
```

对变量默认值（0，false，null）的写，对其它线程对该变量的读可见

具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z ，配合 volatile 的防指令重排，有下面的例子

```java
volatile static int x;
static int y;
new Thread(()->{ 
 y = 10;
 x = 20;
},"t1").start();

new Thread(()->{
 // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
 System.out.println(x); 
},"t2").start();
```

> 变量都是指成员变量或静态成员变量

习题

### **balking** **模式习题**

希望 doInit() 方法仅被调用一次，下面的实现是否有问题，为什么？

```java
public class TestVolatile {
 volatile boolean initialized = false;
    
 void init() {
 if (initialized) { //t1 = false t2=false 两个线程都进来了 用同步代码块解决
 	return;
 } 
 doInit();
 initialized = true;
 }
    
 private void doInit() {
 
 }
}
```

### **线程安全单例习题**

单例模式有很多实现方法，饿汉、懒汉、静态内部类、枚举类，试分析每种实现下获取单例对象（即调用

getInstance）时的线程安全，并思考注释中的问题

> 饿汉式：类加载就会导致该单实例对象被创建
>
> 懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会创建

实现1：

```java
// 问题1：为什么加 final 防止子类继承
// 问题2：如果实现了序列化接口, 还要做什么来防止反序列化破坏单例 反序列化会生成一个新的对象 
public final class Singleton implements Serializable {
 // 问题3：为什么设置为私有? 是否能防止反射创建新的实例? 私有安全 不能防止反射
 private Singleton() {}
 // 问题4：这样初始化是否能保证单例对象创建时的线程安全? 没有 类加载阶段
 private static final Singleton INSTANCE = new Singleton();
 // 问题5：为什么提供静态方法而不是直接将 INSTANCE 设置为 public, 说出你知道的理由 
 public static Singleton getInstance() {
 return INSTANCE;
 }
  //反序列化
 public Object readResolve() {
 	return INSTANCE;
 }
}
```

实现2

```java
// 问题1：枚举单例是如何限制实例个数的 静态成员变量
// 问题2：枚举单例在创建时是否有并发问题 没有 静态类加载的时候
// 问题3：枚举单例能否被反射破坏单例 不能 
// 问题4：枚举单例能否被反序列化破坏单例 默认实现序列化接口 能
// 问题5：枚举单例属于懒汉式还是饿汉式 饿汉式
// 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做 构造方法 普通方法
enum Singleton { 
 INSTANCE; 
}
```

实现3:

```java
public final class Singleton {
 private Singleton() { }
 private static Singleton INSTANCE = null;
 // 分析这里的线程安全, 并说明有什么缺点 性能低
 public static synchronized Singleton getInstance() {
 if( INSTANCE != null ){
 	return INSTANCE;
 } 
 	INSTANCE = new Singleton();
 	return INSTANCE;
 }
}
```

实现4： DCL

```java
public final class Singleton {
 private Singleton() { }
 // 问题1：解释为什么要加 volatile ? 因为 构造方法指令重排序
 private static volatile Singleton INSTANCE = null;
 
 // 问题2：对比实现3, 说出这样做的意义 同步代码块缩小
 public static Singleton getInstance() {
     if (INSTANCE != null) { 
         return INSTANCE;
     }
     synchronized (Singleton.class) { 
     // 问题3：为什么还要在这里加为空判断, 之前不是判断过了吗 防止首次多个线程并发
     if (INSTANCE != null) { // t2 
         return INSTANCE;
     }
         INSTANCE = new Singleton(); 
         return INSTANCE;
     } 
  }
}
```

实现5：

```java
public final class Singleton {
 	private Singleton() { }
 	// 问题1：属于懒汉式还是饿汉式 懒汉式
 	private static class LazyHolder {
 		static final Singleton INSTANCE = new Singleton();
 	}
     // 问题2：在创建时是否有并发问题 类加载 有jvm保护
     public static Singleton getInstance() {
         return LazyHolder.INSTANCE;
     }
}
```

- 本章小结
- 可见性 - 由JVM 缓存优化引起
- 有序性 - 由JVM指令重排序优化引起
- happens - before规则
- 原理方面
  - CPU 指令并行
  - volatile
- 模式方面
  - 两阶段终止模式的 volatile改进
  - 同步模式之 balking



# 6. 共享模型之无锁