# SpringBoot 静态资源

## 一、SSM 中的配置

<font style="color:rgb(74, 74, 74);">要讲 Spring Boot 中的问题，我们得先回到 SSM 环境搭建中，一般来说，我们可以通过 </font><code><font style="color:rgb(255, 56, 96);"><mvc:resources /></font><font style="color:rgb(74, 74, 74);"> </font></code><font style="color:rgb(74, 74, 74);">节点来配置不拦截静态资源，如下：</font>

```xml
<mvc:resources mapping="/js/**" location="/js/"/>
<mvc:resources mapping="/css/**" location="/css/"/>
<mvc:resources mapping="/html/**" location="/html/"/>
```

<font style="color:rgb(74, 74, 74);">由于这是一种Ant风格的路径匹配符，</font><font style="color:rgb(255, 56, 96);">/\*\*</font><font style="color:rgb(74, 74, 74);"> 表示可以匹配任意层级的路径，因此上面的代码也可以像下面这样简写：</font>

```xml
<mvc:resources mapping="/**" location="/"/>
```

<font style="color:rgb(74, 74, 74);">这种配置是在 XML 中的配置，SpringMVC 的配置除了在XML中配置，也可以在 Java 代码中配置，如果在Java代码中配置的话，我们只需要自定义一个类，继承自 </font><code><font style="color:rgb(74, 74, 74);">WebMvcConfigurationSupport</font></code><font style="color:rgb(74, 74, 74);"> 即可：</font>

```java
@Configuration
@ComponentScan(basePackages = "org.sang.javassm")
public class SpringMVCConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("/");
    }
}
```

<font style="color:rgb(74, 74, 74);">重写 </font><code><font style="color:rgb(74, 74, 74);">WebMvcConfigurationSupport </font></code><font style="color:rgb(74, 74, 74);">类中的</font><code><font style="color:rgb(74, 74, 74);">addResourceHandlers</font></code><font style="color:rgb(74, 74, 74);">方法，在该方法中配置静态资源位置即可，这里的含义和上面 xml 配置的含义一致。</font>\ <font style="color:rgb(74, 74, 74);">这是我们传统的解决方案，在Spring Boot 中，其实配置方式和这个一脉相承，只是有一些自动化的配置了。</font>

<font style="color:rgb(74, 74, 74);"></font>

## 二、SpringBoot 中的配置

<font style="color:rgb(74, 74, 74);">在 Spring Boot 中，如果我们是从 </font><font style="color:rgb(255, 56, 96);">https://start.spring.io</font><font style="color:rgb(74, 74, 74);"> 这个网站上创建的项目，默认都会存在 </font><code><font style="color:rgb(74, 74, 74);">resources/static </font></code><font style="color:rgb(74, 74, 74);">目录，很多小伙伴也知道静态资源只要放到这个目录下，就可以直接访问，除了这里还有没有其他可以放静态资源的位置呢？为什么放在这里就能直接访问了呢？</font>

<font style="color:rgb(74, 74, 74);">首先，在 Spring Boot 中，默认情况下，一共有5个位置可以放静态资源，五个路径分别是如下5个：</font>

<font style="color:rgb(74, 74, 74);"></font>

1. <code><font style="color:rgb(74, 74, 74);">classpath:/META-INF/resources/</font></code>
2. <code><font style="color:rgb(74, 74, 74);">classpath:/resources/</font></code>
3. <code><font style="color:rgb(74, 74, 74);">classpath:/static/</font></code>
4. <code><font style="color:rgb(74, 74, 74);">classpath:/public/</font></code>
5. <code><font style="color:rgb(74, 74, 74);">/</font></code>

<font style="color:rgb(74, 74, 74);"></font>

<font style="color:rgb(74, 74, 74);">前四个目录好理解，分别对应了</font><code><font style="color:rgb(74, 74, 74);">resources</font></code><font style="color:rgb(74, 74, 74);">目录下不同的目录，第5个 </font><code><font style="color:rgb(255, 56, 96);">/</font></code><font style="color:rgb(74, 74, 74);"> 是啥意思呢？我们知道，在 Spring Boot 项目中，默认是没有 </font><code><font style="color:rgb(74, 74, 74);">webapp</font></code><font style="color:rgb(74, 74, 74);"> 这个目录的，当然我们也可以自己添加（例如在需要使用JSP的时候），这里第5个</font><code><font style="color:rgb(74, 74, 74);"> </font><font style="color:rgb(255, 56, 96);">/</font><font style="color:rgb(74, 74, 74);"> </font></code><font style="color:rgb(74, 74, 74);">其实就是表示 </font><code><font style="color:rgb(74, 74, 74);">webapp </font></code><font style="color:rgb(74, 74, 74);">目录中的静态资源也不被拦截。如果同一个文件分别出现在五个目录下，那么优先级也是按照上面列出的顺序。</font>

<font style="color:rgb(74, 74, 74);"></font>

<font style="color:rgb(74, 74, 74);">系统默认创建了</font><code><font style="color:rgb(74, 74, 74);"> </font><font style="color:rgb(255, 56, 96);">classpath:/static </font></code><font style="color:rgb(74, 74, 74);">目录， 正常情况下，我们只需要将我们的静态资源放到这个目录下即可，也不需要额外去创建其他静态资源目录，例如我在</font><code><font style="color:rgb(74, 74, 74);"> </font><font style="color:rgb(255, 56, 96);">classpath:/static/</font><font style="color:rgb(74, 74, 74);"> </font></code><font style="color:rgb(74, 74, 74);">目录下放了一张名为 </font><code><font style="color:rgb(74, 74, 74);">1.png</font></code><font style="color:rgb(74, 74, 74);"> 的图片，那么我的访问路径是：</font>

```java
http://localhost:8080/1.png
```

<font style="color:rgb(74, 74, 74);">这里大家注意，请求地址中并不需要 </font><code><font style="color:rgb(74, 74, 74);">static</font></code><font style="color:rgb(74, 74, 74);">，如果加上了</font><code><font style="color:rgb(74, 74, 74);">static</font></code><font style="color:rgb(74, 74, 74);">反而多此一举会报</font><code><font style="color:rgb(74, 74, 74);">404</font></code><font style="color:rgb(74, 74, 74);">错误。很多人会觉得奇怪，为什么不需要添加 </font><code><font style="color:rgb(74, 74, 74);">static</font></code><font style="color:rgb(74, 74, 74);">呢？资源明明放在 </font><code><font style="color:rgb(74, 74, 74);">static </font></code><font style="color:rgb(74, 74, 74);">目录下。其实这个效果很好实现，例如在</font><code><font style="color:rgb(74, 74, 74);">SSM</font></code><font style="color:rgb(74, 74, 74);">配置中，我们的静态资源拦截配置如果是下面这样：</font>

```xml
<mvc:resources mapping="/**" location="/static/"/>
```

<font style="color:rgb(74, 74, 74);">如果我们是这样配置的话，请求地址如果是 </font>`http://localhost:8080/1.png`<font style="color:rgb(74, 74, 74);"> 实际上系统会去 </font>`/static/1.png`<font style="color:rgb(74, 74, 74);"> 目录下查找相关的文件。在 Spring Boot 中也是类似的配置。</font>

比如我们访问放在`static`下的`css`或`js`目录下文件：

```json
<link href="/css/bootstrap.min.css" rel="stylesheet" type="text/css">
<script src="/js/form-validation.js"></script>
```


> 更新: 2022-04-09 16:52:46  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/nd79tm>