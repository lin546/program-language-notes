# Java 集合之 TreeSet

前面介绍了HashSet，我们提到，HashSet有一个重要局限，元素之间没有特定的顺序，我们还提到，Set接口还有另一个重要的实现类TreeSet，它是有序的，与HashSet和HashMap的关系一样，TreeSet是基于TreeMap的，上节我们介绍了TreeMap，本节我们来详细讨论TreeSet。

下面，我们先来看TreeSet的用法，然后看实现原理，最后总结分析TreeSet的特点。

## 一、基本用法
### 1、构造方法
TreeSet的基本构造方法有两个：

```java
public TreeSet()
public TreeSet(Comparator<? super E> comparator)
```

默认构造方法假定元素实现了Comparable接口，第二个使用传入的比较器，不要求元素实现Comparable。

### 2、基本例子
TreeSet经常也只是当做Set使用，只是希望迭代输出有序，如下面代码所示：

```java
Set<String> words = new TreeSet<String>();
words.addAll(Arrays.asList(new String[]{
    "tree", "map", "hash", "map",     
}));
for(String w : words){
    System.out.print(w+" ");
}
```

输出为：

```java
hash map tree
```

TreeSet实现了两点：排重和有序。

如果希望不同的排序，可以传递一个Comparator，如下所示：

```java
Set<String> words = new TreeSet<String>(new Comparator<String>(){

    @Override
    public int compare(String o1, String o2) {
        return o1.compareToIgnoreCase(o2);
    }});
words.addAll(Arrays.asList(new String[]{
    "tree", "map", "hash", "Map",     
}));
System.out.println(words);
```

忽略大小写进行比较，输出为：

```java
[hash, map, tree]
```

需要注意的是，Set是排重的，排重是基于比较结果的，结果为0即视为相同，"map"和"Map"虽然不同，但比较结果为0，所以只会保留第一个元素。

以上就是TreeSet的基本用法，简单易用。不过，因为有序，TreeSet还实现了NavigableSet和SortedSet接口，NavigableSet扩展了SortedSet，此外，TreeSet还有几个构造方法，我们来看下。

## 二、基本实现原理
前面介绍过，HashSet是基于HashMap实现的，元素就是HashMap中的键，值是一个固定的值，TreeSet是类似的，它是基于TreeMap实现的，我们具体来看一下代码，先看其内部组成。

### 1、内部组成
TreeSet的内部有如下成员：

```java
private transient NavigableMap<E,Object> m;
private static final Object PRESENT = new Object();
```

m就是背后的那个TreeMap，这里用的是更为通用的接口类型NavigableMap，PRESENT就是那个固定的共享值。

TreeSet的方法实现主要就是调用m的方法，我们具体来看下。

### 2、构造方法
几个构造方法的代码为：

```java
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

public TreeSet() {
    this(new TreeMap<E,Object>());
}

public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```

### 3、添加元素
add方法的代码为：

```java
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

就是调用map的put方法，元素e用作键，值就是固定值PRESENT，put返回null表示原来没有对应的键，添加成功了。

### 4、检查是否包含元素
代码为：

```java
public boolean contains(Object o) {
    return m.containsKey(o);
}
```

就是检查map中是否包含对应的键。

### 5、删除元素
代码为：

```java
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}
```

就是调用map的remove方法，返回值为PRESENT表示原来有对应的键且删除成功了。

### 6、子集视图
subSet方法的代码：

```java
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                              E toElement,   boolean toInclusive) {
    return new TreeSet<>(m.subMap(fromElement, fromInclusive,
                                   toElement,   toInclusive));
}
```

先调用subMap方法获取NavigatebleMap的子集，然后调用内部的TreeSet构造方法。

### 7、头尾操作
代码为：

```java
public E first() {
    return m.firstKey();
}

public E last() {
    return m.lastKey();
}

 

public E pollFirst() {
    Map.Entry<E,?> e = m.pollFirstEntry();
    return (e == null) ? null : e.getKey();
}

public E pollLast() {
    Map.Entry<E,?> e = m.pollLastEntry();
    return (e == null) ? null : e.getKey();
}
```

代码都比较简单，就不解释了。

### 8、逆序遍历
代码为：

```java
public Iterator<E> descendingIterator() {
    return m.descendingKeySet().iterator();
}

public NavigableSet<E> descendingSet() {
    return new TreeSet<>(m.descendingMap());
}
```

也很简单。

### 实现原理小结
TreeSet的实现代码都比较简单，主要就是调用内部NavigatableMap的方法。

## 三、TreeSet特点分析
与HashSet相比，TreeSet同样实现了Set接口，但内部基于TreeMap实现，而TreeMap基于大致平衡的排序二叉树 - 红黑树，这决定了它有如下特点：

+ 没有重复元素
+ 添加、删除元素、判断元素是否存在，效率比较高，为O(log2(N))，N为元素个数。
+ 有序，TreeSet同样实现了SortedSet和NavigatableSet接口，可以方便的根据顺序进行查找和操作，如第一个、最后一个、某一取值范围、某一值的邻近元素等。
+ 为了有序，TreeSet要求元素实现Comparable接口或通过构造方法提供一个Comparator对象。

## 小结
本节介绍了TreeSet的用法和实现原理，在用法方面，它实现了Set接口，但有序，同样实现了SortedSet和NavigatableSet接口，在内部实现上，它使用了TreeMap，代码比较简单。

至此，我们已经介绍完了Java中主要常见的容器接口和实现类，接口主要有队列(Queue)，双端队列(Deque)，列表(List)，Map和Set，实现类有ArrayList, LinkedList, HashMap, TreeMap, HashSet和TreeSet。

关于接口Queue, Deque, Map和Set，Java容器类中还有其他一些实现类，它们各有特点，让我们在接下来的几节中继续探索。



> 更新: 2022-06-24 00:40:28  
> 原文: <https://www.yuque.com/thinkspace/ulag78/fxq8bk>