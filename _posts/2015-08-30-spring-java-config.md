---
layout: post
title: "Spring Java Config"
date: 2015-08-30 10:47:46
categories: java
tags: 
- java
- spring
---
　　使用JavaConfig (@Configuration), 需要添加依赖，如下所示的最下方：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mkyong.core</groupId>
    <artifactId>Spring3Example</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>Spring3Example</name>
    <url>http://maven.apache.org</url>

    <properties>
        <spring.version>3.0.5.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8.2</version>
            <scope>test</scope>
        </dependency>

        <!-- Spring 3 dependencies -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- JavaConfig need this library -->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>2.2.2</version>
        </dependency>

    </dependencies>
</project>
```
### 一、Spring Bean
```java
package com.mkyong;

public class HelloWorld{

    public void toString(String msg) {

        System.out.println("Hello : " + msg);
    }

}
```

### 二、Spring配置
1、回顾xml配置
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 
    <bean id="helloBean" class="com.mkyong.HelloWorld">
        
</beans>
```

2、JavaConfig，在AppConfig.java中
```java
package com.mkyong;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    
    @Bean(name="helloBean")
    public HelloWorld helloWorld() {
        return new HelloWorld();
    }
    
}
```

### 三、运行，在App.java中
```java
package com.mkyong;
 
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main(String[] args) {
        
        //ApplicationContext context = new ClassPathXmlApplicationContext("SpringBeans.xml");
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        HelloWorld obj = (HelloWorld) context.getBean("helloBean");
        
        obj.toString("Hello,Spring Java Config");

    }
}
```

### 四、结果
```
Hello : Hello,Spring Java Config
```

### 五、目录结构
![](/assets/img/2015/spring-java-config.png){: .img-small}

### 六、补充@Import
当要配置多个文件时候，可以用@Import

#### 1、在xml中
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
 
    <import resource="config/customer.xml"/>
    <import resource="config/scheduler.xml"/>
 
</beans>
```

#### 2、在JavaConfig中
```java
package com.mkyong.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({ CustomerConfig.class, SchedulerConfig.class })
public class AppConfig {

}
```
