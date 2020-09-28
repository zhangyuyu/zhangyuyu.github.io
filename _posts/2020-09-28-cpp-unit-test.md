---
layout: post
title: "C++单元测试指引"
date: 2020-09-28
categories: 测试
tags:
- 测试
comments: true
---

*  目录
{:toc}

# 前言

　　在上一篇[GoLang单元测试指引](https://zhangyuyu.github.io/golang-unit-test/) 中，
首先介绍了单元测试的意义和编写单元测试的一般方法。作为同系列出品，本篇文章则主要介绍如何在C++中写单元测试。

　　本文直接从常用的C++单元测试框架出发，分别对几种框架进行了简单的介绍和小结，然后介绍了Mock的框架，
并以具体代码示例进行说明，最后列举了一些常见问题。

# 一、常用C++单测框架
　　常用的C++单测对比如下：

|Google Test|Catch 2|CppUTest|
|:--- |:--- |:--- |
|特点|<ul><li>成熟、兼容性好</li><li>简洁、有效率</li><li>常用、学习资源多</li></ul>|<ul><li>框架只有一个<code>catch.hpp</code>、集成轻松</li><li>有Given-When-Then分区，适合BDD行为驱动开发</li><li>无自带Mock框架</li></ul>|<ul><li>可以检测内存泄露</li><li>输出更简洁</li><li>适合在嵌入式系统项目中使用</li></ul>|
|Mock框架|Google Mock|无自带Mock框架|CppUMock|
|推荐指数|★★★★★|★★★☆☆|★★☆☆☆|

　　一般情况下，我们推荐使用Google Test搭配Google Mock。如果项目有特殊需求或更适合其他框架，也可以考虑。

　　根据实际使用频率，在以下部分，Google Test和Google Mock的介绍更为详细；对于其他框架，这里介绍它们的主要特点，
具体使用方法，可以查阅各自文档。

# 二. Google Test
　　Google Test是目前比较成熟而且最常用的C++单元测试框架之一。

## 1. 基本概念
　　**断言（Assertions）** 是检查条件是否为真的语句。断言的结果可能是*成功*或者*失败*，
而失败又分为*非致命失败*或*致命失败*。如果发生*致命失败*，测试进程将中止当前运行，否则它将继续运行。

　　**测试（Test）** 使用断言来验证被测试代码的行为。如果测试崩溃或断言失败，则测试失败；否则测试成功。

　　**测试套件（Test Suite）** 包含一个或多个**测试（Test）**。当测试套件中的多个测试需要共享通用对象和子例程时，
可以将它们放入**测试夹具（Test Fixture）**。

　　**测试程序（Test Program）** 可以包含多个测试套件。

## 2. 断言
　　Google Test中，**断言（Assertions）** 是类似函数调用的**宏**。断言失败时，googletest会输出断言的源文件和
行号位置以及失败消息；我们还可以提供自定义失败消息，该消息将附加到googletest消息中。

　　断言成对出现（`ASSERT_*`和`EXPECT_*`），它们测试的对象相同，但对当前运行有不同的影响。`ASSERT_*`版本失败时
会产生致命故障，并中止当前函数（不一定是整个TEST）运行。`EXPECT_*`版本会产生非致命故障，不会停止当前函数运行。
通常`EXPECT_*`是首选，因为可以在测试中报告多个故障。但是如果在断言失败时继续执行没有意义，则应使用`ASSERT_*`。

　　要提供自定义失败消息，只需使用`<<` 运算符或此类运算符的序列将其流式传输到宏中即可 。一个例子：
```
ASSERT_EQ(x.size(), y.size()) << "x和y长度不同";

for (int i = 0; i < x.size(); ++i) {
	EXPECT_EQ(x[i], y[i]) << "x和y元素存在不同：" << i;
}
```
　　以下是一些最常用的断言，如果需要查阅其他断言，可以前往googletest的官方文档。

### 2.1. 基本断言
　　`condition` 是返回`true`/`false`的变量、布尔表达式、函数调用等，以下断言对其进行验证。

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_TRUE(condition);|EXPECT_TRUE(condition);|condition为真|
|ASSERT_FALSE(condition);|EXPECT_FALSE(condition);|condition为假|
　　例如：在`ASSERT_TRUE(condition)`中，当`condition`为`true`时，符合断言，不影响执行；当`condition`
为`false`时，不符合断言，且由于是`ASSERT`，当前执行中断。

### 2.2. 普通比较型断言
　　`val1`和`val2`是两个可用`==`、`!=`、`>`、`<`等运算符进行比较的值，以下断言对其进行比较。

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_EQ(val1, val2);|EXPECT_EQ(val1, val2);|val1 == val2|
|ASSERT_NE(val1, val2);|EXPECT_NE(val1, val2);|val1 != val2|
|ASSERT_LT(val1, val2);|EXPECT_LT(val1, val2);|val1 < val2|
|ASSERT_LE(val1, val2);|EXPECT_LE(val1, val2);|val1 <= val2|
|ASSERT_GT(val1, val2);|EXPECT_GT(val1, val2);|val1 > val2|
|ASSERT_GE(val1, val2);|EXPECT_GE(val1, val2);|val1 >= val2|

　　例如：在`ASSERT_GT(val1, val2)`中，只有当`val1 > val2`时，符合断言，不影响执行；当`val1 <= val2`时，
不符合断言，且由于是`ASSERT`，当前执行中断。

### 2.3. C字符串比较型断言
　　`str1`和`str2`是两个C字符串，以下断言对它们的值进行比较；如果要比较两个`std::string`对象，直接用之前
提到的`EXPECT_NE`，`EXPECT_NE`等。

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_STREQ(str1,str2);|EXPECT_STREQ(str1,str2);|这两个C字符串具有相同的内容|
|ASSERT_STRNE(str1,str2);|EXPECT_STRNE(str1,str2);|两个C字符串的内容不同|
|ASSERT_STRCASEEQ(str1,str2);|EXPECT_STRCASEEQ(str1,str2);|忽略大小写，两个C字符串的内容相同|
|ASSERT_STRCASENE(str1,str2);|EXPECT_STRCASENE(str1,str2)|忽略大小写，两个C字符串的内容不同|

　　例如：`char *str1 = "ABC";``char *str2 = "ABC";`，`EXPECT_STREQ(str1, str2);`断言通过，
因为它们的内容一样；而`EXPECT_EQ(str1, str2);`断言失败，因为它们的地址不一样。

　　注意：一个`NULL`指针和一个空字符串`""`是不同的。

### 2.4. 浮点数比较型断言
　　`val1`和`val2`是两个浮点数，以下断言对其进行比较。

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_FLOAT_EQ(val1, val2);|EXPECT_FLOAT_EQ(val1, val2);|这两个float值几乎相等|
|ASSERT_DOUBLE_EQ(val1, val2);|EXPECT_DOUBLE_EQ(val1, val2);|这两个double值几乎相等|

　　以下断言可以选择可接受的误差范围：

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_NEAR(val1, val2, abs_error);|EXPECT_NEAR(val1, val2, abs_error);|val1和val2的差的绝对值不超过abs_error|

### 2.5. 明确的成功和失败

- 明确生成成功：
	- `SUCCEED();` 生成一个成功，但这不代表整个测试就成功了。
- 明确生成失败：
	- `FAIL();` 生成致命错误
	- `ADD_FAILURE();` 生成非致命错误。
	- `ADD_FAILURE_AT("file_path",line_number);` 生成非致命错误，输出文件名和行号。

例如：
```
if(condition) {
	SUCCEED();
} else{
	FAIL();
}
```
　　效果上等同于

```
ASSERT_TRUE(condition);
```
　　只是`ASSERT_TRUE`失败时可以输出`condition`的具体值。当但我们需要验证的`condition`很复杂时，
或者需要很多个`if..else...`分支来验证彼此互斥的情况以保证覆盖到每一种可能性时，`SUCCEED()`、`FAIL()`等
明确的成功/失败可能是更好的选择。

### 2.6. 异常断言
　　这些断言验证一段代码（`statement`）是否抛出（或不抛出）给定类型的异常：

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_THROW(statement, exception_type);|EXPECT_THROW(statement, exception_type);|statement抛出给定类型的异常|
|ASSERT_ANY_THROW(statement);|EXPECT_ANY_THROW(statement);|statement抛出任何类型的异常|
|ASSERT_NO_THROW(statement);|EXPECT_NO_THROW(statement);|statement不抛出任何异常|

### 2.7. 使用已有布尔函数
　　当`predN`是一个有`N`个参数，返回布尔值的函数时，以下断言可以获取更好的错误信息。

|失败时中断执行的断言|失败时不中断执行的断言|断言成功情况|
|--- |--- |--- |
|ASSERT_PRED1(pred1, val1);|EXPECT_PRED1(pred1, val1);|pred1(val1)为真|
|ASSERT_PRED2(pred2, val1, val2);|EXPECT_PRED2(pred2, val1, val2);|pred2(val1, val2)为真|
|...|...|...|

　　例如：`isComparable(Object o1, Object o2)`是一个返回布尔值的函数。我们可以有以下选择，
都能达到验证函数调用结果的目的：
1. `ASSERT_TRUE(isComparable(obj1, obj2));`
2. `ASSERT_PRED2(isComparable, obj1, obj2);`
　　区别在于：当断言失败时，`ASSERT_TRUE`只会告知函数最后的返回值是`false`；而`ASSERT_TRUE`同时
会输出`val1`、`val2`的值。

## 3. 测试

　　创建一个测试的步骤：
1. 使用`TEST()`宏定义和命名测试功能。
2. 在`TEST()`宏内，构造出达到测试状态的函数、变量
3. 使用断言指定函数、变量期望的返回值、值。
```
TEST(MessageTestSuite, BodyLengthNegative) {
	... 构造 ...
	... 断言 ...
}
```

　　`TEST()`第一个参数是Test Suite的名称，第二个参数是Test Suite内的Test名称。这两个名称都必须是有效的
C++标识符，并且它们不应包含任何下划线（`_`）。测试的全名包括Test Suite名和Test名。来自不同Test Suite的
测试可以具有相同的Test名。它们都不是变量，也不是字符串。

　　在上面的例子中，这个测试的名称是`BodyLengthNegative`，Test Suite的名称是`MessageTestSuite`。

## 4. 测试夹具：多个测试有共有的数据配置

　　如果多个测试有共有的数据配置，可以使用**测试夹具（Test Fixture）**将共用部分提取出来重复利用。

　　要创建一个测试夹具：
1. 创建一个继承`::testing::Test`的类。从`protected`开始这个类，因为我们要从子类访问夹具成员。 
2. 在类内部，声明计划使用的任何对象。 
3. 如有必要，编写默认的`constructor`或`SetUp()`函数为每个测试准备对象。
4. 如有必要，编写一个`destructor`或`TearDown()`函数以释放在`SetUp()`中分配的任何资源。
5. 如有必要，定义一些共享的类函数

　　当使用测试夹具是，需要使用`TEST_F()`而不是`TEST()`

```
class TestFixtureName : public ::testing::Test {
protected:
	virtual void SetUp() {
		...
	}
	virtual void TearDown() {
		...
	}
	virtual int SomeFunction() {
		...
	}
	SomeObject object;
};

TEST_F(TestFixtureName, TestName1) {
	... 构造 ...
	... 断言 ...
}

TEST_F(TestFixtureName, TestName2) {
	... 构造 ...
	... 断言 ...
}
```

　　那上面这个例子来说，对于每个`TEST_F()`测试，googletest将在运行时
1. 创建一个新的**测试夹具（Test Fixture）**对象
2. 通过`SetUp()`对其进行初始化
3. 运行该`TEST_F()`测试
4. 通过调用进行清理`TearDown()`
5. 然后删除该**测试夹具（Test Fixture）**对象

　　所以，虽然多个`TEST_F`共用同一部分代码，但共同代码会每个`TEST_F`都独立执行一次。同一测试套件中的不同测试具有不同的测试夹具对象。一个测试对**测试夹具**所做的任何更改均不会影响其他测试。

# 三、Catch 2

　　Catch2 仅有头部文件（header only），所以它的第一个优点是可以轻易地放入任何项目中进行使用。只需要 `#include "catch.hpp"` 就可以在当前文件使用 Catch

## 1. REQUIRE

　　Catch的基础使用方法也很简单。

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"
unsigned int Factorial( unsigned int number ) {
	return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {   
	REQUIRE( Factorial(0) == 1 );    
	REQUIRE( Factorial(1) == 1 );    
	REQUIRE( Factorial(2) == 2 );    
	REQUIRE( Factorial(3) == 6 );    
	REQUIRE( Factorial(10) == 3628800 );
}
```

## 2. SECTIONS

　　Catch的SECTION相当于GTEST里夹具（fixture）的功能。对每一个SECTION，TEST_CASE 都从头开始执行。

```
TEST_CASE( "vectors can be sized and resized", "[vector]" ) {
    std::vector<int> v( 5 );    
    REQUIRE( v.size() == 5 );    
    REQUIRE( v.capacity() >= 5 );    
    SECTION( "resizing bigger changes size and capacity" ) 
        v.resize( 10 );        
        REQUIRE( v.size() == 10 );        
        REQUIRE( v.capacity() >= 10 );    
    }    
    SECTION( "resizing smaller changes size not capacity" ){
        v.resize( 0 );        
        REQUIRE( v.size() == 0 );        
        REQUIRE( v.capacity() >= 5 );    
    }    
    SECTION( "reserving bigger changes capacity not size" ) { 
        v.reserve( 10 );        
        REQUIRE( v.size() == 5 );        
        REQUIRE( v.capacity() >= 10 );    
    }    
    SECTION( "reserving smaller does not change size" ) {
        v.reserve( 0 );        
        REQUIRE( v.size() == 5 );        
        REQUIRE( v.capacity() >= 5 );    
    }
}
```

## 3. 标签

　　Catch提供标签特性。

```
TEST_CASE( "A", "[widget]" ) { /* ... */ }
TEST_CASE( "B", "[widget]" ) { /* ... */ }
TEST_CASE( "C", "[gadget]" ) { /* ... */ }
TEST_CASE( "D", "[widget][gadget]" ) { /* ... */ }
```

* `"[widget]"` 选取 A、B、D. 
* `"[gadget]"` 选取 C、D. 
* `"[widget][gadget]"` 只选取 D
* `"[widget],[gadget]"` 所有A、B、C、D.
* 还有一些特殊标签指定特殊行为

## 4. 特点总结

- 框架只有一个<code>catch.hpp</code>、集成轻松
- 有Given-When-Then分区，适合BDD行为驱动开发
- 无自带Mock框架

# 四、CppUTest

## 1. main和test

main.cpp:

```
#include "CppUTest/CommandLineTestRunner.h"
int main(int ac, char** av){    
	return CommandLineTestRunner::RunAllTests(ac, av);
}
```

test.cpp:

```
#include "CppUTest/TestHarness.h"
TEST_GROUP(FirstTestGroup){
    void setup(){      
        // Init stuff   
    } 
    void teardown(){      
        // Uninit stuff   
    }
};

TEST(FirstTestGroup, FirstTest){
    FAIL("Fail me!");
}
TEST(FirstTestGroup, SecondTest){
    STRCMP_EQUAL("hello", "world");
}
```

## 2. 断言

- `CHECK(boolean condition)`检查任何布尔结果。
- `CHECK_TEXT(boolean condition, text)`检查任何布尔结果，并在失败时输出文本。
- `CHECK_FALSE(condition)`检查任何布尔结果
- `CHECK_EQUAL(expected, actual)`使用`==`检查实体之间的相等性。因此，如果有一个支持`operator==()`的类，则可以使用此宏比较两个实例。
- `CHECK_COMPARE(first, relop, second)`检查在两个实体之间是否存在关系运算符。失败时，打印两个操作数求和的结果。
- `CHECK_THROWS(expected_exception, expression)`检查表达式是否抛出`expected_exception`（例如`std::exception`）。`CHECK_THROWS`仅在使用标准C ++库（默认）构建CppUTest时可用。
- `STRCMP_EQUAL(expected, actual)`使用`strcmp()`检查`const char *`字符串是否相等。
- `STRNCMP_EQUAL(expected, actual, length)`使用`strncmp()`检查`const char *`字符串是否相等。
- `STRCMP_NOCASE_EQUAL(expected, actual)`不考虑大小写，检查`const char *`字符串是否相等。

## 3. 特点
- 可以检测内存泄露
- 输出更简洁
- 使用在嵌入式系统项目中使用

# 五、Google Mock

　　Google Mock一般来说和Google Test搭配使用，但Google Test也可以和其他Mock框架一起使用。
本部分是Google Mock基础常用的用法，如需要特殊用法，请查阅Google Mock官方文档。

## 1. Fake、Mock、Stub

- Fake对象**有**具体的实现，但采取一些捷径，比如用内存替代真实的数据库读取。
- Stub对象**没有**具体的实现，只是返回提前准备好的数据。
- Mock对象和Stub类似，只是在测试中需要调用时，针对某种输入指定期望的行为。Mock和Stub的区别是，
Mock除了返回数据还可以指定期望以验证行为。

## 2. 简单例子：Mock Turtle

Turtle类:

```
class Turtle {
	...
	virtual ~Turtle() {};
	virtual void PenUp() = 0;
	virtual void PenDown() = 0;
	virtual void Forward(int distance) = 0;
	virtual void Turn(int degrees) = 0;
	virtual void GoTo(int x, int y) = 0;
	virtual int GetX() const = 0;
	virtual int GetY() const = 0;
};
```
MockTurtle类:

```
#include "gmock/gmock.h"

class MockTurtle : public Turtle {
public:
	...
	MOCK_METHOD(void, PenUp, (), (override));
	MOCK_METHOD(void, PenDown, (), (override));
	MOCK_METHOD(void, Forward, (int distance), (override));
	MOCK_METHOD(void, Turn, (int degrees), (override));
	MOCK_METHOD(void, GoTo, (int x, int y), (override));
	MOCK_METHOD(int, GetX, (), (const, override));
	MOCK_METHOD(int, GetY, (), (const, override));
};
```

　　创建Mock类的步骤：

1. `MockTurtle`继承`Turtle`
2. 找到`Turtle`的一个虚函数
3. 在`public:`的部分中，写一个`MOCK_METHOD();`
4. 将虚函数函数签名复制进`MOCK_METHOD();`中，加两个逗号：一个在返回类型和函数名之间另一个在函数名和参数列表之间
		
	例如：`void PenDown()`有三部分：`void`、`PenDown`和`()`，这三部分就是`MOCK_METHOD`的前三个参数
		
5. 如果要模拟`const`方法，添加一个包含`(const) `的第4个参数（必须带括号）。 
6. 建议添加`override`关键字。所以对于`const`方法，第四个参数变为`(const, override)`，对于非`const`方法，第四个参数变为`(override)`。这不是强制性的。
7. 重复步骤直至完成要模拟的所有虚拟函数。

## 3. 在测试中使用Mock

　　在测试中使用Mock的步骤：

1. 从`testing`名称空间导入`gmock.h`的函数名（每个文件只需要执行一次）。
2. 创建一些Mock对象。
3. 指定对它们的期望（方法将被调用多少次？带有什么参数？每次应该做什么（对参数做什么、返回什么值）？等等）。
4. 使用Mock对象；可以使用googletest断言检查结果。如果mock函数的调用超出预期或参数错误，将会立即收到错误消息。
5. 当Mock对象被销毁时，gMock自动检查对模拟的所有期望是否得到满足。

```
#include "path/to/mock-turtle.h"
#include "gmock/gmock.h"
#include "gtest/gtest.h"

using ::testing::AtLeast;                         	// #1

TEST(PainterTest, CanDrawSomething) {
	MockTurtle turtle;                              // #2
	EXPECT_CALL(turtle, PenDown())                  // #3
		.Times(AtLeast(1));

	Painter painter(&turtle);                       // #4

	EXPECT_TRUE(painter.DrawCircle(0, 0, 10));      // #5
}
```

　　在这个例子中，我们期望`turtle`的`PenDown()`至少被调用一次。如果在`turtle`对象被销毁时，`PenDown()`还没有被调用或者调用两次或以上，测试会失败。

## 4. 指定期望

`EXPECT_CALL`（指定期望）是使用Google Mock的核心。`EXPECT_CALL`的作用是两方面的：

1. 告诉这个Mock（假）方法如何模仿原始方法：

	我们在`EXPECT_CALL`中告诉Google Mock，某个对象的某个方法被第一次调用时，会修改某个参数，会返回某个值；第二次调用时，会修改某个参数，会返回某个值.......
	
2. 验证被调用的情况

	我们在`EXPECT_CALL`中告诉Google Mock，某个对象的某个方法总共会被调用N次（或大于N次、小于N次）。如果最终次数不符合预期，会导致测试失败。

### 4.1. 基本语法

```
EXPECT_CALL(mock_object, method(matchers))
	.Times(cardinality)
	.WillOnce(action)
	.WillRepeatedly(action);
```

* `mock_object` 是对象
* `method(matchers)` 用于匹配相应的函数调用
* `cardinality` 指定基数（被调用次数情况）
* `action` 指定被调用时的行为

例子：

```
using ::testing::Return;
...
EXPECT_CALL(turtle, GetX())
	.Times(5)
	.WillOnce(Return(100))
	.WillOnce(Return(150))
	.WillRepeatedly(Return(200));
```

　　这个`EXPECT_CALL()`指定的期望是：在`turtle`这个Mock对象销毁之前，`turtle`的`getX()`函数会被调用五次。第一次返回`100`，第二次返回`150`，第三次及以后都返回`200`。指定期望后，5次对`getX`的调用会有这些行为。但如果最终调用次数不为5次，则测试失败。

### 4.2. 参数匹配：哪次调用

```
using ::testing::_;
using ::testing::Ge;
// 只与Forward(100)匹配
EXPECT_CALL(turtle, Forward(100));
// 与GoTo(x,y)匹配, 只要x>=50
EXPECT_CALL(turtle, GoTo(Ge(50), _));
```

* `_`相当于“任何”。
* `100`相当于`Eq(100)`。
* `Ge(50)`指参数大于或等于50。
* 如果不关心参数，只写函数名就可以。比如`EXPECT_CALL(turtle, GoTo);`。

### 4.3. 基数：被调用几次
　　用`Times(m)`，`Times(AtLeast(n))`等来指定期待的调用次数。

　　`Times`可以被省略。比如整个`EXPECT_CALL`只有一个`WillOnce(action)`相当于也说明了调用次数只能为1。

### 4.4. 行为：该做什么
　　常用模式：如果需要指定前几次调用的特殊情况，并且之后的调用情况相同。使用一系列`WillOnce()`之后有`WillRepeatedly()`

　　除了用来指定调用返回值的`Return()`，Google Mock中常用行为中还有：`SetArgPointee<N>(value)`，
`SetArgPointee`将第`N`个指针参数（从0开始）指向的变量赋值为`value`。

　　比如`void getObject(Object* response){...}`的`EXPECT_CALL`：

```
Object* a = new Object;
EXPECT_CALL(object, request)
	.WillOnce(SetArgPointee<1>(*a));
```
　　就修改了传入的指针`response`，使其指向了一个我们新创建的对象。

　　如果有多个行为，应该使用`DoAll(a1, a2, ..., an)`。`DoAll`执行所有`n`个action并返回`an`的结果。

### 4.5. 使用多个预期

例子：

```
using ::testing::_;
...
EXPECT_CALL(turtle, Forward(_))		// #1
	.Times(3);  	
EXPECT_CALL(turtle, Forward(10))  	// #2
	.Times(2);
...mock对象函数被调用...
	//Forward(10);						// 与#2匹配
	//Forward(20);						// 与#1匹配
```

　　正常情况下，Google Mock以倒序搜索预期：如果和多个`EXPECT_CALL`都可以匹配，只有之前的，
距离调用最近的一个`EXPECT_CALL()`会被匹配。例如：

* 连续三次调用`Forward(10)`会生错误因为它和#2匹配。
* 连续三次调用`Forward(20)`不会有错误因为它和#1匹配。

　　一旦匹配，该预期会被一直绑定，即使执行次数达到上限之后，还是是生效的，这就是为什么三次调用
`Forward(10)`超过了2号`EXPECT_CALL`的上限时，不会去试图绑定1号`EXPECT_CALL`而是报错的原因。

　　为了明确地让某一个`EXPECT_CALL`“退休”，可以加上`RetiresOnSaturation()`，例子：

```
using ::testing::Return;

EXPECT_CALL(turtle, GetX())		// #1
	.WillOnce(Return(10))
	.RetiresOnSaturation();
EXPECT_CALL(turtle, GetX())		// #2
	.WillOnce(Return(20))
	.RetiresOnSaturation();

turtle.GetX()					// 与#2匹配，返回20，然后#2“退休”
turtle.GetX()					// 与#1匹配，返回10
```

　　在这个例子中，第一次`GetX()`调用和#2匹配，返回`20`，然后这个`EXPECT_CALL`就“退休”了；
第二次`GetX()`调用和#1匹配，返回`10`

### 4.6. Sequence
　　可以用sequence来指定期望匹配的顺序。

```
using ::testing::Return;
using ::testing::Sequence;
Sequence s1, s2;
...
EXPECT_CALL(foo, Reset())
    .InSequence(s1, s2)
    .WillOnce(Return(true));
EXPECT_CALL(foo, GetSize())
    .InSequence(s1)
    .WillOnce(Return(1));
EXPECT_CALL(foo, Describe(A<const char*>()))
    .InSequence(s2)
    .WillOnce(Return("dummy"));
```
![](/assets/img/2020/20200928-cpp-sequence.png)

　　在上面的例子中，创建了两个Sequence `s1`和`s2`，属于`s1`的有`Reset()`和`GetSize()`，
所以`Reset()`必须在`GetSize()`之前执行。属于`s2`的有`Reset()`和`Describe(A<const char*>())`，
所以`Reset()`必须在`Describe(A<const char*>())`之前执行。所以，`Reset()`必须在`GetSize()`
和`Describe()`之前执行。而`GetSize()`和`Describe()`这两者之间没有顺序约束。

　　如果需要指定很多期望的顺序，有另一种用法：

```
using ::testing::InSequence;
{
  InSequence seq;

  EXPECT_CALL(...)...;
  EXPECT_CALL(...)...;
  ...
  EXPECT_CALL(...)...;
}
```
　　在这种用法中，scope中（大括号中）的期望必须遵守严格的顺序。


## 5. 更多

* [Google Mock Cheat Sheet](https://github.com/google/googletest/blob/master/googlemock/docs/cheat_sheet.md)
* [Google Mock Cook Book](https://github.com/google/googletest/blob/master/googlemock/docs/cook_book.md)

# 六、情景示例

　　在这部分，我们用一个[示例项目](https://git.code.oa.com/catonzhong/cpp-ut)来演示，如何在不同情景中使用
Google Test和Google Mock写单元测试用例。

## 1. 项目结构

　　示例项目是一个C++命令行聊天室软件，包含服务器和客户端。

```text
.
├── CMakeLists.txt
├── README.md
├── client_main.cpp
├── server_main.cpp
├── include
│   ├── chat_client.hpp
│   ├── chat_message.hpp
│   ├── chat_participant.hpp
│   ├── chat_room.hpp
│   ├── chat_server.hpp
│   ├── chat_session.hpp
│   ├── http_request.hpp
│   ├── http_request_impl.hpp
│   ├── message_dao.hpp
│   └── message_dao_impl.hpp
├── src
│   ├── chat_client.cpp
│   ├── chat_message.cpp
│   ├── chat_room.cpp
│   ├── chat_server.cpp
│   ├── chat_session.cpp
│   ├── http_request_impl.cpp
│   └── message_dao_impl.cpp
└── tests
    ├── chat_message_unittest.cpp
    └── chat_room_unittest.cpp
```

## 2. 普通测试

　　如果被测试的函数不包含外部依赖，用Google Test基础的用法就可以完成用例编写。

原函数：

```
void chat_message::body_length(std::size_t new_length) {
    body_length_ = new_length;
    if (body_length_ > 512)
        body_length_ = 512;
}
```

　　这个函数很简单。就是给`body_length_`赋值但是有最大值限制。测试用例可以这样写：

```
TEST(ChatMessageTest, BodyLengthNegative) {
    chat_message c;
    c.body_length(-50);
    EXPECT_EQ(512, c.body_length());
}

TEST(ChatMessageTest, BodyLength0) {
    chat_message c;
    c.body_length(0);
    EXPECT_EQ(0, c.body_length());
}

TEST(ChatMessageTest, BodyLength100) {
    chat_message c;
    c.body_length(100);
    EXPECT_EQ(100, c.body_length());
}

TEST(ChatMessageTest, BodyLength512) {
    chat_message c;
    c.body_length(512);
    EXPECT_EQ(512, c.body_length());
}

TEST(ChatMessageTest, BodyLength513) {
    chat_message c;
    c.body_length(513);
    EXPECT_EQ(512, c.body_length());
}
```

　　我们可以看到，对于这类函数，用例编写很直接简单，步骤都是构造变量，再用合适的Google Test的
宏来验证变量值或者函数调用返回值。

## 3. 简单 Mock

原函数

```
void chat_room::leave(chat_participant_ptr participant) {
    participants_.erase(participant);
}
```
　　`participants_ ` 的类型是 `std::set<chat_participant_ptr>`。这个函数的目的很明显，将一个`participant`从`set`中移除。

　　真实地创建一个聊天参与者`participant`对象可以条件比较苛刻或者成本比较高。为了有效率地验证这个函数，我们可以新建一些Mock的`chat_participant_ptr`而不用严格地去创建真实的`participant`对象。

`chat_participant`对象：

```
class chat_participant {
public:
    virtual ~chat_participant() {}
    virtual void deliver(const chat_message &msg) = 0;
};
```

Mock对象：

```
class mock_chat_participant : public chat_participant {
public:
    MOCK_METHOD(void, deliver, (const chat_message &msg), (override));
};
```

测试用例：

```
TEST(ChatRoomTest, leave) {
    auto p1 = std::make_shared<mock_chat_participant>();	//新建第一个Mock指针
    auto p2 = std::make_shared<mock_chat_participant>();	//新建第二个Mock指针
    auto p3 = std::make_shared<mock_chat_participant>();	//新建第三个Mock指针
    auto p4 = std::make_shared<mock_chat_participant>();	//新建第四个Mock指针
    chat_room cr;											//新建待测试对象chat_room
    cr.join(p1);
    cr.join(p2);
    cr.join(p3);
    cr.join(p4);
    EXPECT_EQ(cr.participants().size(), 4);
    cr.leave(p4);
    EXPECT_EQ(cr.participants().size(), 3);
    cr.leave(p4);
    EXPECT_EQ(cr.participants().size(), 3);
    cr.leave(p2);
    EXPECT_EQ(cr.participants().size(), 2);
}
```

## 4. Web请求

　　`chat_room`中有一个`log()`，依赖网络请求。原函数：

```
std::string chat_room::log() {
    std::string* response;
    this->requester->execute("request",response);		// web访问，结果存在response指针中
    return *response;
}
```

　　在单元测试中，我们只关心被测试部分的逻辑。为了测试这个函数，我们不应该创建真实的`requester`，应该使用mock。

`http_request`对象：

```
class http_request {
public:
    virtual ~http_request(){}
    virtual bool execute(std::string request, std::string* response)=0;
};
```

Mock对象：

```
class mock_http_request : public http_request {
public:
    MOCK_METHOD(bool, execute, (std::string request, std::string * response), (override));
};
```

测试用例：

```
TEST(ChatRoomTest, log) {
    testing::NiceMock<mock_message_dao> mock_dao;	//在下一部分会提到mock_message_dao
    mock_http_request mock_requester;				//Mock对象
    std::string response = "response";				//期待调用函数的第二个参数将指向这个string对象
    EXPECT_CALL(mock_requester, execute)
    	.WillRepeatedly(							//每次调用都会（WillRepeatedly）执行
    		testing::DoAll(							//每次执行包含多个行为
    			testing::SetArgPointee<1>(response),//将传入参数指针变量response指向response
    			testing::Return(true)));			//返回值为true
    chat_room cr 
    	= chat_room(&mock_dao, &mock_requester);	//将mock对象通过chat_room的constructor注入
    EXPECT_EQ(cr.log(),"response");					//调用和Google Test断言
}
```

## 5. 数据库访问

　　`chat_room`对象会将聊天者发送的消息存储在redis数据库中。当新用户加入时，`chat_room`对象从数据库
获取所有历史消息发送给该新用户。

`join()`函数：

```
void chat_room::join(chat_participant_ptr participant) {
    participants_.insert(participant);
    std::vector<std::string> recent_msg_strs = 
    	this->dao->get_messages(); 			//从数据库中获取历史消息
    for (std::string recent_msg_str: recent_msg_strs) {
										    //将每一个消息发送给该聊天参与者	
        auto msg = chat_message();
        msg.set_body_string(recent_msg_str);
        participant->deliver(msg);
    }
}
```

`message_dao`对象：

```
class message_dao {
public:
    virtual ~message_dao(){}
    virtual bool add_message(std::string m)=0;
    virtual std::vector<std::string> get_messages()=0;
};
```

Mock对象：

```
class mock_message_dao : public message_dao {
public:
    MOCK_METHOD(bool, add_message, (std::string m), (override));
    MOCK_METHOD(std::vector<std::string>, get_messages, (), (override));
};
```

测试用例：

```
TEST(ChatRoomTest, join) {
    mock_message_dao mock_dao;				//创建mock对象（需要注入chat_room）
    http_request_impl requester;			//创建web访问对象（也需要注入chat_room）
    auto mock_p1 = std::make_shared<mock_chat_participant>();
    										//创建participant的mock指针
    EXPECT_CALL(mock_dao, get_messages)
    	.WillOnce(testing::Return(std::vector<std::string>{"test_msg_body_1", "test_msg_body_2", "test_msg_body_3"}));
    										//指定get_messages调用的返回值
    EXPECT_CALL(*mock_p1, deliver).Times(3);
    										//指定deliver调用的次数
    chat_room cr = chat_room(&mock_dao, &requester);
    										//创建chat_room对象，注入dao和requester
    cr.join(mock_p1);						//调用
}
```

　　先创建mock对象，再指定函数调用的预期，最后指向被测试函数。我们可以看到，`mock_dao`指定了`get_messages`的
返回值时一个长度为3的vector，所以有3条消息会被deliver。

# 七、FAQ

## 1. 单元测试源文件应该放在项目的什么位置？

　　一般来说，我们会在根目录创建一个`tests`文件夹，里面放单元测试部分的源代码，从而不会和被测试代码混在一起。

　　如果需要和其他测试（如接口测试、压力测试）等区分开来，可以

1. 把`tests`改成`unittests`、`utests`等，或者
2. 在`tests`创建不同子文件夹存放不同类型的测试代码。

## 2. Google Mock只能Mock虚函数，如果我想Mock非虚函数怎么办？

　　由于Google Mock（及其他大部分Mock框架）通过继承来动态重载机制的限制，一般来说Google Mock只能Mock虚函数。如果要mock非虚函数，官方文档提供这几种思路：

1. Mock类和原类没有继承关系，在测试对象使用函数模板。在测试中，测试对象接受Mock类。
2. 创建一个接口（抽象类），原类继承自这个接口（抽象类）。在测试中Mock这个接口（抽象类）。

　　这两种方法，都需要对代码进行一定的修改或重构。如果不想修改被测试代码。可以考虑使用hook技术替换被mock的部分从而mock一般函数。km上有很多文章分享不同的工具和方法。

1. [如何mock一个没有虚函数的类](http://km.oa.com/group/2804/articles/show/426064?kmref=search&from_page=2&no=4)
2. [单元测试：googletest + googlemock + bhook](http://km.oa.com/articles/show/427350?kmref=search&from_page=4&no=1)
3. [单元测试（UT）之GTEST， GMock，GMOCKPLUS](http://km.oa.com/group/11800/articles/show/284251?kmref=search&from_page=1&no=3)
4. [TMock：结合Hook和GMock进行单元测试](http://km.oa.com/group/17627/articles/show/222211?kmref=search&from_page=1&no=5)
5. [单元测试-gmock、tbase、tmock简介](http://km.oa.com/articles/show/459439?kmref=search&from_page=1&no=7)

使用`TMock`对非虚函数mock的例子：

mock函数

```
# include "tmock.h"

class MockClass
{
public:
    //注册mock类
    TMOCK_CLASS(MockClass);
    //声明mock类函数，TMOCK_METHOD{n}第一个参数与attach_func_lib第一个参数相同，其余参考与MOCK_METHOD{n}一致。
    TMOCK_METHOD1("original", original, uint32_t(const char * str_file_md5) )
};
```
　　单测中应用`tmock`的方法和Google Mock基本一致。但在结束的时候需要使用`TMOCK_CLEAR`清除exception，
detach hook的函数，防止干扰其他单元测试。


## 3. Google Test官方文档中说测测试套件名称、测试夹具名称、测试名称中不应该出现下划线`_`。为什么？

　　`TEST(TestSuiteName, TestName)`生成名为`TestSuiteName_TestName_Test`的类。

　　下划线`_`是特殊的，因为C ++保留以下内容供编译器和标准库使用。所以开头和结尾有下划线很容易让生成的类的标识符不合法。

　　另一方面，下划线可能让不同测试生成相同的类。比如`TEST（Time，Flies_Like_An_Arrow）{...} `和`TEST（Time_Flies，Like_An_Arrow）{...} `都生成名为`Time_Flies_Like_An_Arrow_Test`的类。

## 4. 测试输出里有很多`Uninteresting mock function call`警告怎么办？

　　创建的Mock的对象的某些调用如果没有相应匹配的`EXPECT_CALL`，Google Mock会生成这个警告。

　　为了去除这个警告，可以使用`NiceMock`。比如如果原本使用`MockFoo nice_foo;`新建mock对象的话，可以改成`NiceMock<MockFoo> nice_foo;`。`NiceMock<MockFoo>`是`MockFoo`的子类。


# 八、结语

## 1. 实践小结
　　和GoLang单元测试框架有些区别的是，GoLang自生就提供了自带的测试框架，也有第三方框架进行选择。
而C/C++/php等语言的单元测试框架则需要第三方提供和安装。

　　框架的使用，无非是一些语法糖的差异和使用的难易程度。不管使用什么语言，什么框架，最关键的是利用单元测试的思路，
写出解耦的、可测试的、易于维护的代码，保证代码的质量。

　　单元测试是一种手段，能够一定程度的改善生产力。凡事有度过犹不及，如果一味的盲目的追求测试覆盖率，
忽视了测试代码本身的质量，那么各种无效的单元测试反而带来了沉重的维护负担。因此单测的代码，本身也是代码，
也是和项目本身的代码一样，需要重构、维护的（好好写代码）。

## 2. 特别鸣谢
　　感谢实习生钟梓轩，在暑假实习期间，主导整理了C++单测的代码示例和部分文章内容。

## 3. 相关阅读
- [GoLang单元测试指引](https://zhangyuyu.github.io/golang-unit-test/)