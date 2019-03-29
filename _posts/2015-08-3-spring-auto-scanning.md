---
layout: post
title: "Spring-Auto-scanning"
date: 2015-08-30 11:38:10
categories: java
tags: 
- java
- spring
---
### 一、一般的bean
#### 1、CustomerDAO.java
	package com.mkyong.customer.dao;
	
	public class CustomerDAO 
	{
		@Override
		public String toString() {
			return "Hello , This is CustomerDAO";
		}	
	}
#### 2、CustomerService.java
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
#### 3、Spring-Customer.xml
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
		
		<bean id="customerService" class="com.mkyong.customer.services.CustomerService">
			<property name="customerDAO" ref="customerDAO" />
		</bean>
	
		<bean id="customerDAO" class="com.mkyong.customer.dao.CustomerDAO" />
	
	</beans>
### 二、auto Scanning
#### 1、CustomerDAO.java
	package com.mkyong.customer.dao;
	
	import org.springframework.stereotype.Component;
	
	@Component
	public class CustomerDAO 
	{
		@Override
		public String toString() {
			return "Hello , This is CustomerDAO";
		}	
	}
#### 2、CustomerService.java
	package com.mkyong.customer.services;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	
	import com.mkyong.customer.dao.CustomerDAO;
	
	@Component
	public class CustomerService 
	{
		@Autowired
		CustomerDAO customerDAO;
	
		@Override
		public String toString() {
			return "CustomerService [customerDAO=" + customerDAO + "]";
		}
	}
#### 3、Spring-Customer.xml
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-2.5.xsd">
	
		<context:component-scan base-package="com.mkyong.customer" />
	
	</beans>
### 三、Antotation Types——自动扫描组件的注释类型

有4种注释类型，分别是：

@Component      ——表示一个自动扫描component

@Repository              ——表示持久化层的DAO component

@Service             ——表示业务逻辑层的Service component

@Controller        ——表示表示层的Controller component
Spring将会扫描所有用@Component注释过得组件。

　　实际上，@Repository、@Service、@Controller三种注释它们都用@Component注释过。所以，在项目中，我们可以将所有自动扫描组件都用@Component注释，Spring将会扫描所有用@Component注释过得组件。
　　但是，为了更好的可读性，应该在不同的应用层中，用不同的注释。
### 四、Spring Filter Components —— 在自动扫描中过滤组件
#### 1、Filter Component——include
　　用“filter”自动扫描注册组件，这些组件只要匹配定义的“regex”的命名规则，Clasee前就不需要用@Component进行注释。
##### （1）DAO层，在CustomerDAO.java中
	package com.mkyong.customer.dao;
	
	public class CustomerDAO 
	{
		@Override
		public String toString() {
			return "Hello , This is CustomerDAO";
		}	
	}
##### （2）Service层，在CustomerService.java中
	package com.mkyong.customer.services;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import com.mkyong.customer.dao.CustomerDAO;
	
	public class CustomerService 
	{
		@Autowired
		CustomerDAO customerDAO;
	
		@Override
		public String toString() {
			return "CustomerService [customerDAO=" + customerDAO + "]";
		}
			
	}
##### （3）在Spring-Customer.xml中
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-2.5.xsd">
	
		<context:component-scan base-package="com.mkyong" >
	
			<context:include-filter type="regex" 
	                       expression="com.mkyong.customer.dao.*DAO.*" />
	
			<context:include-filter type="regex" 
	                       expression="com.mkyong.customer.services.*Service.*" />
	
		</context:component-scan>
	
	</beans>
　　以上xml文件中，所有文件名字，只要包含DAO和Service`（*DAO.*，*Service.*）`关键字的，都将被检查注册到Spring容器中。
#### 2、Filter Component——exclude
　　用exclude，制定组件避免被Spring发现并被注册到容器中。
以下配置排除用@Service注释过的组件，注意type="annotation"

	<context:component-scan base-package="com.lei.customer" >
	        <context:exclude-filter type="annotation" 
	            expression="org.springframework.stereotype.Service" />        
	</context:component-scan>
以下配置排除包含“DAO”关键字的组件

	<context:component-scan base-package="com.lei" >
	        <context:exclude-filter type="regex" 
	            expression="com.mkyong.customer.dao.*DAO.*" />        
	</context:component-scan>
