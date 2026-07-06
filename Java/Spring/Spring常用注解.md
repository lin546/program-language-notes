# Spring常用注解

#### 五、Spring事务模块注解


**1、常用的注解**



在处理dao层或service层的事务操作时，譬如删除失败时的回滚操作。使用[**@Transactional **](/Transactional )** ** 作为注解，但是需要在配置文件激活。



```xml
<!-- 开启注解方式声明事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
```



2、举例



```java
@Service
public class CompanyServiceImpl implements CompanyService {
  @Autowired
  private CompanyDAO companyDAO;

  @Transactional(propagation = Propagation.REQUIRED, readOnly = false, rollbackFor = Exception.class)
  public int deleteByName(String name) {

    int result = companyDAO.deleteByName(name);
    return result;
  }
  ...
}
```



3、总结



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621162455.png)



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621162516.png)



#### 六、其他注解


1、[@Import ](/Import ) 



> 我们可以使用Spring JavaConfig来替代传统的XML文件，@Import是被用来整合所有在@Configuration注解中定义的bean配置。这其实很像我们将多个XML配置文件导入到单个文件的情形。
>
>  
>
>  
>

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    Class<?>[] value();
}
```



在下面的例子中，我创建了两个配置文件，然后导入到主配置文件中，最后使用配置文件来创建context.



```java
Car.java
package javabeat.net.basic;
public interface Car {
    public void print();
}
```



```java
Toyota.java

package javabeat.net.basic;
import org.springframework.stereotype.Component;
@Component
public class Toyota implements Car{
    public void print(){
        System.out.println("I am Toyota");
    }
}
```



```java
Volkswagen.java
package javabeat.net.basic;
import org.springframework.stereotype.Component;
@Component
public class Volkswagen implements Car{
    public void print(){
        System.out.println("I am Volkswagen");
    }
}
```



```java
JavaConfigA.java

package javabeat.net.basic;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaConfigA {
    @Bean(name="volkswagen")
    public Car getVolkswagen(){
        return new Volkswagen();
    }
}
```



```java
JavaConfigB.java

package javabeat.net.basic;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaConfigB {
    @Bean(name="toyota")
    public Car getToyota(){
        return new Toyota();
    }
}
```



```java
ParentConfig.java

package javabeat.net.basic;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({JavaConfigA.class,JavaConfigB.class})
public class ParentConfig {
    //Any other bean definitions
}
```



```java
ContextLoader.java

package javabeat.net.basic;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ContextLoader {
    public static void main (String args[]){
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ParentConfig.class);
        Car car = (Toyota)context.getBean("toyota");
        car.print();
        car = (Volkswagen)context.getBean("volkswagen");
        car.print();
        context.close();
    }
}
```



程序执行输出  
I am Toyata  
I am Volkswagen



> 更新: 2022-04-09 16:52:42  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/ngif87>