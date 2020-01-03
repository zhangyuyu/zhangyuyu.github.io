---
layout: post
title: "CMake初识"
date: 2018-06-24 14:34:14
categories: tool
tags:
- cmake
---
 前言
>本文的主要观点、代码片段、图片均源自《The Hitchhiker’s Guide to the CMake》教程的Overview 章节。

主要包含以下方面：

* 认识 CMake
* CMake可以做什么？
* CMake 的特点
* CMake 的 Workflow
* CMake 的组成？
* make 与 ninja

<!-- more -->

## 一、认识 Cmake
　　CMake，是`Cross Platform Make`的缩写，是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。

　　CMake 自己本身并不是构建工具（build tool），它不直接建构出最终的软件。它的职责是从抽象配置代码生成原生构建
工具（native build tool）文件。

>原生构建工具比如：
* Xcode
* Visual Studio
* Ninja
* Make

　　CMake使用指定名为`CMakeLists.txt`的配置文件可以控制软件的构建、测试和打包等流程。同时，通过编写平台无关的
`CMakeLists.txt`文件和需要简单的配置，CMake就能生成对应目标平台的构建文件，例如：类Unix系统的`makefile`文件、
Windows的`Visual Studio`工程或者Mac的`Xcode`工程，大大简化了跨平台和交叉编译方面的工作。

## 二、Cmake可以做什么？

### 1. 跨平台开发

　　假设你有C++的跨平台项目，其代码在不同的平台/IDE共享。比如，Windows的`Visual Studio`、 OSX的`XCode`和 
Linux 的`Makefile`：
![](/assets/img/cmake-native-build.png)

　　如果要添加一个`bar.cpp`源文件，你会怎么做？你不得不将该文件添加到你所使用的每个工具中：
![](/assets/img/cmake-native-build-add.png)

　　为了保证环境的一致性，你不得不将类似的更新操作好几次。更重要的是，你还得手动进行操作（如下图红色的箭头）。
当然这种方式很容易出错，而且很不灵活。

　　CMake 通过在开发过程中增加额外的步骤来解决这一设计缺陷。你可以在`CMakeList.txt`文件中描述项目，并使用 CMake 
通过跨平台的 CMake 代码来生成你感兴趣的构建工具。
![](/assets/img/cmake-generate-native-files.png)

　　同样地，当要添加`bar.cpp`源文件的时候，就只需要一个步骤了：
![](/assets/img/cmake-generate-native-files-add.png)

　　注意，上述图表的底部并没有发生变化，你仍然可以使用你喜欢的工具，比如`Visual Studio/msbuild`, `Xcode/xcodebuild` 
和 `Makefile/make`。

### 2. VCS 友好

　　当你在团队工作时候，你可能想共享、保存更改的历史记录，这就是通常版本控制（VCS）做的事情。在实践中，对于IDE 
的文件（比如*.sln ）该怎么存储呢？
下面是将`bar.cpp`文件加入到`Visual Studio`解决方案之后，可执行文件的差异：

```text
--- /overview/snippets/foo-old.sln
+++ /overview/snippets/foo-new.sln
@@ -4,6 +4,8 @@
 VisualStudioVersion = 14.0.25123.0
 MinimumVisualStudioVersion = 10.0.40219.1
 Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "foo", "foo.vcxproj", "{C8F8C325-ACF3-460E-81DF-8515C72B334A}"
+EndProject
+Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "bar", "..\bar\bar.vcxproj", "{D14B78EA-1ADA-487F-B1ED-42C2B919C000}"
 EndProject
 Global
    GlobalSection(SolutionConfigurationPlatforms) = preSolution
@@ -21,6 +23,14 @@
        {C8F8C325-ACF3-460E-81DF-8515C72B334A}.Release|x64.Build.0 = Release|x64
        {C8F8C325-ACF3-460E-81DF-8515C72B334A}.Release|x86.ActiveCfg = Release|Win32
        {C8F8C325-ACF3-460E-81DF-8515C72B334A}.Release|x86.Build.0 = Release|Win32
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Debug|x64.ActiveCfg = Debug|x64
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Debug|x64.Build.0 = Debug|x64
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Debug|x86.ActiveCfg = Debug|Win32
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Debug|x86.Build.0 = Debug|Win32
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Release|x64.ActiveCfg = Release|x64
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Release|x64.Build.0 = Release|x64
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Release|x86.ActiveCfg = Release|Win32
+       {D14B78EA-1ADA-487F-B1ED-42C2B919C000}.Release|x86.Build.0 = Release|Win32
    EndGlobalSection
    GlobalSection(SolutionProperties) = preSolution
        HideSolutionNode = FALSE
```
　　同时还增加了一个`bar.vcxproj`文件，大概有150行代码，此处省略该代码。当使用`Xcode`的时候，类似地，在`.pbxproj`
文件也有很多更改。

　　我们只是简单的添加了一个源文件，就有上述如此多的改动，于是你不禁会问几个问题：

* 你确定所有 XML 代码是有意添加的，而不是不小心点击造成的？
* 你确定所有这些` x86/x64/Win32`，`Debug/Release`配置是以正确的顺序连接到一起，并且你没有破坏一些东西吗？
* 你确定上面所有的magic number 不是在你执行重要脚本之后从你的环境变量中读取的，或者事实上是一些密钥、令牌或者密码？
* 你认为这个文件的冲突很容易解决吗？

　　幸运地是，我们有 CMake，它可以以一种整洁的方式帮助我们。我们还没涉及任何 CMake 的语法，但是我很确定你能一眼看出改变是什么：

```text
--- /overview/snippets/CMakeLists-old.txt
+++ /overview/snippets/CMakeLists-new.txt
@@ -2,3 +2,4 @@
 project(foo)
 
 add_executable(foo foo.cpp)
+add_executable(bar bar.cpp)
```

### 3. 试验

　　即使你的团队开始并没有计划使用一些原生工具，但是将来可能会发生变化。比如，你之前在使用`Makefile`，现在想尝试`Ninja`。
你会怎么办？手动转换？找到转换工具？从头开始写一个转换工具？还是从头开始写`Ninja`的配置？使用 CMake，你可以将
`cmake -G 'Unix Makefiles`改成`cmake -G Ninja`，完成！

　　这也有助于新 IDE 的开发人员。当开发人员决定使用你的`SuperDuperIDE`而不是他们最喜欢的 IDE时候，他们可能编写了无数的
`SuperDuperIDE <-> Xcode`, `SuperDuperIDE <-> Visual Studio`等转换器，相比于让你的IDE用户陷入该境地，你所需要做的
就是将一个新的生成器`-G SuperDuperIDE`添加到 CMake 即可。

### 4. 工具包

　　Cmake 是一个工具包（Family of tools），可以在开发者的所有阶段（`sources for developers` -> `quality control` -> 
`installers for users`）提供帮助。下面的行为图展示了`CMake`、`CTest`和`CPack `的连接。

![](/assets/img/cmake-native-build-add.png)

## 三、CMake的特点

　　通过上述 CMake 可以做什么的理解，可以总结CMake的特点如下：

* 人类可读的配置
* 兼容所有工具的单一配置
* 跨平台跨工具的友好开发
* 不强制改变你最喜欢的构建工具/IDE
* VCS 友好的开发
* 简单的试验
* 轻松开发新的 IDE

## 四、CMake的 Workflow

### 1. 从 Hello World 说起

　　先引入一个`Hello World`，对应 CMake 的 Workflow。编写 CMakeList.txt 文件如下：

```
# CMakeLists.txt

cmake_minimum_required(VERSION 2.8)
project(foo)

add_executable(foo foo.cpp)

message("Processing CMakeLists.txt")
```

执行`cmake -H. -B_builds`命令：

```
$ cmake -H. -B_builds
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
Processing CMakeLists.txt
-- Configuring done
-- Generating done
-- Build files have been written to: /.../minimal-with-message/_builds
```

　　可以看到上面命令行输出有几个关键字`Processing`，`Configuring`，`Generating`，`Build`，这便是 CMake 的执行过程了。

### 2. Workflow

![](/assets/img/cmake-workflow.png){: .img-large}

　　从上图中，可看出每一步的输出，`Configurting`阶段生成了`CMakeCache.txt`文件，`generating`阶段生成了构建文件
（比如`Makefile`），而后，在`Build`阶段构建工具则可根据构建文件，生成二进制文件。

### 3. 外部构建和内部构建
* 外部构建
　　注意上述的命令指定了`-B_builds`，生成的文件全部都在`_builds`下。`out-of-source build`，分离了源文件和生成的二进制文件，
也是比较推荐的做法。

* 内部构建
　　`in-source build`，如果不指定输出目录，默认情况下会将生成的文件防到和源文件目录下，如果有多个子模块，则每个子模块的
二进制文件都在自己的子模块下，和源文件一起。

Note:
　　外部构建不仅仅是指定`-B_builds`，同时记得要将任何自动生成的文件放到`_builds`里面。比如，C++模板文件`myproject.h.in`，
该文件用于生成`myproject.h`，因此记得要将`myproject.h.in`放至源文件下，而将`myproject.h`放至二进制文件下。

## 五、一些概念
　　在了解 CMake 之前，我经常听到`make` ，`ninja`，`makefile`,`CMakeList.txt`，`CMake`，`CMakeCache.txt`这些词，
下面统一说明一下：

　　`CMakeList.txt`是一个列表文件，它是当前源目录的入口。`CMake`会从源代码树的顶层`CMamkeList.txt`文件读起，
并继续读取通过`add_subdirectory`指令添加的其他子依赖的`CMamkeList.txt`文件。

　　为了优化，存在有一种特殊的变量，它的生命周期不限于一个CMake运行（可能含有子模块）。该变量保存在`CMakeCache.txt`文件中，
并保存在项目构建树的多个运行中。

1. CMake生成器

```
$ cmake --help
...

Generators
  Unix Makefiles               = Generates standard UNIX makefiles.
  Ninja                        = Generates build.ninja files.
  Xcode                        = Generate Xcode project files.
  CodeBlocks - Ninja           = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
  CodeLite - Ninja             = Generates CodeLite project files.
  CodeLite - Unix Makefiles    = Generates CodeLite project files.
  Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                               = Generates Sublime Text 2 project files.
  Kate - Ninja                 = Generates Kate project files.
  Kate - Unix Makefiles        = Generates Kate project files.
  Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.
  KDevelop3                    = Generates KDevelop 3 project files.
  KDevelop3 - Unix Makefiles   = Generates KDevelop 3 project files.
```

　　可以看到，最上面的三个分别是`Unix Makefiles`、`ninja`和`Xcode`。

* 当生成器选择的是`Unix Makefiles`时候，CMake会生成`Makefile`文件，`make` 是与 `Makefile` 对应的，`make`读取`Makefile`文件，
根据源代码自动生成可执行的程序或者库。
* 当生成器选择的是`ninjas`时候，CMake会生成`build.ninja`文件，`ninja`是与`build.ninja`对应的，`ninjia`读取`build.ninja`
文件里的构建规则进行构建。

2. 编译器

　　原生构建工具只会编排我们的构建，我们还需要有编译器才能从我们的C++源中生成二进制文件。
* Visual Studio
　　`Visual Studio`的编译器compiler (也就是cl.exe)伴随着 IDE 一起被安装，无其他额外操作。
* Ubuntu GCC
　　`gcc`编译器通常是`Linux OS`上使用的
* OSX Clang
　　`Clang`编译器通常伴随着`Xcode`一起安装的，或者是在安装`make`的时候被安装。

3. 生成原生构建工具文件

　　我们可以使用 GUI 或者 CMake的命令行版本生成原生文件：
* GUI: Visual Studio
* GUI: Xcode
* GUI: cmake-gui
* CLI: Visual Studio
```
cmake -H. -B_builds -G "Visual Studio 14 2015 Win64"
```
* CLI: Xcode
```
cmake -H. -B_builds -GXcode
```
* CLI: Cake
```
cmake -H. -B_builds -G "Unix Makefiles"
```

Note:

* `-H `, `-H<path-to-source-tree>`，无空格，必须和`-B`一起使用，该路径会被存在变量` CMAKE_SOURCE_DIR`里
* `-B` , `-B<path-to-binary-tree>`，无空格，必须和`-H`一起使用，该路径会被存在变量`CMAKE_BINARY_DIR`里

## 最后
　　本篇文章不在于教你如何使用 CMake，CMake 是一个工具，相应的教程，手册可以很容易找到。因此，本文主要在于理解 CMake 
的特点及其 workflow，这对于之后使用 CMake 的时候很有帮助。

## Reference
* [CGold: The Hitchhiker’s Guide to the CMake](https://cgold.readthedocs.io/en/latest/index.html)
* [Cmake 官网教程](https://cmake.org/cmake-tutorial/)
* [Cmake 参考手册 - devdocs.io](http://devdocs.io/cmake~3.6/manual/cmake-buildsystem.7#introduction)
