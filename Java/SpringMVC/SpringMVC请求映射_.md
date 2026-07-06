# SpringMVC 请求映射

## 一、自动扫描

将controller beans注册到 `WebApplicationContext`,使用XML配置方式:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->
</beans>
```

## 二、请求映射

### 1、[@RequestMapping ](/RequestMapping)

`@RequestMapping` 注解映射请求到控制器的方法.它通过 URL, HTTP方法, request参数, headers, 和媒体类型，来匹配方法

```java
@RestController
public class Application {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello World!";
    }
}
```

`@RequestMapping`可以注解`@Controller`类：

```java
@RestController
@RequestMapping("/classPath")
public class Application {
    @RequestMapping("/methodPath")
    public String method() {
        return "mapping url is /classPath/methodPath";
    }
}
```

`method`方法匹配的URL是`/classPath/methodPath`"。

### 2、[@PathVariable ](/PathVariable)

```java
@RequestMapping("/users/{username}")
public String userProfile(@PathVariable("username") String username) {
    return String.format("user %s", username);
}

@RequestMapping("/posts/{id}")
public String post(@PathVariable("id") int id) {
    return String.format("post %d", id);
}
```

在上述例子中，URL中的变量可以用`{variableName}`来表示，同时在方法的参数中加上`@PathVariable("variableName")`，那么当请求被转发给该方法处理时，对应的URL中的变量会被自动赋值给被`@PathVariable`注解的参数（能够自动根据参数类型赋值，例如上例中的int）。

### 3、支持HTTP方法

对于HTTP请求除了其URL，还需要注意它的方法（Method）。例如我们在浏览器中访问一个页面通常是GET方法，而表单的提交一般是POST方法。`@Controller`中的方法同样需要对其进行区分：

```java
@RequestMapping(value = "/login", method = RequestMethod.GET)
public String loginGet() {
    return "Login Page";
}

@RequestMapping(value = "/login", method = RequestMethod.POST)
public String loginPost() {
    return "Login Post Request";
}
```

Spring MVC最新的版本中提供了一种更加简洁的配置HTTP方法的方式，增加了四个标注：

* `@PutMapping`
* `@GetMapping`
* `@PostMapping`
* `@DeleteMapping`

在Web应用中常用的HTTP方法有四种：

* PUT方法用来添加的资源
* GET方法用来获取已有的资源
* POST方法用来对资源进行状态转换
* DELETE方法用来删除已有的资源\
  这四个方法可以对应到CRUD操作（`Create`、`Read`、`Update`和`Delete`）。

每一个Web请求都是属于其中一种，在Spring MVC中如果不特殊指定的话，默认是GET请求。


> 更新: 2022-04-09 16:52:45  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ayrnot>