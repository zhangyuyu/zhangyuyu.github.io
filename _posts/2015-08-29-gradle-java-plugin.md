---
layout: post
title: "Gradle Java Plugin"
date: 2015-08-29 19:42:22
categories: tool
tags: gradle
---
　　java plugin给项目添加了java编译，测试和捆绑的能力，是很多其他gradle plugin的基础。
在build.gradle中加入`apply plugin: 'java'`，即可使用java plugin。

### 一、java Plugin引入的主要Task
执行`gradle build`，我们已经可以看到java Plugin所引入的主要Task：
```
>:compileJava
>:processResources
>:classes
>:jar
>:assemble
>:compileTestJava
>:processTestResources
>:testClasses
>:test
>:check
>:build
```

　　build也是java Plugin所引入的一个Task，它依赖于其他Task，其他Task又依赖于另外的Task，所以有了以上Task执行列表。以上Task执行列表基本上描述了java Plugin向项目中所引入的构建生命周期概念。
　　除了定义众多的Task外，java Plugin还向Project中加入了一些额外的Property。比如，sourceCompatibility用于指定在编译Java源文件时所使用的Java版本，archivesBaseName用于指定打包成Jar文件时的文件名称。
　　
### 二、Source sets
　　source set是一组编译和执行的源文件，和编译路径与执行路径有联系。Gradle的每个source set都包含有一个名字，并且包含有一个名为java的Property和一个名为resources的Property，他们分别用于表示该source set所包含的Java源文件集合和资源文件集合。
```groovy
sourceSets {
    main {
        java {
            srcDir 'src/java'
        }
        resources {
            srcDir 'src/resources'
        }
    }
}
```
　　Gradle会自动地为每一个新创建的source set创建相应的Task。对于名为mySourceSet的source set，Gradle将为其创建compile<mySourceSet>Java、process<mySourceSet>Resources和<mySourceSet>Classes这3个Task。对于这里api而言，Gradle会为其创建名为compileApiJava、processApiResource和apiClasses Task。我们可以在命令行中执行"gradle apiClasses"。<br>
>注意，对于main而言，Gradle并没有相应的compileMainJava，原因在于：由于main是Gradle默认创建的source set，并且又是及其重要的source set，Gradle便省略掉了其中的"Main"，而是直接使用了compileJava作为main的编译Task。对于test来说，Gradle依然采用了compileTestJava。

　　通常的情况是，我们自己创建的名为api的source set会被其他source set所依赖，比如main中的类需要实现api中的某个接口等。此时我们需要做两件事情。
　　* 第一，我们需要在编译main之前对api进行编译，即编译main中Java源文件的Task应该依赖于api中的Task：`classes.dependsOn apiClasses`
　　* 第二，在编译main时，我们需要将api编译生成的class文件放在main的classpath下。此时，我们可以对main和test做以下配置：
 
 ```groovy
sourceSets {
  main {
     compileClasspath = compileClasspath + files(api.output.classesDir)
  }
  test {
     runtimeClasspath = runtimeClasspath + files(api.output.classesDir)
  }
}
```

### 三、依赖管理
#### 1、repositories
　　在声明对第三方类库的依赖时，我们需要告诉Gradle在什么地方去获取这些依赖，即配置Gradle的Repository。在配置好依赖之后，Gradle会自动地下载这些依赖到本地。Gradle可以使用Maven和Ivy的Repository，同时它还可以使用本地文件系统作为Repository。
配置Maven的Repository如下：
```groovy
repositories {
  mavenCentral()
}
```
	
#### 2、configurations
　　Gradle将对依赖进行分组，每一组依赖称为一个Configuration，在声明依赖时，我们实际上是在设置不同的Configuration。
##### (1）定义一个Configuration：
```groovy
configurations {
  myDependency
}
``` 
 
##### (2) 加入实际的依赖项：
 ```groovy
dependencies {
  myDependency 'org.apache.commons:commons-lang3:3.0'
}
```
 
>java Plugin定义了compile和testCompile这两个Configuration，分别用于编译Java源文件和编译Java测试源文件。也定义了runtime和testRuntime，分别用于在程序运行和测试运行时加入所配置的依赖。

　　因此，下述声明依赖时，可以直接使用dependencies：
```groovy
dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

##### (3）获取classpath
```groovy
task showMyDependency << {
  println configurations.myDependency.asPath
}
``` 
