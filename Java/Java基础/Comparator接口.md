# Comparator 接口

在使用Java自带的排序函数时，往往需要根据自己的需求自定义比较器。以前一直对Comparator的升序降序疑惑。现在记录一下，加深下印象。先给结论：实现Comparator接口，必须实现下面这个函数：

```java
@Override
public int compare(CommentVo o1, CommentVo o2) {
           return o1.getTime().compareTo(o2.getTime());
}
```

这里o1表示位于前面的对象，o2表示后面的对象返回-1（0或负数），表示不需要交换o1和o2的位置，o1排在o2前面，asc返回1（或正数），表示需要交换01和02的位置，o1排在o2后面，desc举例说明：（分析说明在运行结果之后）package com.zhb.test;

```java

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

class A {
    int a;
    public A(int a) {
        this.a = a;
    }
    @Override
    public String toString() {
        return "[a=" + a + "]";
    }
}

class MyComparator implements Comparator<A> {
    @Override
    public int compare(A o1, A o2) {
        //升序
        //return o1.a - o2.a;
        //降序：后面会具体分析为什么降序
        return o2.a - o1.a;
    }
}

public class ComparatorTest {
    public static void main(String[] args) {
        A a1 = new A(5);
        A a2 = new A(7);
        List<A> list = new ArrayList<A>();
        list.add(a1);
        list.add(a2);
        Collections.sort(list, new MyComparator());
        System.out.println(list);
    }
}
```

输出结果：下面来用我们之前的结论解释为什么return o2.a - o1.a;就是降序了：\
首先o2是第二个元素，o1是第一个元素。无非就以下这些情况：

* `o2.a > o1.a` : 那么此时返回正数，表示需要调整o1,o2的顺序，也就是需要把o2放到o1前面，这不就是降序了么。
* `o2.a <= o1.a` : 那么此时返回负数，表示不需要调整，也就是此时o1 比 o2大， 不还是降序么。这样一分析下来，就很明白了。


> 更新: 2022-06-24 00:38:17  
> 原文: <https://www.yuque.com/thinkspace/ulag78/kwk0cu>