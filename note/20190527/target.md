### JEP 286: 本地变量类型推断
```$xslt
    Author	Brian Goetz
    Owner	Dan Smith
    Type	Feature
    Scope	SE
    Status	Closed / Delivered
    Release	10
    Component	tools
    Discussion	amber dash dev at openjdk dot java dot net
    Effort	M
    Duration	S
    Relates to	JEP 323: Local-Variable Syntax for Lambda Parameters
        JEP 301: Enhanced Enums
    Reviewed by	Alex Buckley, Mark Reinhold
    Endorsed by	Mark Reinhold
    Created	2016/03/08 15:37
    Updated	2018/10/12 01:28
    Issue	8151454
```
### 摘要
    增强Java语言以使用初始值设定项将类型推断扩展为局部变量的声明。
### 目标
     我们通过减少与编写Java代码相关的编程范式来寻求改善开发者的使用体验，同时保持Java对静态类型安全的承诺,
     允许开发人员忽略一些往往没有必要的局部变量类型的显式声明。这种特性将会被允许，举个例子，
     可以使用以下方式声明变量：
```$xslt
        var list = new ArrayList<String>();  // 指定list变量为ArrayList<String>类型
        var stream = list.stream();          // 指定stream变量为Stream<String>类型
```
     这种处理方式将局限于可以初始化值的局部变量、增强的for循环中的索引、以及传统for循环内声明的变量；它不适用于
     方法形式、构造函数形式、方法返回类型、字段、catch形式或其他任意类型的变量声明。
### 成功的标准
    数量上，我们希望实际代码库中的大部分局部变量声明都能够使用此特性转换，进而推断出合适的类型。
    
    客观上， 我们希望局部变量类型推断的局限性，以及这些限制的动机，可以供普通用户访问。(当然，这通常是不可能实现的
    ；我们不仅无法推断出所有局部变量的合理类型，而且有些用户认为类型推断是一种脑海里阅读的形式，而不是一种约束求解的算法，
    在这种情况下，任何解释似乎都不是明智的。)，不过我们试图以这样的方式绘制线条，以便可以清楚地说明为什么某个特定
    的构造会在线上，并且以这种方式，编译器可以有效地联系到用户代码的复杂性，而不是语言上的任意限制。
### 动机
    开发者经常抱怨Java中所需的样板编码程度。本地的一些很明显的类型声明通常被认为是没有必要的，甚至还有妨碍；给出良好的
    变量命名，通常很清楚这个变量做了什么。
    
    为每个变量提供一个清楚的类型的需要也意外地鼓励开发者使用过于复杂的表达式；而使用较低的编程范式声明语法，将复杂的链式
    或嵌套式表达式分解为更简单的表达式的可能性更小。
    
    几乎所有其他流行的静态类型 "{}" 语言，无论是否在JVM上，都已经支持某些形式的局部变量类型推断：C++ (auto),C# (var)
    ,Scala (var/val),Go (用 := 声明)。Java几乎是唯一一种没有采用局部变量类型推断的流行的静态类型语言；在这一点上，这应该
    不再是一个有争议的特性。
    
    在Java SE 8中，类型推断的范围显著被放宽，包括嵌套和链式泛型方法调用的推断扩展，和lambda形式的推断。这使得构建用于
    调用链的API变得容易得多，而这样的API(如 Streams)非常流行，这些都已经表明开发者已经习惯推断出中间类型。比如在下面的
    调用链中：
```$xslt
        int maxWeight = blocks.stream()
                      .filter(b -> b.getColor() == BLUE)
                      .mapToInt(Block::getWeight)
                      .max();
```
    
     
   







