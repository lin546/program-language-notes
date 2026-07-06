# SpringMVC 快速入门

转载于[https://www.tianmaying.com/tutorial/spring-mvc-quickstart](https://www.tianmaying.com/tutorial/spring-mvc-quickstart)



## 一、一个最简单的Web应用


使用Spring Boot框架可以大大加速Web应用的开发过程，首先在Maven项目依赖中`spring-boot-starter-web`：



`pom.xml`



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.tianmaying</groupId>
  <artifactId>spring-web-demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-web-demo</name>
  <description>Demo project for Spring WebMvc</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.2.5.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```



接下来创建`src/main/java/com.tmy.Application.java`:



```java
package com.tmy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String greeting() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



运行应用：`mvn spring-boot:run`或在IDE中运行`main()`方法，在浏览器中访问http://localhost:8080，Hello World!就出现在了页面中。只用了区区十几行Java代码，一个Hello World应用就可以正确运行了，那么这段代码究竟做了什么呢？我们从程序的入口`SpringApplication.run(Application.class, args)`;开始分析：



`SpringApplication`是Spring Boot框架中描述Spring应用的类，它的`run()`方法会创建一个Spring应用上下文（`Application Context`）。另一方面它会扫描当前应用类路径上的依赖，例如本例中发现`spring-webmvc`（由 `spring-boot-starter-web`传递引入）在类路径中，那么Spring Boot会判断这是一个Web应用，并启动一个内嵌的Servlet容器（默认是Tomcat）用于处理HTTP请求。



Spring WebMvc框架会将Servlet容器里收到的HTTP请求根据路径分发给对应的`@Controller`类进行处理，`@RestController`是一类特殊的`@Controller`，它的返回值直接作为`HTTP Response`的`Body`部分返回给浏览器。



`@RequestMapping`注解表明该方法处理那些URL对应的HTTP请求，也就是我们常说的URL路由（`routing`)，请求的分发工作是有Spring完成的。例如上面的代码中http://localhost:8080/ 根路径就被路由至`greeting()`方法进行处理。如果访问http://localhost:8080/hello ，则会出现 404 Not Found错误，因为我们并没有编写任何方法来处理/hello`请求。



## 二、使用@Controller实现URL路由


现代Web应用往往包括很多页面，不同的页面也对应着不同的URL。对于不同的URL，通常需要不同的方法进行处理并返回不同的内容。



### 1、匹配多个URL


```java
@RestController
public class Application {

    @RequestMapping("/")
    public String index() {
        return "Index Page";
    }

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



method方法匹配的URL是`/classPath/methodPath`"。



提示:可以定义多个`@Controller`将不同URL的处理方法分散在不同的类中。



### 2、URL中的变量——PathVariable


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



+ `@PutMapping`
+ `@GetMapping`
+ `@PostMapping`
+ `@DeleteMapping`



这四个方法可以对应到CRUD操作（`Create`、`Read`、`Update`和`Delete`）。



每一个Web请求都是属于其中一种，在Spring MVC中如果不特殊指定的话，默认是GET请求。



## 三、模板渲染


在之前所有的`@RequestMapping`注解的方法中，返回值字符串都被直接传送到浏览器端并显示给用户。但是为了能够呈现更加丰富、美观的页面，我们需要将HTML代码返回给浏览器，浏览器再进行页面的渲染、显示。



一种很直观的方法是在处理请求的方法中，直接返回HTML代码，但是这样做的问题在于——一个复杂的页面HTML代码往往也非常复杂，并且嵌入在Java代码中十分不利于维护。更好的做法是将页面的HTML代码写在模板文件中，渲染后再返回给用户。为了能够进行模板渲染，需要将`@RestController`改成`@Controller`：



```java
import org.springframework.ui.Model;

@Controller
public class HelloController {

    @RequestMapping("/hello/{name}")
    public String hello(@PathVariable("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello";
    }
}
```



在上述例子中，返回值`"hello"`并非直接将字符串返回给浏览器，而是寻找名字为`hello`的模板进行渲染，我们使用Thymeleaf模板引擎进行模板渲染，需要引入依赖：



```plain
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```



接下来需要在默认的模板文件夹`src/main/resources/templates/`目录下添加一个模板文件`hello.html`：



```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```



`th:text="'Hello, ' + ${name} + '!'"`也就是将我们之前在`@Controller`方法里添加至`Model`的属性`name`进行渲染，并放入`<p>`标签中（因为`th:text`是`<p>`标签的属性）。



## 四、处理静态文件


浏览器页面使用HTML作为描述语言，那么必然也脱离不了CSS以及JavaScript。为了能够浏览器能够正确加载类似`/css/style.css`, `/js/main.js`等资源，默认情况下我们只需要在`src/main/resources/static`目录下添加`css/style.css`和`js/main.js`文件后，Spring MVC能够自动将他们发布，通过访问`/css/style.css`, `/js/main.js`也就可以正确加载这些资源。



```html
<link type="text/css" rel="styleSheet" href="/css/test.css" />
```



## 五、文件上传


Spring MVC还能够支持更为复杂的HTTP请求——文件资源。我们在网站中经常遇到上传图片、附件一类的需求，就是通过文件上传技术来实现的。



处理文件的表单和普通表单的唯一区别在于设置`enctype`——multipart编码方式则需要设置`enctype`为`multipart/form-data`。



```html
<form method="post" enctype="multipart/form-data">
    <input type="text" name="title" value="hello">
    <input type="file" name="avatar">
    <input type="submit">
</form>
```



该表单将会显示为一个文本框、一个文件按钮、一个提交按钮。然后我们选择一个文件：`chrome.png`，点击表单提交后产生的请求可能是这样的：



请求头：



```plain
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
```



请求体：



```plain
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="title"

hello
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="avatar"; filename="chrome.png"
Content-Type: image/png

 ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```



这便是一个multipart编码的表单。`Content-Type`中还包含了`boundary`的定义，它用来分隔请求体中的每个字段。正是这一机制，使得请求体中可以包含二进制文件（当然文件中不能包含`boundary`）。文件上传正是利用这种机制来完成的。



如果不设置`<form>`的`enctype`编码，同样可以在表单中设置`type=file`类型的输入框，但是请求体和传统的表单一样，这样服务器程序无法获取真正的文件内容。



在服务端，为了支持文件上传我们还需要进行一些配置。



### 1、控制器逻辑


对于表单中的文本信息输入，我们可以通过`@RequestParam`注解获取。对于上传的二进制文件（文本文件同样会转化为`byte[]`进行传输），就需要借助Spring提供的`MultipartFile`类来获取了：



```java
@Controller
public class FileUploadController {

    @PostMapping("/upload")
    @ResponseBody
    public String handleFileUpload(@RequestParam("file") MultipartFile file) {
        byte[] bytes = file.getBytes();

        return "file uploaded successfully."
    }
}
```



通过`MultipartFile`的`getBytes()`方法即可以得到上传的文件内容（`<form>`中定义了一个`type="file"`的，在这里我们可以将它保存到本地磁盘。另外，在默认的情况下Spring仅仅支持大小为128KB的文件，为了调整它，我们可以修改Spring的配置文件`src/main/resources/application.properties：`



```plain
multipart.maxFileSize: 128KB
multipart.maxRequestSize: 128KB
```



### 2、HTML表单


HTML中支持文件上传的表单元素仍然是，只不过它的类型是file：



```html
<html>
<body>
  <form method="POST" enctype="multipart/form-data" action="/upload">
    File to upload: <input type="file" name="file"><br />
    Name: <input type="text" name="name"><br /> <br />
    <input type="submit" value="Upload"> Press here to upload the file!
  </form>
</body>
</html>
```



multipart/form-data表单既可以上传文件类型，也可以和普通表单一样提交其他类型的数据，在Spring MVC的@RequestMapping方法参数中用@RequestParam标注即可（也可以利用数据绑定机制，绑定一个对象)



## 六、拦截器Interceptor


Spring MVC框架中的Interceptor，与Servlet API中的Filter十分类似，用于对Web请求进行预处理/后处理。通常情况下这些预处理/后处理逻辑是通用的，可以被应用于所有或多个Web请求，例如：



+ 记录Web请求相关日志，可以用于做一些信息监控、统计、分析
+ 检查Web请求访问权限，例如发现用户没有登录后，重定向到登录页面
+ 打开/关闭数据库连接——预处理时打开，后处理关闭，可以避免在所有业务方法中都编写类似代码，也不会忘记关闭数据库连接



### 1、Spring MVC请求流程


![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20200320162255.png)



上图是Spring MVC框架处理Web请求的基本流程，请求会经过`DispatcherServlet`的分发后，会按顺序经过一系列的`Interceptor`并执行其中的预处理方法，在请求返回时同样会执行其中的后处理方法。



在`DispatcherServlet`和`Controller`之间哪些竖着的彩色细条，是拦截请求进行额外处理的地方，所以命名为**拦截器（Interceptor）**。



### 2、HandlerInterceptor


Spring MVC中拦截器是实现了HandlerInterceptor接口的Bean：



```java
public interface HandlerInterceptor {
    boolean preHandle(HttpServletRequest request, 
                      HttpServletResponse response, 
                      Object handler) throws Exception;

    void postHandle(HttpServletRequest request, 
                    HttpServletResponse response, 
                    Object handler, ModelAndView modelAndView) throws Exception;

    void afterCompletion(HttpServletRequest request, 
                         HttpServletResponse response, 
                         Object handler, Exception ex) throws Exception;
}
```



+ `preHandle()`：预处理回调方法，若方法返回值为`true`，请求继续（调用下一个拦截器或处理器方法）；若方法返回值为`false`，请求处理流程中断，不会继续调用其他的拦截器或处理器方法，此时需要通过`response`产生响应；
+ `postHandle()`：后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时可以通过ModelAndView对模型数据进行处理或对视图进行处理
+ `afterCompletion()`：整个请求处理完毕回调方法，即在视图渲染完毕时调用



`HandlerInterceptor`有三个方法需要实现，但大部分时候可能只需要实现其中的一个方法，`HandlerInterceptorAdapter`是一个实现了`HandlerInterceptor`的抽象类，它的三个实现方法都为空实现（或者返回true），继承该抽象类后可以仅仅实现其中的一个方法：



```java
public class Interceptor extends HandlerInterceptorAdapter {

    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        // 在controller方法调用前打印信息
        System.out.println("This is interceptor.");
        // 返回true，将强求继续传递（传递到下一个拦截器，没有其它拦截器了，则传递给Controller）
        return true;
    }
}
```



### 3、配置Interceptor


定义`HandlerInterceptor`后，需要创建`WebMvcConfigurerAdapter`在MVC配置中将它们应用于特定的URL中。一般一个拦截器都是拦截特定的某一部分请求，这些请求通过URL模型来指定。



下面是一个配置的例子：



```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleInterceptor());
        registry.addInterceptor(new ThemeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```



## 八、异常处理


Spring MVC框架提供了多种机制用来处理异常，初次接触可能会对他们用法以及适用的场景感到困惑。现在以一个简单例子来解释这些异常处理的机制。



假设现在我们开发了一个博客应用，其中最重要的资源就是文章（Post），应用中的URL设计如下：



+ 获取文章列表：`GET /posts/`
+ 添加一篇文章：`POST /posts/`
+ 获取一篇文章：`GET /posts/{id}`
+ 更新一篇文章：`PUT /posts/{id}`
+ 删除一篇文章：`DELETE /posts/{id}`



这是非常标准的复合RESTful风格的URL设计，在Spring MVC实现的应用过程中，相应也会有5个对应的用`@RequestMapping`注解的方法来处理相应的URL请求。在处理某一篇文章的请求中（获取、更新、删除），无疑需要做这样一个判断——请求URL中的文章id是否在于系统中，如果不存在需要返回404 Not Found。



### 1、使用HTTP状态码


在默认情况下，Spring MVC处理Web请求时如果发现存在没有应用代码捕获的异常，那么会返回HTTP 500（Internal Server Error）错误。但是如果该异常是我们自己定义的并且使用`@ResponseStatus`注解进行修饰，那么Spring MVC则会返回指定的HTTP状态码：



```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "No Such Post")//404 Not Found
public class PostNotFoundException extends RuntimeException {
}
```



在Controller中可以这样使用它：



```java
@RequestMapping(value = "/posts/{id}", method = RequestMethod.GET)
public String showPost(@PathVariable("id") long id, Model model) {
    Post post = postService.get(id);
    if (post == null) throw new PostNotFoundException("post not found");
    model.addAttribute("post", post);
    return "postDetail";
}
```



这样如果我们访问了一个不存在的文章，那么Spring MVC会根据抛出的PostNotFoundException上的注解值返回一个HTTP 404 Not Found给浏览器。



### 2、最佳实践


上述场景中，除了获取一篇文章的请求，还有更新和删除一篇文章的方法中都需要判断文章id是否存在。在每一个方法中都加上`if (post == null) throw new PostNotFoundException("post not found")`;是一种解决方案，但如果有10个、20个包含`/posts/{id}`的方法，虽然只有一行代码但让他们重复10次、20次也是非常不优雅的。



为了解决这个问题，可以将这个逻辑放在Service中实现：



```java
@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    public Post get(long id) {
        return postRepository.findById(id)
                .orElseThrow(() -> new PostNotFoundException("post not found"));
    }
}

这里`PostRepository`继承了`JpaRepository`，可以定义`findById`方法返回一个`Optional<Post>`——如果不存在则Optional为空，抛出异常。
```



这样在所有的Controller方法中，只需要正常获取文章即可，所有的异常处理都交给了Spring MVC。



### 3、在Controller中处理异常


Controller中的方法除了可以用于处理Web请求，还能够用于处理异常处理——为它们加上`@ExceptionHandler`即可：



```java
@Controller
public class ExceptionHandlingController {

  // @RequestHandler methods
  ...

  // Exception handling methods

  // Convert a predefined exception to an HTTP Status code
  @ResponseStatus(value=HttpStatus.CONFLICT, reason="Data integrity violation")  // 409
  @ExceptionHandler(DataIntegrityViolationException.class)
  public void conflict() {
    // Nothing to do
  }

  // Specify the name of a specific view that will be used to display the error:
  @ExceptionHandler({SQLException.class,DataAccessException.class})
  public String databaseError() {
    // Nothing to do.  Returns the logical view name of an error page, passed to
    // the view-resolver(s) in usual way.
    // Note that the exception is _not_ available to this view (it is not added to
    // the model) but see "Extending ExceptionHandlerExceptionResolver" below.
    return "databaseError";
  }

  // Total control - setup a model and return the view name yourself. Or consider
  // subclassing ExceptionHandlerExceptionResolver (see below).
  @ExceptionHandler(Exception.class)
  public ModelAndView handleError(HttpServletRequest req, Exception exception) {
    logger.error("Request: " + req.getRequestURL() + " raised " + exception);

    ModelAndView mav = new ModelAndView();
    mav.addObject("exception", exception);
    mav.addObject("url", req.getRequestURL());
    mav.setViewName("error");
    return mav;
  }
}
```



首先需要明确的一点是，在Controller方法中的`@ExceptionHandler`方法只能够处理同一个Controller中抛出的异常。这些方法上同时也可以继续使用`@ResponseStatus`注解用于返回指定的HTTP状态码，但同时还能够支持更加丰富的异常处理：



+ 渲染特定的视图页面
+ 使用ModelAndView返回更多的业务信息



大多数网站都会使用一个特定的页面来响应这些异常，而不是直接返回一个HTTP状态码或者显示Java异常调用栈。当然异常信息对于开发人员是非常有用的，如果想要在视图中直接看到它们可以这样渲染模板（以JSP为例）：



```plain
<h1>Error Page</h1>
<p>Application has encountered an error. Please contact support on ...</p>

<!--
Failed URL: ${url}
Exception:  ${exception.message}
<c:forEach items="${exception.stackTrace}" var="ste">    ${ste} 
</c:forEach>
-->
```



### 4、全局异常处理


`@ControllerAdvice`提供了和上一节一样的异常处理能力，但是可以被应用于Spring应用上下文中的所有`@Controller`：



```java
@ControllerAdvice
class GlobalControllerExceptionHandler {
    @ResponseStatus(HttpStatus.CONFLICT)  // 409
    @ExceptionHandler(DataIntegrityViolationException.class)
    public void handleConflict() {
        // Nothing to do
    }
}
```



Spring MVC默认对于没有捕获也没有被`@ResponseStatus`以及`@ExceptionHandler`声明的异常，会直接返回500，这显然并不友好，可以在`@ControllerAdvice`中对其进行处理（例如返回一个友好的错误页面，引导用户返回正确的位置或者提交错误信息）：



```java
@ControllerAdvice
class GlobalDefaultExceptionHandler {
    public static final String DEFAULT_ERROR_VIEW = "error";

    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        // If the exception is annotated with @ResponseStatus rethrow it and let
        // the framework handle it - like the OrderNotFoundException example
        // at the start of this post.
        // AnnotationUtils is a Spring Framework utility class.
        if (AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class) != null)
            throw e;

        // Otherwise setup and send the user to a default error-view.
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception", e);
        mav.addObject("url", req.getRequestURL());
        mav.setViewName(DEFAULT_ERROR_VIEW);
        return mav;
    }
}
```



### 5、总结


Spring在异常处理方面提供了一如既往的强大特性和支持，那么在应用开发中我们应该如何使用这些方法呢？以下提供一些经验性的准则：



+ 不要在`@Controller`中自己进行异常处理逻辑。即使它只是一个Controller相关的特定异常，在`@Controller`中添加一个`@ExceptionHandler`方法处理。  
对于自定义的异常，可以考虑对其加上`@ResponseStatus`注解
+ 使用`@ControllerAdvice`处理通用异常（例如资源不存在、资源存在冲突等）



> 更新: 2022-04-09 16:52:44  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ewndzk>