# Spring Cloud Config

> <font style="color:rgb(44, 62, 80);">Spring Cloud Config 是用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持，它分为服务端与客户端两个部分。</font>
>
> * <font style="color:rgb(44, 62, 80);">服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密 / 解密信息等访问接口。</font>
>
> * <font style="color:rgb(44, 62, 80);">而客户端则是微服务架构中的各个微服务应用或基础设施，它们通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。</font>
>
> <font style="color:rgb(44, 62, 80);">由于Spring Cloud Config 实现的配置中心默认采用 Git 来存储配置信息，所以使用 Spring Cloud Config 构建的配置服务器，天然就支持对微服务应用配置信息的版本管理，并且可以通过 Git 客户端工具来方便的管理和访问配置内容。当然它也提供了对其他存储方式的支持，比如：SVN 仓库、本地化文件系统。</font>
>
> <font style="color:rgb(44, 62, 80);">在本文中，我们将学习如何构建一个基于 Git 存储的分布式配置中心，并对客户端进行改造，并让其能够从配置中心获取配置信息并绑定到代码中的整个过程。最后，我们还将了解如何能让客户端获取到修改后的最新配置。</font>

## 一、准备工作

准备一个 Git 仓库，在 Github 或者 Gitee 上面创建一个文件夹 `config-repo` 用来存放配置文件，为了模拟生产环境，我们创建以下三个配置文件：

```plain
// 开发环境
config-client-dev.yml
// 测试环境
config-client-test.yml
// 生产环境
config-client-prod.yml
```

<font style="color:rgb(44, 62, 80);">每个配置文件中都写一个属性 neo.hello, 属性值分别是 dev、test、prod。</font>

```yaml
neo:
    hello: dev
```

<font style="color:rgb(44, 62, 80);">下面我们开始配置 Server 端。</font>

## 二、Server 端

创建一个基础的 SpringBoot 工程，命名为：`config-server-git`

### 1、添加依赖

只需要在 pom.xml 中加入`spring-cloud-config-server`即可。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 2、配置文件

<font style="color:rgb(44, 62, 80);">在 </font><code><font style="color:rgb(44, 62, 80);">application.yml</font></code><font style="color:rgb(44, 62, 80);"> 中添加配置服务的基本信息以及 Git 仓库的相关信息</font>

```yaml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/nanmu486/spring-cloud-study.git # 配置git仓库的地址
          search-paths: config-repo # git仓库地址下的相对地址，可以配置多个，用,分割。
server:
  port: 12000
```

> <font style="color:rgb(113, 128, 150);">如果我们的 Git 仓库需要权限访问，那么可以通过配置下面的两个属性来实现；</font><code><font style="color:rgb(113, 128, 150);">spring.cloud.config.server.git.username</font></code><font style="color:rgb(113, 128, 150);">：访问 Git 仓库的用户名</font><code><font style="color:rgb(113, 128, 150);">spring.cloud.config.server.git.password</font></code><font style="color:rgb(113, 128, 150);">：访问 Git 仓库的用户密码</font>

### 3、启动类

<font style="color:rgb(44, 62, 80);">启动类添加 </font><code><font style="color:rgb(44, 62, 80);">@EnableConfigServer</font></code><font style="color:rgb(44, 62, 80);">，激活对配置中心的支持</font>

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

<font style="color:rgb(44, 62, 80);">到此 Server 端相关配置已经完成。</font>

### 4、测试

<font style="color:rgb(86, 90, 95);">完成了这些准备工作之后，我们就可以通过浏览器、POSTMAN或CURL等工具直接来访问到我们的配置内容了。访问配置信息的URL与配置文件的映射关系如下：</font>

* <code><font style="color:rgb(86, 90, 95);">/{application}/{profile}[/{label}]</font></code>
* <code><font style="color:rgb(86, 90, 95);">/{application}-{profile}.yml</font></code>
* <code><font style="color:rgb(86, 90, 95);">/{label}/{application}-{profile}.yml</font></code>
* <code><font style="color:rgb(86, 90, 95);">/{application}-{profile}.properties</font></code>
* <code><font style="color:rgb(86, 90, 95);">/{label}/{application}-{profile}.properties</font></code>

<font style="color:rgb(86, 90, 95);">上面的url会映射 </font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">{application}-{profile}.properties </font><font style="color:rgb(86, 90, 95);">或者 </font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">{application}-{profile}.yml</font><font style="color:rgb(86, 90, 95);">对应的配置文件，其中</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">{label}</font><font style="color:rgb(86, 90, 95);">对应Git上不同的分支，默认为 master。</font>

首先我们测试Server端是否可以读取 github 上面的配置信息。直接访问 <http://localhost:12000/config-client/dev> 返回信息如下：

```json
{
	"name": "config-client",
	"profiles": ["dev"],
	"label": null,
	"version": "e3c2c2934a40ce6d1c54c1112fd052be51ea5528",
	"state": null,
	"propertySources": [{
		"name": "https://gitee.com/lin546/config-repo.git/config-client-dev.yml",
		"source": {
			"neo.hello": "dev"
		}
	}]
}
```

<font style="color:rgb(44, 62, 80);">上述的返回的信息包含了配置文件的位置、版本、配置文件的名称以及配置文件中的具体内容，说明 Server 端已经成功获取了 Git 仓库的配置信息。</font>

## 三、Client 端

<font style="color:rgb(44, 62, 80);">在完成了上述验证之后，确定配置服务中心已经正常运作，下面我们尝试如何在微服务应用中获取上述的配置信息。再创建一个基础的 Spring Boot 应用，命名为 </font><code><font style="color:rgb(44, 62, 80);">config-client</font></code><font style="color:rgb(44, 62, 80);">。</font>

### 1、添加依赖

在 pom.xml 中添加下述依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 2、配置文件

<font style="color:rgb(44, 62, 80);">需要配置两个配置文件，</font><code><font style="color:rgb(44, 62, 80);">application.yml</font></code><font style="color:rgb(44, 62, 80);"> 和</font><code><font style="color:rgb(44, 62, 80);"> bootstrap.yml</font></code><font style="color:rgb(44, 62, 80);">，配置分别如下：</font>

**<font style="color:rgb(44, 62, 80);">application.yml</font>**

```yaml
spring:
  application:
    name: config-client
server:
  port: 13000
```

**<font style="color:rgb(44, 62, 80);">bootstrap.yml</font>**

```yaml
spring:
  cloud:
    config:
        uri: http://localhost:12000 # 配置中心的具体地址，即 config-server
      name: config-client # 对应 {application} 部分
      profile: dev # 对应 {profile} 部分
      label: master # 对应 {label} 部分，即 Git 的分支。如果配置中心使用的是本地存储，则该参数无用
```

### 3、启动类

<font style="color:rgb(44, 62, 80);">启动类不用修改，只用 </font><code><font style="color:rgb(44, 62, 80);">@SpringBootApplication</font></code><font style="color:rgb(44, 62, 80);"> 就行了</font>

```java
@SpringBootApplication
public class ConfigClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}

}
```

### 4、Controller

<font style="color:rgb(44, 62, 80);">在 </font><code><font style="color:rgb(44, 62, 80);">Controller</font></code><font style="color:rgb(44, 62, 80);"> 中使用</font><code><font style="color:rgb(44, 62, 80);">@Value</font></code><font style="color:rgb(44, 62, 80);">注解来获取 Server 端参数的值</font>

```java
@RestController
public class HelloController {

    @Value("${neo.hello}")
    private String profile;

    @GetMapping("/info")
    public String hello() {
        return profile;
    }
}
```

### 5、测试

<font style="color:rgb(44, 62, 80);">启动项目后访问 </font><http://localhost:13000/info><font style="color:rgb(44, 62, 80);"> 返回</font><font style="color:rgb(44, 62, 80);">dev</font><font style="color:rgb(44, 62, 80);">说明已经正确的从 Server 端获取到了参数。到此一个完整的服务端提供配置服务，客户端获取配置参数的例子就完成了。</font>

<font style="color:rgb(44, 62, 80);">我们再做一个小实验，手动修改 </font><code><font style="color:rgb(44, 62, 80);">config-client-dev.yml</font></code><font style="color:rgb(44, 62, 80);"> 中配置信息为：</font><code><font style="color:rgb(44, 62, 80);">dev update</font></code><font style="color:rgb(44, 62, 80);"> 提交到 Github, 再次在浏览器访问</font><http://localhost:13000/info><font style="color:rgb(44, 62, 80);">返回：</font><code><font style="color:rgb(44, 62, 80);">dev</font></code><font style="color:rgb(44, 62, 80);">，说明获取的信息还是旧的参数，这是为什么呢？</font>

<font style="color:rgb(44, 62, 80);">因为 Spring Cloud Config 分服务端和客户端，服务端负责将 Git 中存储的配置文件发布成 REST 接口，客户端可以从服务端 REST 接口获取配置。但客户端并不能主动感知到配置的变化，从而主动去获取新的配置。客户端如何去主动获取新的配置信息呢，Spring Cloud 已经给我们提供了解决方案，每个客户端通过 POST 方法触发各自的 </font><code><font style="color:rgb(44, 62, 80);">/actuator/refresh</font></code><font style="color:rgb(44, 62, 80);">。</font>

## 四、Refresh

<font style="color:rgb(44, 62, 80);">仅修改客户端即</font><code><font style="color:rgb(44, 62, 80);"> config-client</font></code><font style="color:rgb(44, 62, 80);"> 项目，就可以实现 refresh 的功能。</font>

### 1、添加依赖·

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2、开启更新机制

<font style="color:rgb(44, 62, 80);">需要给加载变量的类上面加载 </font><code><font style="color:rgb(44, 62, 80);">@RefreshScope</font></code><font style="color:rgb(44, 62, 80);">，在客户端执行 </font><code><font style="color:rgb(44, 62, 80);">/actuator/refresh</font></code><font style="color:rgb(44, 62, 80);">的时候就会更新此类下面的变量值。</font>

```java
@RefreshScope
@RestController
public class HelloController {

    @Value("${neo.hello}")
    private String profile;

    @GetMapping("/info")
    public String hello() {
        return profile;
    }
}
```

### 3、配置

<font style="color:rgb(44, 62, 80);">Spring Boot 1.5.X 以上默认开通了安全认证，所以要在配置文件 </font><code><font style="color:rgb(44, 62, 80);">application.yml</font></code><font style="color:rgb(44, 62, 80);"> 中添加以下配置以将 </font><code><font style="color:rgb(44, 62, 80);">/actuator/refresh</font></code><font style="color:rgb(44, 62, 80);"> 这个 Endpoint 暴露出来</font>

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

### 4、测试

<font style="color:rgb(44, 62, 80);">改造完之后，我们重启 config-client，我们以 POST 请求的方式来访问</font><http://localhost:13000/actuator/refresh><font style="color:rgb(44, 62, 80);"> 就会更新配置文件至最新版本。这时访问 </font><http://localhost:13000/info> 返回 `dev update` <font style="color:rgb(44, 62, 80);">这就说明客户端已经得到了最新的值，Refresh 是有效的。</font>

## 参考

* <https://www.haoyizebo.com/posts/8f2a7e9d/>
* <https://blog.didispace.com/spring-cloud-starter-dalston-3/>


> 更新: 2022-04-09 16:53:00  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ovec0b>