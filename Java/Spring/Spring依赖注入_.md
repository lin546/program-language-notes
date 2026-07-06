# Spring依赖注入

## 一、概念

我们经常说的控制反转(Inversion of Control-IOC)和依赖注入(Dependency injection-DI)在Spring环境下是等同的概念，控制反转是通过依赖注入实现的。所谓依赖注入指的是容器负责创建对象和维护对象间的依赖关系，而不是通过对象本身的创建和解决自己的依赖。

Spring IoC容器（ApplicationContext）负责创建Bean，并通过容器将功能类Bean注入到你需要的Bean中。Spring提供使用XML、注解、Java配置实现Bean的创建和注入。

## 二、相关注解

### 1、声明Bean的注解

* [@Component ](/Component) 组件，没有明确的角色。
* [@Service ](/Service) 在业务层(service层)使用。
* [@Repository ](/Repository) 在数据访问层(dao层)使用。
* @Controller在展示层(MVC->Spring MVC)使用。

启动了自动扫描后，被声明的类都自动会注册为Spring Beans。在Spring上下文中，每个Bean都有自己的ID。Spring扫描过程中所创建的Bean都有默认的ID，这个ID为将类名首字母小写形成的字符串。如果我们希望给某个类对应的Bean取特点的名字，则可以给@Component标注传入指定的参数。比如：

```java
@Component("customizedName")
public class UserDao {
    ...
}
```

### 2、注入bean的注解

(1)、[\*\*@Autowired \*\*](/Autowired)\*\* \*\*：Spring提供的注解,默认按类型匹配的方式，在容器查找匹配的Bean，当有且仅有一个匹配的Bean时，Spring将其注入@Autowired标注的变量中。

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}
```

(2)、[\*\*@Inject \*\*](/Inject)\*\* \*\*：JSR-330提供的注解。

(3)、@**Resource**：JSR-250提供的注解。

```java
public class AnotationExp {

    @Resource(name = "HappyClient")
    private HappyClient happyClient;
    
    @Resource(type = HappyPlayAno .class)
    private HappyPlayAno happyPlayAno;
}
```

* @Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配
* 指定了name或者type则根据指定的类型去匹配bean
* 指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错

上面三个注解都可以用在set方法上或者属性上，建议用在属性上，优点是代码更少，层次更清晰。

使用使用自动装配时，如果在应用上下文中，对应类型的Bean有且只有一个，则会自动装配到该属性上。

如果没有找到对应的Bean，应用会抛出对应的异常，如果想避免抛出这个异常，则需要设置`@Autowired(required=false)`。

### 3、[@Qualifier ](/Qualifier)

@Autowired注解**默认按照类型装配**，如果容器中包含多个同一类型的Bean，那么启动容器时会报找不到指定类型bean的异常，解决办法是结合`@Qualifier`注解进行限定，指定注入的bean名称。

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {

	String value() default "";

}
```

## 三、启动自动扫描

### 1、SpringBoot

在Spring Boot应用中，加入`@SpringBootApplication`标注就能启动自动扫描。因为`@SpringBootApplication`包含一个`@ComponentScan`注解。

### 2、XML配置方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

<context:component-scan base-package="com.example" />
</beans>
```

`<context:component-scan>`标签将会开启Spring Beans的自动扫描，并可设置`base-package`属性，表示Spring将会扫描该目录以及子目录下所有被`@Component`标注修饰的类，对它们进行装配。

### 3、Java配置方式

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example")
public class BlogSystemConfig {
}
```

* `@Configuration`表示这个Java文件是一个配置文件，这类Java代码的作用在于配置，要和普通的Java代码区分开发，因此一般我们需要将其放在一个专门的包中，比如com.example.config。
* `@ComponentScan`表示开启Spring Beans扫描，类似于XML配置，这里也可以可以设置basePackages属性。

下面详细介绍Java配置。

## 四、Java配置

Java配置是Spring推荐的配置方式，可以完全代替xml配置，Java配置是通过`@Configuration`和`@Bean`来实现的。

* `@Configuration`声明当前类是一个配置类，相当于一个Spring配置的xml文件。
* `@Bean`注解在方法上，声明当前方法的返回值为一个Bean，当于xml配置中的

一般项目会混同Java配置和注解配置，原则是：全局配置使用Java配置（如数据库相关配置、MVC相关配置），业务Bean的配置注解配置（@Service、@Component、@Repositry、@Controller）。


> 更新: 2022-04-09 16:52:43  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/cpa1ky>