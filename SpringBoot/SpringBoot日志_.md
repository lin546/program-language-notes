# SpringBoot 日志

[SpringBoot Docs](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-logging)

## 一、日志格式

默认情况下，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台。

```plain
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

从上面可以看到，日志输出内容元素具体如下：

* 时间日期：精确到毫秒
* 日志级别：`ERROR`, `WARN`, `INFO`, `DEBUG` or `TRACE`
* 进程ID
* 分隔符：--- 标识实际日志的开始
* 线程名：方括号括起来（可能会截断控制台输出）
* Logger名：通常使用源代码的类名
* 日志内容

## 二、控制台输出

默认情况下，日志信息输出到控制台，且只有 `ERROR-level`, `WARN-level`, 和 `INFO-level` 的信息会被输出. 你可以通过下面命令在run项目时开启`"debug"`或`"trace"`模式.

```plain
java -jar myapp.jar --debug|--trace
```

当然也可以在`application.properties`文件中指定`debug=true`或`debug=trace`,

> 注：如果你的 terminal 支持 ANSI, 可以配置 `spring.output.ansi.enabled=ALWAYS`来开启输出日志高亮.

## 三、Log Levels

所有支持的日志记录系统都可以在Spring环境中设置记录级别（例如在`application.properties`中）\
格式为：`'logging.level.* = LEVEL'`

* `logging.level`：日志级别控制前缀，`*`为包名或`Logger`名
* `LEVEL`：选项`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`, `OFF`

举例：

* `logging.level.cn.lin=DEBUG`：cn.lin包下所有class以DEBUG级别输出
* `logging.level.root=WARN`：root日志以WARN级别输出
* `logging.level.org.springframework.web=DEBUG`
* `logging.level.org.hibernate=ERROR`


> 更新: 2022-04-09 16:52:47  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/glyb2a>