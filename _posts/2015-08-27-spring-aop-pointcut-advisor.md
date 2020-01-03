---
layout: post
title: "Spring AOP Pointcut & Advisor"
date: 2015-08-27 22:56:33
categories: java
tags: 
- java
- spring
---
　　上一篇的Spring AOP Advice例子中，Class（CustomerService）中的全部method都被自动的拦截了。但是大多情况下，你只需要去拦截一两个method。这样就引入了Pointcut（切入点）的概念，它允许你根据method的名字去拦截指定的method。另外，一个Pointcut必须结合一个Advisor来使用。
>在Spring AOP中，有3个常用的概念:
* Advices：表示一个method执行前或执行后的动作。
* Pointcut：表示根据method的名字或者正则表达式去拦截一个method。
* Advisor：Advice和Pointcut组成的独立的单元，并且能够传给proxy factory 对象。

### 一、回顾Around advice
#### 1、在CustomerService.java中
```java
package com.zhangyu.customer.services;

public class CustomerService {
    private String name;
    private String occupation;

    public void setName(String name) {	this.name = name;}

    public void printName() {System.out.println("Customer name : " + this.name);}

    public void printOccupation() {System.out.println("Customer occupation : " + this.occupation);}

    public void setOccupation(String occupation) {this.occupation = occupation;}

    public void printThrowException() {
        throw new IllegalArgumentException();
    }	
}
```

#### 2、 在HelloAroundMethod.java中
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
            System.out.println("HelloAroundMethod : Before after ,hello!");
            return result;
        } catch (IllegalArgumentException e) {
            // same with ThrowsAdvice
            System.out.println("HelloAroundMethod : Throw exception ,hello!");
            throw e;
        }
    }
}
```
	
#### 3、 在Spring-Customer.xml中
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
        
    <bean id="customerServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="customerService" />
        <property name="interceptorNames">
            <list>
                <value>helloAroundMethodBean</value>
            </list>
        </property>
    </bean>
</beans>
```

#### 4、运行，在App.java中
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
　
#### 5、运行结果
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

### 二、Pointcut
　　可以用名字匹配法和正则表达式匹配法去匹配要拦截的method。

#### 1、Pointcut——名字匹配法
##### （1）通过pointcut和advisor拦截printName()方法。
　　* 创建一个NameMatchMethodPointcut的bean，将你想拦截的方法的名字printName注入到属性mappedName。
　　* 创建一个DefaultPointcutAdvisor的advisor bean，将pointcut和advice关联起来。
　　* 更改代理的interceptorNames的value值为customerAdvisor。
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

    <bean id="customerServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="customerService" />
        <property name="interceptorNames">
            <list>
                <value>customerAdvisor</value>
            </list>
        </property>
    </bean>

    <bean id="customerPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
        <property name="mappedName" value="printName" />
    </bean>

    <bean id="customerAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="pointcut" ref="customerPointcut" />
        <property name="advice" ref="helloAroundMethodBean" />
    </bean>
</beans>
```

#####（2）运行，结果如下：
```
*************************
Method name : printName
HelloAroundMethod : Before method ,hello!
Customer name : Zhang Yu
HelloAroundMethod : After method ,hello!
*************************
Customer occupation : developer
*************************
```

　　可以看到，只拦截了printName（），在其前后分别输出了“Before method ，hello!”和“After method ,hello!”

#####（3）另外
　　以上配置中pointcut和advisor可以合并在一起配置，即不用单独配置customerPointcut和customerAdvisor，只要配置customerAdvisor时class选择NameMatchMethodPointcutAdvisor如下：
```xml
<bean id="customerAdvisor" class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
    <property name="mappedName" value="printName" />
    <property name="advice" ref="helloAroundMethodBean" />
</bean>
```
	
　　但是，如果将method名字单独配置成pointcut（切入点），advice和pointcut的结合会更灵活，使一个pointcut可以和多个advice结合，更符合松耦合理念。

#### 2、Pointcut——正则表达式匹配法
##### (1) 你可以配置用正则表达式匹配需要拦截的method：
```xml
<bean id="customerAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="patterns">
        <list>
            <value>.*Occu.*</value>
        </list>
    </property>
    <property name="advice" ref="helloAroundMethodBean" />
</bean>
```

##### (2) 结果
```
*************************
Customer name : Zhang Yu
*************************
Method name : printOccupation
HelloAroundMethod : Before method ,hello!
Customer occupation : developer
HelloAroundMethod : After method ,hello!
*************************
```
　可以看到，拦截名字中包含了Occu字符的method，这里是printOccupation()。
