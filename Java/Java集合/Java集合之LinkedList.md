# Java 集合之 LinkedList

## 一、用法
### 构造方法
LinkedList的构造方法与ArrayList类似，有两个，一个是默认构造方法，另外一个可以接受一个已有的Collection，如下所示：

```java
public LinkedList()
public LinkedList(Collection<? extends E> c)
```

比如，可以这么创建：

```java
List<String> list = new LinkedList<>();
List<String> list2 = new LinkedList<>(
        Arrays.asList(new String[]{"a","b","c"}));
```

## 二、实现原理
### 内部组成
我们知道，ArrayList 内部是数组，元素在内存是连续存放的，但LinkedList不是。LinkedList直译就是链表，确切的说，它的内部实现是双向链表，每个元素在内存都是单独存放的，元素之间通过链接连在一起，类似于小朋友之间手拉手一样。

为了表示链接关系，需要一个节点的概念，节点包括实际的元素，但同时有两个链接，分别指向前一个节点(前驱)和后一个节点(后继)，节点是一个内部类，具体定义为：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

Node类表示节点，item指向实际的元素，next指向下一个节点，prev指向前一个节点。

LinkedList内部组成就是如下三个实例变量：

```java
transient int size = 0;
transient Node<E> first;
transient Node<E> last;
```

我们暂时忽略transient关键字，size表示链表长度，默认为0，first指向头节点，last指向尾节点，初始值都为null。

LinkedList的所有public方法内部操作的都是这三个实例变量，具体是怎么操作的？链接关系是如何维护的？我们看一些主要的方法，先来看add方法。

### Add 方法
add方法的代码为：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

主要就是调用了linkLast，它的代码为：

```java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

代码的基本步骤是：

1. 创建一个新的节点newNode。prev指向原来的尾节点，如果原来链表为空，则为null。代码为：

```java
Node<E> newNode = new Node<>(l, e, null);
```

2. 修改尾节点last，指向新的最后节点newNode。代码为：

```java
last = newNode;
```

3. 修改前节点的后向链接，如果原来链表为空，则让头节点指向新节点，否则让前一个节点的next指向新节点。代码为：

```java
if (l == null)
    first = newNode;
else
    l.next = newNode;
```

4. 增加链表大小。代码为：

```java
size++
```

modCount++的目的与ArrayList是一样的，记录修改次数，便于迭代中间检测结构性变化。

### 删除元素
我们再来看删除元素，代码为：

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

通过node方法找到节点后，调用了unlink方法，代码为：

```java
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

删除x节点，基本思路就是让x的前驱和后继直接链接起来，next是x的后继，prev是x的前驱，具体分为两步：

+ 第一步是让x的前驱的后继指向x的后继。如果x没有前驱，说明删除的是头节点，则修改头节点指向x的后继。
+ 第二步是让x的后继的前驱指向x的前驱。如果x没有后继，说明删除的是尾节点，则修改尾节点指向x的前驱。

## 三、LinkedList 总结
+ 按需分配空间，不需要预先分配很多空间
+ 不可以随机访问，按照索引位置访问效率比较低，必须从头或尾顺着链接找，效率为O(N/2)。
+ 不管列表是否已排序，只要是按照内容查找元素，效率都比较低，必须逐个比较，效率为O(N)。
+ 在两端添加、删除元素的效率很高，为O(1)。
+ 在中间插入、删除元素，要先定位，效率比较低，为O(N)，但修改本身的效率很高，效率为O(1)。

理解了LinkedList和ArrayList的特点，我们就能比较容易的进行选择了，如果列表长度未知，添加、删除操作比较多，尤其经常从两端进行操作，而按照索引位置访问相对比较少，则LinkedList就是比较理想的选择。

## 参考
+ [https://www.cnblogs.com/swiftma/p/5914153.html](https://www.cnblogs.com/swiftma/p/5914153.html)



> 更新: 2022-06-23 22:50:34  
> 原文: <https://www.yuque.com/thinkspace/ulag78/ggt3ie>