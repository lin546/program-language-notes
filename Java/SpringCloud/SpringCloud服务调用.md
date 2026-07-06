# Spring Cloud 服务调用

> <font style="color:rgb(36, 41, 46);">在上一篇文章，讲了服务的注册和发现。在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于http restful的。</font>

<font style="color:rgb(36, 41, 46);">创建服务消费者根据使用的API的不同，大致分为三种方式。虽然大家在实际使用中用的应该都是</font><code><font style="color:rgb(36, 41, 46);">Feign</font></code><font style="color:rgb(36, 41, 46);"> ,但是这里把三种都介绍一下。</font>

<font style="color:rgb(36, 41, 46);">三种方式均使用同一配置文件，不再单独说明。</font>

```yaml
spring:
  application:
    name: eureka-consumer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/ # 指定 Eureka 注册中心的地址
server:
  port: 9000 # 分别为 9000、9001、9002
```

## 一、使用 LoadBalancerClient

> <font style="color:rgb(44, 62, 80);">从</font>`LoadBalancerClient`<font style="color:rgb(44, 62, 80);">接口的命名中，我们就知道这是一个负载均衡客户端的抽象定义，下面我们就看看如何使用 Spring Cloud 提供的负载均衡器客户端接口来实现服务的消费。</font>

### 1、引入依赖

> <font style="color:rgb(44, 62, 80);">我们先来创建一个服务消费者工程，命名为：</font><code>**<font style="color:rgb(44, 62, 80);">eureka-consumer</font>**</code>**<font style="color:rgb(44, 62, 80);">, </font>**<code><font style="color:rgb(44, 62, 80);">pom.xml</font></code><font style="color:rgb(44, 62, 80);"> 同 Producer 的。</font>

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2、启动类

> 初始化 `RestTemplate` ，用来发起 `REST` 请求。

```java
@SpringBootApplication
public class EurekaConsumerApplication {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }
}
```

### 3、Controller

> 创建一个接口用来消费 <code><font style="color:rgb(44, 62, 80);">eureka-producer</font></code> 提供的接口：

```java
@RequestMapping("/hello")
@RestController
public class HelloController {

    @Autowired
    private LoadBalancerClient client;

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/")
    public String hello(@RequestParam String name) {
        name += "!";
        ServiceInstance instance = client.choose("eureka-producer");
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/hello/?name=" + name;
        return restTemplate.getForObject(url, String.class);
    }

}
```

<font style="color:rgb(44, 62, 80);">可以看到这里，我们注入了</font>LoadBalancerClient<font style="color:rgb(44, 62, 80);">和</font>RestTemplate<font style="color:rgb(44, 62, 80);">，并在</font>hello<font style="color:rgb(44, 62, 80);">方法中，先通过</font>loadBalancerClient<font style="color:rgb(44, 62, 80);">的</font>choose<font style="color:rgb(44, 62, 80);">方法来负载均衡的选出一个</font>eureka-producer<font style="color:rgb(44, 62, 80);">的服务实例，这个服务实例的基本信息存储在</font>ServiceInstance<font style="color:rgb(44, 62, 80);">中，然后通过这些对象中的信息拼接出访问服务调用者的</font>/hello/<font style="color:rgb(44, 62, 80);">接口的详细地址，最后再利用</font>RestTemplate<font style="color:rgb(44, 62, 80);">对象实现对服务提供者接口的调用。</font>

<font style="color:rgb(44, 62, 80);">另外，为了在调用时能从返回结果上与服务提供者有个区分，在这里我简单处理了一下，</font>name+="!"<font style="color:rgb(44, 62, 80);">，即服务调用者的 response 中会比服务提供者的多一个感叹号（!）。</font>

<font style="color:rgb(44, 62, 80);">访问</font><http://localhost:9000/hello/?name=yibo><font style="color:rgb(44, 62, 80);">以验证是否调用成功</font>

```plain
Hello, yibo! Fri Apr 13 18:44:55 CST 2018
```

## 二、使用 Spring Cloud Ribbon

> <font style="color:rgb(44, 62, 80);">Ribbon 是一个基于 HTTP 和 TCP 的客户端负载均衡器。它可以通过在客户端中配置 </font><code><font style="color:rgb(44, 62, 80);">ribbonServerList</font></code><font style="color:rgb(44, 62, 80);"> 来设置服务端列表去轮询访问以达到均衡负载的作用。</font>
>
> <font style="color:rgb(44, 62, 80);">当 Ribbon 与 Eureka 联合使用时，</font><code><font style="color:rgb(44, 62, 80);">ribbonServerList</font></code><font style="color:rgb(44, 62, 80);"> 会被 </font><code><font style="color:rgb(44, 62, 80);">DiscoveryEnabledNIWSServerList </font></code><font style="color:rgb(44, 62, 80);">重写，扩展成从 Eureka 注册中心中获取服务实例列表。同时它也会用 </font><code><font style="color:rgb(44, 62, 80);">NIWSDiscoveryPing</font></code><font style="color:rgb(44, 62, 80);"> 来取代 </font><code><font style="color:rgb(44, 62, 80);">IPing</font></code><font style="color:rgb(44, 62, 80);">，它将职责委托给 Eureka 来确定服务端是否已经启动。</font>

### 1、引入依赖

创建一个基础的 SpringBoot 工程，命名为 <code><font style="color:rgb(44, 62, 80);">eureka-consumer-ribbon</font></code><font style="color:#565A5F;">，并在</font><font style="color:#E96900;background-color:#F8F8F8;">pom.xml</font><font style="color:#565A5F;">中引入需要的依赖内容。</font>

> <font style="color:rgb(44, 62, 80);">至于 </font><code><font style="color:rgb(44, 62, 80);">spring-cloud-starter-ribbon</font></code><font style="color:rgb(44, 62, 80);">，因为我使用的 Spring Cloud 版本是 </font><code><font style="color:rgb(44, 62, 80);">Finchley.RC1</font></code><font style="color:rgb(44, 62, 80);">，</font><code><font style="color:rgb(44, 62, 80);">spring-cloud-starter-netflix-eureka-client</font></code><font style="color:rgb(44, 62, 80);"> 里边已经包含了 </font><code><font style="color:rgb(44, 62, 80);">spring-cloud-starter-netflix-ribbon</font></code><font style="color:rgb(44, 62, 80);"> 了</font>

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2、启动类

修改应用主类，为 `RestTemplate` 添加 `@LoadBalanced` 注解。

```java
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

### 3、Controller

<font style="color:rgb(44, 62, 80);">修改 controller，去掉</font><code><font style="color:rgb(44, 62, 80);">LoadBalancerClient</font></code><font style="color:rgb(44, 62, 80);">，并修改相应的方法，直接用 </font><code><font style="color:rgb(44, 62, 80);">RestTemplate</font></code><font style="color:rgb(44, 62, 80);">发起请求</font>

```java
@RequestMapping("/hello")
@RestController
public class HelloController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/")
    public String hello(@RequestParam String name) {
        name += "!";
        String url = "http://eureka-producer/hello/?name=" + name;
        return restTemplate.getForObject(url, String.class);
    }
}
```

<font style="color:rgb(44, 62, 80);">可能你已经注意到了，这里直接用服务名</font><code><font style="color:rgb(44, 62, 80);">eureka-producer</font></code><font style="color:rgb(44, 62, 80);">取代了之前的具体的</font><code><font style="color:rgb(44, 62, 80);">host:port</font></code><font style="color:rgb(44, 62, 80);">。那么这样的请求为什么可以调用成功呢？因为 Spring Cloud Ribbon 有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，并将这里的服务名替换成实际要请求的 IP 地址和端口，从而完成服务接口的调用。</font>

<font style="color:rgb(44, 62, 80);">另外，为了在调用时能从返回结果上与服务提供者有个区分，在这里我简单处理了一下，</font>`name+="!"`<font style="color:rgb(44, 62, 80);">，即服务调用者的 response 中会比服务提供者的多一个感叹号（!）。</font>

<font style="color:rgb(44, 62, 80);">访问</font><http://localhost:9000/hello/?name=yibo><font style="color:rgb(44, 62, 80);">以验证是否调用成功</font>

```java
Hello, yibo! Fri Apr 13 18:44:55 CST 2018
```

## 三、Spring Cloud Feign

> <font style="color:rgb(44, 62, 80);">在实际工作中，我们基本上都是使用 Feign 来完成调用的。我们通过一个例子来展现 Feign 如何方便的声明对 eureka-producer 服务的定义和调用。</font>

### 1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2、启动类

<font style="color:rgb(44, 62, 80);">在启动类上加上</font><code><font style="color:rgb(44, 62, 80);">@EnableFeignClients</font></code>

```java
@EnableFeignClients
@SpringBootApplication
public class EurekaConsumerFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerFeignApplication.class, args);
    }
}
```

### 3、Feign 调用实现

<font style="color:rgb(44, 62, 80);">创建一个 Feign 的客户端接口定义。使用</font><code><font style="color:rgb(44, 62, 80);">@FeignClient</font></code><font style="color:rgb(44, 62, 80);">注解来指定这个接口所要调用的服务名称，接口中定义的各个函数使用 Spring MVC 的注解就可以来绑定服务提供方的 REST 接口，比如下面就是绑定 eureka-producer 服务的</font><code><font style="color:rgb(44, 62, 80);">/hello/</font></code><font style="color:rgb(44, 62, 80);">接口的例子：</font>

```java
@FeignClient(name = "eureka-producer")
public interface HelloRemote {

    @GetMapping("/hello/")
    String hello(@RequestParam(value = "name") String name);

}
```

<font style="color:rgb(44, 62, 80);">此类中的方法和远程服务中 Contoller 中的方法名和参数需保持一致。</font>

> 这里有几个坑，后边有详细说明。

### 4、Controller

<font style="color:rgb(44, 62, 80);">修改 Controller，将 HelloRemote 注入到 controller 层，像普通方法一样去调用即可。</font>

```java
@RequestMapping("/hello")
@RestController
public class HelloController {

    @Autowired
    HelloRemote helloRemote;

    @GetMapping("/{name}")
    public String index(@PathVariable("name") String name) {
        return helloRemote.hello(name + "!");
    }

}
```

<font style="color:rgb(44, 62, 80);">通过 Spring Cloud Feign 来实现服务调用的方式非常简单，通过</font><font style="color:rgb(44, 62, 80);">@FeignClient</font><font style="color:rgb(44, 62, 80);">定义的接口来统一的声明我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可。由于 Feign 是基于 Ribbon 实现的，所以它自带了客户端负载均衡功能，也可以通过 Ribbon 的 IRule 进行策略扩展。另外，Feign 还整合的 Hystrix 来实现服务的容错保护，这个在后边会详细讲。（在 Finchley.RC1 版本中，Feign 的 Hystrix 默认是关闭的。参考</font>[Spring Cloud OpenFeign](https://github.com/spring-cloud/spring-cloud-openfeign/blob/30d92458218067dfc84f529d8bb519450b268297/docs/src/main/asciidoc/spring-cloud-openfeign.adoc)<font style="color:rgb(44, 62, 80);">和</font>[Disable HystrixCommands For FeignClients By Default](https://github.com/spring-cloud/spring-cloud-netflix/issues/1277)<font style="color:rgb(44, 62, 80);">）。</font>

<font style="color:rgb(44, 62, 80);">访问</font><http://localhost:9002/hello/xiaoming><font style="color:rgb(44, 62, 80);">以验证是否调用成功</font>

## 参考

* <https://www.haoyizebo.com/posts/1066a978/#>


> 更新: 2022-04-09 16:52:59  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ozflgq>