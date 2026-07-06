# Spring IOC 基础

## 一、IOC 概念

**<font style="color:rgb(34, 34, 34);">Ioc （Inversion of Control）</font>**<font style="color:rgb(34, 34, 34);">，中文叫做控制反转。这是一个概念，也是一种思想。控制反转，实际上就是指对一个对象的控制权的反转。例如，如下代码：</font>

```java
public class Book {
    private Integer id;
    private String name;
    private Double price;
//省略getter/setter
}

public class User {
    private Integer id;
    private String name;
    private Integer age;

    public void doSth() {
        Book book = new Book();
        book.setId(1);
        book.setName("故事新编");
        book.setPrice((double) 20);
    }
}
```

<font style="color:rgb(34, 34, 34);">在这种情况下，Book 对象的控制权在 </font><code><font style="color:rgb(34, 34, 34);">User </font></code><font style="color:rgb(34, 34, 34);">对象里边，这样，</font><code><font style="color:rgb(34, 34, 34);">Book </font></code><font style="color:rgb(34, 34, 34);">和 </font><code><font style="color:rgb(34, 34, 34);">User </font></code><font style="color:rgb(34, 34, 34);">高度耦合，如果在其他对象中需要使用 Book 对象，得重新创建，也就是说，对象的创建、初始化、销毁等操作，统统都要开发者自己来完成。如果能够将这些操作交给容器来管理，开发者就可以极大的从对象的创建中解脱出来。</font>

<font style="color:rgb(34, 34, 34);"></font>

<font style="color:rgb(34, 34, 34);">使用 Spring 之后，我们可以将对象的创建、初始化、销毁等操作交给 Spring 容器来管理。就是说，在项目启动时，所有的 Bean 都将自己注册到 Spring 容器中去（如果有必要的话），然后如果其他 Bean 需要使用到这个 Bean ，则不需要自己去 new，而是直接去 Spring 容器去要。</font>

<font style="color:rgb(34, 34, 34);">通过一个简单的例子看下这个过程。</font>

<font style="color:rgb(34, 34, 34);"></font>

## 二、实例

<font style="color:rgb(34, 34, 34);">首先创建一个普通的 Maven 项目，然后引入 </font><code><font style="color:rgb(34, 34, 34);">spring-context</font></code><font style="color:rgb(34, 34, 34);"> 依赖，如下：</font>

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
</dependencies>
```

<font style="color:rgb(34, 34, 34);">接下来，在 </font><code><font style="color:rgb(34, 34, 34);">resources </font></code><font style="color:rgb(34, 34, 34);">目录下创建一个 spring 的配置文件 </font>`applicationContext.xml`<font style="color:rgb(34, 34, 34);">（注意，一定要先添加依赖，后创建配置文件，否则创建配置文件时，没有模板选项）：</font>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

  
</beans>
```

<font style="color:rgb(34, 34, 34);">在这个文件中，我们可以配置所有需要注册到 Spring 容器的 Bean：</font>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book"/>
</beans>
```

<code><font style="color:rgb(34, 34, 34);">class </font></code><font style="color:rgb(34, 34, 34);">属性表示需要注册的 bean 的全路径，</font><code><font style="color:rgb(34, 34, 34);">id </font></code><font style="color:rgb(34, 34, 34);">则表示 bean 的唯一标记，也开可以 </font><code><font style="color:rgb(34, 34, 34);">name </font></code><font style="color:rgb(34, 34, 34);">属性作为 bean 的标记，在超过 99% 的情况下，</font><code><font style="color:rgb(34, 34, 34);">id </font></code><font style="color:rgb(34, 34, 34);">和 </font><code><font style="color:rgb(34, 34, 34);">name </font></code><font style="color:rgb(34, 34, 34);">其实是一样的，特殊情况下不一样。</font>

<font style="color:rgb(34, 34, 34);">接下来，加载这个配置文件：</font>

```xml
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
}
```

<font style="color:rgb(34, 34, 34);">执行 main 方法，配置文件就会被自动加载，进而在 Spring 中初始化一个 </font><code><font style="color:rgb(34, 34, 34);">Book </font></code><font style="color:rgb(34, 34, 34);">实例。此时，我们显式的指定 </font><code><font style="color:rgb(34, 34, 34);">Book </font></code><font style="color:rgb(34, 34, 34);">类的无参构造方法，并在无参构造方法中打印日志，可以看到无参构造方法执行了，进而证明对象已经在 Spring 容器中初始化了。</font>

<font style="color:rgb(34, 34, 34);"></font>

<font style="color:rgb(34, 34, 34);">最后，通过 </font><code><font style="color:rgb(34, 34, 34);">getBean </font></code><font style="color:rgb(34, 34, 34);">方法，可以从容器中去获取对象：</font>

```xml
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Book book = (Book) ctx.getBean("book");
        System.out.println(book);
    }
}
```

加载方式，除了`ClassPathXmlApplicationContext `之外（去 classpath 下查找配置文件），另外也可以使用 `FileSystemXmlApplicationContext `，`FileSystemXmlApplicationContext `会从操作系统路径下去寻找配置文件。

```xml
public class Main {
    public static void main(String[] args) {
        FileSystemXmlApplicationContext ctx = new FileSystemXmlApplicationContext("F:\\workspace5\\workspace\\spring\\spring-ioc\\src\\main\\resources\\applicationContext.xml");
        Book book = (Book) ctx.getBean("book");
        System.out.println(book);
    }
}
```

## 三、Bean 的获取

<font style="color:rgb(34, 34, 34);">在上一小节中，我们通过 </font><code><font style="color:rgb(34, 34, 34);">ctx.getBean</font></code><font style="color:rgb(34, 34, 34);"> 方法来从 Spring 容器中获取 Bean，传入的参数是 Bean 的 </font><code><font style="color:rgb(34, 34, 34);">name</font></code><font style="color:rgb(34, 34, 34);"> 或者 </font><code><font style="color:rgb(34, 34, 34);">id</font></code><font style="color:rgb(34, 34, 34);"> 属性。除了这种方式之外，也可以直接通过 Class 去获取一个 Bean。</font>

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Book book = ctx.getBean(Book.class);
        System.out.println(book);
    }
}
```

<font style="color:rgb(34, 34, 34);">这种方式有一个很大的弊端，如果存在多个实例，这种方式就不可用，例如，xml 文件中存在两个 Bean：</font>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book"/>
    <bean class="org.javaboy.Book" id="book2"/>
</beans>
```

<font style="color:rgb(34, 34, 34);">此时，如果通过 Class 去查找 Bean，会报如下错误：</font>

```xml
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'Book' available: expected single matching bean but found 2: book,book2
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveNamedBean(DefaultListableBeanFactory.java:1144)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveBean(DefaultListableBeanFactory.java:411)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:344)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:337)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1123)
	at Main.main(Main.java:8)
```

## 四、属性的注入

<font style="color:rgb(34, 34, 34);"></font>

<font style="color:rgb(34, 34, 34);">在 XML 配置中，属性的注入存在多种方式。</font>

<font style="color:rgb(34, 34, 34);"></font>

### 1、构造方法注入

<font style="color:rgb(34, 34, 34);">通过 Bean 的构造方法给 Bean 的属性注入值。</font>

<font style="color:rgb(34, 34, 34);"></font>

1. <font style="color:rgb(34, 34, 34);">第一步首先给 Bean 添加对应的构造方法：</font>

```java
public class Book {
    private Integer id;
    private String name;
    private Double price;

    public Book() {
        System.out.println("-----book init----");
    }

    public Book(Integer id, String name, Double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}
```

2. <font style="color:rgb(34, 34, 34);">在 xml 文件中注入 Bean</font>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="三国演义"/>
        <constructor-arg index="2" value="30"/>
    </bean>
</beans>
```

<font style="color:rgb(34, 34, 34);">这里需要注意的是，</font><code><font style="color:rgb(34, 34, 34);">constructor-arg</font></code><font style="color:rgb(34, 34, 34);"> 中的 </font><code><font style="color:rgb(34, 34, 34);">index </font></code><font style="color:rgb(34, 34, 34);">和 </font><code><font style="color:rgb(34, 34, 34);">Book</font></code><font style="color:rgb(34, 34, 34);"> 中的构造方法参数一一对应。写的顺序可以颠倒，但是 </font><code><font style="color:rgb(34, 34, 34);">index</font></code><font style="color:rgb(34, 34, 34);"> 的值和 </font><code><font style="color:rgb(34, 34, 34);">value</font></code><font style="color:rgb(34, 34, 34);"> 要一一对应。</font>

<font style="color:rgb(34, 34, 34);">另一种构造方法中的属性注入，则是通过直接指定参数名来注入：</font>

```xml
<bean class="org.javaboy.Book" id="book2">
    <constructor-arg name="id" value="2"/>
    <constructor-arg name="name" value="红楼梦"/>
    <constructor-arg name="price" value="40"/>
</bean>
```

### 2、set 方法注入

<font style="color:rgb(34, 34, 34);">除了构造方法之外，我们也可以通过 </font><code><font style="color:rgb(34, 34, 34);">set</font></code><font style="color:rgb(34, 34, 34);"> 方法注入值。</font>

```xml
<bean class="org.javaboy.Book" id="book3">
    <property name="id" value="3"/>
    <property name="name" value="水浒传"/>
    <property name="price" value="30"/>
</bean>
```

<code><font style="color:rgb(34, 34, 34);">set</font></code><font style="color:rgb(34, 34, 34);"> 方法注入，有一个很重要的问题，就是属性名。很多人会有一种错觉，觉得属性名就是你定义的属性名，这个是不对的。在所有的框架中，凡是涉及到反射注入值的，属性名统统都不是 Bean 中定义的属性名，而是通过 Java 中的内省机制分析出来的属性名，简单说，就是根据 </font><code><font style="color:rgb(34, 34, 34);">get/set</font></code><font style="color:rgb(34, 34, 34);"> 方法分析出来的属性名。</font>

<font style="color:rgb(34, 34, 34);"></font>

<font style="color:rgb(34, 34, 34);"></font>


> 更新: 2022-04-09 16:53:09  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/qsi46e>