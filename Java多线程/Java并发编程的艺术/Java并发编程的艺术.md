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