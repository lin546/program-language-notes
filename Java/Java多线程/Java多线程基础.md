# Java 多线程基础

## 一、创建线程

线程表示一条单独的执行流，它有自己的程序执行计数器，有自己的栈。下面，我们通过创建线程来对线程建立一个直观感受，在Java中创建线程有两种方式，一种是继承`Thread`，另外一种是实现`Runnable`接口，我们先来看第一种。

### 继承 Thread

Java中 java.lang.Thread 这个类表示线程，一个类可以继承Thread并重写其run方法来实现一个线程，如下所示：

```java
public class HelloThread extends Thread {
    
    @Override
    public void run() {
        System.out.println("hello");
    }
}
```

HelloThread这个类继承了Thread，并重写了run方法。run方法的方法签名是固定的，public，没有参数，没有返回值，不能抛出受检异常。run方法类似于单线程程序中的main方法，线程从run方法的第一条语句开始执行直到结束。

定义了这个类不代表代码就会开始执行，线程需要被启动，启动需要先创建一个HelloThread对象，然后调用Thread的start方法，如下所示：

```java
public static void main(String[] args) {
    Thread thread = new HelloThread();
    thread.start();
}
```

我们在main方法中创建了一个线程对象，并调用了其start方法，调用start方法后，HelloThread的run方法就会开始执行，屏幕输出：

```java
hello
```

为什么调用的是start，执行的却是run方法呢？start表示启动该线程，使其成为一条单独的执行流，背后，操作系统会分配线程相关的资源，每个线程会有单独的程序执行计数器和栈，操作系统会把这个线程作为一个独立的个体进行调度，分配时间片让它执行，执行的起点就是run方法。

如果不调用start，而直接调用run方法呢？屏幕的输出并不会发生变化，但并不会启动一条单独的执行流，run方法的代码依然是在main线程中执行的，run方法只是main方法调用的一个普通方法。

怎么确认代码是在哪个线程中执行的呢？Thread有一个静态方法currentThread，返回当前执行的线程对象：

```java
public static native Thread currentThread();
```

每个Thread都有一个id和name：

```java
public long getId()
public final String getName()
```

这样，我们就可以判断代码是在哪个线程中执行的，我们在HelloThead的run方法中加一些代码：

```java
@Override
public void run() {
    System.out.println("thread name: "+ Thread.currentThread().getName());
    System.out.println("hello");
}
```

如果在main方法中通过start方法启动线程，程序输出为：

```java
thread name: Thread-0
hello
```

如果在main方法中直接调用run方法，程序输出为：

```java
thread name: main
hello
```

调用start后，就有了两条执行流，新的一条执行run方法，旧的一条继续执行main方法，两条执行流并发执行，操作系统负责调度，在单CPU的机器上，同一时刻只能有一个线程在执行，在多CPU的机器上，同一时刻可以有多个线程同时执行，但操作系统给我们屏蔽了这种差异，给程序员的感觉就是多个线程并发执行，但哪条语句先执行哪条后执行是不一定的。当所有线程都执行完毕的时候，程序退出。

### 实现 Runnable 接口

通过继承Thread来实现线程虽然比较简单，但我们知道，Java中只支持单继承，每个类最多只能有一个父类，如果类已经有父类了，就不能再继承Thread，这时，可以通过实现java.lang.Runnable接口来实现线程。

Runnable接口的定义很简单，只有一个run方法，如下所示：

```java
public interface Runnable {
    public abstract void run();
}
```

一个类可以实现该接口，并实现run方法，如下所示：

```java
public class HelloRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("hello");
    }
}
```

仅仅实现Runnable是不够的，要启动线程，还是要创建一个Thread对象，但传递一个Runnable对象，如下所示：

```java
public static void main(String[] args) {
    Thread helloThread = new Thread(new HelloRunnable());
    helloThread.start();
}
```

无论是通过继承Thead还是实现Runnable接口来实现线程，启动线程都是调用Thread对象的start方法。

## 二、线程的基本属性和方法

### id和name

前面我们提到，每个线程都有一个id和name，id是一个递增的整数，每创建一个线程就加一，name的默认值是"Thread-"后跟一个编号，name可以在Thread的构造方法中进行指定，也可以通过setName方法进行设置，给Thread设置一个友好的名字，可以方便调试。

### 优先级

线程有一个优先级的概念，在Java中，优先级从1到10，默认为5，相关方法是：

```java
public final void setPriority(int newPriority)
public final int getPriority()
```

这个优先级会被映射到操作系统中线程的优先级，不过，因为操作系统各不相同，不一定都是10个优先级，Java中不同的优先级可能会被映射到操作系统中相同的优先级，另外，优先级对操作系统而言更多的是一种建议和提示，而非强制，简单的说，在编程中，不要过于依赖优先级。

### 状态

线程有一个状态的概念，Thread有一个方法用于获取线程的状态：

```java
public State getState()
```

返回值类型为Thread.State，它是一个枚举类型，有如下值：

```java
public enum State {
  NEW,
  RUNNABLE,
  BLOCKED,
  WAITING,
  TIMED_WAITING,
  TERMINATED;
}
```

关于这些状态，我们简单解释下：

* NEW: 没有调用start的线程状态为NEW
* TERMINATED: 线程运行结束后状态为TERMINATED
* RUNNABLE: 调用start后线程在执行run方法且没有阻塞时状态为RUNNABLE，不过，RUNNABLE不代表CPU一定在执行该线程的代码，可能正在执行也可能在等待操作系统分配时间片，只是它没有在等待其他条件
* BLOCKED、WAITING、TIMED\_WAITING：都表示线程被阻塞了，在等待一些条件，其中的区别我们在后续章节再介绍

Thread还有一个方法，返回线程是否活着：

```java
public final native boolean isAlive()
```

线程被启动后，run方法运行结束前，返回值都是true。

### 是否daemo线程

Thread有一个是否daemo线程的属性，相关方法是：

```java
public final void setDaemon(boolean on)
public final boolean isDaemon()
```

前面我们提到，启动线程会启动一条单独的执行流，整个程序只有在所有线程都结束的时候才退出，但daemo线程是例外，当整个程序中剩下的都是daemo线程的时候，程序就会退出。

daemo线程有什么用呢？它一般是其他线程的辅助线程，在它辅助的主线程退出的时候，它就没有存在的意义了。在我们运行一个即使最简单的"hello world"类型的程序时，实际上，Java也会创建多个线程，除了main线程外，至少还有一个负责垃圾回收的线程，这个线程就是daemo线程，在main线程结束的时候，垃圾回收线程也会退出。

### sleep方法

Thread有一个静态的sleep方法，调用该方法会让当前线程睡眠指定的时间，单位是毫秒：

```java
public static native void sleep(long millis) throws InterruptedException;
```

睡眠期间，该线程会让出CPU，但睡眠的时间不一定是确切的给定毫秒数，可能有一定的偏差，偏差与系统定时器和操作系统调度器的准确度和精度有关。

睡眠期间，线程可以被中断，如果被中断，sleep会抛出InterruptedException，关于中断以及中断处理，我们后续章节再介绍。

### yield方法

Thread还有一个让出CPU的方法：

```java
public static native void yield();
```

这也是一个静态方法，调用该方法，是告诉操作系统的调度器，我现在不着急占用CPU，你可以先让其他线程运行。不过，这对调度器也仅仅是建议，调度器如何处理是不一定的，它可能完全忽略该调用。

### join方法

在前面HelloThread的例子中，HelloThread没执行完，main线程可能就执行完了，Thread有一个join方法，可以让调用join的线程等待该线程结束，join方法的声明为：

```java
public final void join() throws InterruptedException
```

在等待线程结束的过程中，这个等待可能被中断，如果被中断，会抛出InterruptedException。

join方法还有一个变体，可以限定等待的最长时间，单位为毫秒，如果为0，表示无期限等待：

```java
public final synchronized void join(long millis) throws InterruptedException
```

在前面的HelloThread示例中，如果希望main线程在子线程结束后再退出，main方法可以改为：

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread = new HelloThread();
    thread.start();
    thread.join();
}
```

### 过时方法

Thread类中还有一些看上去可以控制线程生命周期的方法，如：

```java
public final void stop()
public final void suspend()
public final void resume()
```

这些方法因为各种原因已被标记为了过时，我们不应该在程序中使用它们。

## 三、线程的优点与成本

### 优点

为什么要创建单独的执行流？或者说线程有什么优点呢？至少有以下几点：

* 充分利用多CPU的计算能力，单线程只能利用一个CPU，使用多线程可以利用多CPU的计算能力。
* 充分利用硬件资源，CPU和硬盘、网络是可以同时工作的，一个线程在等待网络IO的同时，另一个线程完全可以利用CPU，对于多个独立的网络请求，完全可以使用多个线程同时请求。
* 在用户界面(GUI)应用程序中，保持程序的响应性，界面和后台任务通常是不同的线程，否则，如果所有事情都是一个线程来执行，当执行一个很慢的任务时，整个界面将停止响应，也无法取消该任务。
* 简化建模及IO处理，比如，在服务器应用程序中，对每个用户请求使用一个单独的线程进行处理，相比使用一个线程，处理来自各种用户的各种请求，以及各种网络和文件IO事件，建模和编写程序要容易的多。

### 成本

关于线程，我们需要知道，它是有成本的。创建线程需要消耗操作系统的资源，操作系统会为每个线程创建必要的数据结构、栈、程序计数器等，创建也需要一定的时间。

此外，线程调度和切换也是有成本的，当有当量可运行线程的时候，操作系统会忙于调度，为一个线程分配一段时间，执行完后，再让另一个线程执行，一个线程被切换出去后，操作系统需要保存它的当前上下文状态到内存，上下文状态包括当前CPU寄存器的值、程序计数器的值等，而一个线程被切换回来后，操作系统需要恢复它原来的上下文状态，整个过程被称为上下文切换，这个切换不仅耗时，而且使CPU中的很多缓存失效，是有成本的。

当然，这些成本是相对而言的，如果线程中实际执行的事情比较多，这些成本是可以接受的，但如果只是执行本节示例中的counter++，那相对成本就太高了。

另外，如果执行的任务都是CPU密集型的，即主要消耗的都是CPU，那创建超过CPU数量的线程就是没有必要的，并不会加快程序的执行。


> 更新: 2022-06-24 00:19:00  
> 原文: <https://www.yuque.com/thinkspace/ulag78/vkmuwa>