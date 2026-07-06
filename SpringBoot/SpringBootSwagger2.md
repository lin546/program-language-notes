# Spring Boot Swagger2

## 一、准备工作

<font style="color:#565A5F;">首先，我们需要一个Spring Boot实现的RESTful API工程，这里</font><font style="color:#565A5F;">直接使用上一篇中的样例作为基础。</font>

<font style="color:#565A5F;"></font>

github地址：[https://github.com/lin546/springboot-demos/tree/main/demo](https://github.com/lin546/springboot-demos/tree/main/demo1)2

## 二、整合 Swagger2

下面，我们以上面仓库中的demo2 工程进行整合改造.

### 1、添加 Swagger2 依赖

在`pom.xml`中加入依赖，具体如下：

```xml
		<!-- swagger -->
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.9.2</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.10.5</version>
		</dependency>
```

**依赖说明：**

**（1）springfox-swagger2**

检测spring的web请求信息，生成检测结果（`json格式`）。

**（2）springfox-swagger-ui**

根据springfox-swagger2生成的数据，生成`可视化`的友好页面。

### 2、创建 Swagger2 配置类

<font style="color:#565A5F;"></font>

<font style="color:#565A5F;">在主类</font><font style="color:#565A5F;">同级或子级创建Swagger2的配置类</font>`Swagger2`<font style="color:#565A5F;">。</font>

```java

@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.lin"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("标题")
                .description("描述")
                .termsOfServiceUrl("服务条款URL")
                .contact("维护人")
                .version("1.0")
                .build();
    }

}
```

<font style="color:#565A5F;">如上代码所示，通过</font>`@Configuration`<font style="color:#565A5F;">注解，让Spring来加载该类配置。再通过</font>`@EnableSwagger2`<font style="color:#565A5F;">注解来启用Swagger2。</font>

<font style="color:#565A5F;"></font>

<font style="color:#565A5F;">再通过</font>`createRestApi`<font style="color:#565A5F;">函数创建</font>`Docket`<font style="color:#565A5F;">的Bean之后，</font>`apiInfo()`<font style="color:#565A5F;">用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。</font>`select()`<font style="color:#565A5F;">函数返回一个</font>`ApiSelectorBuilder`<font style="color:#565A5F;">实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被</font>`@ApiIgnore`<font style="color:#565A5F;">指定的请求）。</font>

### 3、访问 Swagger UI

<font style="color:#565A5F;">启动应用，访问：</font>`http://localhost:8080/swagger-ui.html`

<font style="color:#565A5F;"></font>

## 参考

* <https://blog.didispace.com/springbootswagger2/>

## 源代码

<https://github.com/lin546/springboot-demos/tree/main/demo3>


> 更新: 2022-04-09 16:52:57  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/qcxfqq>