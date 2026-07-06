# PHP 中的NULL

<font style="color:rgb(36,91,219);">php中很多人还不懂php中 0 , '' , null 和 false 之间的区别,这些区别有时会影响到数据判断的正确性和安全性，给程序的测试运行造成很多麻烦。另外在面试题中也会遇到这些问题，如下:</font>

```plain
<?php
    $str1 = null;
    $str2 = false;
    echo $str1==$str2 ? ‘相等’ : ‘不相等’;
    $str3 = "";
    $str4 = 0;
    echo $str3==$str4 ? ‘相等’ : ‘不相等’;
    $str5 = 0;
    $str6 = '0';
    echo $str5===$str6 ? ‘相等’ : ‘不相等’;
    $str7=0;
    $str8=false;
    echo $str7==$str8 ? ‘相等’ : ‘不相等’;
?>
运行结果：
//相等，相等，不相等,相等。
```

原因是在PHP中变量是以C语言的结构体来存储的，<u>空字符串和NULL,false都是以值为0存储的</u>，其中这个结构体有个zend\_uchar type;这样的成员变量，他是用来保存变量的类型的，而<u>空字符串的类型是string，NULL的类型是NULL,false是boolean。</u>

这一点可以用`echo gettype('')`;和`echo gettype(NULL)`;来打印看看！而`===`运算符是不单比较值，还有比较类型的，所以第三个为false！另外再说下，php中 ：

\= 一个等号是赋值

\== 两个等号是判断相等且只比较值，不比较类型

\=== 三个等号是判断值和类型都相等

!= 不等于符号，只比较值，不管类型

!== 不全等符号，比较值和类型

所以空字符串`('')`，`false`,`NULL`和`0`是值相等而类型不一样！

注意：NULL是一种特殊的类型.两种情况下为NULL。

1. $var = NULL;
2. $var;
3. ""、0、"0"、NULL、FALSE、`array()`、var $var; 以及没有任何属性的对象都将被认为是空的。


> 更新: 2024-10-03 13:46:35  
> 原文: <https://www.yuque.com/thinkspace/du51gc/fz5ham9eo3a4t6ky>