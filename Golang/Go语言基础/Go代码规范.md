# Go 代码规范

## 一、代码风格

### 1、代码格式

* 代码都必须用 `gofmt`和 `goimports` 进行格式化<font style="color:rgb(59, 67, 81);">（建议将代码Go代码编辑器设置为：保存时运行 </font><code><font style="color:rgb(59, 67, 81);">gofmt</font></code><font style="color:rgb(59, 67, 81);">和</font><code><font style="color:rgb(59, 67, 81);">goimports</font></code><font style="color:rgb(59, 67, 81);">）</font>。
* <font style="color:rgb(59, 67, 81);">不要使用相对路径引入包，例如 </font><code><font style="color:rgb(59, 67, 81);">import …/util/net</font></code><font style="color:rgb(59, 67, 81);">。</font>
* <font style="color:rgb(59, 67, 81);">包名称和导入路径的最后一个目录名不匹配时，或者多个相同包冲突时，则必须使用别名导入。</font>

```go
"github.com/dgrijalva/jwt-go/v4"// bad

jwt "github.com/dgrijalva/jwt-go/v4"//good
```

* 导入的包建议进行分组，匿名包的引用使用一个新的分组，并对匿名包进行说明。

```go
import (
    // go 标准包
    "fmt"

    // 第三方包
    "github.com/jinzhu/gorm"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"

    // 匿名包单独分组，并对匿名包引用进行说明
    // import mysql driver
    _ "github.com/jinzhu/gorm/dialects/mysql"

    // 内部包
    v1 "github.com/marmotedu/api/apiserver/v1"
    metav1 "github.com/marmotedu/apimachinery/pkg/meta/v1"
    "github.com/marmotedu/iam/pkg/cli/genericclioptions"
)
```

## 二、命名规范

### 1、包命名

* 对于Go中的包，一般<font style="background-color:#CEF5F7;">建议使用小写形式的单个单词</font>命名。

> 我们在给包命名的时候不要有是否与其他包重名的顾虑。因为在Go中，包名是不唯一的。比如：fool项目有名为`log`的包，bar项目也可以有自己的`log`的包。每个包的导入路径是唯一的，对于包冲突的情况，可以在导入包时使用一个显式包名来指代导入的包，并且在这个源文件中使用显式包名来引用包中的元素。

```go
import "github.com/bigwhite/foo/log"
import barlog "github.com/bigwhite/bar/log"
```

* Go语言建议，<font style="background-color:#CEF5F7;">包名尽量与包导入路径的最后一个路径分段保持一致</font>。例如：包导入路径`golang.org/x/test/encoding`的最后路径分段时`encoding`，改路径下包名就应该为`encoding`。
* 此外，我们在给包命名的时候，不仅要考虑包自身的名字，还要考虑包自身的名字，还要兼顾包导出的标识符（如变量、常量、类型、函数等）的命名。由于对这些包导出标识符的引用必须以包名为前缀，因此对包导出标识符命名时，在名字中不要再包含包名，比如：

```go
string.Reader			//good
string.StringReader  	//bad
```

### 2、变量、结构体、函数和方法

* GO语言官方要求标志符采用驼峰命名法。由于首字母大写的标志符在GO语言中被视为包导出标识符，因此只有在涉及包导出的情况下才会用到大驼峰拼写法，其他情况使用小驼峰拼写法。
* 【变量】：若干变量为bool类型，则名称应以Has，Is，Can或Allow开头，例如：

```go
var hasConflict bool
var isExist bool
var canManage bool
var allowGitHook bool
```

### 3、项目和文件命名

* <font style="color:rgb(59, 67, 81);">项目名可以通过中划线来连接多个单词。</font>
* 文件名应小写，并使用下划线分割单词。

### 4、接口

* 在Go语言中，对于接口类型优先以单个单词命名。对于拥有唯一方法或通过多个唯一方法的接口组合而成的接口，Go语言的惯例是用"方法名+er"命名。比如：

```go
type Writer interface{
	Write(p []byte)(n int,err error)
}
type Reader interface{
	Read(p []byte)(n int,err error)
}
type Closer interface{
	Close() error
}
type ReadWriterCloser interface{
    Reader
	Writer
    Closer
}

```

### 5、常量

* 在C语言家族中，常量通常使用全大写的单词命名，但在Go语言中，常量在命名方式上与变量并无较大差别，根据访问控制决定使用大写或小写。
* 在Go中数值型常量无需显示赋予类型，常量会在使用时根据左值类型和其他运算操数的类型进行自动转换。

```go
const(
	keepHostHeader = false
)
```

### 6、Error 的命名

* Error 类型应该写成 FooError 的形式。

```go
type ExitError struct {
	// ....
}
```

Error变量写成ErrFoo的形式。

```go
var ErrFormat = errors.New("unknown format")
```

## 三、类型

### 1、字符串

* 空字符串判断

```go
// bad
if s == "" {
    // normal code
}

// good
if len(s) == 0 {
    // normal code
}
```

* `[]byte`相等比较

```go
// bad
var s1 []byte
var s2 []byte
...
bytes.Equal(s1, s2) == 0
bytes.Equal(s1, s2) != 0

// good
var s1 []byte
var s2 []byte
...
bytes.Compare(s1, s2) == 0
bytes.Compare(s1, s2) != 0
```

### 2、切片

* 声明 slice

```go
// bad
s := []string{}
s := make([]string, 0)

// good
var s []string
```

* slice 赋值

```go
// bad
var b1, b2 []byte
for i, v := range b1 {
   b2[i] = v
}
for i := range b1 {
   b2[i] = b1[i]
}

// good
copy(b2, b1)
```

* slice 新增

```go
// bad
var a, b []int
for _, v := range a {
    b = append(b, v)
}

// good
var a, b []int
b = append(b, a...)
```

### 3、结构体

* struct 初始化

struct 以多行格式初始化。

```go
type user struct {
	Id   int64
	Name string
}

//bad
u1 := user{100, "Colin"}	

//good
u2 := user{		
    Id:   200,
    Name: "Lex",
}
```

### 4、控制语句

* <font style="color:rgb(59, 67, 81);">【if】 对于bool类型的变量，应直接进行真假判断。</font>

```go
var isAllow bool
if isAllow {
	// normal code
}
```

* 【range】如果只需要第一项（key），就丢弃第二个。如果只需要第二项，则把第一项置为下划线。

```go
for key := range keys {
// normal code
}

sum := 0
for _, value := range array {
    sum += value
}
```

## 四、函数

* 传入变量和返回变量以小写字母开头。
* 尽量采用值传递，而非指针传递。
* 传入参数是 map、slice、chan、interface，不要传递指针。

### 1、函数参数

* 如果函数返回相同类型的两个或三个参数，或者如果从上下文中不清楚结果的含义，使用命名返回，其他情况不建议使用命名返回，例如：

```go
func coordinate() (x, y float64, err error) {
	// normal code
}
```

* <font style="color:rgb(59, 67, 81);">传入变量和返回变量都以小写字母开头。</font>
* <font style="color:rgb(59, 67, 81);">尽量用值传递，非指针传递。</font>
* <font style="color:rgb(59, 67, 81);">多返回值最多返回三个，超过三个请使用 struct。</font>

### <font style="color:rgb(59, 67, 81);">2、defer</font>

* 当存在资源创建时，应紧跟defer释放资源。先判断是否错误，再defer释放资源。

```go
rep, err := http.Get(url)
if err != nil {
    return err
}

defer resp.Body.Close()
```

### 3、变量命名

* 变量声明尽量放在变量第一次使用的前面，遵循就近原则。
* 如果魔法数字出现超过两次，则禁止使用，改用一个常量替代。

## 五、再谈变量声明

Go 语言在使用变量前需要进行变量的声明，下面是Go语言常见的变量声明形式：

```go
var a int32
var a int32 = 1024
var a = 1024
a := 1024
var(
	a = 1024
    b = 2048
)
```

Go语言有两类变量：

* **包级变量**：在package级别可见的变量。如果是导出变量，则该包级变量也可以视为全局变量。
* **局部变量**：函数或方法体内声明的变量，仅在函数或方法体内可见。

下面来分别说明实现这两类变量在声明形式选择上保持一致性的一些最佳实践。

### 1、包级变量的声明形式

包级变量只能使用带有var关键字的变量声明形式，根据<font style="background-color:#D9EAFC;">声明变量时是否延迟初始化</font>这个角度又可以继续划分。

* 对于<font style="background-color:#D9EAFC;">在声明变量的同时进行显式初始化的这类包级变量</font>，实践中多使用下面的格式：

```go
var variableName = InitExpression
```

Go编译器自动根据等号右侧的`InitExpression`表达式求值的类型确定左侧所声明变量的类型。如果`InitExpression`采用的是不带有类型信息的常量表达式，比如

```go
var a = 1024
var b = 3.33
```

则包级变量会被设置为常量表达式的默认类型：如Go编译器会将`a`设置为`int64`，将`b`设置为`float64`。如果不接受默认类型，可以采用以下方式声明：

```go
var a = int32(1024)
var b = float32(3.33)
```

* 对于声明时并不显式初始化的包级变量，我们使用最基本的声明形式：

```go
var a int32
var b float64
```

虽然没有显式初始化，但Go语言会让这些变量拥有初始的零值。如果是自定义的类型，保证其零值可用是非常必要的。

* 聚类与就近原则：
  * **声明聚类**：如果一个包级变量在包内部被多处使用，那么这个变量放在源文件头部使用var块声明比较合适。
  * **就近原则**：尽可能在靠近第一次使用变量的位置声明该变量。

### 2、局部变量的声明形式

* <font style="background-color:#D9EAFC;">对于延迟初始化的局部变量声明，采用var关键字的声明形式</font>：

```go
var buf []byte
var err error
```

* <font style="background-color:#D9EAFC;">对于声明且显式初始化的局部变量，建议使用短变量声明形式</font>，对于接受默认类型的变量，可以使用下面的形式：

```go
a := 1024
b := 3.14
```

对于不接受默认类型的变量，依然可以使用短变量声明形式，只是要进行显式转型。

```go
a := int32(1024)
b := float32(3.33)
```

* 尽量在分支控制时应用短变量声明形式

```go
for _,v : range *v{
	...
}
```

##


> 更新: 2023-08-27 18:24:31  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/mbgraeaqihmngqbs>