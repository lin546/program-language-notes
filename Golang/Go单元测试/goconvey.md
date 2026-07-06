# goconvey

文档：<https://github.com/smartystreets/goconvey/wiki>

### 一、快速开始

#### 1、安装

```bash
go get github.com/smartystreets/goconvey
go install github.com/smartystreets/goconvey
```

#### 2、第一个例子

```go
func TestIntegerStuff(t *testing.T) {
    Convey("Given some integer with a starting value", t, func() {
        x := 1

        Convey("When the integer is incremented", func() {
            x++

            Convey("The value should be greater by one", func() {
                So(x, ShouldEqual, 2)
            })
        })
    })
}

```

### 二、断言

#### 1、<font style="color:rgb(31, 35, 40);">General </font>

```go
So(thing1, ShouldEqual, thing2)     	
So(thing1, ShouldNotEqual, thing2)		
So(thing1, ShouldResemble, thing2)		// a deep equals for arrays, slices, maps, and structs
So(thing1, ShouldNotResemble, thing2)   
So(thing1, ShouldPointTo, thing2)		
So(thing1, ShouldNotPointTo, thing2)
So(thing1, ShouldBeNil)
So(thing1, ShouldNotBeNil)
So(thing1, ShouldBeTrue)
So(thing1, ShouldBeFalse)
So(thing1, ShouldBeZeroValue)
```

#### 2、<font style="color:rgb(31, 35, 40);">Numeric </font>

```go
So(1, ShouldBeGreaterThan, 0)
So(1, ShouldBeGreaterThanOrEqualTo, 0)
So(1, ShouldBeLessThan, 2)
So(1, ShouldBeLessThanOrEqualTo, 2)
So(1.1, ShouldBeBetween, .8, 1.2)
So(1.1, ShouldNotBeBetween, 2, 3)
So(1.1, ShouldBeBetweenOrEqual, .9, 1.1)
So(1.1, ShouldNotBeBetweenOrEqual, 1000, 2000)
```

#### 3、Collections

```go
So([]int{2, 4, 6}, ShouldContain, 4)
So([]int{2, 4, 6}, ShouldNotContain, 5)
So(4, ShouldBeIn, ...[]int{2, 4, 6})
So(4, ShouldNotBeIn, ...[]int{1, 3, 5})
So([]int{}, ShouldBeEmpty)
So([]int{1}, ShouldNotBeEmpty)
So(map[string]string{"a": "b"}, ShouldContainKey, "a")
So(map[string]string{"a": "b"}, ShouldNotContainKey, "b")
So(map[string]string{"a": "b"}, ShouldNotBeEmpty)
So(map[string]string{}, ShouldBeEmpty)
So(map[string]string{"a": "b"}, ShouldHaveLength, 1) // supports map, slice, chan, and string
```

#### 4、<font style="color:rgb(31, 35, 40);">Strings</font>

```go
So("asdf", ShouldStartWith, "as")
So("asdf", ShouldNotStartWith, "df")
So("asdf", ShouldEndWith, "df")
So("asdf", ShouldNotEndWith, "df")
So("asdf", ShouldContainSubstring, "sd")		// optional 'expected occurences' arguments?
So("asdf", ShouldNotContainSubstring, "er")
So("adsf", ShouldBeBlank)
So("asdf", ShouldNotBeBlank)
```

#### 5、Panic

```go
So(func(), ShouldPanic)
So(func(), ShouldNotPanic)
So(func(), ShouldPanicWith, "")		// or errors.New("something")
So(func(), ShouldNotPanicWith, "")	// or errors.New("something")
```

#### 6、<font style="color:rgb(31, 35, 40);">Type checking</font>

```go
So(1, ShouldHaveSameTypeAs, 0)
So(1, ShouldNotHaveSameTypeAs, "asdf")
```

#### 7、Time

```go
So(time.Now(), ShouldHappenBefore, time.Now())
So(time.Now(), ShouldHappenOnOrBefore, time.Now())
So(time.Now(), ShouldHappenAfter, time.Now())
So(time.Now(), ShouldHappenOnOrAfter, time.Now())
So(time.Now(), ShouldHappenBetween, time.Now(), time.Now())
So(time.Now(), ShouldHappenOnOrBetween, time.Now(), time.Now())
So(time.Now(), ShouldNotHappenOnOrBetween, time.Now(), time.Now())
So(time.Now(), ShouldHappenWithin, duration, time.Now())
So(time.Now(), ShouldNotHappenWithin, duration, time.Now())
```

> Convey 支持定义自己的Assert，详见[Custom Assertions](https://github.com/smartystreets/goconvey/wiki/Custom-Assertions)

### 三、执行

```go
go test -v
```

执行顺序

```go
Convey A
    So 1
    Convey B
        So 2
    Convey C
        So 3
```

> 执行顺序为A1B2 A1C3

### 四、作用域

```go
Convey("Setup", func() {
    foo := &Bar{} // 外层变量声明
    
    Convey("Test case 1", func() {
        foo := &Bar{} // 新建变量，与外层 foo 无关
    })
    
    Convey("Test case 2", func() {
        foo = &Bar{} // 修改外层的变量
    })
})
```

### 五、<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">资源清理</font>

```go
Convey("Top-level", t, func() {    // 外层作用域
    db.Open()          // 🟢 Setup 1: 每个子测试前执行
    db.Initialize()    // 🟢 Setup 2

    Convey("Test A", func() {      // 子测试1
        db.Query()     // 🟡 测试逻辑
    })

    Convey("Test B", func() {      // 子测试2
        db.Insert()    // 🟡 测试逻辑
    })

    Reset(func() {     // 🔴 清理逻辑：每个子测试后执行
        db.Close()    
    })
})
```

* <code><font style="background-color:rgb(252, 252, 252);">Reset()</font></code><font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);"> 的特性</font>
  * **<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">触发时机</font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">：同一作用域内的每个 </font><code><font style="color:rgba(0, 0, 0, 0.9);">Convey()</font></code><font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);"> </font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">执行完毕后立即运行</font>**
  * **<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">嵌套规则</font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">：内层 </font><code><font style="background-color:rgb(252, 252, 252);">Reset()</font></code><font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);"> 优先于外层执行（若存在嵌套）</font>

```go
Convey("Parent", t, func() {
    Reset(func() { println("Parent Reset") })

    Convey("Child", func() {
        Reset(func() { println("Child Reset") })
    })
})
```

> 执行顺序：Child Reset → Parent Reset

* 调试技巧:<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">在 </font><code><font style="color:rgba(0, 0, 0, 0.9);">Reset()</font></code><font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);"> 中添加打印</font>

```plain
Reset(func() {
    println("Reset called at:", time.Now())
    db.Close()
})
```


> 更新: 2025-03-09 22:37:13  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/tozb7cdn6vs12uzh>