---
layout: post
title: "Gradle Property"
date: 2015-08-29 10:38:22
categories: tool
tags: gradle
---
　　很多Plugin都会向Project中加入额外的Property，在使用这些Plugin时，我们需要对这些Property进行赋值。
　　Gradle在默认情况下已经为Project定义了很多Property，其中比较常用的有：
>project：Project本身
name：Project的名字
path：Project的绝对路径
description：Project的描述信息
buildDir：Project构建结果存放目录
version：Project的版本号


### 一、project属性配置
	version = 'this is the project version'
	description = 'this is the project description'
	task showProjectProperties << {
	  println version
	  println project.description
	}
	
　　注意，在打印description时，我们使用了project.description，而不是直接使用description。原因在于，Project和Task都拥有description属性，而定义Task的闭包将delegate设置成了当前的Task，故如果直接使用description，此时打印的是showProjectProperties的description，而不是Project的，所以我们需要显式地指明project。
### 二、Task属性配置
#### 1、对desctiption属性的配置
##### （1）定义时候对description属性的配置
build.gradle:

	task myTask << {
	  description = "this is myProperty"
	  println description
	}

##### （2）myTask.description
	task myTask << {
		  println description
	}
	myTask.description = "this is myProperty"
##### (3) 同名的方法中设置description
	task myTask << {
		  println description
	}
	myTask{
		  description = "this is myProperty"
	}
##### (4）调用Task的configure()方法,配置description属性
build.gradle

	task myTask << {
	  println description
	}
	myTask.configure {
	  description = "this is myProperty"
	}

运行，结果：

	$ gradle -q myTask
	this is myProperty
#### 2、自定义属性的配置
##### (1) 额外自定义属性的配置
build.gradle:

	task myTask {
	    ext.myProperty = "this is myProperty"
	}
	
	task printTaskProperties << {
	    println myTask.myProperty
	}
也可以使用闭包的方式：

	ext { myProperty = "this is myProperty"}
Terminal:

	$ gradle -q printTaskProperties
	this is myProperty
##### (2) 通过命令行参数自定义Property
build.gradle

	task myTask << {
		  println myProperty
	}
此时，会报错：

	* What went wrong:
	Execution failed for task ':myTask'.
	> Could not find property 'myProperty' on task ':myTask'.
　　表示myProperty并没有被定义，在调用gradle命令时，通过-P参数传入该Property：

	$ gradle -PmyProperty="this is myProperty" myTask                  
	:myTask
	this is myProperty
