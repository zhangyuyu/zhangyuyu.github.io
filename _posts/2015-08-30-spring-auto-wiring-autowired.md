---
layout: post
title: "Spring Auto Wiring @Autowired"
date: 2015-08-30 12:52:14
categories: java
tags: 
- java
- spring
---
### 一、bean
Customer.java
```java
package com.mkyong.common;

import org.springframework.beans.factory.annotation.Autowired;

public class Customer {
    @Autowired
    private Person person;
    private int type;
    private String action;
    
    //getter and setter methods
    
    @Override
    public String toString() {
        return "Customer [person=" + person + ", type=" + type + ", action="
                + action + "]";
    }
}
```

Person.java
```java
package com.mkyong.common;
    
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person [name=" + name + "]";
    }
}
```

### 二、 AutowiredAnnotationBeanPostProcessor
　　使用@Autowired, 必须在`SpringBeans.xml`登记 `AutowiredAnnotationBeanPostProcessor`,下面有两种方式：

#### 1、context:annotation-config
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-2.5.xsd">

    <context:annotation-config />

    <bean id="CustomerBean" class="com.mkyong.common.Customer">
        <property name="action" value="buy" />
        <property name="type" value="1" />
    </bean>

    <bean id="PersonBean" class="com.mkyong.common.Person">
        <property name="name" value="mkyong" />
    </bean>
    
</beans>
```

#### 2、直接包含AutowiredAnnotationBeanPostProcessor
将`<context:annotation-config />`替换成
```
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
```

### 三、运行
在App.java中
```java
package com.mkyong.common;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                "SpringBeans.xml");

        Customer cust = (Customer) context.getBean("customer");
        System.out.println(cust);
    }
}
```

### 四、运行结果
```Customer [person=Person [name=mkyongA], type=1, action=buy]```

### 五、Dependency checking
　　默认情况下，`@Autowired`是使用Dependency checking的，确保属性总是可以成功注入。如果Spring没有找到匹配的bean去wire，就会扔出异常。可以使用`@Autowired(required=false)`去关闭Dependency checking。

### 六、@Qualifier
　　`@Qualifier`用来控制哪个bean应该被autowire。例如，在下面的例子的`SpringBeans.xml`中配置了`PersonBean1`和`PersonBean2`.
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-2.5.xsd">

    <context:annotation-config />

    <bean id="CustomerBean" class="com.mkyong.common.Customer">
        <property name="action" value="buy" />
        <property name="type" value="1" />
    </bean>

    <bean id="PersonBean1" class="com.mkyong.common.Person">
        <property name="name" value="mkyong1" />
    </bean>
    
    <bean id="PersonBean2" class="com.mkyong.common.Person">
        <property name="name" value="mkyong2" />
    </bean>
    
</beans>
```

此时，在`Customer.java`中用`@Qualifier("PersonBean1")`就可以将`PersonBean1`注入到`Customer`的`person`属性中。
```java
package com.mkyong.common;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

public class Customer 
{
    @Autowired
    @Qualifier("PersonBean1")
    private Person person;
    private int type;
    private String action;
    //getter and setter methods
}
```
