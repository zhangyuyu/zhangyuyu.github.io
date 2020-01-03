---
layout: post
title: "函数式编程初探（二）"
date: 2016-11-12 19:35:10
categories: java
tags: 
- java
- RxJava
---

　　[上一篇](http://zhangyuyu.github.io/functional-programming-1/)文章中介绍函数式编程的概念以及三个具有普遍意义的基本构造单元，本篇文章会接着讲述一些柯里化与部分施用、缓存、缓求值、函数式的数据结构。

### 一、柯里化与部分施用
　　柯里化（currying）和函数的部分施用（partial application）都是从数学里借用过来的编程语言技法（基于20世纪haskell Curry等数学家的研究成果）。它们两者都有能力操纵函数的参数条目，一般是通过向一部分参数带入一个或多个默认值的办法来实现的。

　　柯里化指的是从一个多参数函数变成一连串单参数函数的变化。调用者可以决定对多少个参数实施变换，余下的部分将衍生为一个参数数目较少的新函数。如函数process(x,y,z)完全柯里化之后变成process(x)(y)(z)。
部分施用是指提前带入一部分参数值，使一个多参数函数得以省略部分参数，从而转化为一个参数数目较少的函数。如在函数process(x,y,z)上部分施用一个参数，那么我们将得到process(y,z)。

　　Groovy语言中部分施用与柯里化的对比：
```
def volume = {h, w, l -> h * w * l}
def area = volume.curry(1)
def lengthPA = volume.curry(1, 1)         // <1>
def lengthC = volume.curry(1).curry(1)    // <2>

println "The volume of the 2x3x4 rectangular solid is ${volume(2, 3, 4)}"
println "The area of the 3x4 rectangle is ${area(3, 4)}"
println "The length of the 6 line is ${lengthPA(6)}"
```
　　代码可参见《函数式编程思想》的[github](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/groovy/currying.groovy)
柯里化常见的应用场景：函数工厂、模板方法模式、隐藏参数。

### 二、缓存
　　缓存是很常见的一种需求（同时也是制造隐晦错误的源头）。缓存有两种实现方式：一种是手工进行状态管理，另一种是采用记忆机制。

#### 手工进行状态管理
　　在代码里面增加一个filed，每次计算之前之前先去检查是否存在于缓存中。但是当我们将缓存的取值范围增大时，可能报出OutOfMemoryError的错误。

　　缓存可以提高性能，但是缓存有代价：它提高了代码的非本质复杂性和维护负担。同时，编写缓存的代码还要兼顾执行的环境（比如缓存范围）。
代码示例参考下文[完美数的手工缓存实现](#完美数的手工缓存实现)

#### 记忆机制
　　Groovy语言记忆函数的办法是，先将要记忆的函数定义成闭包，然后对该闭包执行memoize()方法来获得一个新函数，以后我们调用这个新函数的时候，其结果就会被缓存起来。
请保证所有被记忆的函数：
* 没有副作用
* 不依赖任何外部信息
代码示例参考下文[完美数的记忆实现](#完美数的记忆实现)

### 三、缓求值
　　缓求值指尽可能地推迟求解表达式。缓求值的集合不会预先算好所有的元素，而是在用到的时才落实下来。这样做有几个好处：
* 昂贵的运算只有到了绝对必要的时候才执行
* 我们可以建立无限大的集合，只要一直接到请求，就一直送出元素
* 按缓求值的方式来使用映射、筛选等函数式概念，可以产生更高效的代码。

　　例如：`print length([2+1, 3*2, 1/0, 5-4])`
对于严格求值的编程语言，执行（甚至编译）时，会发生`被零除`的异常；  
对于非严格（也叫缓求值）的里，则会得出`4`的结果。  

缓求值在Groovy中的应用：
```
def prepend(val, closure) { new LazyList(val, closure) }
def integers(n) { prepend(n, { integers(n + 1) }) }

@Test
public void lazy_list_acts_like_a_list() {
    def naturalNumbers = integers(1)
    assertEquals('1 2 3 4 5 6 7 8 9 10', naturalNumbers.getHead(10).join(' '))
    def evenNumbers = naturalNumbers.filter { it % 2 == 0 }
    assertEquals('2 4 6 8 10 12 14 16 18 20', evenNumbers.getHead(10).join(' '))
```

### 四、函数式的数据结构
　　函数式语言里经常遇到返回两种截然不同的值的需求，它们用来建模这种行为的常用数据结构是Either。
使用Either表示两种结果的返回值，使用Option来表示有为空返回值的类型。Option类可近似看作为Either的子类。

用Java的泛型来自己实现Either类如下：
```
public class Either<A,B> {
    private A left = null;
    private B right = null;

    private Either(A a,B b) {
        left = a;
        right = b;
    }

    public static <A,B> Either<A,B> left(A a) {
        return new Either<A,B>(a,null);
    }

    public A left() {
        return left;
    }

    public boolean isLeft() {
        return left != null;
    }

    public boolean isRight() {
        return right != null;
    }

    public B right() {
        return right;
    }

    public static <A,B> Either<A,B> right(B b) {
        return new Either<A,B>(null,b);
    }

    public void fold(F<A> leftOption, F<B> rightOption) {
        if(right == null)
            leftOption.f(left);
        else
            rightOption.f(right);
    }
}
```
　　代码可参见《函数式编程思想》的[github](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/java/errorhandling/Either.java)
使用：
```
public static Either<Exception, Integer> parseNumberDefaults(final String s) {
    if (不满足)
        return Either.left(new Exception("Invalid Number"));
    else {
        int number = new RomanNumeral(s).toInt();
        return Either.right(new RomanNumeral(number >= MAX ? MAX : number).toInt());
    }
}

```

### 五、完美数的示例
案例：
>自然数分类规则：
完美数： 真约数之和 = 数本身
过剩数： 真约数之和 > 数本身
不足数： 真约数之和 < 数本身
真约数和：除了数本身之外，其余正约数的和。

　　代码比较长，可以直接查看《函数式编程思想》的[github](https://github.com/oreillymedia/functional_thinking/)总结如下：

1、完美数的命令式解法
[ImpNumberClassifierSimple.java](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/java/number_classifier/ImpNumberClassifierSimple.java)

2、稍向函数式靠拢的完美数分类实现
[NumberClassifier.java](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/java/number_classifier/NumberClassifier.java)

3、完美数分类的Java8实现
[NumberClassifier.java](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/java/number_classifier8/NumberClassifier.java)

<span id="完美数的手工缓存实现">4、完美数的手工缓存实现</span>
[ClassifierCached.groovy](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/groovy/memoization/ClassifierCached.groovy)

<span id="完美数的记忆实现">5、完美数的记忆实现</span>
[ClassifierMemoized.groovy](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/groovy/memoization/ClassifierMemoized.groovy)

