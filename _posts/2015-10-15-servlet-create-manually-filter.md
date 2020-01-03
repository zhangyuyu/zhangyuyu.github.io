---
layout: post
title: "手动创建Servlet过滤器"
date: 2015-10-15 20:18:09
categories: java
tags:
- java
- servlet
---
　　Servlet 过滤器是可用于 Servlet 编程的 Java 类，主要用于：
* 在客户端的请求访问后端资源之前，拦截这些请求。
* 在服务器的响应发送回客户端之前，处理这些响应。

　　在前面《手动创建Servlet》实现的基础上，增加一个filter,在服务器的响应发送回客户端之前，打印log。
### 1、创建Filter
在test/WEB_INF/classes下新建一个LogFilter.java
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import java.util.*;

public class LogFilter implements Filter  {
    public void  init(FilterConfig config) 
                         throws ServletException{
        // 获取初始化参数
        String testParam = config.getInitParameter("test-param"); 

        // 输出初始化参数
        System.out.println("********Test Param: " + testParam); 

   }
    public void  doFilter(ServletRequest request, 
                 ServletResponse response,
                 FilterChain chain) 
                 throws java.io.IOException, ServletException {
    
        PrintWriter out = response.getWriter();
        out.println("[ Time : "+ new Date().toString()+" ]");	
        // 把请求传回过滤链
        chain.doFilter(request,response);
   }
   public void destroy( ){
      /* 在 Filter 实例被 Web 容器从服务移除之前调用 */
   }
 } 
```

　　过滤器是一个实现了 javax.servlet.Filter 接口的Java类.javax.servlet.Filter 接口定义了三个方法:
* （1）`public void doFilter (ServletRequest, ServletResponse, FilterChain)`
该方法在每次一个请求/响应对因客户端在链的末端请求资源而通过链传递时由容器调用。
* （2）`public void init(FilterConfig filterConfig)`
该方法由Web容器调用，指示一个过滤器被放入服务。
* （3）`public void destroy()`
该方法由Web容器调用，指示一个过滤器被取出服务。

### 2、编译
　　编译ServletExample以及LogFilter：
```
javac -cp /usr/local/Cellar/tomcat/8.0.26/libexec/lib/servlet-api.jar /usr/local/Cellar/tomcat/8.0.26、libexec/webapps/test/WEB-INF/classes/LogFilter.java
```

### 3、部署
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app>
   
    <servlet>
        <servlet-name>ServletExample</servlet-name>
        <servlet-class>ServletExample</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>ServletExample</servlet-name>
        <url-pattern>/test</url-pattern>
    </servlet-mapping>
      
    <filter>
       <filter-name>LogFilter</filter-name>
       <filter-class>LogFilter</filter-class>
       <init-param>
          <param-name>test-param</param-name>
          <param-value>Initialization Paramter</param-value>
       </init-param>
    </filter>

    <filter-mapping>
       <filter-name>LogFilter</filter-name>
       <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
　　过滤器被部署在部署web.xml中然后映射到部署的Servlet名称或 URL模式。有多个过滤器时，执行的顺序是按它们在声明的顺序。
　　
### 4、启动tomcat
```
$ cd /usr/local/Cellar/tomcat/8.0.26/libexec/bin
$ ./startup.sh	
```
	
### 5、运行
　　在浏览器输入`http://localhost:8080/test/hello`
　　
### 6、结果
```
[ Time : Thu Oct 15 20:56:29 CST 2015 ]
Hello World
```
	
### 7、目录结构
```
|---test
    |---WEB-INF
        |---classes
            |---ServletExample.java
            |---LogFilter.java
        |---lib
        |---web.xml		
```
