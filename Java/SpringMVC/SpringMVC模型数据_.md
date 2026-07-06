# SpringMVC模型数据

Spring MVC 通过 `@RequestMapping`将请求引导到处理方法上，使用合适的方法签名将请求消息绑定到入参中。方法入参绑定请求消息只是处理方法的第一步，还有更重要的任务等待完成，即根据入参执行相应的逻辑，产生模型数据，导向到特定视图中。



如何将模型数据暴露给视图是 SpringMVC 框架的一项重要工作，SpringMVC 提供了多种途径输出模型数据：



+ [**@ModelAndView **](/ModelAndView )** **：处理方法返回值类型为 ModelAndView时，方法体即可通过该对象添加模型数据。
+ [**@ModelAttribute **](/ModelAttribute )** **：方法入参标注该注解后，入参的对象就会放到数据模型中。
+ **@Map及Model**:入参为 org.springframework.ui.Model、org.soringframework.ui.ModelMap 或 java.util.Map 时，处理方法返回时，Map中的数据会自动添加到模型中。
+ [**@SessonAttribute **](/SessonAttribute )** **:将模型中的某个属性暂时存到 HttpSession 中，以便多个请求之间可以共享这个属性。



## 一、ModelAndView


控制器处理方法的返回值如果为 ModelAndView，则其既包含视图信息，也包含模型数据信息，这样 SpringMVC 就可以使用视图对模型数据进行渲染了。可以简单地将模型数据看成一个 Map<String, Object>对象。



在处理方法的方法体中，可以使用如下方法添加模型对象：



+ ModelAndView addObject(String attributeName, Object attributeValue)
+ ModelAndView addAllObject(Map<String, ?> modelMap)



可以通过如下方法设置视图



+ void setView(View view)：指定一个具体的视图对象
+ void setViewName(String viewName)：指定一个逻辑视图名



```java
@RequestMapping("/springmvc")
@Controller
public class SpringTest
{    
    //SpringMVC 会把 ModelAndView 的模型数据放到请求域中去
    @RequestMapping("/testModelAndView")
    public ModelAndView testModelAndView()
    {
        String viewName = "success";
　　　　 //指定跳转的逻辑视图名为 success
        ModelAndView modelAndView = new ModelAndView(viewName);
        //添加模型数据到ModelAndView
        modelAndView.addObject("time", new Date());
        return modelAndView;
    }
}
```



success.jsp



```html
<body>
    <h4>Success Page</h4>
    
    time: ${requestScope.time }
</body>
```



### 1、[@ModelAttribute ](/ModelAttribute ) 


#### （1）在方法入参前使用@ModelAttribute注解


如果希望将方法入参对象添加到模型中，则仅需在相应入参前使用 `@ModelAttribute` 注解即可。



```java
@RequestMapping("/handle")
public String handle($ModelAndView("user") User user){
    user.setUserId("1000");
    return "/user/createSuccess";
}
```



SpringMVC 将请求消息绑定到User对象中，然后再以user为键将User对象放到模型中。在准备对视图进行渲染前，SpringMVC 还会进一步将模型中的数据转储到视图的上下文中并暴露给视图对象。



#### （2）在方法定义处使用@ModelAttribute注解


标注在方法上的 `@ModelAttribute` 说明方法是用于添加一个或多个属性到model上。这样的方法能接受与`@RequestMapping` 标注相同的参数类型，只不过不能直接被映射到具体的请求上。



**在同一个控制器中，标注了 **`**@ModelAttribute**`** 的方法实际上会在 **`**@RequestMapping**`** 方法之前被调用**。



`@ModelAttribute` 标注方法有两种风格：



+ 在第一种写法中，方法通过返回值的方式默认地将添加一个属性；
+ 在第二种写法中，方法接收一个Model对象，然后可以向其中添加任意数量的属性。  
可以在根据需要，在两种风格中选择合适的一种。



```java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// Add multiple attributes
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}
```



`@ModelAttribute` 方法通常被用来填充一些公共需要的属性或数据，。



`@ModelAttribute` 方法也可以定义在 `@ControllerAdvice` 标注的类中，并且这些 `@ModelAttribute` 可以同时对许多控制器生效。



属性名没有被显式指定的时候，框架将根据属性的类型给予一个默认名称。举个例子，若方法返回一个Account类型的对象，则默认的属性名为"account"。可以通过设置 `@ModelAttribute` 标注的值来改变默认值。当向 Model 中直接添加属性时，请使用合适的重载方法 `addAttribute(..)`-即带或不带属性名的方法。



`@ModelAttribute` 标注也可以被用在 `@RequestMapping` 方法上。这种情况下，`@RequestMapping` 方法的返回值将会被解释为 model 的一个属性，而非一个视图名，此时视图名将以视图命名约定来方式来确定。



### 二、Map 及 Model


SpringMVC 在内部使用一个 org.springframework.ui.Model 接口存储模型数据，它的功能类似于 java.util.Map，但它比Map易用。org.springframework.ui.ModelMap实现了 Map 接口，而 org.springframework.ui.ExtendedModelMap 继承于 ModelMap 同时实现了 Model 接口。



SpringMVC 在调用方法前会创建一个隐含的模型对象，作为模型数据的存储容器，我们称之为 “隐含模型”。如果处理方法的入参为 Map 或 Model 类型，SpringMVC 会将隐含模型的引用传递给这些入参。在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型数据中添加新的属性数据。



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20200405150531.png)



SpringMVC 一旦发现处理方法有 Map 或 Model 类型的入参，就会请求内在的隐含模型对象传递给这些参数，因此在方法体中可以通过这个入参对模型对象的数据进行读写操作。



### 二，[@ModelAttribute ](/ModelAttribute ) 


> 在方法定义上使用 [_**@ModelAttribute **_](/ModelAttribute )_**  **_注解：SpringMVC 在调用目标处理方法前，会逐个调用在方法上标注了 [@ModelAttribute ](/ModelAttribute ) 的方法 
>
>  
>
> 在方法的入参前使用 [_**@ModelAttribute **_](/ModelAttribute )_** **_ 注解：
>
>  
>
> 1. 可以从隐含对象中获取隐含的模型数据对象，再将请求参数绑定到对象中，再传入入参
>
>  
>
> 2. 将方法入参对象添加到模型中
>



如果希望将方法入参对象添加到模型中，仅需在相应的入参前使用@ModelAttribute注解即可，



```java
public String list(@ModelAttribute("teachet") Teacher teachet){  

        Teacher teacher = new Teacher();  

        teacher.setAge(24);  

        teacher.setId(1);  

        teacher.setName("json");  

        return "teacher/list";  

    }
```



SpringMVC将请求消息绑定到Teacher对象中，然后再以“teacher”为键将Teacher对象放到隐藏模型中，最后在视图对象list.jsp中就可以使用${teacher.name}取值显示了。



@ModelAttribute也可以在方法上使用，SpringMVC在调用目标方法前，会先逐个调用在方法级上标注了@ModelAttribute的方法，并将这些方法的返回值添加到隐藏模型中。



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621162733.png)



注意：在②处，当获得整合版本的user对象后，会将其添加到模型中。



### 四，[@SessionAttributes ](/SessionAttributes ) 


如果希望在多个请求之间共用某个模型属性数据，则可以在控制器类标注个 [_**@SessionAttributes **_](/SessionAttributes )_** **_，SpringMVC 会将模型中对应的属性暂存到 HTTPSession 中。



[@SessionAttributes ](/SessionAttributes ) 除了可以通过 **属性名**指定需要放到会话中的属性外，还可以通过模型属性的**对象类型**指定哪些模型属性需要放到会话中。



> 1. @SessionAttributes(**types**=User.class)会将隐含模型中所有类型为 User 的属性添加到会话中
>
>  
>
> 2. @SessionAttributes(**value**={"user1", "user2"})将名为 user1 和 user2 的模型属性添加到会话中
>
>  
>
> 3. @SessionAttributes(types={"User.class", "Dept.class"})将模型中所有类型为 User 及 Dept 的属性添加到会话中
>
>  
>
> 4. @SessionAtributes(value={"user1", "user2"}, types={Dept.class})将名为 user1 和 user2 的模型属性添加到会话中，同时将所有类型为 Dept 的模型属性添加到会话中
>



```java
@SessionAttributes(value={"user"}, types={String.class})
@RequestMapping("/springmvc")
@Controller
public class SpringTest
{
    private static final String SUCCESS = "success";
    
　　@RequestMapping("/testSessionAttributes")
    public String testSessionAttributes(Map<String, Object> map)
    {
        User user = new User("Jack", "123");
        map.put("user", user);
        map.put("msg", "hello");
        return SUCCESS;
    }
}



<a href="springmvc/testSessionAttributes">Test SessionAttributes</a>



request user: ${requestScope.user }
<br><br>
request msg: ${requestScope.msg }
<br><br>
session user: ${sessionScope.user }
<br><br>
session msg: ${sessionScope.msg }
```



由结果可以看出，被 [@SessionAttributes ](/SessionAttributes ) 注解修饰后，模型属性不仅存在于请求域还存在于会话域。需要注意的是 [**@SessionAttributes **](/SessionAttributes )** 注解只能用于修饰类而不能用于方法上 **。



### 补充：


如果Model中没有key为user的属性，并且没写@ModelAttribute("user")，由于参数列表中有User user对象入参，则Spring会将该对象放入model，并且key值为首字母小写的类名，也就是说对于方法：



```java
 @RequestMapping(value="/helloworld")
 public String helloWorld(User user)
 {
     return "helloworld";
 }
```



框架提前帮你写了一句model.addAttribute("user",user)。  
**而且特别需要注意注意，无论model中是否有key为user的属性，都要求User类有无参构造方法**



## 参考：


+ [Spring MVC 处理模型数据](https://www.cnblogs.com/2015110615L/p/5629461.html)
+ [Spring MVC @ModelAttribute详解](https://www.tianmaying.com/tutorial/spring-mvc-modelattribute)



> 更新: 2022-04-09 16:52:44  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ghq0o3>