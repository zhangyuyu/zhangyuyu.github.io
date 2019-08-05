---
layout: post
title: "SpringMVC_Gradle_xml"
date: 2015-08-28 12:35:55
categories: java
tags: 
- java
- spring
---

### 一、Gradle Build

	apply plugin: 'java'
	apply plugin: 'war'
	apply plugin: 'idea'
	apply plugin: 'jetty'
	
	// JDK 7
	sourceCompatibility = 1.7
	targetCompatibility = 1.7
	
	repositories {
	    mavenLocal()
	    mavenCentral()
	}
	
	dependencies {
	 
		compile 'ch.qos.logback:logback-classic:1.1.3'
		compile 'org.springframework:spring-webmvc:4.1.6.RELEASE'
		compile 'javax.servlet:jstl:1.2'
		
	}
	
	// Embeded Jetty for testing
	jettyRun{
		contextPath = "spring4"
		httpPort = 8080
	}
	
	jettyRunWar{
		contextPath = "spring4"
		httpPort = 8080
	}

　　然后，在Terminal里运行```gradle idea```,下载相关依赖，支持intellij IDE。
### 二、Spring MVC

#### 1、WelcomeController.java

#### 2、HelloWorldService.java

#### 3、logback.xml

#### 4、spring-core-config.xml

#### 5、spring-mvc-config.xml

#### 6、web.xml

### 三、运行，输出结果
　　在terminal中运行`gradle jettyRun`：

	$ gradle jettyRun                    
	:compileJava
	:processResources
	:classes
	:jettyRun
	Building 75% > :jettyRun > Running at http://localhost:8080/spring4
	
　　然后，在浏览器输入http://localhost:8080/spring4，可以看到页面。
### 四、目录结构
![](/assets/img/spring-mvc-gradle-xml.png)


