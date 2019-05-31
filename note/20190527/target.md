### JEP 286: 局部变量类型推断 [原文链接]()
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
    增强Java语言以使用初始化值将类型推断扩展为声明局部变量。
### 目标
     我们通过减少与编写Java代码相关的编程范式来寻求改善开发者的使用体验，同时保持Java对静态类型安全的承诺,
     允许开发人员忽略一些往往没有必要的局部变量类型的显式声明。这种功能将会被允许，举个例子，
     可以使用以下方式声明变量：
```$xslt
        var list = new ArrayList<String>();  // 指定list变量为ArrayList<String>类型
        var stream = list.stream();          // 指定stream变量为Stream<String>类型
```
     这种处理方式将局限于可以初始化值的局部变量、增强的for循环中的索引、以及传统for循环内声明的变量；它不适用于
     方法形式、构造函数形式、方法返回类型、字段、catch形式或其他任意类型的变量声明。
### 成功的标准
    定量上，我们希望实际代码库中的大部分局部变量声明都能够使用此功能转换，进而推断出合适的类型。
    
    定性上， 我们希望局部变量类型推断的局限性，以及这些限制的动机，可以供普通用户访问。(当然，这通常是不可能实现的
    ；我们不仅无法推断出所有局部变量的合理类型，而且有些用户认为类型推断是一种脑海里阅读的形式，而不是一种约束求解的算法，
    在这种情况下，任何解释似乎都是不明智的。)，不过我们试图以这样的方式绘制一条线，以便可以清楚地说明为什么某个特定
    的构造会在线上，并且以这种方式，编译器可以有效地联系到用户代码的复杂性，而不是语言上的任意限制。
### 动机
    开发者经常抱怨Java中所需的样板编码程度。本地的一些很明显的类型声明通常被认为是没有必要的，甚至还有妨碍；给出良好的
    变量命名，通常很清楚这个变量做了什么。
    
    为每个变量提供一个清楚的类型的需要也意外地鼓励开发者使用过于复杂的表达式；而使用较低的编程范式声明语言，将复杂的链式
    或嵌套式表达式分解为更简单的表达式的可能性就更小了。
    
    几乎所有其他流行的静态类型 "{}" 语言，无论是否在JVM上，都已经支持某些形式的局部变量类型推断：C++ (auto),C# (var)
    ,Scala (var/val),Go (用 := 声明)。Java几乎是唯一一种没有采用局部变量类型推断的流行的静态类型语言；在这一点上，这应该
    不再是一个有争议的功能。
    
    在Java SE 8中，类型推断的范围显著被放宽，包括嵌套和链式泛型方法调用的推断扩展，还有lambda格式的推断。这使得构建用于
    调用链的API变得容易得多，而这样的API(如 Streams)非常流行，这些都已经表明开发者已经习惯推断出中间类型。比如在下面的
    链式调用中：
```$xslt
        int maxWeight = blocks.stream()
                      .filter(b -> b.getColor() == BLUE)
                      .mapToInt(Block::getWeight)
                      .max();
```
    没有人会困扰到(或者注意到)中间类型Stream<Block> 和 IntStream，以及lambda表达式中变量b的类型，也不会在源程序中明确
    显示。    
    
    局部变量类型推断允许在结构不太紧密的API中产生类似的效果；局部变量基本都用在链式调用上，并从类型推断上获益，例如：
```$xslt
        var path = Paths.get(fileName);
        var bytes = Files.readAllBytes(path);
``` 
### 描述  
    对于使用初始化值的局部变量声明、增强for循环里的索引、和在传统for循环中声明的索引变量，允许它们接受保留类型名称var来
    代替可见的变量类型：
```$xslt
        var list = new ArrayList<String>(); // infers ArrayList<String>
        var stream = list.stream();         // infers Stream<String>
```
    标识符var不是关键字，而是一个保留类型名称。这意味着可以使用var作为变量、方法或包名称的代码不会受到影响；使用var作为
    类或接口名称的代码将受到影响（不过这些名称在实际情况中很少见，因为它们违反了通常的命名惯例）。
    
    缺少初始化值的局部变量声明、声明多个变量，有额外的数组维度符号[]、或引用正在初始化的变量，这些都是不允许的。
    拒绝未初始化值的局部变量会缩小这个功能的作用范围，避免"action at distance"推断错误，在典型的程序中仅仅只排除了一小
    部分的局部变量。
    
    在推断过程中基本上只给变量赋予其初始化表达式的类型。另外还有一些要注意的：
    
*  初始化程序没有目标类型（因为我们还没有推断出来）。需要这种类型的Poly表达式，如lambda表达式、方法引用和数组初始化值，
    会引起错误。    
*  如果初始化值具有null类型，会引发错误，就像没有初始化值的变量一样，这个变量可能稍后要进行初始化，但我们还不知道想要什
    么类型。
*  捕获变量和内嵌的捕获变量，会被映射到没有涉及到捕获变量的超类型上。这种映射将捕获变量替换为其上界，用有界通配符替换
    涉及到捕获变量的类型参数。（然后不断重复）。这保留了传统的捕获变量的限制范围，这些变量仅在单个语句中考虑。
*   除上述例外情况外，不可见类型、包括匿名类类型和交集类型，可以被推断出来。编译器以及编译工具需要考虑这种可能性。

### 适用性和影响力
    通过浏览OpenJDK代码库来查看局部变量的声明，我们发现13%的局部变量不能使用var编写，因为它们没有初始化值、初始化的值是
    null类型、或者在初始化的时候很少需要目标类型。在剩余的局部变量声明中：
*  无论如何，具有初始化值的局部变量大多数（在JDK和更广泛的语法库中超过75%）已经是有效不变的，意味着任何"推动"远离这个
    功能提供的可变性都会受到限制。 
*   lambdas/内部类的可捕获性已经为有效final变量提供了重要的推动力。
*   在一个有7个有效final变量和2个可变变量的代码块中，可变变量的类型会在视觉上不和谐，破坏了该功能的大部分。
                
                    
     另一方面，我们可以扩展这个功能包括相当于空的final的局部变量（即：不需要初始化值，而是依靠明确的分配分析），我们选择
     限制为"仅适用初始化值的变量"，因为它涵盖了很大一部分候选项，同时保持功能的简单并减少"action in distance"错误。
     
     同样地，我们也可以在推断类型的时候考虑所有的分配任务，而不仅仅是初始化值；虽然这会进一步增加局部变量利用此功能的百
     分比。它同样也会增加"action at a distance"的错误风险。

### 语法选择
    在语法上有多样的选择。 这里的两个主要自由度是使用的关键字（var,auto等），以及是否为不可变的局部变量设置单独的格式
    （val,let）。我们考虑了一下语法选项：
*   var x = expr only (如 C#)
*   var, plus val 用于不可变局部变量 (如 Scala, Kotlin)
*   var, plus let 用于不可变局部变量 (如 Swift)
*   auto x = expr (如 C++)
*   const x = expr (已经是一个保留字)
*   final x = expr (已经是一个保留字)
*   let x = expr
*   def x = expr (如 Groovy)
*   x := expr (如 Go)
        
        
    在收集了大量的意见后，var显然比Groovy，C++或Go方法更受欢迎。关于不可变的局部变量的第二种语法格式（val,let），
    存在着相当多的意见；这将是额外设计中的一种权衡，最后，我们选择了仅支持var。
   你可以在[这里](http://mail.openjdk.java.net/pipermail/platform-jep-discuss/2016-December/000066.html)找到关于
   基本原理的一些细节
### 不可见类型
    有时候，初始化值的类型是不可见类型，例如捕获变量类型、交集类型、匿名类类型。在这种情况下，我们可以选择:
    i)是否要推断这个类型, ii)拒绝这个表达式 ,iii)推断一个可见的超类型。
    
    编译器（以及细心的程序员）必定是已经对不可见的类型进行了合理的推断。然而，这些不可见类型的变量作为局部变量类型使用
    将会显著增加他们的曝光率。泄露编译/规范错误进而迫使程序员更频繁地面对它们。在教学上，在显式类型和隐式类型声明之间
    进行简单的语法转换是很好的。
    
    也就是说，简单拒绝不可见类型变量的初始化是无用的（这经常令程序员惊讶，例如这样的声明 var c = getClass()）,
    并且映射到超类型可能会意外受损。
    
    这些考虑使我们得到了不同的答案：
*   null类型的变量几乎没用，并且对于推断类型也没有好的替代方法，所以我们拒绝这些变量。
*   允许捕获类型的变量进入后续的语句会增加语言的表现力，但这不是此功能的目标。相反，这种建议的映射操作是我们无论如何都
    需要用来解决类型推断系统中的各种Bug（参照例子：[JDK-8016196](https://bugs.openjdk.java.net/browse/JDK-8016196)）,
    在这里应用它是合理的。
*   交集类型特别难以映射到超类型，它们没有预先安排，所以在交集中的每个元素并不比其他元素"更好"。对于超类型的稳定选择是
    所有元素的润滑剂，不过它们的超类型往往是Object或其他同样无益的类，所以我们允许它们这样做。
*   匿名类类型无法命名，但它们很容易理解，它们仅仅是class类。允许变量具有匿名类类型引入了一种有用的简写方式，用于声明
    class类型的局部变量的单例实例。我们也允许它们这样做。
### 风险和假设
    风险：因为Java已经在RHS上做了重要的类型推断（lambda格式、泛型方法类型参数、<>操作符），尝试在这样的表达式的LHS上使
    用var会有失败的风险，并且可能存在难以阅读的错误信息。   
    
    我们已经通过在推断LHS时，使用简化的错误消息来缓解这种情况。
    例如：
```$xslt
        Main.java:81: error: cannot infer type for local 无法推断局部变量类型
        variable x
                var x;
                    ^
          (cannot use 'val' on variable without initializer) 不能在未初始化的变量上使用var修饰
        
        Main.java:82: error: cannot infer type for local 无法推断局部变量类型
        variable f
                var f = () -> { };
                    ^
          (lambda expression needs an explicit target-type) lambda表达式需要一个明确的目标类型
        
        Main.java:83: error: cannot infer type for local 无法推断局部变量类型
        variable g
                var g = null;
                    ^
          (variable initializer is 'null') 变量初始化值是null
        
        Main.java:84: error: cannot infer type for local 无法推断局部变量类型
        variable c
                var c = l();
                    ^
          (inferred type is non denotable) 推断类型不是<>表达式
        
        Main.java:195: error: cannot infer type for local variable m 无法推断局部变量 m 的类型
                var m = this::l;
                    ^
          (method reference needs an explicit target-type) 引用方法需要一个明确的返回类型
        
        Main.java:199: error: cannot infer type for local variable k 无法推断局部变量 k 的类型
                var k = { 1 , 2 };
                    ^
          (array initializer needs an explicit target-type) 初始化的数组需要一个明确的目标类型
          
```
    风险：来源不兼容（有可能已将var用作类型名称）  
    
    使用保留类型名称进行缓解；var之类的名称不符合类型的命名约定，因此不太可能用作类型。var通常用作标识符；我们依然允许
    它这样做。
    
    风险：可读性降低，重构时出现意外。

   像任何其他语言的功能一样，局部变量类型推断可用于编写清晰和不清晰的代码；最终编写清晰代码的责任在于用户，请参阅
   [var风格指南](http://openjdk.java.net/projects/amber/LVTIstyle.html)
   以及[常见问题](http://openjdk.java.net/projects/amber/LVTIFAQ.html)使用var。                            
                    
    
    
    
    
   
     
    
        






