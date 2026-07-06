# fmt

fmt包实现了类似C语言printf和scanf的格式化I/O。主要分为向外输出内容和获取输入内容两大部分。

## 一、向外输出

### 1、Print

`Print`系列函数会将内容输出到系统的标准输出，区别在于`Print`函数直接输出内容，`Printf`函数支持格式化输出字符串，`Println`函数会在输出内容的结尾添加一个换行符。

```plain
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

举个简单的例子

```plain
func main() {
    fmt.Print("在终端打印该信息。")
    name := "沙河小王子"        
    fmt.Printf("我是：%s\n", name)
    fmt.Println("在终端打印单独一行显示")
}
```

### 2、Fprint

`Fprint`系列函数会将内容输出到一个`io.Writer`接口类型的变量`w`中，我们通常用这个函数往文件中写入内容。

```plain
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

举个例子：

```plain
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
        fmt.Println("打开文件出错，err:", err)
        return
}
name := "沙河小王子"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

注意，只要满足`io.Writer`接口的类型都支持写入。

### 3、Sprint

`Sprint`系列函数会把传入的数据生成并返回一个字符串。

```plain
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

简单的示例代码如下：

```plain
s1 := fmt.Sprint("沙河小王子")
name := "沙河小王子"
age := 18
s2 := fmt.Sprintf("name:%s,age:%d", name, age)
s3 := fmt.Sprintln("沙河小王子")
fmt.Println(s1, s2, s3)
```

### 4、Errorf

`Errorf`函数根据format参数生成格式化字符串并返回一个包含该字符串的错误。

```plain
func Errorf(format string, a ...interface{}) error
```

通常使用这种方式来自定义错误类型，例如：

```plain
err := fmt.Errorf("这是一个错误")
```

Go1.13版本为`fmt.Errorf`函数新加了一个`%w`占位符用来生成一个可以包裹Error的Wrapping Error。

## 二、格式化占位符

`*printf`系列函数都支持format格式化参数，在这里我们按照占位符将被替换的变量类型划分，方便查询和记忆。

### 1、通用占位符

* %v：值的默认格式表示

The default format for %v is:

```plain
bool:                    %t
int, int8 etc.:          %d
uint, uint8 etc.:        %d, %#x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p
pointer:                 %p
```

* %+v：类似%v，但输出结构体时会添加字段名
* %#v：值的Go语法表示
* %T：打印值的类型
* %%：百分号

### 2、布尔型

* %t：true或false

### 3、整型

* %b 表示为二进制
* %c 该值对应的unicode码值
* %d 表示为十进制
* %o 表示为八进制
* %x 表示为十六进制，使用a-f
* %X 表示为十六进制，使用A-F
* %U 表示为Unicode格式：U+1234，等价于"U+%04X"
* %q 该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示

### 4、浮点数与复数

* %b 无小数部分、二进制指数的科学计数法，如-123456p-78
* %e 科学计数法，如-1234.456e+78
* %E 科学计数法，如-1234.456E+78
* %f 有小数部分但无指数部分，如123.456
* %F 等价于%f
* %g 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）
* %G 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）

### 5、字符串和\[]byte

* %s 直接输出字符串或者\[]byte
* %q 该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示
* %x 每个字节用两字符十六进制数表示（使用a-f
* %X 每个字节用两字符十六进制数表示（使用A-F）

### 6、Slice

* %p address of 0th element in base 16 notation, with leading 0x

### 6、指针

* %p base 16 notation, with leading 0x

The %b, %d, %o, %x and %X verbs also work with pointers,formatting the value exactly as if it were an integer.

### 7、宽度标识符

宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。举例如下：

* %f 默认宽度，默认精度
* %9f 宽度9，默认精度
* %.2f 默认宽度，精度2
* %9.2f 宽度9，精度2
* %9.f 宽度9，精度0

### 8、其他flag

* '+' always print a sign for numeric values; guarantee ASCII-only output for %q (%+q)
* '-' pad with spaces on the right rather than the left (left-justify the field)
* '#' alternate format: add leading 0b for binary (%#b), 0 for octal (%#o),
  * 0x or 0X for hex (%#x or %#X); suppress 0x for %p (%#p);for %q, print a raw (backquoted) string if strconv.CanBackquote
  * returns true;
  * always print a decimal point for %e, %E, %f, %F, %g and %G;
  * do not remove trailing zeros for %g and %G;
  * write e.g. U+0078 'x' if the character is printable for %U (%#U).
* ' ' (space) leave a space for elided sign in numbers (% d);

put spaces between bytes printing strings or slices in hex (% x, % X)

* '0' pad with leading zeros rather than spaces;for numbers, this moves the padding after the sign;

ignored for strings, byte slices and byte arrays

## 参考：

* https://devdocs.io/go/fmt/index#pkg-overview


> 更新: 2024-10-03 22:05:28  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/crvub6ft8inhpe05>