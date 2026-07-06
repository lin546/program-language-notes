# SpringMVC数据绑定

> Spring会根据请求方法签名的不同，将请求消息中的消息以一定的方式转换并绑定到请求方法的入参中，当请求消息到达真正需要调用的方法时，SpringMVC还有很多工作要做，包括数据转换，数据格式化及数据校验。
>



## 一、数据绑定流程


![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163240.png)



数据绑定流程：



+ SpringMVC主框架将`ServletRequest`对象及目标方法的入参实例传递给`WebDateBinderFactory`实例，以创建 `DataBinder` 实例对象。
+ `DataBinder` 调用装配在SpringMVC上下文中的 `ConversionService` 组件进行数据类型转换，数据格式化工作。将Servlet中的请求信息填充到入参对象中。
+ 调用 Validator 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果 `BindingData` 对象。
+ SpringMVC抽取 `BindingResult` 中的入参对象和校验对象，将它们赋给处理方法的响应入参。



## 二、数据转换


### 1、ConversionService


`ConversionService` 是Spring类型转换体系的核心接口，该接口定义了以下四个方法。



SpringMVC 有支持的默认参数类型，我们直接在形参上给出这些默认类型的声明，就能直接使用了。如下：



> ①、HttpServletRequest 对象
>
>  
>
> ②、HttpServletResponse 对象
>
>  
>
> ③、HttpSession 对象
>
>  
>
> ④、Model/ModelMap 对象
>



Controller 代码：



```java
@RequestMapping("/defaultParameter")
    public ModelAndView defaultParameter(HttpServletRequest request,HttpServletResponse response,
                            HttpSession session,Model model,ModelMap modelMap) throws Exception{
        request.setAttribute("requestParameter", "request类型");
        response.getWriter().write("response");
        session.setAttribute("sessionParameter", "session类型");
        //ModelMap是Model接口的一个实现类，作用是将Model数据填充到request域
        //即使使用Model接口，其内部绑定还是由ModelMap来实现
        model.addAttribute("modelParameter", "model类型");
        modelMap.addAttribute("modelMapParameter", "modelMap类型");
         
        ModelAndView mv = new ModelAndView();
        mv.setViewName("view/success.jsp");
        return mv;
    }
```



表单代码：



```html
<body>
    request:${requestParameter}
    session:${sessionParameter}
    model:${modelParameter}
    modelMap:${modelMapParameter}
</body>
```



### 3、基本数据类型的绑定


哪些是基本数据类型，我们这里重新总结一下：



```plain
一、byte，占用一个字节，取值范围为 -128-127，默认是"\u0000"，表示空
二、short，占用两个字节，取值范围为 -32768-32767
三、int，占用四个字节，-2147483648-2147483647
四、long，占用八个字节，对 long 型变量赋值时必须加上"L"或"l",否则不认为是 long 型
五、float，占用四个字节，对 float 型进行赋值的时候必须加上"F"或"f"，如果不加，会产生编译错误，因为系统
自动将其定义为 double 型变量。double转换为float类型数据会损失精度。float a = 12.23产生编译错误的，float a = 12是正确的
六、double，占用八个字节，对 double 型变量赋值的时候最好加上"D"或"d"，但加不加不是硬性规定
七、char,占用两个字节，在定义字符型变量时，要用单引号括起来
八、boolean，只有两个值"true"和"false"，默认值为false，不能用0或非0来代替，这点和C语言不同
```



我们以 int 类型为例：



JSP 页面代码：



```html
<form action="basicData" method="post">
    <input name="age" value="10" type="text"/>
    <input type="submit" value="提交">
</form>9
```



Controller 代码：



```java
@RequestMapping("/basicData")
    public void basicData(int age){
        System.out.println(age);//10
    }
```



结果是打印出了表单里面的 value 的值。



注意：表单中input的name值和Controller的参数变量名保持一致，就能完成数据绑定。那么如果不一致呢？我们使用 `@RequestParam` 注解来完成，如下：



```java
@RequestMapping("/basicData")
    public void basicData(@RequestParam(value="age") int myage){
        System.out.println(myage);//10
    }
```



> 注解里面的 value 属性值要和表单的name属性值一样。
>



### 4、包装数据类型的绑定


问题：我们这里的参数是基本数据类型，如果从前台页面传递的值为 null 或者 ""的话，那么会出现数据转换的异常，所以，在开发过程中，对可能为空的数据，最好将参数数据类型定义成包装类型，具体参见下面的例子。



包装类型如 `Integer、Long、Byte、Double、Float、Short`，（String 类型在这也是适用的）这里我们以 Integer 为例



```java
@RequestMapping("/basicData")
    public void basicData(Interger age){
        System.out.println(age);//10
    }
```



如果表单中num为""或者表单中无num这个input，那么，Controller方法参数中的num值则为null。



### 五、POJO类型的绑定


User.java



```java
package com.ys.po;
 
import java.util.Date;
 
public class User {
    private Integer id;
 
    private String username;
 
    private String sex;
 
    private Date birthday;
 
    public Integer getId() {
        return id;
    }
 
    public void setId(Integer id) {
        this.id = id;
    }
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username == null ? null : username.trim();
    }
 
    public String getSex() {
        return sex;
    }
 
    public void setSex(String sex) {
        this.sex = sex == null ? null : sex.trim();
    }
 
    public Date getBirthday() {
        return birthday;
    }
 
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```



JSP页面：注意输入框的 name 属性值和上面 POJO 实体类的属性保持一致即可映射成功。



```html
<form action="pojo" method="post">
        用户id:<input type="text" name="id" value="2"></br>
        用户名:<input type="text" name="username" value="Marry"></br>
        性别:<input type="text" name="sex" value="女"></br>
        出生日期:<input type="text" name="birthday" value="2017-08-25"></br>
        <input type="submit" value="提交">
</form>
```



注意看：这里面我们数据都写死了，直接提交。有Integer类型的，String类型的，Date类型的。



Controller ：



```java
@RequestMapping("/pojo")
    public void pojo(User user){
        System.out.println(user);
    }
```



运行：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163318.png)



上面是报错了，User.java 的birthday 属性是 Date 类型的，而我们输入的是字符串类型，故绑定不了



那么问题来了，Date 类型的数据绑定失败，如何解决这样的问题呢？这就是我们前面所说的需要自定义Date类型的转换器。



**①、定义由String类型到 Date 类型的转换器**



```java
package com.ys.util;
 
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
 
import org.springframework.core.convert.converter.Converter;
 
//需要实现Converter接口，这里是将String类型转换成Date类型
public class DateConverter implements Converter<String, Date> {
 
    @Override
    public Date convert(String source) {
        //实现将字符串转成日期类型(格式是yyyy-MM-dd HH:mm:ss)
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            return dateFormat.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        //如果参数绑定失败返回null
        return null;
    }
}
```



**②、在 springmvc.xml 文件中配置转换器**



```xml
<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
     
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <!-- 自定义转换器的类名 -->
            <bean class="com.ys.util.DateConverter"></bean>
        </property>
    </bean>
```



输入 URL，再次运行就不会报错了。



自定义类型转换器具体操作请参考：[https://my.oschina.net/GL24568/blog/1647383](https://my.oschina.net/GL24568/blog/1647383)



### 六、复合POJO类型的绑定


这里我们增加一个实体类，ContactInfo.java



```java
package com.ys.po;
 
public class ContactInfo {
    private Integer id;
 
    private String tel;
 
    private String address;
 
    public Integer getId() {
        return id;
    }
 
    public void setId(Integer id) {
        this.id = id;
    }
 
    public String getTel() {
        return tel;
    }
 
    public void setTel(String tel) {
        this.tel = tel == null ? null : tel.trim();
    }
 
    public String getAddress() {
        return address;
    }
 
    public void setAddress(String address) {
        this.address = address == null ? null : address.trim();
    }
}
```



然后在上面的User.java中增加一个属性 private ContactInfo contactInfo



```plain
public class User {
    private Integer id;
 
    private String username;
 
    private String sex;
 
    private Date birthday;
 
    private ContactInfo contactInfo;

    set,get方法.....
}
```



JSP 页面：注意属性name的命名，User.java 的复合属性名.字段名



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163402.png)



User对象中有ContactInfo属性，但是，在表单代码中，需要使用"属性名(对象类型的属性).属性名"来命名input的name。



Controller



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163418.png)



### 七、数组类型的绑定


数组的绑定指的是前台传来多个同一类型的数据，我们在controller中使用数组形参来接收前台传来的数据。比如现在我们要批量删除用户，那么我们需要勾选好几个用户，这些商品都有userId，在controller中，我们需要将这些userId全部获取并放到一个数组中，然后再根据数组中的id号挨个删除数据库中对应的项。那么该如何绑定呢？其实也很简单，如下：



JSP 页面：**注意用户id的name值定义为 userId**



```html
<form action="array" method="post">
  <p><input type="checkbox" name="userId" value="1" /> I am jerry</p>
  <p><input type="checkbox" name="userId" value="2" /> I am tom</p>
  <input type="submit" value="Submit" />
</form>
```



Controller.java



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163523.png)



### 八、List类型的绑定


SpringMVC不支持list类型的直接转换，需要封装成Object。



需求：批量修改 User 用户的信息



第一步：创建 UserVo.java，封装 List 属性



```java
public class UserVo {
     
    private List<User> userList;
    public List<User> getUserList() {
        return userList;
    }
    public void setUserList(List<User> userList) {
        this.userList = userList;
    }
}
```



Controller



```java

@RequestMapping("list")
public void testList(UserVo userVo){  //引入封装好的类

  System.out.println(userVo.getUserList());
}
```



Jsp页面



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163610.png)



### 九、Map类型的绑定


SpringMVC不支持map类型的直接转换，需要封装成Object。



首先在 UserVo 里面增加一个属性 Map<String,User> userMap



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163641.png)



第二步：JSP页面，注意看 输入框 name 的属性值



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621163652.png)



第三步：Controller 中获取页面的属性



```java
@RequestMapping("map")
public void testList(UserVo userVo){  //引入封装好的类

  System.out.println(userVo.getUserMap());
}
```



> 更新: 2022-04-09 16:52:45  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/bqnw8q>