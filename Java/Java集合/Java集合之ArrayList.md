# Java 集合之 ArrayList

## 一、基本用法
### 1、新建 ArrayList
ArrayList是一个泛型容器，新建ArrayList需要实例化泛型参数，比如：

```java
ArrayList<Integer> intList = new ArrayList<Integer>();
ArrayList<String> strList = new ArrayList<String>();
```

### 2、添加元素
add方法添加元素到末尾

```java
ArrayList<Integer> intList = new ArrayList<Integer>();
intList.add(123);
intList.add(456);
ArrayList<String> strList = new ArrayList<String>();
strList.add("老马");
strList.add("编程");
```

### 3、长度方法
```java
//判断是否为空
public boolean isEmpty()

//获取长度
public int size()
```

### 4、访问指定位置的元素
```java
public E get(int index)
```

如：

```java
ArrayList<String> strList = new ArrayList<String>();
strList.add("老马");
strList.add("编程");
for(int i=0; i<strList.size(); i++){
    System.out.println(strList.get(i));
}
```

### 5、查找元素
```java
public int indexOf(Object o) //如果找到，返回索引位置，否则返回-1。
public int lastIndexOf(Object o)  //从后往前找
```

是否包含指定元素，相同的依据是equals方法返回true。如果传入的元素为null，则找null的元素。

```java
public boolean contains(Object o)
```

### 6、删除元素
```java
public E remove(int index)   //删除指定对象
```

与indexOf一样，比较的依据的是equals方法，如果o为null，则删除值为null的元素。另外，remove只删除第一个相同的对象，也就是说，即使ArrayList中有多个与o相同的元素，也只会删除第一个。返回值为boolean类型，表示是否删除了元素。

### 7、删除所有元素
```java
public void clear()
```

### 8、插入元素
在指定位置插入元素

```java
public void add(int index, E element)
```

index为0表示插入最前面，index为ArrayList的长度表示插到最后面。

### 9、修改元素
修改指定位置的元素内容

```java
public E set(int index, E element)
```

## 二、基本原理
### 1、内部构成
可以看出，ArrayList的基本用法是比较简单的，它的基本原理也是比较简单的，内部有一个数组elementData，一般会有一些预留的空间，有一个整数size记录实际的元素个数，如下所示：

```java
private transient Object[] elementData;
private int size;
```

我们暂时可以忽略transient这个关键字。各种public方法内部操作的基本都是这个数组和这个整数，elementData会随着实际元素个数的增多而重新分配，而size则始终记录实际的元素个数。

### 2、add 方法
虽然基本思路是简单的，但内部代码有一些比较晦涩，我们来看下add方法的代码：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

它首先调用ensureCapacityInternal确保数组容量是够的，ensureCapacityInternal的代码是：

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```

它先判断数组是不是空的，如果是空的，则首次至少要分配的大小为DEFAULT_CAPACITY，DEFAULT_CAPACITY的值为10，接下来调用ensureExplicitCapacity，代码为：

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

modCount++ 是什么意思呢？modCount表示内部的修改次数，modCount++当然就是增加修改次数，为什么要记录修改次数呢？我们待会解释。

如果需要的长度大于当前数组的长度，则调用grow方法。接下来，看grow方法：

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

排除边缘情况，长度增长的主要代码为：

```java
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

右移一位相当于除2，所以，newCapacity相当于oldCapacity的1.5倍。

### 3、remove 方法
我们再来看Remove方法的代码：

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

## 三、ArrayList 的其他方法
## 1、构造方法
ArrayList还有两个构造方法

```java
public ArrayList(int initialCapacity)
public ArrayList(Collection<? extends E> c)
```

第一个方法以指定的大小initialCapacity初始化内部的数组大小，代码为：

```java
this.elementData = new Object[initialCapacity];
```

在事先知道元素长度的情况下，或者，预先知道长度上限的情况下，使用这个构造方法可以避免重新分配和拷贝数组。

第二个构造方法以一个已有的Collection构建，数据会新拷贝一份。

## 2、与数组的相互转换
ArrayList中有两个方法可以返回数组

```java
public Object[] toArray()
public <T> T[] toArray(T[] a)
```

第一个方法返回是Object数组，代码为：

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

第二个方法返回对应类型的数组，如果参数数组长度足以容纳所有元素，就使用该数组，否则就新建一个数组，比如：

```java
ArrayList<Integer> intList = new ArrayList<Integer>();
intList.add(123);
intList.add(456);
intList.add(789);

Integer[] arrA = new Integer[3];
intList.toArray(arrA);
Integer[] arrB = intList.toArray(new Integer[0]);

System.out.println(Arrays.equals(arrA, arrB));
```

输出为true，表示两种方式都是可以的。

Arrays中有一个静态方法asList可以返回对应的List，如下所示：

```java
Integer[] a = {1,2,3};
List<Integer> list = Arrays.asList(a);
```

需要注意的是，这个方法返回的List，它的实现类并不是本节介绍的ArrayList，而是Arrays类的一个内部类，在这个内部类的实现中，内部用的的数组就是传入的数组，没有拷贝，也不会动态改变大小，所以对数组的修改也会反映到List中，对List调用add/remove方法会抛出异常。

要使用ArrayList完整的方法，应该新建一个ArrayList，如下所示：

```java
List<Integer> list = new ArrayList<Integer>(Arrays.asList(a));
```

## 参考
+ [https://www.cnblogs.com/swiftma/p/5894874.html](https://www.cnblogs.com/swiftma/p/5894874.html)



> 更新: 2022-06-21 23:16:49  
> 原文: <https://www.yuque.com/thinkspace/ulag78/uoq7pp>