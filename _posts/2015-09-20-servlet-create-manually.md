---
layout: post
title: "手动创建Servlet"
date: 2015-09-20 09:34:01
categories: java
tags:
- java
- servlet
---
### 1、创建servlet
　　（笔者的tomcat安装在`/usr/local/Cellar/tomcat/8.0.26/`）  
　　在tomcat下的webapps下创建一个名为test的文件夹，然后在test里创建一个名为*WEB-INF*的文件（文件名严格遵守），在`WEB-INF`中再创建一个`classes`文件夹、一个`lib`文件夹、一个`web.xml`文件，然后在`classes`文件夹下创建一个Java文件`ServletExample.java`
```java
import javax.servlet.*;
import java.io.IOException;
import java.io.PrintWriter;

public class ServletExample implements Servlet {
    private int count;

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        PrintWriter out = servletResponse.getWriter();
        out.print("Hello World");

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

### 2、编译生成class文件
　　进行编译，这时候要手动指明jar包的位置，不然的话会找不到。
```
javac -cp /usr/local/Cellar/tomcat/8.0.26/libexec/lib/servlet-api.jar /usr/local/Cellar/tomcat/8.0.26/libexec/webapps/test/WEB-INF/classes/ServletExample.java
```

### 3、部署servlet
　　在上述新建的web.xml中，
	
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app>
    <!-- Define servlets that are included in the example application -->

    <servlet>
        <servlet-name>ServletExample</servlet-name>
        <servlet-class>ServletExample</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>ServletExample</servlet-name>
        <url-pattern>/test</url-pattern>
    </servlet-mapping>
</web-app>
```
　　其中，<servlet-name>对应创建的servlet的java文件名，<servlet-class>指明servlet所在的的路径包括包和类名，<url-pattern>可以任意取。每添加一个servlet时就必须在web.xml中进行部署。

### 4、启动tomcat
　　到tomcat的bin目录下，运行startup.sh文件。(保险起见，可以在运行startup.sh之前，先运行shutdown.sh)
```
$ cd /usr/local/Cellar/tomcat/8.0.26/libexec/bin
$ ./startup.sh	
```

### 5、运行及结果
　　在浏览器输入`http://localhost:8080/test/test`，结果为显示`Hello World`　

### 6、目录结构
```
    |---test
        |---WEB-INF
            |---classes
                |---ServletExample.java
            |---lib
            |---web.xml
```
　　
>此外，可以在tomcat下的conf里的tomcat-users.xml配置username和password进行manager webapp。
>
	<role rolename="zhang'yu"/>
	<user username="用户名" password="密码" roles="manager-gui"/>
