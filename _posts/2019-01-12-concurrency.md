---
title: Concurrency Notes
categories: Java
tags: [Java, Concurrency]
---
# Concurrency

## 一般概念

* 进程（ process ）：计算机中执行中的程序，拥有独立的内存等资源。
* 线程 （ thread ）：进程中多个同时运行中的任务（并发，concurrency ），线程共享进程中的各种资源，在单核处理器中，线程的同时运行是通过线程调度器将 CPU 时间片在多个任务之间来回切换，这就不是真正意义上的同时运行。相反，多核处理器则存在真正意义上的同时运行。
* 并发的作用：
  * 提高程序的运行速度，比如网站服务器会给每个请求分配一个线程。按顺序执行的程序在没有阻塞（ block ）的情况下，使用多线程并没有速度上的优势，相反，在线程来回切换的过程中会降低程序的运行速度。但是，但程序中的部分存在阻塞时，比如网络请求、硬盘读写、等待用户输入信息等，使用多线程的速度优势能够提现出来。
  * 简化程序的设计复杂度。在设计 GUI 界面时，线程的使用，可将计算任务分配给一个线程，而对用户的操作用其他线程予以反应，这样就不用将对用户操作反应的代码插入到其他代码中，提高代码的简洁性和可维护性。
* Java中的线程和任务：线程是通过调用 `Thread` 类的 `start()` 方法启动的，但是具体的任务却是在 `run()` 方法中定义。
* 线程安全级别：
  * 不可变（Immutable）:不可变类的实例是线程安全的，这些类包括：`String`, `Integer`, `Long`等。
  * 无条件线程安全（Unconditionally thread-safe）：这些类的实例是可变的，但却是线程安全的，这些类包括：`AtomicInteger`, `ConcurrentHashMap`。
  * 

## 实现线程的方式

### 继承`Thread`，重写`run()`方法

```java
public class ThreadTest extends Thread{
    public void run(){
        //...
    }
    
    public static void main(String[] args){
        Thread t = new ThreadTest();
        t.start();
    }
}
```

### 实现`Runnable接口`，重写`run()`方法

实现`Runnable`接口，将实际任务与线程之间进行解藕，Java类中的单继承特点，可以不再继承`Thread`类。

```java
public class RunnableTest implements Runnable{
    public void run(){
        //...
    }
    
    public static void main(String[] args){
        Runnable r = new RunnableTest();
        Thread t = new Thread(r);
        t.start();
    }
}
```

### 实现`Callable`接口，重写`call（）`方法

`Callable` 接口的 `call()` 方法可以指定返回值，在实现该接口时，通过泛型类型参数指定返回参数的类型。返回类型为 `Future<T>` ，可以调用 `Future<T>` 对象的 `isDone()` 方法确定返回已经完成，即可调用 `get()` 方法（也可以直接调用，此时该方法会处于阻塞状态），获得返回值。

与 `Runnable` 使用 `execute()` 方法添加到线程池不同的是，实现 `Callable` 接口的类，添加到线程池时使用 `ExecutorService` 类的 `submit()` 方法。

```java
public class CallableTest implements Callable<String>{
    public String call(){
        //...
        return "from CallableTest call()";
    }
    
    public static void main(String[] args){
        List<Future<String>> results = new ArrayList<Future<String>>();
        ExecutorSerivice exec = Executors.newCachedThreadPool();
        for(int i = 0; i < 5; i++){
            results.add(exec.sumbit(new CallalbeTest()));
        }
        
        for(Future<String> result : results){
            try{
                //get() block until compelete
                System.out.println(result.get());
            }catch(InterrutpedException e){
                System.out.println(e);
                return;
            }catch(ExecutionException e){
                System.out.println(e);
            }finally{
                exec.shutdown();
            }
        }
    }
}
```

### 线程池

```java
public class ThreadPollTest{
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        for(int i = 0; i < 100; i++){
            //匿名内部类
            Runnable r = new Runnable(){
                public void run(){
                    //获得当前方法的线程信息
                    System.out.println(Thread.currentThread());
                }
            }
            threadPool.execute(r);
        }
        //向线程池中的线程发送 interrupt() 方法
        //threadPool.shutdown() 表示线程池将不再接受新的 Runnable 对象。
        threadPool.shutdownNow();
    }
}
```

`Executors`中常用的方法：

* `newFixedThreadPool()` ：事先创建数量固定的线程，线程会被重复利用。
* `newCachedThreadPool()` ：根据需要创建足够多的线程，当线程数量满足需求后，重复利用。
* `newSingleThreadExecutor()` ：根据线程的放入顺序依次执行。

### 守护线程

守护线程又称为后台线程，线程创建出来之后默认是前台线程，前台线程通过调用`setDaemon()`方法之后变为前台线程。

守护线程在进程结束之后结束，进程在所有的前台进程结束后结束。

## `Thread` 常用方法

### 静态方法

* `Thread.currentThread()` ：获得运行该方法的线程。
* `Thread.sleep(nTime)` ：运行该方法的线程休眠一段时间，线程处于阻塞状态。
* `Thread.yield()` ：运行该方法的线程自愿放弃CPU时间片，由Running状态变为Runnable状态。
* `Thread.interrupted()` ：判断当前线程是否中断。

### 实例方法

* `join()` ：该方法所在的线程在调用该方法的线程结束之前处于阻塞状态，可以设定一个 `timeout` 数值，表示等待的最长时间。

  ```java
  public class JoinTest{
      public static void main(String[] args){
          Thread t = new Thread(){
              public void run(){
                  Thread.sleep(3000);
                  System.out.println("Thread 0000");
              }
          }；
          
          t.start();
          //main线程将等待 t 线程结束后继续执行。
          t.join();
          System.out.println("main thread")
      }
  }
  ```

  

* `start()`：启动线程的方法，创建新的线程。

* `run()` :在当前线程中执行该方法，并不会创建新的线程。

* `interrupt()` ：

* `getId()`, `getName()`, `isAlive()`, `getPriority()`, `setPriority()`, `isDaemon()`, `setDaemon()`

## 资源的同步

### `synchronized`

### `ReentrantLock`

### `volatile` 关键字

* 原子性（ atomicity ）：是指不会被线程调度打断的操作，确保一次性完成，对除 `long` 和 `double` 之外的基本变量进行读写操作可以保证是不可分割的原子操作。因为 Java 中原子操作是针对32位的，而 `long` 和 `double` 变量是64位，所以对其操作不为原子性。Java中 `i++`， `i += 3` 都不是原子操作。
* 可见性（ visibility ）：多线程任务中，其中一个线程对某个变量所做的更该是存储在缓冲区的，不会立即更新到主内存中，这就导致其他线程不能立即读取到变量的最新值。
* 声明变量时，指定关键字 `volatile` 可以保证变量读写的原子性和可见性，该关键字的作用某种程度上和 `synchronized` 相同，只是仅仅针对某个变量。
* 只有多个线程对某个变量进行读写操作时才考虑使用 `volatile` 关键字，如果变量的读写操作被 `synchronized` 关键字包围，则不必用 `volatile`。

### `ThreadLocal`

* `ThreadLocal` 是为线程中某个相同的变量自动分配独立的存储区，防止线程之间资源共享的冲突的机制。

  ```java
  class ThreadLocal<T>{
      T get();	//获得当前线程的 thread-local 变量
      protected T initialValue();	//返回当前线程的 thread-local 变量的初始值
      void set(T value);	//设置当前线程的 thread-loacal 变量
      void remove();	//移除当前线程的 thread-local 变量。
  }
  ```

* `ThreadLocal` 变量一般声明为 `static`。

## 线程间通信

### 线程的状态

- new
- runnable
- running
- block
- dead

线程进入阻塞状态的情形：

- 线程调用了 `sleep` 方法。（ interruptible blocking ）
- 线程调用了 `wait` 方法，这需要 timeout 时间到期，或者线程收到 `notify` 或 `notifyAll` 信息才会重新回到runnable状态。
- 线程处于 I/O 阻塞状态。（ uninterruptible blocking ）。
- 线程在等待 `synchronized` 锁的释放。（ uninterruptible block ）。

给线程发送 `interrupt`

-  `ExecutorService` 的`shutdownNow` 方法同时向线程池中的所有线程发送 `interrupt()` 方法。
- 将`Future<?>` 的 `cancel()`设置为 `true`，可以向指定的线程发送 `interrupt()` 方法

## Java内存模型

Java内存模型是一系列的用于规定某个线程对**共享变量**的写入何时可以被其他线程读取到的规则。

共享变量：类变量，成员变量存储在堆内存中的变量为线程之间共享的变量，这类变量需要多线程并发控制访问。对于线程的局部变量是不被共享的，所以不存在线程安全问题。

java内存有主内存和工作内存之分，主内存是线程间共享的，而工作内存是每个线程独占的。变Java内存模型定义了8个原子操作，用于共享变量在主内存和工作内存之间的读写，这8个操作是：lock, unlock, read, load, use, assign, store, write。

Informal overview of Java Memory Model shows it to be a set of rules for when writes by one thread are visible to another thread.

JMM规则由以下4个方面构成：

* 变量读写的原子性；
* 先行发生原则（happens-before）
* well-formed execution and commit order
* final 关键字语义

