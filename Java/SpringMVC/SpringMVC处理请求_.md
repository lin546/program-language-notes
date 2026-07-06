# SpringMVC处理请求

使用 `@RequestMapping` 标注的处理方法可以拥有非常灵活的方法签名，它支持的方法参数及返回值类型种类极其丰富。大多数参数都可以任意的次序出现，除了唯一的一个例外：`BindingResult`参数。

## 一、支持的方法参数类型

`@RequestMapping` 方法方法所支持的常见参数类型：

* `WebRequest, NativeWebRequest`:可以直接访问请求参数，请求和会话属性，而无需使用Servlet API。
* `javax.servlet.ServletRequest/ServletResponse`（Servlet API）。可以是任何具体的请求或响应类型的对象，如`HttpServletRequest`。
* `HttpSession` 类型的会话对象（Servlet API）。
* `java.util.Locale/TimeZone`，当前请求的地区信息/时区信息。
* `org.springframework.http.HttpMethod` 。可以拿到HTTP请求方法
* `java.security.Principal`，当前被认证用户信息
* `@PathVariable` 标注的方法参数，其存放了URI模板变量中的值。
* `@RequestParam` 标注的方法参数，其存放了Servlet请求中所指定的参数。参数的值会被转换成方法参数所声明的类型。
* `@RequestHeader` 标注的方法参数，其存放了Servlet请求中所指定的HTTP请求头的值。参数的值会被转换成方法参数所声明的类型。
* `@RequestBody` 标注的参数，提供了对HTTP请求体的存取。参数的值通过 `HttpMessageConverter` 被转换成方法参数所声明的类型。
* `@RequestPart` 标注的参数，提供了对一个"`multipart/form-data` 请求块（request part）内容的存取。
* `HttpEntity<?>` 类型的参数，其提供了对HTTP请求头和请求内容的存取。请求流是通过 `HttpMessageConverter` 被转换成 `entity` 对象的。
* `java.util.Map/org.springframework.io.Model/org.springframework.ui.ModelMap` 类型的参数，用以访问 model（用在视图渲染）。
* `@ModelAndView` 访问已经存在于 model 中的属性（如果该属性不存在则实例化它）。
* `RedirectAttributes` 类型的参数，用以指定重定向下要使用到的属性集。

## 二、支持的方法返回类型

`@RequestMapping` 方法方法支持的常见返回类型：

* `ModelAndView` 对象，其中 model 隐含填充了命令对象，以及标注了 `@ModelAttribute` 字段的存取器被调用所返回的值。
* `java.util.Map, org.springframework.ui.Model`，添加到隐含 model 中的属性，其中视图名称默认由 `RequestToViewNameTranslator`决定。
* View 对象。其中 model 隐含填充了命令对象，以及标注了 `@ModelAttribute` 字段的存取器被调用所返回的值。handler方法也可以增加一个Model类型的方法参数来增强model
* String 对象，其值会被解析成一个逻辑视图名。其中，model 将默认填充了命令对象以及标注了 `@ModelAttribute` 字段的存取器被调用所返回的值。handler方法也可以增加一个Model类型的方法参数来增强model
* void。如果处理器方法中已经对response响应数据进行了处理（比如在方法参数中定义一个 `ServletResponse` 或 `HttpServletResponse` 类型的参数并直接向其响应体中写东西），那么方法可以返回void。handler方法也可以增加一个Model类型的方法参数来增强model
* `@ResponseBody` ：返回值会被 `HttpMessageConverters` 转换成所方法声明的参数类型，并写入到 response。
* `HttpEntity<?>` 或 `ResponseEntity<?>` 对象，用于提供对 Servlet HTTP响应头和响应内容的存取。对象体会被 `HttpMessageConverters` 转换成响应流。\
  `HttpHeaders` 对象，返回一个不含响应体的`response`
* 如果返回类型不是Spring MVC默认识别的类型，则会被处理成 model 的一个属性并返回给视图，该属性的名称为方法级的 `@ModelAttribute` 所标注的字段名（或者以返回类型的类名作为默认的属性名）。model隐含填充了命令对象以及标注了`@ModelAttribute` 字段的存取器被调用所返回的值

注：

> * SpringMVC 在调用方法前会创建一个隐含的模型对象，作为模型数据的存储容器，我们称之为 “隐含模型”。如果处理方法的入参为 Map 或 Model 类型，SpringMVC 会将隐含模型的引用传递给这些入参。在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型数据中添加新的属性数据。

## 三、方法参数/返回类型详解

### 1、[@RequestParam ](/RequestParam)

将请求的参数绑定到方法中的参数上，有required参数，默认情况下，required=true，也就是该参数必须要传。

```java
@RequestMapping("/happy")
  public String sayHappy(
  @RequestParam(value = "name", required = false) String name,
  @RequestParam(value = "age", required = true) String age) {
  //age参数必须传 ，name可传可不传
  ...
  }
```

（1）如果目标方法参数是 `String` 类型，则类型转换（Type Conversion）会自动生效。\
（2）如果目标方法参数是 `Map<String, String>` 或 `MultiValueMap<String, String>` 类型, 则请求参数填充该 Map.\
（3）注意是否使用 `@RequestParam` 有时是可选的。如简单类型会自动匹配：我们以 int 类型为例：

JSP 页面代码：

```html
<form action="basicData" method="post">
    <input name="age" value="10" type="text"/>
    <input type="submit" value="提交">
</form>9
```

Controller 代码：

```java
@RequestMapping("/basicData")
    public void basicData(int age){
        System.out.println(age);//10
    }
```

结果是 打印出了表单里面的 value 的值。

### 2、[@RequestHeader ](/RequestHeader)

使用 `@RequestHeader` 注解把请求头绑定到控制器方法参数.

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```

（1）如果目标方法参数是 `String` 类型，则类型转换（Type Conversion）会自动生效。\
（2）如果目标方法参数是 `Map<String, String>` 、 `MultiValueMap<String, String>` 或 `HttpHeaders` 类型, 则请求头填充该 Map.

### 3、[@CookieValue ](/CookieValue)

使用 `@CookieValue` 注解把HTTP cookie的值绑定到控制器方法参数.

如获取cookie中的"JSESSIONID"值。

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) {
    //...
}
```

（1）如果目标方法参数是 `String` 类型，则类型转换（Type Conversion）会自动生效。

### 4、[@SessionAttributes ](/SessionAttributes)

`@SessionAttributes` 被用来在请求之间的会话存储 model 属性.如

```java
@Controller
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

当名为“pet”的属性添加到model中时，该属性会自动添加到session中。

#### [@SessionAttribute ](/SessionAttribute)

如果需要访问已经存在会话中的属性

If you need access to pre-existing session attributes that are managed globally, i.e. outside the controller (e.g. by a filter), and may or may not be present use the [@SessionAttribute ](/SessionAttribute) annotation on a method parameter:

#### [@PathVariable ](/PathVariable)

@PathVariable用来接收路径参数，此出街放在参数前。

```java
@RequestMapping(value="/happy/{dayid}",method=RequestMethod.GET)
public String findPet(@PathVariable String dayid, Model mode) {
//使用@PathVariable注解绑定 {dayid} 到String dayid
}
```

## [@RequestBody ](/RequestBody)

@RequestBody允许request的参数在request体中，而不是直接连接在地址后面。此注解配置在参数前。

```java
@RequestMapping(value = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body，@RequestBody User user){
   //可以绑定自定义的对象类型
}
```

### [@ResponseBody ](/ResponseBody)

@ResponseBody与@RequestBody类似，它的作用是将返回类型直接输入到HTTP response body中，而不是返回一个页面。@ResponseBody在输出JSON格式的数据时，会经常用到。此注解可放置在返回值或者方法上。

```java
@RequestMapping(value = "/happy", method =RequestMethod.POST)
@ResponseBody
public String helloWorld() {    
return "Hello World";//返回String类型
}
```

#### [@RestController ](/RestController)

它相当于@Controller+@ResponseBody。这意味着当你开发一个和页面交互数据的控制的时候，需要使用此注解。

## 参考

* [Spring Docs](https://docs.spring.io/spring/docs/5.0.16.RELEASE/spring-framework-reference/web.html#mvc-controller)
* [Spring MVC@RequestMapping 方法所支持的参数类型和返回类型详解](https://www.tianmaying.com/tutorial/request-mapping-parameter-type)


> 更新: 2022-04-09 16:52:43  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/sg0aqk>