---
layout: post
title: "Gradle中buildscript"
date: 2015-08-30
categories: tool
tags: 
- gradle
---

## 一、buildscript、根级别、allprojects的区别
　　在编写Gradle脚本的时候，在build.gradle文件中经常看到这样的代码：
```groovy build.gradle
buildscript {
    repositories {
        mavenCentral()
    }
}

repositories {
    mavenCentral()
}

allprojects {
    repositories {
        mavenCentral()
    }
}

```

1. buildscript
buildscript的repositories，主要是为了Gradle脚本自身的执行，获取脚本依赖插件。

2. 根级别
根级别的repositories，主要是为了当前项目提供所需依赖包，比如log4j、spring-core等依赖包可从mavenCentral仓库获得。

3. allprojects
allprojects块的repositories，用于多项目构建，为所有项目提供共同所需依赖包。而子项目可以配置自己的repositories以获取自己独需的依赖包。


## 二、示例
### 1. buildscript
> 需求：假设我们要在build.gradle里面编写一个task，用于解析csv文件并输出其内容。

虽然我们可以使用gradle编写解析csv文件的代码，但其实apache有个库已经实现了一个解析csv文件的库供我们直接使用。
我们如果想要使用这个库，需要在gradle.build文件中加入对该库的引用。

```groovy
buildscript {
    repositories {
        mavenLocal()
        mavenCentral()
    }

    dependencies {
        classpath 'org.apache.commons:commons-csv:1.0'
    }
}

import org.apache.commons.csv.*

task printCSV() {
    doLast {
        def records = CSVFormat.EXCEL.parse(new FileReader('config/sample.csv'))
        for (item in records) {
            print item.get(0) + ' '
            println item.get(1)
        }

    }
}
```
　　这个时候，是在buildscript里面加入对apache的common-csv库的引用。需要注意的是在buildscript代码块中，对dependencies使用的是classpath声明。  
　　该classpath声明说明了在执行其余的build脚本时，class loader可以使用这些你提供的依赖项。这也正是我们使用buildscript代码块的目的。

### 2. 根级别
> 如果你的项目中也需要使用该类库

　　就需要定义在buildscript代码块之外的dependencies代码块中，所以有可能会看到在build.gradle中出现以下代码：
```groovy
repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile 'org.springframework.ws:spring-ws-core:2.2.0.RELEASE',
            'org.apache.commons:commons-csv:1.0'
}


buildscript {
    repositories {
        mavenLocal()
        mavenCentral()
    }

    dependencies {
        classpath 'org.apache.commons:commons-csv:1.0'
    }
}

import org.apache.commons.csv.*

task printCSV() {
    doLast {
        def records = CSVFormat.EXCEL.parse(new FileReader('config/sample.csv'))
        for (item in records) {
            print item.get(0) + ' '
            println item.get(1)
        }

    }
}
```

## 参考
* [Gradle中的buildscript代码块](http://www.huangbowen.net/blog/2014/08/27/buildscript-in-gradle/)
* [External dependencies for the build script](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:build_script_external_dependencies)
