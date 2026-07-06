# SpringMVC数据校验

> 数据校验是所有Web应用必须处理的问题。因为Web应用的开放性，网络上所有的浏览者都可以自由使用该应用因此该应用通过输入页面收集的数据是非常复杂的，不仅包括正常用户的误输入，还可能包含恶意用户的恶意输入。一个健壮的应用系统必须将这些非法输入阻止再看应用之外，防止这些非法输入进入系统，这样才可以保证系统不受影响。
>
>  
>
> 异常的输入，轻则会导致系统非正常中断，重则会导致系统崩溃。应用程序必须能正确处理表现层接收的各种数据，通常的做法是遇到异常输入时应用程序直接返回，提示用户必须重新输入，这就是输入校验，也成为数据校验。
>
>  
>
> 输入校验分为客户端校验和服务端校验，客户端校验主要是过滤正常用户的误操作，通常使用JavaScript来实现，服务端校验是整个应用阻止非法数据的最后防线，主要通过在应用中编程实现。
>
>  
>
> SpringMVC提供了强大的数据检验功能，其中有两种方法可以验证输入：一种是利用Spring自带的Validation校验框架，另一种是利用JSR 303（Java验证规范）实现校验功能。本篇博文，只以第二种为例。
>



### 一、JSR 303 校验


JSR 303是一个规范，它的核心接口是javax.validation.Validator,该接口根据目标对象类中所标注的校验注解进行数据校验，并得到校验结果。JSR 303目前有两个实现，一个是Hibernate Validator，一个是Apache bval。



JSR 303中定义了一套可标注在成员变量、属性方法上的校验注解

| 注释 | 功能 |
| --- | --- |
| [@NULL](https://my.oschina.net/u/561366) | 被注释的元素必须为null |
| [@NotNull](https://my.oschina.net/notnull) | 被注释的元素必须不为null |
| [@AssertTrue ](/AssertTrue ) | 被注释的元素必须为true |
| [@AssertFalse](https://my.oschina.net/u/2430840) | 被注释的元素必须为false |
| @Max（value） | Number和String对象必须小于等于指定的值 |
| @Min（value） | Number和String对象必须大于等于指定的值 |
| [@DecimalMin(value) ](/value) | 被标注的值必须小于等于约束中指定的最大值，这个约束的参数是一个通过BigDecimal定义的浮点数。 |
| [@DecimalMax(value) ](/value) | 被标注的值必须大于等于约束中指定的最大值，这个约束的参数是一个通过BigDecimal定义的浮点数。 |
| @Digits(interger,fraction) | 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。 |
| @Size(min,max) | 验证对象（Array，Collection，Map，String）长度是否在给定的范围之内 |
| [@Past ](/Past ) | Date和Calender对象要在当前时间之前 |
| [@Future ](/Future ) | Date和Calender对象要在当前时间之后 |
| [@Pattern ](/Pattern ) | 验证String对象是否符合正则表达式的规则 |




### 二、Hibernate Validator


Hibernate Validator是JSR 303的一个参考实现，除了支持所有标准的校验注解之外，它还拓展了以下的注解。

| 注解 | 功能 |
| --- | --- |
| [@NotBlank ](/NotBlank ) | 检查约束字符串是不是NULL，被Trim的长度是否大于0，只对字符串，且会去掉前后空格。 |
| [@URL ](/URL ) | 判断是否是合法的url |
| [@Email ](/Email ) | 验证是否是合法的邮件地址 |
| [@CreditCardNumber ](/CreditCardNumber ) | 验证是否是合法的信用卡号码 |
| @Length(min,max) | 验证字符串的长度必须在指定的范围内 |
| [@NotEmpty ](/NotEmpty ) | 检查元素是否为NULL或者Empty。用于ArrayCollection，Map，String |
| @Range(min,max,message) | 验证属性值必须在合适的范围内 |




### 三、具体使用


①、导入Jar包



如果是maven项目只需要在pom文件加入



```xml
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>5.4.1.Final</version>
		</dependency>
```



maven会为我们导入剩下的依赖包：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621165518.png)



②、在pojo加入校验注解



我们以@pattern为例



```java
public class Employee {

	private Integer empId;

	@Pattern(regexp = "(^[a-zA-Z0-9_-]{6,16}$)|(^[\\u2E80-\\u9FFF]{2,5})", 
             message = "用户名可以是2-5位中文或者6-16位英文和数字的组合")
	private String empName;

	@Pattern(regexp = "^([a-z0-9_\\.-]+)@([\\da-z\\.-]+)\\.([a-z\\.]{2,6})$", 
             message = "邮箱格式不正确")
	private String email;
	
    ...get...set..
}
```



③、 捕获校验错误信息



上面已经将校验相关的配置都配好了，接下来需要在Controller的方法中捕获校验结果中的信息，然后将这些信息传到前台去显示。那么controller的方法中该如何获取呢？



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621165539.png)



④、配置校验器



mvc:annotation-driven注解默认帮我们注入了校验器，所以我们只需要在配置文件里加入这个标签即可。



> 更新: 2022-04-09 16:52:46  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/gsvral>