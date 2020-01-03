---
layout: post
title: "手动创建Servlet转发页面"
date: 2015-09-20 10:12:41
categories: java
tags:
- java
- servlet
---
　　在上一篇的基础上，本节实现转发页面。
### 1、创建servlet
#### （1）在test目录下，新建一个index.html
```xml
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html">
    <head>
        <title>Hello index!</title>
    </head>
    <body>
        <form action="hello" method="GET">
            First Name: <input type="text" name="first_name" /><br />
            Last Name: <input type="text" name="last_name"   /><br />
            <input type="submit" value="Submit" />
        </form>
    </body>
</html>
```

#### （2）在test/WEB_INF/classes下新建一个HEllo.java
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class Hello extends HttpServlet {
 
    public void doGet(HttpServletRequest request,HttpServletResponse response)throws ServletException, IOException{
        response.setContentType("text/html");
        
        PrintWriter out = response.getWriter();
        String title = "Using GET Method to Read Form Data";
        String docType =
        "<!doctype html public \"-//w3c//dtd html 4.0 " +
        "transitional//en\">\n";
        out.println(docType +
                  "<html>\n" +
                  "<head><title>" + title + "</title></head>\n" +
                  "<body>\n" +
                  "<h1 align=\"center\">" + title + "</h1>\n" +
                  "<b>First Name</b>: "
                  + request.getParameter("first_name") + "\n" +
                  "<b>Last Name</b>: "
                  + request.getParameter("last_name") + "\n" +
                  "</body>"+"</html>");
  }
}
```

### 2、编译生成class文件
```
javac -cp /usr/local/Cellar/tomcat/8.0.26/libexec/lib/servlet-api.jar /usr/local/Cellar/tomcat/8.0.26/libexec/webapps/test/WEB-INF/classes/Hello.java
```

### 3、部署
　在web.xml中
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app>
    <!-- Define servlets that are included in the example application -->

    <servlet>
        <servlet-name>Hello</servlet-name>
        <servlet-class>Hello</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>Hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

### 4、启动tomcat
```
$ cd /usr/local/Cellar/tomcat/8.0.24/libexec/bin
$ ./startup.sh	
```

### 5、运行及结果
　　在浏览器输入`http://localhost:8080/test/index.html`，填写内容，点击submit，跳转到了hello页面。

### 6、目录结构
```
|---test
    |---index.html
    |---WEB-INF
        |---classes
            |---Hello.java
        |---lib
        |---web.xml
```		
