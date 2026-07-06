# SpringMVC处理Json

---



title: SpringMVC处理json  
layout: post  
categories:



+ Spring MVC  
tags:
+ Spring MVC

---

一、具体实现



1、加入jar包



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621170002.png)



2、编写目标方法，使其返回JSON对应的对象或集合



3、在编写的方法上面添加@ResponseBody注解



```java
@Controller
public class testJson {

	@ResponseBody
	@RequestMapping("/testJson")
	public List<Department> testJson() {

		List<Department> list = new ArrayList<>();
		Department department1 = new Department(1, "GL");
		Department department2 = new Department(2, "LG");
		list.add(department1);
		list.add(department2);
		return list;
	}
}
```



这样我们就可以返回一个JSON数组。



二、原理



SpringMVC在处理JSON数据时，需要使用HttpMessageConverter接口，其是Spring3.0新添加的接口，主要负责**将请求信息转换为一个Java对象（类型为 T）**，或**将Java对象（类型为 T）输出为响应信息**。   
　　HttpMessageConverter的基本流程如下图所示：   
![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621170014.png)



HttpMessageConverter的实现类主要有：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621170035.png)



DispatcherServlet默认装配有ReuqestMappingHandlerAdapter



而ReuqestMappingHandlerAdapter默认装配的HttpMessageConverter如下：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621170051.png)



加入Jackson所需要的jar包后，RequestMappingHandlerAdapter装配的HttpMessageConverter如下：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621170116.png)



由此可见，在加入Jackson所需要的jar包后，SpringMVC会自动将MappingJackson2HttpMessageConverter装配上，以实现对JSON数据的处理。



三、HttpMessageConverter



使用`HttpMessageConverter<T>`将请求信息转化并绑定到处理方法的入参中或将响应结果转化为对应类型的响应信息，SpringMVC提供了两种途径：   
　　①. 使用[**@RequestBody **](/RequestBody )** **标注处理方法的入参或使用[**@ResponseBody **](/ResponseBody )** **标注处理方法；   
　　②. 使用`**HttpEntity<T>**`作为处理方法的入参或使用`**ResponseEntity<T>**`作为处理方法的返回值。



>  
>
> 1. [@Responsebody ](/Responsebody ) 注解表示该方法的返回的结果直接写入 HTTP 响应正文（ResponseBody）中，一般在异步获取数据时使用； 
> 2. 在使用 [@RequestMapping ](/RequestMapping ) 后，返回值通常解析为跳转路径，加上 [@Responsebody ](/Responsebody ) 后返回结果不会被解析为跳转路径，而是直接写入HTTP 响应正文中。例如，异步获取 json 数据，加上 [@Responsebody ](/Responsebody ) 注解后，就会直接返回 json 数据。 
> 3. [@RequestBody ](/RequestBody ) 注解则是将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。 
>
>  
>



那么HttpMessageConverter的匹配过程是什么样的呢？



@RequestBody注解时： 根据Request对象header部分的Content-Type类型，逐一匹配合适的HttpMessageConverter来读取数据。



spring 3.1源代码如下：



```java
private Object readWithMessageConverters(MethodParameter methodParam, HttpInputMessage inputMessage, Class paramType) throws Exception {  

    MediaType contentType = inputMessage.getHeaders().getContentType();  
    if (contentType == null) {  
        StringBuilder builder = new StringBuilder(ClassUtils.getShortName(methodParam.getParameterType()));  
        String paramName = methodParam.getParameterName();  
        if (paramName != null) {  
            builder.append(' ');  
            builder.append(paramName);  
        }  
        throw new HttpMediaTypeNotSupportedException("Cannot extract parameter (" + builder.toString() + "): no Content-Type found");  
    }  

    List<MediaType> allSupportedMediaTypes = new ArrayList<MediaType>();  
    if (this.messageConverters != null) {  
        for (HttpMessageConverter<?> messageConverter : this.messageConverters) {  
            allSupportedMediaTypes.addAll(messageConverter.getSupportedMediaTypes());  
            if (messageConverter.canRead(paramType, contentType)) {  
                if (logger.isDebugEnabled()) {  
                    logger.debug("Reading [" + paramType.getName() + "] as \"" + contentType  + "\" using [" + messageConverter + "]");  
                }  
                return messageConverter.read(paramType, inputMessage);  
            }  
        }  
    }  
    throw new HttpMediaTypeNotSupportedException(contentType, allSupportedMediaTypes);  
}
```



@ResponseBody注解时：根据Request对象header部分的Accept属性（逗号分隔），逐一按accept中的类型，去遍历找到能处理的HttpMessageConverter。



spring 3.1 源代码如下：



```java
private void writeWithMessageConverters(Object returnValue,  HttpInputMessage inputMessage, HttpOutputMessage outputMessage)  
                throws IOException, HttpMediaTypeNotAcceptableException {  
    List<MediaType> acceptedMediaTypes = inputMessage.getHeaders().getAccept();  
    if (acceptedMediaTypes.isEmpty()) {  
        acceptedMediaTypes = Collections.singletonList(MediaType.ALL);  
    }  
    MediaType.sortByQualityValue(acceptedMediaTypes);  
    Class<?> returnValueType = returnValue.getClass();  
    List<MediaType> allSupportedMediaTypes = new ArrayList<MediaType>();  
    if (getMessageConverters() != null) {  
        for (MediaType acceptedMediaType : acceptedMediaTypes) {  
            for (HttpMessageConverter messageConverter : getMessageConverters()) {  
                if (messageConverter.canWrite(returnValueType, acceptedMediaType)) {  
                    messageConverter.write(returnValue, acceptedMediaType, outputMessage);  
                    if (logger.isDebugEnabled()) {  
                        MediaType contentType = outputMessage.getHeaders().getContentType();  
                        if (contentType == null) {  
                            contentType = acceptedMediaType;  
                        }  
                        logger.debug("Written [" + returnValue + "] as \"" + contentType +  
                                "\" using [" + messageConverter + "]");  
                    }  
                    this.responseArgumentUsed = true;  
                    return;  
                }  
            }  
        }  
        for (HttpMessageConverter messageConverter : messageConverters) {  
            allSupportedMediaTypes.addAll(messageConverter.getSupportedMediaTypes());  
        }  
    }  
    throw new HttpMediaTypeNotAcceptableException(allSupportedMediaTypes);  
}
```



> 更新: 2022-04-09 16:52:43  
> 原文: <https://www.yuque.com/thinkspace/afrw3l/rxng0t>