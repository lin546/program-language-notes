# SpringBoot应用监控

SpringBoot提供了运行时的应用监控和管理的功能。我们可以通过Http和JMX协议来进行操作。本文基于SpringBoot版本为2.2.5

## 一、开启应用监控功能

开启该功能只需要引入`spring-boot-starter-actuator`依赖即可。

maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

gradle:

```groovy
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

## 二、端点 Endpoints

加入`actuator`依赖后，应用启动后会创建一些基于Web的Endpoint，通过/actuator/EndpointId来访问各端点。

| 端点 | function |
| --- | --- |
| actuator | 所有Endpoint的列表，需加入spring HATEOAS支持 |
| autoconfig | 用来查看Spring Boot的框架自动配置信息 |
| beans | 显示应用上下文的Bean列表 |
| configprops | 当前应用中所有的配置属性 |
| mappings | 显示所有的@RequestMapping映射的路径 |
| shutdown | 关闭该Web应用 |

## 三、Enabing Endpoints

默认情况下，除了`shutdown`,其它的端点都默认可用，对于不可用的端点可以通过配置`management.endpoint.<id>.enabled`开启，如开启`shutdown`端点：

```properties
management.endpoint.shutdown.enabled=true
```

## 四、Exposing Endpoints

因为端点可能包含一些敏感的信息，所以通过http模式访问会有很多限制，默认只开启了`info`和`health`的端口访问。通过配置`management.endpoints.web.exposure.include`配置端口访问，如：

```properties
management.endpoints.web.exposure.include=*
```

更多信息访问：[SpringBoot Docs](https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/html/production-ready-features.html#production-ready-enabling)


> 更新: 2022-04-09 16:52:48  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/xgf44l>