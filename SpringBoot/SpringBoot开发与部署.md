# SpringBoot 开发与部署

在开发Spring Boot应用的过程中，Spring Boot直接执行`public static void main()`函数并启动一个内嵌的应用服务器（取决于类路径上的依赖是`Tomcat`还是`jetty`）来处理应用请求。对于生产环境，这样的部署方式同样有效，同时Spring Boot也支持传统的部署方式——将war包放入应用服务器中启动运行。

## 一、热部署

### 1、spring-boot-devtools

<code><font style="color:rgb(51, 51, 51);">spring-boot-devtools</font></code><font style="color:rgb(51, 51, 51);"> 为应用提供一些开发时特性，包括默认值设置，自动重启，livereload等。</font>

<font style="color:rgb(51, 51, 51);">想要使用</font><code><font style="color:rgb(51, 51, 51);">devtools</font></code><font style="color:rgb(51, 51, 51);">热部署功能，maven 添加依赖如下：</font>

#### （1）引入依赖

```json
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
```

<font style="color:rgb(51, 51, 51);">将依赖关系标记为可选</font>`<optional>true</optional>`<font style="color:rgb(51, 51, 51);">是一种最佳做法，可以防止使用项目将devtools 传递性地应用于其他模块。</font>

<font style="color:rgb(51, 51, 51);"></font>

#### （2）默认属性

<font style="color:rgb(51, 51, 51);">在Spring Boot集成Thymeleaf时，</font>`spring.thymeleaf.cache`<font style="color:rgb(51, 51, 51);"> 属性设置为</font><code><font style="color:rgb(51, 51, 51);">false</font></code><font style="color:rgb(51, 51, 51);">可以禁用模板引擎编译的缓存结果。现在，</font><code><font style="color:rgb(51, 51, 51);">devtools</font></code><font style="color:rgb(51, 51, 51);">会自动帮你做到这些，禁用所有模板的缓存，包括</font><code><font style="color:rgb(51, 51, 51);">Thymeleaf</font></code><font style="color:rgb(51, 51, 51);">, </font><code><font style="color:rgb(51, 51, 51);">Freemarker</font></code><font style="color:rgb(51, 51, 51);">,  </font><code><font style="color:rgb(51, 51, 51);">Velocity</font></code><font style="color:rgb(51, 51, 51);"> 等。</font>

<font style="color:rgb(51, 51, 51);"></font>

<font style="color:rgb(51, 51, 51);">更多的属性，请参考</font>[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/blob/v1.5.3.RELEASE/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)<font style="color:rgb(51, 51, 51);">。</font>

<font style="color:rgb(51, 51, 51);"></font>

#### （4）自动重启

<font style="color:rgb(51, 51, 51);">自动重启的原理在于 spring boot 使用两个</font><code><font style="color:rgb(51, 51, 51);">classloader</font></code><font style="color:rgb(51, 51, 51);">：不改变的类（如第三方jar）由</font><code><font style="color:rgb(51, 51, 51);">base</font></code><font style="color:rgb(51, 51, 51);">类加载器加载，正在开发的类由</font><code><font style="color:rgb(51, 51, 51);">restart</font></code><font style="color:rgb(51, 51, 51);">类加载器加载。应用重启时，</font><code><font style="color:rgb(51, 51, 51);">restart</font></code><font style="color:rgb(51, 51, 51);">类加载器被扔掉重建，而</font><code><font style="color:rgb(51, 51, 51);">base</font></code><font style="color:rgb(51, 51, 51);">类加载器不变，这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为</font><code><font style="color:rgb(51, 51, 51);">base</font></code><font style="color:rgb(51, 51, 51);">类加载器已经可用并已填充。</font>

<font style="color:rgb(51, 51, 51);">所以，当我们开启</font><code><font style="color:rgb(51, 51, 51);">devtools</font></code><font style="color:rgb(51, 51, 51);">后，</font><code><font style="color:rgb(51, 51, 51);">classpath</font></code><font style="color:rgb(51, 51, 51);">中的文件变化会导致应用自动重启。\ </font><font style="color:rgb(51, 51, 51);">当然不同的IDE效果不一样，Eclipse 中保存文件即可引起</font><code><font style="color:rgb(51, 51, 51);">classpath</font></code><font style="color:rgb(51, 51, 51);">更新(注：需要打开自动编译)，从而触发重启。而 IDEA 则需要自己手动 CTRL+F9 重新编译一下（感觉IDEA这种更好，不然每修改一个地方就重启，好蛋疼）</font>

<font style="color:rgb(51, 51, 51);"></font>

<font style="color:rgb(51, 51, 51);"></font>

## 二、生产部署

### 1、jar形式

若我们在新建SpringBoot项目的时候，选择打包为jar，则只需要使用：

```plain
mvn package
```

生成的jar包，可直接使用下面命令运行：

```plain
java -jar xx.jar
```

### 2、war包形式

新建SpringBoot项目时可选择打包方式为war形式，打包的方式和jar包一致,执行：

```plain
mvn package
```

生成的war文件可以放在Servlet容器上运行。

### 3、打包方式为jar时

若我们新建SpringBoot项目时选择打包方式为jar，部署时又想用war包形式部署，我们只需比较两者的不同，这时我们可以做如下修改（反过来也适用）:

* 修改pom.xml文件

```plain
<packaging>war</packaging>
```

* 增加下面依赖覆盖内嵌的Tomcat依赖

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
  </dependency>
```

* 在入口类同目录下增加ServletInitializer类，内容如下

```java
public class ServletInitializer extends SpringBootServletInitializer {
  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
      return application.sources(DemoApplication.class);
  }
}
```

## 基于Docker的部署

## 参考

* <http://tengj.top/2017/06/01/springboot10/>


> 更新: 2022-04-09 16:52:48  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/wl25no>