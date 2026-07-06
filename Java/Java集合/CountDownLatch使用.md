# CountDownLatch 使用

## 一、什么是 CountDownLatch

Java的`concurrent`包里面的`CountDownLatch`其实可以把它看作一个计数器，只不过这个计数器的操作是原子操作，同时只能有一个线程去操作这个计数器，也就是同时只能有一个线程去减这个计数器里面的值。

你可以向`CountDownLatch`对象设置一个初始的数字作为计数值，任何调用这个对象上的`await()`方法都会阻塞，直到这个计数器的计数值被其他的线程减为`0`为止。

`CountDownLatch`的一个非常典型的应用场景是：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。假如我们这个想要继续往下执行的任务调用一个`CountDownLatch`对象的`await()`方法，其他的任务执行完自己的任务后调用同一个`CountDownLatch`对象上的`countDown()`方法，这个调用`await()`方法的任务将一直阻塞等待，直到这个`CountDownLatch`对象的计数值减到 **0** 为止。

比如：客户端一次请求5个统计数据，服务器需要全部统计完成后，才返回客户端，可以使用`CountDownLatch` 。

## 二、怎么使用 CountDownLatch

### 1、构造方法

```java
//参数count为计数值
public CountDownLatch(int count) {  };
```

### 2、重要方法

```java
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };   

//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  

//将count值减1
public void countDown() { };
```

### 3、使用案例

* 首先是创建实例 `CountDownLatch countDown = new CountDownLatch(2)`
* 需要同步的线程执行完之后，计数\*\*-1\*\*； `countDown.countDown()`
* 需要等待其他线程执行完毕之后，再运行的线程，调用 `countDown.await()`实现阻塞同步

```java
import java.util.concurrent.CountDownLatch;

public class Test {
    
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        final CountDownLatch latch = new CountDownLatch(2);
        
        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName()
                                       + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName()
                                       + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        
        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName()
                                       + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName()
                                       + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        
        try {
            System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
}
```

打印结果：

```plain
子线程Thread-0正在执行
等待2个子线程执行完毕...
子线程Thread-1正在执行
子线程Thread-0执行完毕
子线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```

## 三、CountDownLatch 使用场景

电商的详情页，由众多的数据拼装组成，如可以分成一下几个模块

* 交易的收发货地址，销量
* 商品的基本信息（标题，图文详情之类的）
* 推荐的商品列表
* 评价的内容
* ....

上面的几个模块信息，都是从不同的服务获取信息，且彼此没啥关联；所以为了提高响应，完全可以做成并发获取数据，如

* 线程1获取交易相关数据
* 线程2获取商品基本信息
* 线程3获取推荐的信息
* 线程4获取评价信息
* ....

但是最终拼装数据并返回给前端，需要等到上面的所有信息都获取完毕之后，才能返回，这个场景就非常的适合 `CountDownLatch` 来做了

在拼装完整数据的线程中调用 `CountDownLatch#await(long, TimeUnit)` 等待所有的模块信息返回\
每个模块信息的获取，由一个独立的线程执行；执行完毕之后调用 `CountDownLatch#countDown()` 进行计数\*\*-1\*\*

## 参考

* <https://www.cnblogs.com/crazymakercircle/p/13906922.html>


> 更新: 2022-06-21 23:13:35  
> 原文: <https://www.yuque.com/thinkspace/ulag78/anavkc>