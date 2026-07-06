# DispatcherServlet

[Spring Docs](https://docs.spring.io/spring/docs/5.0.16.RELEASE/spring-framework-reference/web.html#mvc-servlet)



Spring MVC框架，与其他很多web的MVC框架一样：请求驱动；所有设计都围绕着一个中央Servlet来展开，它负责把所有请求分发到控制器；同时提供其他web应用开发所需要的功能。不过Spring的中央处理器，`DispatcherServlet`，能做的比这更多。



`DispatcherServlet`和其他Servlet一样需要使用java配置类或者web.xml声明和映射路径。



下面是使用`web.xml`配置注册和初始化`DispatcherServlet`的一个例子:



```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```



## 一、Context Hierarchy


`DispatcherServlet`需要一个`WebApplicationContext`,对于许多应用来说使用一个 `WebApplicationContext`就足够了，但是你也可以将上下文分层，即包含一个`root WebApplicationContext`用来在多个`DispatcherServlet`实例间共享信息，每个`DispatcherServlet`拥有一个`child WebApplicationContext`.如下图：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20200328223412.png)



使用`Web.xml`配置如下：



```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```



## 二、Special Bean Types


Spring的`DispatcherServlet`使用了特殊的bean来处理请求、渲染视图等，这些特定的bean是Spring MVC框架的一部分。



### 1、HandlerMapping


处理器映射。它会根据某些规则将进入容器的请求映射到具体的处理器以及一系列处理器拦截器上。具体的规则视`HandlerMapping`类的实现不同而有所不同。



主要有两个实现：



+ `RequestMappingHandlerMapping` (which supports [@RequestMapping ](/RequestMapping ) annotated methods) 
+ `SimpleUrlHandlerMapping` (which maintains explicit registrations of URI path patterns to handlers)



### 2、HandlerAdapter


处理器适配器。拿到请求所对应的处理器后，适配器将负责去调用该处理器，这使得 `DispatcherServlet` 无需关心具体的调用细节。



### 3、HandlerExceptionResolver


处理异常的策略，可能会把异常映射到handlers，HTML错误页面等。



### 4、ViewResolver


视图解析器。它负责将一个代表逻辑视图名的字符串（String）映射到实际的视图类型View上。



### 5、LocaleResolver, LocaleContextResolver


地区解析器 和 地区上下文解析器。它们负责解析客户端所在的地区信息甚至时区信息，为国际化的视图定制提供了支持。



### 6、ThemeResolver


主题解析器。它负责解析你web应用中可用的主题，比如，提供一些个性化定制的布局等。



### 7、MultipartResolver


解析multi-part的传输请求，比如支持通过HTML表单进行的文件上传等。



### 8、FlashMapManager


FlashMap管理器。它能够存储并取回两次请求之间的FlashMap对象。后者可用于在请求之间传递数据，通常是在请求重定向的情境下使用。



## 三、Web MVC Config


`DispatcherServlet`首先在Web上下文检查每一个特殊的Bean类型实现，如果没有找到，就会使用bean类型默认的实现：



```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
    org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping,\
    org.springframework.web.servlet.function.support.RouterFunctionMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
    org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
    org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter,\
    org.springframework.web.servlet.function.support.HandlerFunctionAdapter


org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
    org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
    org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```



> 更新: 2022-04-09 16:52:43  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/mgxrcf>