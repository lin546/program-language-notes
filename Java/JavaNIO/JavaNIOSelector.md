# Java NIO Selector

## 一、为什么使用Selector?

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">仅用单个线程来处理多个</font><code><font style="color:rgb(102, 102, 102);">Channels</font></code><font style="color:rgb(102, 102, 102);">的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">下面是单线程使用一个Selector处理3个</font><code><font style="color:rgb(102, 102, 102);">channel</font></code><font style="color:rgb(102, 102, 102);">的示例图：</font>

<font style="color:rgb(102, 102, 102);"></font>

![1633615211233-a9b5b965-7121-49b6-a122-8a408f758234.png](./img/mnEaQ6SQo-qHdXxV/1633615211233-a9b5b965-7121-49b6-a122-8a408f758234-417070.png)

## 二、Selector的创建

<font style="color:rgb(102, 102, 102);">通过调用</font><code><font style="color:rgb(102, 102, 102);">Selector.open()</font></code><font style="color:rgb(102, 102, 102);">方法创建一个 Selector，如下：</font>

```c
Selector selector = Selector.open();
```

### <font style="color:rgb(102, 102, 102);">三、向Selector注册通道</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">为了将 </font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);"> 和 </font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);"> 配合使用，必须将</font><code><font style="color:rgb(102, 102, 102);">channel</font></code><font style="color:rgb(102, 102, 102);">注册到</font><code><font style="color:rgb(102, 102, 102);">selector</font></code><font style="color:rgb(102, 102, 102);">上。通过</font><code><font style="color:rgb(102, 102, 102);">SelectableChannel.register()</font></code><font style="color:rgb(102, 102, 102);">方法来实现，如下：</font>

```c
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

<font style="color:rgb(102, 102, 102);">与</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">一起使用时，</font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);">必须处于非阻塞模式下。这意味着不能将</font><code><font style="color:rgb(102, 102, 102);">FileChannel</font></code><font style="color:rgb(102, 102, 102);">与</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">一起使用，因为</font><code><font style="color:rgb(102, 102, 102);">FileChannel</font></code><font style="color:rgb(102, 102, 102);">不能切换到非阻塞模式。而套接字通道都可以。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">注意</font><code><font style="color:rgb(102, 102, 102);">register()</font></code><font style="color:rgb(102, 102, 102);">方法的第二个参数。这是一个“</font><code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);">集合”，意思是在通过</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">监听</font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);">时对什么事件感兴趣。可以监听四种不同类型的事件：</font>

<font style="color:rgb(102, 102, 102);"></font>

1. <code><font style="color:rgb(102, 102, 102);">Connect</font></code> : 客户端连接成功时触发
2. <code><font style="color:rgb(102, 102, 102);">Accept</font></code> : 服务器端成功接受连接时触发
3. <code><font style="color:rgb(102, 102, 102);">Read</font></code> : 数据可读入时触发，有因为接收能力弱，数据暂不能读入的情况
4. <code><font style="color:rgb(102, 102, 102);">Write</font></code> : 数据可写出时触发，有因为发送能力弱，数据暂不能写出的情况

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">通道触发了一个事件意思是该事件已经就绪。所以，某个</font><code><font style="color:rgb(102, 102, 102);">channel</font></code><font style="color:rgb(102, 102, 102);">成功连接到另一个服务器称为“连接就绪”。一个</font><code><font style="color:rgb(102, 102, 102);">server socket channel</font></code><font style="color:rgb(102, 102, 102);">准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">这四种事件用SelectionKey的四个常量来表示：</font>

<font style="color:rgb(102, 102, 102);"></font>

1. <code><font style="color:rgb(102, 102, 102);">SelectionKey.OP_CONNECT</font></code>
2. <code><font style="color:rgb(102, 102, 102);">SelectionKey.OP_ACCEPT</font></code>
3. <code><font style="color:rgb(102, 102, 102);">SelectionKey.OP_READ</font></code>
4. <code><font style="color:rgb(102, 102, 102);">SelectionKey.OP_WRITE</font></code>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：</font>

```c
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

<font style="color:rgb(102, 102, 102);">在下面还会继续提到</font><code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);">集合。</font>

<font style="color:rgb(102, 102, 102);"></font>

## 三、SelectionKey

<font style="color:rgb(102, 102, 102);">在上一小节中，当向 </font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);"> 注册 </font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);"> 时，</font><code><font style="color:rgb(102, 102, 102);">register()</font></code><font style="color:rgb(102, 102, 102);"> 方法会返回一个 </font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);"> 对象。这个对象包含了一些你感兴趣的属性：</font>

<font style="color:rgb(102, 102, 102);"></font>

* <code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);">集合</font>
* <code><font style="color:rgb(102, 102, 102);">ready</font></code><font style="color:rgb(102, 102, 102);">集合</font>
* <code><font style="color:rgb(102, 102, 102);">Channel</font></code>
* <code><font style="color:rgb(102, 102, 102);">Selector</font></code>
* <font style="color:rgb(102, 102, 102);">附加的对象（可选）</font>

<font style="color:rgb(102, 102, 102);">下面我会描述这些属性。</font>

<font style="color:rgb(102, 102, 102);"></font>

### 1、interest 集合

<font style="color:rgb(102, 102, 102);">就像</font>向 `Selector` 注册通道<font style="color:rgb(102, 102, 102);">一节中所描述的，</font><code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);"> 集合是你所选择的感兴趣的事件集合。可以通过</font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);">读写 </font><code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);"> 集合，像这样：</font>

```c
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = SelectionKey.OP_ACCEPT  == (interests & SelectionKey.OP_ACCEPT);
boolean isInterestedInConnect = SelectionKey.OP_CONNECT == (interests & SelectionKey.OP_CONNECT);
boolean isInterestedInRead    = SelectionKey.OP_READ    == (interests & SelectionKey.OP_READ);
boolean isInterestedInWrite   = SelectionKey.OP_WRITE   == (interests & SelectionKey.OP_WRITE);
```

<font style="color:rgb(102, 102, 102);">可以看到，用“位与”操作</font><code><font style="color:rgb(102, 102, 102);">interest </font></code><font style="color:rgb(102, 102, 102);">集合和给定的</font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);">常量，可以确定某个确定的事件是否在</font><code><font style="color:rgb(102, 102, 102);">interest </font></code><font style="color:rgb(102, 102, 102);">集合中。</font>

<font style="color:rgb(102, 102, 102);"></font>

### 2、ready 集合

<code><font style="color:rgb(102, 102, 102);">ready</font></code><font style="color:rgb(102, 102, 102);"> 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个 ready set。Selection 将在下一小节进行解释。可以这样访问 ready 集合：</font>

```c
int readySet = selectionKey.readyOps();
```

<font style="color:rgb(102, 102, 102);">可以用像检测</font><code><font style="color:rgb(102, 102, 102);">interest</font></code><font style="color:rgb(102, 102, 102);">集合那样的方法，来检测</font><code><font style="color:rgb(102, 102, 102);">channel</font></code><font style="color:rgb(102, 102, 102);">中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：</font>

```c
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

### 3、Channel + Selector

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">从</font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);">访问</font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);">和</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">很简单。如下：</font>

```c
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

### 4、附加的对象

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">可以将一个对象或者更多信息附着到</font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);">上，这样就能方便的识别某个给定的通道。例如，可以附加与通道一起使用的 </font><code><font style="color:rgb(102, 102, 102);">Buffer</font></code><font style="color:rgb(102, 102, 102);">，或是包含聚集数据的某个对象。使用方法如下：</font>

```c
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

<font style="color:rgb(102, 102, 102);">还可以在用</font><code><font style="color:rgb(102, 102, 102);">register()</font></code><font style="color:rgb(102, 102, 102);">方法向</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">注册</font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);">的时候附加对象。如：</font>

```c
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

## 四、通过 Selector 选择通道

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">一旦向</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">注册了一或多个通道，就可以调用几个重载的</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法会返回读事件已经就绪的那些通道。</font>

<font style="color:rgb(102, 102, 102);">下面是</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法：</font>

<font style="color:rgb(102, 102, 102);"></font>

* <code><font style="color:rgb(102, 102, 102);">int select()</font></code> ：<font style="color:rgb(102, 102, 102);">阻塞到至少有一个通道在你注册的事件上就绪了。</font>
* <code><font style="color:rgb(102, 102, 102);">int select(long timeout)</font></code> ： 阻塞直到一个通道在你注册的事件就绪，或是超时（时间单位为 ms）
* <code><font style="color:rgb(102, 102, 102);">int selectNow()</font></code> ：不会阻塞，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件

<code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法返回的</font><code><font style="color:rgb(102, 102, 102);">int</font></code><font style="color:rgb(102, 102, 102);">值表示有多少通道已经就绪。亦即，自上次调用</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法后有多少通道变成就绪状态。</font>

<font style="color:rgb(102, 102, 102);"></font>

### 1、selectedKeys()

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">一旦调用了</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用</font><code><font style="color:rgb(102, 102, 102);">selector</font></code><font style="color:rgb(102, 102, 102);">的</font><code><font style="color:rgb(102, 102, 102);">selectedKeys()</font></code><font style="color:rgb(102, 102, 102);">方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：</font>

```c
Set selectedKeys = selector.selectedKeys();
```

<font style="color:rgb(102, 102, 102);">当向 </font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);"> 注册 </font><code><font style="color:rgb(102, 102, 102);">Channel</font></code><font style="color:rgb(102, 102, 102);"> 时，</font><code><font style="color:rgb(102, 102, 102);">Channel.register()</font></code><font style="color:rgb(102, 102, 102);">方法会返回一个 </font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);"> 对象。这个对象代表了注册到该</font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);">的通道。可以通过</font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);">的</font><code><font style="color:rgb(102, 102, 102);">selectedKeySet()</font></code><font style="color:rgb(102, 102, 102);">方法访问这些对象。</font>

<font style="color:rgb(102, 102, 102);">可以遍历这个已选择的键集合来访问就绪的通道。如下：</font>

```c
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```

<font style="color:rgb(102, 102, 102);">这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">注意每次迭代末尾的</font><code><font style="color:rgb(102, 102, 102);">keyIterator.remove()</font></code><font style="color:rgb(102, 102, 102);">调用。Selector 不会自己从已选择键集中移除 SelectionKey 实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector 会再次将其放入已选择键集中。</font>

<code><font style="color:rgb(102, 102, 102);">SelectionKey.channel()</font></code><font style="color:rgb(102, 102, 102);">方法返回的通道需要转型成你要处理的类型，如</font><code><font style="color:rgb(102, 102, 102);">ServerSocketChannel</font></code><font style="color:rgb(102, 102, 102);">或</font><code><font style="color:rgb(102, 102, 102);">SocketChannel</font></code><font style="color:rgb(102, 102, 102);"> 等。</font>

<font style="color:rgb(102, 102, 102);"></font>

#### 2、wakeUp()

<font style="color:rgb(102, 102, 102);">某个线程调用</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法后阻塞了，即使没有通道已经就绪，也有办法让其从</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法返回。只要让其它线程在第一个线程调用</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法的那个对象上调用</font><code><font style="color:rgb(102, 102, 102);">Selector.wakeup()</font></code><font style="color:rgb(102, 102, 102);">方法即可。阻塞在</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法上的线程会立马返回。</font>

<font style="color:rgb(102, 102, 102);"></font>

<font style="color:rgb(102, 102, 102);">如果有其它线程调用了</font><code><font style="color:rgb(102, 102, 102);">wakeup()</font></code><font style="color:rgb(102, 102, 102);">方法，但当前没有线程阻塞在</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法上，下个调用</font><code><font style="color:rgb(102, 102, 102);">select()</font></code><font style="color:rgb(102, 102, 102);">方法的线程会立即“醒来（wake up）”。</font>

<font style="color:rgb(102, 102, 102);"></font>

#### 3、close()

<font style="color:rgb(102, 102, 102);">用完 Selector 后调用其</font><code><font style="color:rgb(102, 102, 102);">close()</font></code><font style="color:rgb(102, 102, 102);">方法会关闭该 Selector，且使注册到该 </font><code><font style="color:rgb(102, 102, 102);">Selector</font></code><font style="color:rgb(102, 102, 102);"> 上的所有 </font><code><font style="color:rgb(102, 102, 102);">SelectionKey</font></code><font style="color:rgb(102, 102, 102);"> 实例无效。通道本身并不会关闭。</font>

<font style="color:rgb(102, 102, 102);"></font>

## 五、完整的示例

<font style="color:rgb(102, 102, 102);">这里有一个完整的示例，打开一个 Selector，注册一个通道注册到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件（接受，连接，读，写）是否就绪。</font>

```c
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
  }
}
```

*<font style="color:rgb(102, 102, 102);"></font>*

> *<font style="color:rgb(102, 102, 102);">转载自 </font>*[并发编程网 – ifeve.com](http://ifeve.com/)***<font style="color:rgb(102, 102, 102);">本文链接地址:</font>**\_\_<font style="color:rgb(102, 102, 102);"> </font>*[Java NIO系列教程（六） Selector](http://ifeve.com/selectors/)


> 更新: 2022-04-09 16:53:15  
> 原文: <https://www.yuque.com/thinkspace/ulag78/sqxqwp>