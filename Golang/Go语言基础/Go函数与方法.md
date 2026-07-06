# Go 函数与方法

## **一、defer 用法**
defer是Go语言提供的一种延迟调用机制，defer的运作离不开函数。怎么理解呢？这句话至少有以下两点含义：

+ 在Go中，只有在函数（和方法）内部才能使用defer；
+ defer关键字后面只能接函数（或方法），这些函数被称为deferred函数。defer将它们注册到其所在Goroutine中，用于存放deferred函数的栈数据结构中，这些deferred函数将在执行defer的函数退出前，按后进先出（LIFO）的顺序被程序调度执行。
+ 而且，无论是执行到函数体尾部返回，还是在某个错误处理分支显式return，又或是出现panic，已经存储到deferred函数栈中的函数，都会被调度执行。所以说，deferred函数是一个可以在任何情况下为函数进行收尾工作的好“伙伴”。

除了释放资源这个最基本、最常见的用法之外，defer 的运行机制决定了它还可以在其他一些场合发挥作用。

### **1、拦截 panic**
defer 的第二个重要用途就是拦截 panic，并按需要对 panic 进行处理，可以尝试从 panic 中恢复（这也是 Go 语言中唯一的从 panic 中恢复的手段），也可以如下面标准库代码中这样触发一个新 panic，但为新 panic 传一个新的 error 值。

```plain
func makeSlice(n int) []byte {
        defer func() {
                if recover() != nil {
                        panic(ErrTooLarge)
                }
        }
        return make([]byte, n)
}
```

下面的代码则通过 deferred 函数拦截 panic 并恢复了程序的运行。

```plain
func bar() {
        fmt.Println("raise a panic")
        panic(-1)
}
func foo() {
        defer func() {
                if e := recover(); e != nil {
                        fmt.Println("recovered from a panic")
                }
        }()
        bar()
}
```

### **2、修改函数的具名返回值**
下面是 Go 标准库中通过 deferred 函数访问函数具名返回值变量的两个例子:

```plain
func foo(a, b int) (x, y int) {
        defer func() {
                x = x * 5
                y = y * 10
        }()
        x = a + 5
        y = b + 6
        return
}
```

运行这个程序，可以看到，x，y 在返回给调用者之前，会分别乘以5倍和10倍。

## **二、Go 方法**
和函数相比，Go 语言中的方法在声明形式上仅仅多了一个参数，Go 称之为 receiver 参数。receiver 参数是方法与类型之间的按钮。

Go 方法的一般声明形式如下：

```plain
func (receiver T/*T) 方法名(参数列表) (返回参数) {
    函数体
}
```

无论receiver参数的类型为*T还是T，我们都把一般声明形式中的T叫做receiver参数t的基类型。通过 receiver，上述方法被绑定到类型 T上。换句话说，上述方法是类型 T 的一个方法，我们可以通过类型 T 或 *T 的实例调用该方法。

 Go 方法具有以下特点：

+ 方法名的首字母是否大写决定了该方法是不是导出方法
+ 方法定义要与类型定义放在同一个包内。由此我们可以推出：
    - 不能为原生类型（如 int、float64、map等）添加方法，只能为自定义类型定义方法。
    - 不能横跨 Go 包为其他包内的自定义类型定义方法。
+ receiver 参数的基类型本身不能为指针类型或接口类型。
+ Go 语法糖使得我们在通过类型实例调用类型方法时无须考虑实例类型与receiver参数类型是否一致，编译器会为我们做自动转换。

### **1、Go 方法的本质**
Go 方法的本质：一个以方法所绑定类型实例为第一个参数的普通函数。

```plain
fun (t T) M1() <=> M1(t T)
fun (t *T) M1() <=> M1(t *T)
```

理解了Go方法的本质，就可以以此来选择正确的 receiver 类型。

+ 如果要对类型实例进行修改，那么选择 *T 类型的 receiver 参数。
+ 如果没有对类型实例修改的需求，那么为 receiver 选择 T 类型或 *T 类型均可。但考虑到Go方法调用时，receiver 是以值复制的形式传入到方法中的，如果类型的size较大，以值传递传入会导致较大损失，这时选择 *T 作为 receiver 类型会好点。

关于receiver类型的选择其实还有一个重要因素，那就是类型是否要实现某个接口。

+ 方法集合是类型与接口间隐式关系的纽带，只有当类型的方法集合是某接口类型的超集时，我们才说该类型实现了某接口。
+ 类型T的方法集合是以T为receiver类型的所有方法的集合，类型*T的方法集合是*T为receiver类型的所有方法的集合与类型T的方法集合的并集。

### **2、变长参数函数**
变长参函数就是指调用时可以接受零个、一个或多个实际参数的函数。就像下面对fmt.Println的调用那样。

```plain
fmt.Println()
fmt.Println("hello","world")
fmt.Println("hello","world","!")
```

对照下面 fmt.Println函数的原型：

```plain
func Println(a ...interface)(n int,err error)
```

这种接受...T类型形式参数的函数被称为变长参数函数。变长参数函数需要注意以下几点：

+ 一个变长参数函数只能有一个...T，并且只能为函数参数列表中的最后一个形式参数。
+ ...T在函数体内呈现为[]T类型的变量，在函数外部，...T类型形式参数可匹配和接受的实参类型有两种（只能选同时择一种）：
    - 多个T类型变量
    - t...（t为[]T类型变量）



> 更新: 2024-10-03 22:02:47  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/fqxga9lbxhy0vfh8>