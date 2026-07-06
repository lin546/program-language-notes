# Spring Boot 基础知识

## 一、Starter Parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>
```

需要注意的是这里需要指定版本号，但是如果你需要引入可选的starter，则可以忽略版本号。如果你想指定特定的版本号可以在pom.xml中配置，如：

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

完整的配置列表见[spring-boot-dependencies](https://github.com/spring-projects/spring-boot/blob/v2.2.2.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)

<code>**starters**</code>\*\* \*\*包含了大量的依赖集合，你可以使用将其加入到项目中达到快速开发的目的。这些`starters`由Springboot提供，位于`org.springframework.boot` group。关于`starters`的详细信息访问[官方文档](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)

\[

]\(https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)

## 二、@SpringBootApplication

@SpringBootApplication是Sping Boot的核心注解，它是一个组合注解，源码如下：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {
    
    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

`@SpringBootApplication`注解主要组合了三个注解。

* `@SpringBootConfiguration`： 将该类声明为配置类。这个注解实际上是`@Configuration` 注解的特殊形式。
* `@EnableAutoConfiguration `：启用 Spring Boot 的自动配置。
* `ComponentScan`：启用组件扫描。这样我们能够通过想 `@Component`、`@Controller`、`@Service` 这样的注解声明其他类，以便让 Spring 自动发现它们并将它们注册为 Spring 应用程序上下文中的组件。

## 三、配置文件

SpringBoot 使用一个全局的配置文件`application.properties`或`application.yml`，放置在`src/main/resources`目录或者类路径的`/config`下。

SpringBoot的全局配置文件的作用是对一些默认配置的配置值进行修改。如修改内置服务器端口：

```properties
server.port=8080
```

### 1、配置基础

<font style="color:#565A5F;">Spring Boot的默认配置文件位置为： </font>`src/main/resources/application.properties`<font style="color:#565A5F;">。据我们引入的不同Starter模块，可以在这里定义诸如：容器端口名、数据库链接信息、日志级别等各种配置信息。比如，我们需要自定义web模块的服务端口号，可以在</font>`application.properties`<font style="color:#565A5F;">中添加</font>`server.port=8888`<font style="color:#565A5F;">来指定服务端口为8888</font><font style="color:#565A5F;">。</font>

<font style="color:#565A5F;">Spring Boot的配置文件除了可以使用传统的properties文件之外，还支持现在被广泛推荐使用的YAML文件。比如：下面的一段YAML配置信息</font>

```yaml
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App

```

<font style="color:#565A5F;">与其等价的properties配置如下。</font>

```yaml
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
```

### 2、自定义参数

<font style="color:#565A5F;">我们除了可以在Spring Boot的配置文件中设置各个Starter模块中预定义的配置属性，也可以在配置文件中定义一些我们需要的自定义属性。比如在</font>`application.properties`<font style="color:#565A5F;">中添加：</font>

```yaml
name=lin546
word=hello
```

<font style="color:#565A5F;">然后，在应用中我们可以通过</font>`@Value`<font style="color:#565A5F;">注解来加载这些自定义的参数，比如：</font>

<font style="color:#565A5F;"></font>

```java
@RestController
public class UserController {

    @Value("${name}")
    private  String name;

    @RequestMapping("/")
    public String hello(){
        return "hello+ "+name;
    }
}
```

### 3、常用属性配置

#### （1）数据库连接信息

```properties
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

YAML格式更加简洁：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/test
    username: dbuser
    password: dbpass
    driver-class: com.mysql.jdbc.Driver
```

一旦发现这些信息，Spring Boot就会根据它们创建`DataSource`对象。

#### （2）Web应用服务器配置

```yaml
# Server settings (ServerProperties)
server:
  port: 8080
  address: 127.0.0.1
  sessionTimeout: 30
  contextPath: /

  # Tomcat specifics
  tomcat:
    accessLogEnabled: false
    protocolHeader: x-forwarded-proto
    remoteIpHeader: x-forwarded-for
    basedir:
    backgroundProcessorDelay: 30 # secs
```

通过`port`和`address`可以修改服务器监听的地址和端口，`sessionTimeout`配置session过期时间（再也不用修改web.xml了，因为它根本不存在）。同时如果在生产环境中使用内嵌Tomcat，当然希望能够配置它的日志、线程池等信息，这些现在都可以通过Spring Boot的属性文件配置，而不再需要再对生产环境中的Tomcat实例进行单独的配置管理了。

更详尽的配置列表访问[SpringBoot Docs](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/appendix-application-properties.html)

## 参考

* <http://tengj.top/2017/02/28/springboot2/>
* <https://blog.didispace.com/spring-boot-learning-21-1-3/>


> 更新: 2024-08-27 19:53:34  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/mzzglr>