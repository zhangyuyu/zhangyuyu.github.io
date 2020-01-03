---
layout: post
title: "Spring AOP Advice"
date: 2015-08-27 20:57:00
categories: java
tags: 
- java
- spring
---
　　Spring AOP即Aspect-oriented programming，面向切面编程，专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题。简单地说，就是一个拦截器（interceptor）拦截一些处理过程。例如，当一个method被执行，Spring AOP能够劫持正在运行的method，在method执行前或者后加入一些额外的功能。
>在Spring AOP中，支持4中类型的通知（Advice）:<br>
* Before advice      ——method执行前通知<br>
* After returning advice ——method返回一个结果后通知<br>
* After throwing advice – method抛出异常后通知<br>
* Around advice – 环绕通知，结合了以上三种

### 一、不使用AOP的简单例子

#### 1、在CustomerService.java中
```java
package com.zhangyu.customer.services;

public class CustomerService {
    private String name;
    private String occupation;

    public void setName(String name) {this.name = name;}

    public void printName() {System.out.println("Customer name : " + this.name);}

    public void printOccupation() {System.out.println("Customer occupation : " + this.occupation);}

    public void setOccupation(String occupation) {this.occupation = occupation;}

    public void printThrowException() {
        throw new IllegalArgumentException();
    }
}
```

#### 2、在Spring-Customer.xml中
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean id="customerService" class="com.zhangyu.customer.services.CustomerService">
        <property name="name" value="Zhang Yu" />
        <property name="occupation" value="developer" />
    </bean>
</beans>
```

#### 3、运行，在App.java中
```java
package com.zhangyu.common;

import com.zhangyu.customer.services.CustomerService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext appContext = new ClassPathXmlApplicationContext(
                new String[] { "Spring-Customer.xml" });

        CustomerService cust = (CustomerService) appContext
                .getBean("customerService");

        System.out.println("*************************");
        cust.printName();
        System.out.println("*************************");
        cust.printOccupation();
        System.out.println("*************************");
        try {
            cust.printThrowException();
        } catch (Exception e) {

        }
    }
}
```

#### 4、结果
```
*************************
Customer name : Zhang Yu
*************************
Customer occupation : developer
*************************
```

### 二、使用AOP
#### 1、Before Advice

##### （1）HelloBeforeMethod.java中
　　创建一个实现了接口MethodBeforeAdvice的class，method运行前，将运行HelloBeforeMethod.java　　

```java
package com.zhangyu.aop;

import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;

public class HelloBeforeMethod implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target)
            throws Throwable {
        System.out.println("HelloBeforeMethod : Before method ,hello");
    }
}
```

##### (2) Spring-Customer.xml中
　　在Spring-Customer.xml中加入新的bean配置`HelloBeforeMethodBean`，然后创建一个新的代理（proxy），命名为`customerServiceProxy`。其中,`target`定义你想劫持哪个bean，`interceptorNames`定义你想用哪个class(advice)劫持target。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
    <bean id="customerService" class="com.zhangyu.customer.services.CustomerService">
        <property name="name" value="Zhang Yu" />
        <property name="occupation" value="developer" />
    </bean>

    <bean id="helloAroundMethodBean" class="com.zhangyu.aop.HelloAroundMethod" />
    <bean id="helloBeforeMethodBean" class="com.zhangyu.aop.HelloBeforeMethod" />

    <bean id="customerServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="customerService" />
        <property name="interceptorNames">
            <list>
                <value>helloBeforeMethodBean</value>
            </list>
        </property>
    </bean>
</beans>
```

##### (3) 运行，在App.java中　　
将`CustomerService cust`改为从`customerServiceProxy`中getBean:
```
CustomerService cust = (CustomerService)appContext.getBean("customerServiceProxy");
```

##### (4) 结果

```
*************************
HelloBeforeMethod : Before method ,hello
Customer name : Zhang Yu
*************************
HelloBeforeMethod : Before method ,hello
Customer occupation : developer
*************************
HelloBeforeMethod : Before method ,hello
```

#### 2、After Advice
##### (1) 在HelloAfterMethod.java中

```java
package com.zhangyu.aop;

import java.lang.reflect.Method;
import org.springframework.aop.AfterReturningAdvice;

public class HelloAfterMethod implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method,
            Object[] args, Object target) throws Throwable {
        System.out.println("HelloAfterMethod : After method ,hello!");
    }
}
```

##### (2) 在Spring-Customer.xml中
　　在Spring-Customer.xml中加入新的bean配置`HelloAfterMethodBean`，然后在customerServiceProxy中设置"interceptorNames"的value为`helloAfterMethodBean`。
　　
##### （3）运行结果

```
*************************
Customer name : Zhang Yu
HelloAfterMethod : After method ,hello
*************************
Customer occupation : developer
HelloAfterMethod : After method ,hello
*************************
```
　　执行到cust.printThrowException()后，直接抛出异常，方法没有正常执行完毕（或者说没有返回结果），所以不运行切入的afterReturning方法。

#### 3、After throwing advice

　　创建一个实现了ThrowsAdvice接口的class，劫持IllegalArgumentException异常，目标method运行时，抛出IllegalArgumentException异常后，运行切入的方法。

##### (1) 在HelloThrowException.java中
```java
package com.zhangyu.aop;

import org.springframework.aop.ThrowsAdvice;

public class HelloThrowException implements ThrowsAdvice {
    public void afterThrowing(IllegalArgumentException e) throws Throwable {
        System.out.println("HelloThrowException : Throw exception ,hello");
    }
}
```

##### (2) 在Spring-Customer.xml中
　　在Spring-Customer.xml中加入新的bean配置`HelloThrowExceptionBean`，然后在customerServiceProxy中设置"interceptorNames"的value为`helloThrowExceptionBean`。
　　
##### （3）运行结果
```
*************************
Customer name : Zhang Yu
*************************
Customer occupation : developer
*************************
HelloThrowException : Throw exception ,hello
```

者说没有返回结果），所以不运行切入的afterReturning方法。


#### 4、Around advice

##### (1) 在HelloAroundMethod.java中
```java
package com.zhangyu.aop;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class HelloAroundMethod implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {

        System.out.println("Method name : " + methodInvocation.getMethod().getName());
        
        // same with MethodBeforeAdvice
        System.out.println("HelloAroundMethod : Before method ,hello");

        try {
            Object result = methodInvocation.proceed();
            
            // same with AfterReturningAdvice
            System.out.println("HelloAroundMethod : After method ,hello!");
            return result;
        } catch (IllegalArgumentException e) {
            // same with ThrowsAdvice
            System.out.println("HelloAroundMethod : Throw exception ,hello!");
            throw e;
        }
    }
}
```

##### (2) 在Spring-Customer.xml中
　　在Spring-Customer.xml中加入新的bean配置`HelloAroundMethod`，然后在customerServiceProxy中设置"interceptorNames"的value为`helloAroundMethodBean`。
　　
##### （3）运行结果
```
*************************
Method name : printName
HelloAroundMethod : Before method ,hello!
Customer name : Zhang Yu
HelloAroundMethod : After method ,hello!
*************************
Method name : printOccupation
HelloAroundMethod : Before method ,hello!
Customer occupation : developer
HelloAroundMethod : After method ,hello!
*************************
Method name : printThrowException
HelloAroundMethod : Before method ,hello!
HelloAroundMethod : Throw exception ,hello!
```

### 三、目录结构
![](/assets/img/spring-aop-advice.png){: .img-medium}
