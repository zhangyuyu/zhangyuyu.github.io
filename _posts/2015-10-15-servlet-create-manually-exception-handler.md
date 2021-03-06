---
layout: post
title: "手动创建Servlet异常处理"
date: 2015-10-15 21:19:45
categories: java
tags:
- java
- servlet
---
### 1、创建产生异常的servlet
在ExceptionServlet.java中：
```java
import javax.servlet.*;
import java.io.IOException;
import java.io.PrintWriter;

public class ExceptionServlet implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        throw new ServletException("hello");  
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

### 2、创建异常处理类
在ErrorHandle.java中：
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import java.util.*;

// 扩展 HttpServlet 类
public class ErrorHandler extends HttpServlet {
 
  // 处理 GET 方法请求的方法
  public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
  {
      PrintWriter out = response.getWriter();
      // 分析 Servlet 异常       
      Throwable throwable = (Throwable)request.getAttribute("javax.servlet.error.exception");
      Integer statusCode = (Integer)request.getAttribute("javax.servlet.error.status_code");
      String servletName = (String)request.getAttribute("javax.servlet.error.servlet_name");
      String requestUri = (String)request.getAttribute("javax.servlet.error.request_uri");

      if (throwable != null){
         out.println("Exception Type : " + throwable.getClass( ).getName( ));
         out.println("The exception message: " + throwable.getMessage( ));
      }
      if (statusCode != null){
         out.println("The status code : " + statusCode);
      }
      out.println("Servlet Name : " + servletName);
      out.println("The request URI: " + requestUri);
         
  }
  // 处理 POST 方法请求的方法
  public void doPost(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
     doGet(request, response);
  }
}
```

### 3、编译
```
javac -cp /usr/local/Cellar/tomcat/8.0.26/libexec/lib/servlet-api.jar /usr/local/Cellar/tomcat/8.0.26/libexec/webapps/test/WEB-INF/classes/*.java
```

### 4、部署
在web.xml中
```xml
<servlet>
    <servlet-name>ErrorHandler</servlet-name>
    <servlet-class>ErrorHandler</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>ErrorHandler</servlet-name>
    <url-pattern>/error</url-pattern>
</servlet-mapping>

<error-page>
    <error-code>404</error-code>
    <location>/error</location>
</error-page>

<error-page>
    <error-code>403</error-code>
    <location>/error</location>
</error-page>

<error-page>
    <exception-type>javax.servlet.ServletException</exception-type >
    <location>/error</location>
</error-page>

<error-page>
    <exception-type>java.io.IOException</exception-type >
    <location>/errorr</location>
</error-page>
```

### 5、启动tomcat
```
$ cd /usr/local/Cellar/tomcat/8.0.26/libexec/bin
$ ./startup.sh	
```
	
### 6、运行及结果
　　使用上面会产生异常的Servlet，或者输入一个错误的URL，会触发Web容器调用`ErrorHandler的Servlet`，并显示适当的信息。</br>  
　　如果使用上面会产生异常的`ExceptionServlet`(`http://localhost:8080/test/exception`)，那么它将显示：
```
Exception Type : javax.servlet.ServletException</br>
The exception message: hello</br>
The status code : 500</br>
Servlet Name : ExceptionServlet</br>
The request URI: /test/exception
```

　　如果输入了一个错误的URL(`http://localhost:8080/test/ex`)，那么它将显示`The status code : 404`
　　
### 7、目录结构
```
|---test
    |---WEB-INF
        |---classes
            |---ServletException.java
            |---ErrorHandler.java
        |---lib
        |---web.xml
```
