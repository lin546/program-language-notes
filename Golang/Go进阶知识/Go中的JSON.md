# Go 中的 JSON

## 一、基本的序列化

首先我们来看一下Go语言中`json.Marshal`（序列化）与`json.Unmarshal`（反序列化）的基本用法。

```go
type Person struct {
	Name   string
	Age    int64
	Weight float64
}

func main() {
	p1 := Person{
		Name:   "七米",
		Age:    18,
		Weight: 71.5,
	}
	// struct -> json string
	b, err := json.Marshal(p1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	// json string -> struct
	var p2 Person
	err = json.Unmarshal(b, &p2)
	if err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("p2:%#v\n", p2)
}
```

输出：

```go
str:{"Name":"七米","Age":18,"Weight":71.5}
p2:main.Person{Name:"七米", Age:18, Weight:71.5}
```

### 1、结构体 tag 介绍

<font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">Tag</font><font style="color:rgb(68, 68, 68);">是结构体的元信息，可以在运行的时候通过反射的机制读取出来。 </font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">Tag</font><font style="color:rgb(68, 68, 68);">在结构体字段的后方定义，由一对</font>**<font style="color:red;">反引号</font>**<font style="color:rgb(68, 68, 68);">包裹起来，具体的格式如下：</font>

```go
`key1:"value1" key2:"value2"`
```

<font style="color:rgb(68, 68, 68);">结构体tag由一个或多个键值对组成。键与值使用</font>**<font style="color:red;">冒号</font>**<font style="color:rgb(68, 68, 68);">分隔，值用</font>**<font style="color:red;">双引号</font>**<font style="color:rgb(68, 68, 68);">括起来。同一个结构体字段可以设置多个键值对tag，不同的键值对之间使用</font>**<font style="color:red;">空格</font>**<font style="color:rgb(68, 68, 68);">分隔。</font>

### <font style="color:rgb(68, 68, 68);">2、使用 json tag 指定字段名</font>

<font style="color:rgb(68, 68, 68);">序列化与反序列化默认情况下使用结构体的字段名，我们可以通过给结构体字段添加tag来指定json序列化生成的字段名。</font>

```go
// 使用json tag指定序列化与反序列化时的行为
type Person struct {
	Name   string `json:"name"` // 指定json序列化/反序列化时使用小写name
	Age    int64
	Weight float64
}
```

### 3、忽略某个字段

<font style="color:rgb(68, 68, 68);">如果你想在json序列化/反序列化的时候忽略掉结构体中的某个字段，可以按如下方式在tag中添加</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">-</font><font style="color:rgb(68, 68, 68);">。</font>

```go
// 使用json tag指定json序列化与反序列化时的行为
type Person struct {
	Name   string `json:"name"` // 指定json序列化/反序列化时使用小写name
	Age    int64
	Weight float64 `json:"-"` // 指定json序列化/反序列化时忽略此字段
}
```

### 4、忽略空值字段

<font style="color:rgb(68, 68, 68);">当 struct 中的字段没有值时， </font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">json.Marshal()</font></code><font style="color:rgb(68, 68, 68);"> 序列化的时候不会忽略这些字段，而是默认输出字段的类型零值（例如</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">int</font></code><font style="color:rgb(68, 68, 68);">和</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">float</font></code><font style="color:rgb(68, 68, 68);">类型零值是 0，</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">string</font></code><font style="color:rgb(68, 68, 68);">类型零值是</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">""</font><font style="color:rgb(68, 68, 68);">，对象类型零值是 nil）。如果想要在序列序列化时忽略这些没有值的字段时，可以在对应字段添加</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">omitempty</font></code><font style="color:rgb(68, 68, 68);"> tag。</font>

```go
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email"`
	Hobby []string `json:"hobby"`
}

func omitemptyDemo() {
	u1 := User{
		Name: "七米",
	}
	// struct -> json string
	b, err := json.Marshal(u1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
}
```

<font style="color:rgb(68, 68, 68);">输出结果：</font>

```go
str:{"name":"七米","email":"","hobby":null}
```

<font style="color:rgb(68, 68, 68);">如果想要在最终的序列化结果中去掉空值字段，可以像下面这样定义结构体：</font>

```go
// 在tag中添加omitempty忽略空值
// 注意这里 hobby,omitempty 合起来是json tag值，中间用英文逗号分隔
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
}
```

<font style="color:rgb(68, 68, 68);">此时，再执行上述的</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">omitemptyDemo</font><font style="color:rgb(68, 68, 68);">，输出结果如下：</font>

```go
str:{"name":"七米"} // 序列化结果中没有email和hobby字段
```

### 5、忽略嵌套结构体空值字段

<font style="color:rgb(68, 68, 68);">首先来看几种结构体嵌套的示例：</font>

```go
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
	Profile
}

type Profile struct {
	Website string `json:"site"`
	Slogan  string `json:"slogan"`
}

func nestedStructDemo() {
	u1 := User{
		Name:  "七米",
		Hobby: []string{"足球", "双色球"},
	}
	b, err := json.Marshal(u1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
}
```

<font style="color:rgb(68, 68, 68);">匿名嵌套</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">Profile</font><font style="color:rgb(68, 68, 68);">时序列化后的json串为单层的：</font>

```go
str:{"name":"七米","hobby":["足球","双色球"],"site":"","slogan":""}
```

<font style="color:rgb(68, 68, 68);">想要变成嵌套的json串，需要改为具名嵌套或定义字段tag：</font>

```go
type User struct {
	Name    string   `json:"name"`
	Email   string   `json:"email,omitempty"`
	Hobby   []string `json:"hobby,omitempty"`
	Profile `json:"profile"`
}
// str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

<font style="color:rgb(68, 68, 68);">想要在嵌套的结构体为空值时，忽略该字段，仅添加</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">omitempty</font><font style="color:rgb(68, 68, 68);">是不够的：</font>

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

<font style="color:rgb(68, 68, 68);">还需要使用嵌套的结构体指针：</font>

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	*Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"]}
```

## 二、GO 序列化技巧

### 1、优雅处理字符串格式的数字

<font style="color:rgb(68, 68, 68);">有时候，前端在传递来的json数据中可能会使用字符串类型的数字，这个时候可以在结构体tag中添加</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">string</font><font style="color:rgb(68, 68, 68);">来告诉json包从字符串中解析相应字段的数据：</font>

```go
type Card struct {
	ID    int64   `json:"id,string"`    // 添加string tag
	Score float64 `json:"score,string"` // 添加string tag
}

func intAndStringDemo() {
	jsonStr1 := `{"id": "1234567","score": "88.50"}`
	var c1 Card
	if err := json.Unmarshal([]byte(jsonStr1), &c1); err != nil {
		fmt.Printf("json.Unmarsha jsonStr1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("c1:%#v\n", c1) // c1:main.Card{ID:1234567, Score:88.5}
}
```

### 2、整数变浮点数

<font style="color:rgb(68, 68, 68);">因为在 JSON 协议中是没有整型和浮点型之分的，它们统称为number。json字符串中的数字经过Go语言中的json包反序列化之后都会成为</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">float64</font><font style="color:rgb(68, 68, 68);">类型。</font>

<font style="color:rgb(68, 68, 68);">通常这并不会有什么问题，但是在某些特殊场景下就会产生意想不到的结果。比如，将JSON格式的数据反序列化为</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">map\[string]interface{}</font><font style="color:rgb(68, 68, 68);">时，数字都变成科学计数法表示的浮点数。</font>

```go
// useNumberDemo 使用json.UseNumber
// 解决将JSON数据反序列化成map[string]interface{}时
// 数字变为科学计数法表示的浮点数问题
func useNumberDemo(){
	type student struct {
		ID int64 `json:"id"`
		Name string `json:"q1mi"`
	}
	s := student{ID: 123456789,Name: "q1mi"}
	b, _ := json.Marshal(s)
	var m map[string]interface{}
	// decode
	json.Unmarshal(b, &m)
	fmt.Printf("id:%#v\n", m["id"])  // 1.23456789e+08
	fmt.Printf("id type:%T\n", m["id"])  //float64

	// use Number decode
	decoder := json.NewDecoder(bytes.NewReader(b))
	decoder.UseNumber()
	decoder.Decode(&m)
	fmt.Printf("id:%#v\n", m["id"])  // "123456789"
	fmt.Printf("id type:%T\n", m["id"]) // json.Number
}
```

<font style="color:rgb(68, 68, 68);">这种问题通常出现在将JSON格式数据反序列化为</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">map[string]interface{}</font></code><font style="color:rgb(68, 68, 68);">时，再来一个示例。</font>

```go
func jsonDemo() {
	// map[string]interface{} -> json string
	var m = make(map[string]interface{}, 1)
	m["count"] = 1 // int
	b, err := json.Marshal(m)
	if err != nil {
		fmt.Printf("marshal failed, err:%v\n", err)
	}
	fmt.Printf("str:%#v\n", string(b))
	// json string -> map[string]interface{}
	var m2 map[string]interface{}
	err = json.Unmarshal(b, &m2)
	if err != nil {
		fmt.Printf("unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("value:%v\n", m2["count"]) // 1
	fmt.Printf("type:%T\n", m2["count"])  // float64
}
```

<font style="color:rgb(68, 68, 68);">这种场景下如果想更合理的处理数字就需要使用</font><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">decoder</font><font style="color:rgb(68, 68, 68);">去反序列化，示例代码如下：</font>

```go
func decoderDemo() {
	// map[string]interface{} -> json string
	var m = make(map[string]interface{}, 1)
	m["count"] = 1 // int
	b, err := json.Marshal(m)
	if err != nil {
		fmt.Printf("marshal failed, err:%v\n", err)
	}
	fmt.Printf("str:%#v\n", string(b))
	// json string -> map[string]interface{}
	var m2 map[string]interface{}
	// 使用decoder方式反序列化，指定使用number类型
	decoder := json.NewDecoder(bytes.NewReader(b))
	decoder.UseNumber()
	err = decoder.Decode(&m2)
	if err != nil {
		fmt.Printf("unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("value:%v\n", m2["count"]) // 1
	fmt.Printf("type:%T\n", m2["count"])  // json.Number
	// 将m2["count"]转为json.Number之后调用Int64()方法获得int64类型的值
	count, err := m2["count"].(json.Number).Int64()
	if err != nil {
		fmt.Printf("parse to int64 failed, err:%v\n", err)
		return
	}
	fmt.Printf("type:%T\n", int(count)) // int
}
```

<code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">json.Number</font></code><font style="color:rgb(68, 68, 68);">的源码定义如下：</font>

```go
// A Number represents a JSON number literal.
type Number string

// String returns the literal text of the number.
func (n Number) String() string { return string(n) }

// Float64 returns the number as a float64.
func (n Number) Float64() (float64, error) {
	return strconv.ParseFloat(string(n), 64)
}

// Int64 returns the number as an int64.
func (n Number) Int64() (int64, error) {
	return strconv.ParseInt(string(n), 10, 64)
}
```

<font style="color:rgb(68, 68, 68);">我们在处理</font><code><font style="color:rgb(68, 68, 68);">number</font></code><font style="color:rgb(68, 68, 68);">类型的json字段时需要先得到</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">json.Number</font></code><font style="color:rgb(68, 68, 68);">类型，然后根据该字段的实际类型调用</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">Float64()</font></code><font style="color:rgb(68, 68, 68);">或</font><code><font style="color:rgb(68, 68, 68);background-color:rgb(248, 248, 248);">Int64()</font></code><font style="color:rgb(68, 68, 68);">。</font>


> 更新: 2023-08-27 19:40:06  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/klqtzr0l0abv1i0d>