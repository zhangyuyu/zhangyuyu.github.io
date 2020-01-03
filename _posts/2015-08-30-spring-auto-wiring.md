---
layout: post
title: "Spring Auto Wiring"
date: 2015-08-30 12:09:07
categories: java
tags: 
- java
- spring
---
Spring支持5种自动装配模式，如下：

|自动装配模式   | 描述           |
|:------------|:---------------|
| no          | 默认情况下，不自动装配，通过"ref"属性手动设定。|
| byName      | 根据Property的Name自动装配，如果一个bean的name，和另一个bean中的Property的name相同，则自动装配这个bean到Property中。|
| byType      | 根据Property的数据类型（Type）自动装配，如果一个bean的数据类型，兼容另一个bean中Property的数据类型，则自动装配。|
| constructor | 根据构造函数参数的数据类型，进行byType模式的自动装配。|
| autodetect  | 如果发现默认的构造函数，用constructor模式，否则，用byType模式。|

### 一、bean
CustomerDAO.java
```java
package com.mkyong.customer.dao;

public class CustomerDAO 
{
    @Override
    public String toString() {
        return "Hello , This is CustomerDAO";
    }	
}
```

CustomerService.java
```java
package com.mkyong.customer.services;

import com.mkyong.customer.dao.CustomerDAO;

public class CustomerService 
{
    CustomerDAO customerDAO;

    public void setCustomerDAO(CustomerDAO customerDAO) {
        this.customerDAO = customerDAO;
    }

    @Override
    public String toString() {
        return "CustomerService [customerDAO=" + customerDAO + "]";
    }			
}
```

### 二、装配

#### 1、Auto-Wiring `no`
　　默认情况下，需要通过`ref`来装配bean，如下：
```xml
<bean id="customerService" class="com.mkyong.customer.services.CustomerService">
    <property name="customerDAO" ref="customerDAO" />
</bean>

<bean id="customerDAO" class="com.mkyong.customer.dao.CustomerDAO" />
```

#### 2、Auto-Wiring `byName`
　　根据属性Property的名字装配bean，这种情况，CustomerService设置了`autowire="byName"`，Spring会自动寻找与属性名字`customerDAO`相同的bean，找到后，通过调用`setCustomerDAO(CustomerDAO customerDAO)`将其注入属性。
```xml
<bean id="customerService" class="com.mkyong.customer.services.CustomerService" autowire="byName">
<bean id="customerDAO" class="com.mkyong.customer.dao.CustomerDAO" />
```

#### 3、Auto-Wiring `byType`
　　根据属性Property的数据类型自动装配，这种情况，Customer设置了`autowire="byType"`，Spring会总动寻找与属性类型相同的bean。
```xml
<bean id="customerService" class="com.mkyong.customer.services.CustomerService" autowire="byType">
<bean id="customerDAO" class="com.mkyong.customer.dao.CustomerDAO" />
```
　　如果配置文件中有两个类型相同的bean，将抛出UnsatisfiedDependencyException异常，所以，一旦选择了`byType`类型的自动装配，请确认你的配置文件中每个数据类型定义一个唯一的bean。

#### 4、Auto-Wiring `constructor`
　　Spring会寻找与参数数据类型相同的bean，通过构造函数public Customer(Person person)将其注入。

#### 5、Auto-Wiring `autodetect`
　　这种情况下，Spring会先寻找Customer中是否有默认的构造函数，如果有相当于上边的`constructor`这种情况，用构造函数注入，否则，用`byType`这种方式注入，
>项目中autowire结合dependency-check一起使用是一种很好的方法，这样能够确保属性总是可以成功注入。

```xml
<bean id="customer" class="com.mkyong.customer.services.CustomerService" autowire="autodetect" dependency-check="objects" />
<bean id="person" class="com.mkyong.customer.dao.CustomerDAO" />
```

　　**最后，Auto-Wiring虽然使开发变得更加快速，但是增加了配置文件的复杂性，因此可以选择用手工装配、或者用[@Autowired](http://zhangyuyu.github.io/2015/08/30/Spring-Auto-Wiring-Autowired/)、或者结合[@Component](http://zhangyuyu.github.io/2015/08/30/Spring-Auto-scanning/)。**
