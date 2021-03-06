---
layout: post
title: "手动创建Servlet连接数据库"
date: 2015-09-20 10:20:41
categories: java
tags:
- java
- servlet
---
### 1、准备数据库
#### (1)在本地mysql中create 一个数据库
```
$ mysql.server start
$ mysql -uroot -p
mysql> show databases;
mysql> create database test;
mysql> use test;
mysql> create table raw_report(
       name varchar(20),
       sex varchar(10),
       age varchar(10),
       birthday varchar(20)
  );
mysql> INSERT raw_report(name, sex, age, birthday) VALUES('Alex','M','20','08/07/1995');
mysql> INSERT raw_report(name, sex, age, birthday) VALUES('Alice','F','18','08/07/1997');
mysql> INSERT raw_report(name, sex, age, birthday) VALUES('Bob','M','21','08/07/1994');
mysql> INSERT raw_report(name, sex, age, birthday) VALUES('Mary','F','23','08/07/1992');
```

### 2、创建servlet
　　在test/WEB_INF/classes下新建一个DBServlet.java
```java
import java.io.*;                                    //导入java.io包
import java.util.*;
import java.sql.*;
import javax.servlet.*;
import javax.servlet.http.*;
public class DBServlet extends HttpServlet{            //定义一个继承HttpServlet的公有类
    ServletConfig config=null;                        //定义一个ServletConfig对象
    private String driverName="";                    //定义私有字符串常量并初始化
    private String username="";                    //定义的数据库用户名
    private String password="";                    //定义的数据库连接密码
    private String dbName="";                        //定义的数据库名
    private Connection conn;                        //初始化连接
    private Statement stmt;                        //初始化数据库操作
    ResultSet rs=null;   
                             //初始化结果集
    public void init(ServletConfig config)throws ServletException{
        super.init(config);                            //继承父类的init()方法
        this.config=config;                            //获取配置信息
        driverName=config.getInitParameter("driverName");//从配置文件中获取JDBC驱动名
        username=config.getInitParameter("username");    //获取数据库用户名
        password=config.getInitParameter("password");    //获取数据库连接密码
        dbName=config.getInitParameter("dbName");    //获取要连接的数据库
    }

    public void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException,ServletException{
        resp.setContentType("text/html;charset=GBK");    //设置字符编码格式
        PrintWriter out=resp.getWriter();                //实例化对象，用于页面输出
        out.println("<html>");                    //实现生成静态Html
        out.println("<head>");
        out.println("<meta http-equiv=\"Content-Type\"content=\"text/html;charset=GBK\">");
        out.println("<title>DataBase Connection</title>");
        out.println("</head>");
        out.println("<body bgcolor=\"white\">");
        out.println("<center>");
        String url="jdbc:mysql://localhost:3306/test";
        try{
                Class.forName("com.mysql.jdbc.Driver");
                conn=DriverManager.getConnection(url,username,password);
                stmt=conn.createStatement();
                String sql="select * from raw_report";
                rs=stmt.executeQuery(sql);

                out.println("Servlet访问数据库成功");
                out.println("<table border=1 bordercolorlight=#000000>");
                out.println("<tr><td width=40>name</td>");
                out.println("<td>sex</td>");
                out.println("<td>age</td>");
                out.println("<td>birthday</td>");
                while(rs.next()){
                    out.println("<tr><td>"+rs.getString(1)+"</td>");
                    out.println("<td>"+rs.getString(2)+"</td>");
                    out.println("<td>"+rs.getString(3)+"</td>");
                    out.println("<td>"+rs.getString(4)+"</td>");
                    out.println("<tr>");
                }
                out.println("</table>");
                
                rs.close();
                stmt.close();
                conn.close();    
                   
            }catch(Exception e){
                e.printStackTrace();
                out.println(e.toString());    
            }
            out.println("</center>");
            out.println("</body>");
            out.println("</html>");
        }

        public void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException,ServletException{
            this.doGet(req,resp);
        }
        
        public void destory(){
            config=null;
            driverName=null;
            username=null;
            password=null;
            dbName=null;
            conn=null;
            stmt=null;
            rs=null;
        }
}
```

### 3、编译生成class文件
```
javac -cp /usr/local/Cellar/tomcat/8.0.24/libexec/lib/servlet-api.jar /usr/local/Cellar/tomcat/8.0.24/libexec/webapps/test/WEB-INF/classes/DBServlet.java
```

### 4、部署
　　在web.xml中
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app>
    <servlet>
    <description>This is the description of my J2EE component</description>
    <display-name>This is the display name of my J2EE component</display-name>
    <servlet-name>DBServlet</servlet-name>
    <servlet-class>DBServlet</servlet-class>
    <init-param>
            <param-name>driverName</param-name>
            <param-value>com.mysql.jdbc.Driver</param-value>
        </init-param>
        <init-param>
        <param-name>username</param-name>            
            <param-value>用户名</param-value>                
        </init-param>
        <init-param>
            <param-name>password</param-name>        
            <param-value>密码</param-value>                
        </init-param>
        <init-param>
            <param-name>dbName</param-name>    
            <param-value>test</param-value>    
        </init-param>

  </servlet>

  <servlet-mapping>
     <servlet-name>DBServlet</servlet-name>
     <url-pattern>/DB</url-pattern>
   </servlet-mapping>

</web-app>
```

### 5、启动tomcat
```
$ cd /usr/local/Cellar/tomcat/8.0.24/libexec/bin
$ ./startup.sh	
```

### 6、运行
　　在浏览器输入`http://localhost:8080/test/DB`

### 7、目录结构
```
|---test
    |---WEB-INF
        |---classes
            |---DBServlet.java
        |---lib
        |---web.xml		
```
