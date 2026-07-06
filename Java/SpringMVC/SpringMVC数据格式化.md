# SpringMVC数据格式化

### 一、数据格式化


数据格式化，从本质上讲属于数据转换的范畴。Spring就是基于数据转换框架植入“格式化”功能的。



### 二、如何使用


下面我们具体使用@NumberFormat和[@DateTimeFormat ](/DateTimeFormat ) 



1、配置**FormattingConversionServiceFactoryBean。**



```xml
<mvc:annotation-driven conversion-service="conversionService"/>
	
<bean id="conversionService"
		class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<list>
				<!-- 这里我们可以引入自定义的数据转换器 -->
			</list>
		</property>
</bean>
```



2、使用@DateTimeFormat和@NumberFormat注解对象属性



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621164358.png)



3、编写Controller



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621164420.png)



4、前台请求



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621164644.png)



5、运行结果



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621164707.png)



### 三、解析补充


![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621164806.png)



下面的表格列出了当使用属性style时可用的选择以及对应的输出的例子。

| 描述 | 字符串值 | 示例输出 |
| --- | --- | --- |
| 短格式（这是缺省值） | SS | 8/30/64 11:24 AM |
| 中等格式 | MM | Aug 30, 1964 11:24:41 AM |
| 长格式 | LL | August 30, 1964 11:24:41 AM CDT |
| 完整格式 | FF | Sunday, August 30, 1964 11:24:41 AM CDT |
| 使用短横线省略日期或时间 | M- | Aug 30, 1964 |




下面的表列出了使用iso属性值时可能的值与相应的输出。

| ISO枚举值 | 输出 |
| --- | --- |
| DATE | 2000-10-31 |
| TIME | 01:30:00.000-05:00（最后的是时区) |
| DATE_TIME | 2000-10-31 01:30:00.000-05:00. |
| NONE | 不进行ISO标准的格式化 |




![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621165220.png)



> 更新: 2022-04-09 16:52:45  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/rme9og>