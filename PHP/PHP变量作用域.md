# PHP 变量作用域

变量的作用域是脚本中变量可被引用和使用的部分，PHP 中有四种不同的变量作用域:

+ local：局部作用域
+ global：全局作用域
+ static：静态作用域
+ parameter：函数参数作用域

### 1、**local（本地的）**
在 PHP 函数内部声明的变量是局部变量，仅能在函数内部访问。

```plain
<?php
function test()
{
    $a = 15;
    echo "内部输出结果：" . $a;
}
echo "外部输出结果：" . $a;  // 无法访问变量 a
echo PHP_EOL;
test();
?>
```

### 2、global（总体的）
在所有函数外部定义的变量是全局变量，除了函数外，全局变量可以被脚本中的任何部分访问、要在一个函数中访问一个全局变量，需要使用 global 关键字。

```plain
<?php
$x = 5;
$y = 10;
$z = 0;

function test()
{
    global $x,$y,$z;
    $z = $x + $y;
}

test();
echo $z;
?>
```

结果：**15**

### 3、**static（静态的）**
当一个函数执行完成时，它的所有变量通常都会被删除。

然而，有时需要局部变量不要被删除，要做到这一点，请在您第一次声明变量时使用 static 关键字。

```plain
<?php
function test() {
    static $x=0;
    echo $x . " ";
    $x++;
}

test();
test();
test();
test();
?>
```

结果：**0 1 2 3**

每次调用函数时， 该变量将会保留请前函的前被调用的值一次。

### 4、**parameter（参数）**
```plain
<?php
function myTest($x)
{
    echo $x;
}
myTest('Galois');
myTest(8888);
```



> 更新: 2024-10-03 13:47:32  
> 原文: <https://www.yuque.com/thinkspace/du51gc/uzn1cglb6w9htygr>