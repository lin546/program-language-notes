# Java 集合之 Vector

## 一、Vector 简述

Vector是 JDK1.0 版本就推出的一个类，和ArrayList一样，继承自`AbstractList`，实现了`List`、`RandomAccess`、`Cloneable`、`java.io.Serializable`接口，底层也是基于数组实现的，不同的是它是一个线程安全的类。

## 二、Vector 实现

### 1、核心属性

```java
//底层存储数组
protected Object[] elementData;
//数组内实际存储元素个数
protected int elementCount;
//扩容容量
protected int capacityIncrement;
```

与ArrayList相比，核心属性多了一个扩容容量，可在初始化时指定，若不指定默认扩容2倍。

### 2、构造函数

```java
//initialCapacity：指定初始容量，capacityIncrement：指定扩容容量
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
 
//只指定初始容量，每次扩容2倍
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}
 
//无参构造，默认初始容量10，每次扩容2倍
  public Vector() {
    this(10);
}
 
//传入一个集合c，初始容量与c等于c的大小，每次扩容2倍
public Vector(Collection<? extends E> c) {
    elementData = c.toArray();
    elementCount = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
}
```

### 3、存储元素

和`ArrayList`不同的是，`Vector`里每个存储元素的方法都是`synchronized`修饰的，所以`Vector`是线程安全的，但是由于锁的存在，效率会比较低。

当对`Vector`进行元素添加的时候，都会检查底层数组的容量是否足够，若是不够则进行自动扩容，每次对数组进行增删改的时候都会增加`modCount`（继承自`AbstractList`的属性，用来统计修改次数），添加单个元素时一般会扩容2倍（如果构造函数中指定了扩容容量则以指定容量进行扩容），添加集合时则要取要添加的集合大小与自动扩容的大小的最大值进行扩容。

存储元素分为三种类型：追加，插入，替换，其中第二种和第三种类型都会检查下标是否越界（根据size属性而不是数组的长度）

（1）追加：这种方式最常用，直接添加到数组中最后一个元素的后面。

```java
public synchronized boolean add(E e) {
    modCount++;　　　　 //容量小于 elementCount+1 时自动扩容
    ensureCapacityHelper(elementCount + 1);　//将元素追加到数组末端的同时将elementCount+1
    elementData[elementCount++] = e;
    return true;
}
```

### 4、调整数组容量

（1）扩容：手动扩容和自动扩容，手动扩容用于加入Vector的数据量较大时，提前扩容至需要的容量，避免添加时递增式反复调用System.arraycopy带来的性能损耗，自动扩容则由添加元素时内存不够时触发。

```java
//对外开放的同步方法，提供手动扩容的功能，内部没有进行调用
public synchronized void ensureCapacity(int minCapacity) {
    if (minCapacity > 0) {
        modCount++;
        ensureCapacityHelper(minCapacity);
    }
}
//私密方法，内部自动扩容调用
private void ensureCapacityHelper(int minCapacity) {
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
//核心扩容逻辑
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //如果构造函数传入了扩容容量并且大于0，则按照指定的扩容容量进行扩容，否则默认扩容2倍（这里是本方法和ArrayList的唯一区别之处）
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
    //如果扩容后的容量小于新数组需要的最小容量，则扩容至新数组需要的最小容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //处理扩容后的容量大于预设的list最大值的情况
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
private static int hugeCapacity(int minCapacity) {
    //大于int最大值，抛出内存溢出异常
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //新数组需要的最小容量大于预设的list最大值则扩容至int的最大值，否则扩容至预设的list最大值
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

## 参考

* <https://www.cnblogs.com/despacito/p/10826934.html>


> 更新: 2022-06-23 22:48:10  
> 原文: <https://www.yuque.com/thinkspace/ulag78/trdxb4>