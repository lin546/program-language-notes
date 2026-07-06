# Java 集合之 HashSet

上节介绍了HashMap，提到了Set接口，Map接口的两个方法keySet和entrySet返回的都是Set，本节，我们来看Set接口的一个重要实现类HashSet。

与HashMap类似，字面上看，HashSet由两个单词组成，Hash和Set，Set表示接口，实现Set接口也有多种方式，各有特点，HashSet实现的方式利用了Hash。

下面，我们先来看HashSet的用法，然后看实现原理，最后我们总结分析下HashSet的特点。

## 一、用法
### 接口
Set表示的是没有重复元素、且不保证顺序的容器接口，它扩展了Collection，但没有定义任何新的方法，不过，对于其中的一些方法，它有自己的规范。

Set接口的完整定义为：

```java
public interface Set<E> extends Collection<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean retainAll(Collection<?> c);
    boolean removeAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();
}
```

与Collection接口中定义的方法是一样的，不过，一些方法有一些不同的规范要求。

### 添加元素
```java
boolean add(E e);
```

如果集合中已经存在相同元素了，则不会改变集合，直接返回false，只有不存在时，才会添加，并返回true。

### 批量添加
```java
boolean addAll(Collection<? extends E> c);
```

重复的元素不添加，不重复的添加，如果集合有变化，返回true，没变化返回false。

### 迭代器
```java
Iterator<E> iterator();
```

迭代遍历时，不要求元素之间有特别的顺序。HashSet的实现就是没有顺序，但有的Set实现可能会有特定的顺序，比如TreeSet，我们后续章节介绍。

### HashSet
与HashMap类似，HashSet的构造方法有：

```java
public HashSet()
public HashSet(int initialCapacity)
public HashSet(int initialCapacity, float loadFactor)
public HashSet(Collection<? extends E> c)
```

initialCapacity和loadFactor的含义与HashMap中的是一样的，待会我们再细看。

HashSet的使用也很简单，比如：

```java
Set<String> set = new HashSet<String>();
set.add("hello");
set.add("world");
set.addAll(Arrays.asList(new String[]{"hello","老马"}));

for(String s : set){
    System.out.print(s+" ");
}
```

输出为：

```java
hello 老马 world
```

"hello"被添加了两次，但只会保存一份，输出也没有什么特别的顺序。

### hashCode与equals
与HashMap类似，HashSet要求元素重写hashCode和equals方法，且对两个对象，equals相同，则hashCode也必须相同，如果元素是自定义的类，需要注意这一点。

### 应用场景
HashSet有很多应用场景，比如说：

+ 排重，如果对排重后的元素没有顺序要求，则HashSet可以方便的用于排重。
+ 保存特殊值，Set可以用于保存各种特殊值，程序处理用户请求或数据记录时，根据是否为特殊值，进行特殊处理，比如保存IP地址的黑名单或白名单。
+ 集合运算，使用Set可以方便的进行数学集合中的运算，如交集、并集等运算，这些运算有一些很现实的意义。比如用户标签计算，每个用户都有一些标签，两个用户的标签交集就表示他们的共同特征，交集大小除以并集大小可以表示他们的相似长度。

## 二、实现原理
内部组成

HashSet内部是用HashMap实现的，它内部有一个HashMap实例变量，如下所示：

```java
private transient HashMap<E,Object> map;
```

我们知道，Map有键和值，HashSet相当于只有键，值都是相同的固定值，这个值的定义为：

```java
private static final Object PRESENT = new Object();
```

理解了这个内部组成，它的实现方法也就比较容易理解了，我们来看下代码。

### 构造方法
HashSet的构造方法，主要就是调用了对应的HashMap的构造方法，比如：

```java
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

public HashSet() {
    map = new HashMap<>();
}
```

接受Collection参数的构造方法稍微不一样，代码为：

```java
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

也很容易理解，c.size()/.75f用于计算initialCapacity，0.75f是loadFactor的默认值。

### 添加元素
我们看add方法的代码：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

就是调用map的put方法，元素e用于键，值就是那个固定值PRESENT，put返回null表示原来没有对应的键，添加成功了。HashMap中一个键只会保存一份，所以重复添加HashMap不会变化。

### 检查是否包含元素
代码为：

```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

就是检查map中是否包含对应的键。

删除元素

代码为：

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

就是调用map的remove方法，返回值为PRESENT表示原来有对应的键且删除成功了。

### 迭代器
代码为：

```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```

就是返回map的keySet的迭代器。

## 三、HashSet特点分析
HashSet实现了Set接口，内部是通过HashMap实现的，这决定了它有如下特点：

+ 没有重复元素
+ 可以高效的添加、删除元素、判断元素是否存在，效率都为O(1)。
+ 没有顺序  
如果需求正好符合这些特点，那HashSet就是一个理想的选择。

## 小结
本节介绍了HashSet的用法和实现原理，它实现了Set接口，不含重复元素，内部实现利用了HashMap，可以方便高效地实现如去重、集合运算等功能。

同HashMap一样，HashSet没有顺序，如果要保持添加的顺序，可以使用HashSet的一个子类LinkedHashSet。Set还有一个重要的实现类，TreeSet，它可以排序。这两个类，我们留待后续章节介绍。

HashMap和HashSet的共同实现机制是哈希表，Map和Set还有一个重要的共同实现机制，树，实现类分别是TreeMap和TreeSet，让我们在接下来的两节中探讨。



> 更新: 2022-06-17 15:04:25  
> 原文: <https://www.yuque.com/thinkspace/ulag78/hsxpqr>