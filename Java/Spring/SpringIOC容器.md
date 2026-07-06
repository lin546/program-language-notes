# Spring IOC 容器



## 一、常见注解


### 1、用于创建对象


<font style="color:rgb(52, 73, 94);">我们一般使用</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Autowired</font><font style="color:rgb(52, 73, 94);">注解自动装配 bean，要想把类标识成可用于</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Autowired</font><font style="color:rgb(52, 73, 94);">注解自动装配的 bean 的类,采用以下注解可实现：</font>

+ <font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Component</font><font style="color:rgb(52, 73, 94);">：通用的注解，可标注任意类为</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">Spring</font><font style="color:rgb(52, 73, 94);">组件。如果一个Bean不知道属于哪个层，可以使用</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Component</font><font style="color:rgb(52, 73, 94);">注解标注。</font>
+ <font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Repository</font><font style="color:rgb(52, 73, 94);">: 对应持久层即 Dao 层，主要用于数据库相关操作。</font>
+ <font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Service</font><font style="color:rgb(52, 73, 94);">: 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。</font>
+ <font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Controller</font><font style="color:rgb(52, 73, 94);">: 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。</font>



### 2、@Component 和 @Bean 的区别


1. <font style="color:rgb(52, 73, 94);">作用对象不同:</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Component</font><font style="color:rgb(52, 73, 94);">注解作用于类，而</font><font style="color:rgb(233, 105, 0);background-color:rgb(248, 248, 248);">@Bean</font><font style="color:rgb(52, 73, 94);">注解作用于方法。</font>



> 更新: 2022-04-09 16:52:59  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ukhlz9>