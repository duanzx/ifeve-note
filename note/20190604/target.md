## (翻译)Java中局部变量类型推断编写风格指南 
原文链接：[Style Guidelines for Local Variable Type Inference in Java](http://openjdk.java.net/projects/amber/LVTIstyle.html)        
Stuart W.Marks  
2018-03-22  


### 简介
Java SE 10引入了局部变量的类型推断。早先，所有的局部变量声明都需要在左侧声明明确的类型。
使用类型推断，一些可见类型可以替换为具有初始化值的局部变量保留类型var，这种作为局部变量类型
的var的类型，是从初始化程序的类型中推断出来的。

关于此功能存在一定的争议。有些人对它的简洁性表示欢迎，其他人担心它剥夺了阅读者看重的类型信息
，从而损害了代码的可读性。这两个团体都是对的。它可以通过消除冗余信息使代码更具有可读性，也可以
通过删除有用的信息来降低代码的可读性。另外一个团体担心它会被滥用，从而导致编写更糟糕的Java代码。
这也是事实，但它也可能导致编写更好的代码。像所有功能一样，它必须与判断一起使用。何时该使用，
何时该不使用，没有一揽子规则。

局部变量声明不是孤立存在的；周边的代码可以影响甚至压倒使用var的影响。本文档的目的是检查周边代码
对var声明的影响，解释一些权衡，并提供有效使用var的指南

### 使用原则

#### P1. 阅读代码比编写代码更重要   
代码的读取频率远高于编写代码。此外，在编写代码时，我们通常要有整体思路，并且要花费时间；
在阅读代码的时候，我们经常是上下文切换，而且可能更加匆忙。是否以及如何使用特定语言功能应该取决
于它们对未来读者的影响，而不是它的作者。较短的程序可能比较长的程序更可取，但是过多地简写程序会
省略对理解程序有用的信息。这里的核心问题是为程序找到合适的大小，以便最大化程序的可阅读性。

 我们在这里特别关注在输入或编辑程序时所需要的码字量。虽然简洁性可能会是对作者的一个很好的鼓励，但
 专注于它会忽略主要的目标，即提高程序的可读性。
 
 #### P2. 在经过局部变量类型推断后，代码应该变得清晰
 读者应该能够查看var声明，并且随着局部变量声明的使用，可以立刻了解代码里正在发生了什么事情。理想
 情况下，只通过代码片段或上下文就可以轻松理解代码。如果读懂一个var声明需要读者去查看代码周边的
 若干位置，此时使用var可能不是一个好的情况。而且，可能它还表明代码本身存在问题。
 
 #### P3. 代码的可读性不应该依赖于IDE
 代码通常在IDE中编写和读取，所以很容易依赖IDE的代码分析功能。对于类型声明，在任何地方使用var,都可以通过IDE指向一个变量来
 确定它的类型，但是为什么不这么做呢？
 
 这里有两个原因。代码经常在IDE外部读取。代码出现在IDE设施不可用的许多地方，例如文档中的片段，浏览互联网上的仓库或补丁文件
 。为了理解这些代码的作用，必须将代码导入IDE。这样做是适得其反的。
 
 第二个原因是即使是在IDE中读取代码时也是如此，通常需要明确的操作来查询IDE以获取有关变量的更多信息。例如，查询使用var声明
 的变量类型，可能必须将鼠标悬停在变量上并等待弹出信息，这可能只需要片刻时间，但是它会扰乱阅读流程。
 
 代码应该是自我暴露的，它的表面应该是可以理解的，无需工具的帮助。
 
 #### P4. 显式类型是一种权衡
Java历来要求局部变量声明里要明确包含该类型，显然显式类型可能非常有用，但它们有时候不是很重要，有时候还可以忽略。要求一个
明确的类型可能还会混淆一些有用的信息。

省略显式类型可以减少这种混淆，但只有在这种混淆不会损害其可理解性的情况下。这种类型不是向读者传达信息的唯一方式。其他方法
包括变量的名称和初始化表达式。在确定是否可以将其中一个频道静音时，我们应该考虑所有可用的频道。

### 指南

#### G1. 选择提供有用信息的变量名称
通常这是一个好习惯，但在var的背景下它更为重要。在一个var的声明中，可以使用变量的名称来传达有关变量含义和用法的信息。
使用var替换显式类型的同时也要改进变量的名称。例如：

    //原始写法
    List<Customer> x = dbconn.executeQuery(query);
    
    //改进写法
    var custList = dbconn.executeQuery(query);
在这种情况下，无用的变量名已被替换为一个能够唤起变量类型的名称，该名称现在隐含在var的声明中。根据其逻辑结论对变量的类型
进行编码，得出了"匈牙利表示法"。就像显式类型一样，这有时是有帮助的，有时候只是杂乱无章。在此示例中，名称custList表示正在
返回List。这可能不重要。和显式类型不同，变量的名称有时可以更好地表达变量的角色或性质，比如"customers":

    //原始写法    
    try (Stream<Customer> result = dbconn.executeQuery(query)) {
         return result.map(...)
                      .filter(...)
                      .findAny();
     }
    //改进写法
    try (var customers = dbconn.executeQuery(query)) {
        return customers.map(...)
                        .filter(...)
                        .findAny();
    }

#### G2.最小化局部变量的范围
最小化局部变量的范围通常也是一个好的习惯。这种做法在Effective Java (第三版)，第57项中有所描述。 如果使用var，它会是额外
助力。

在下面的例子中，add方法明确地将特殊项添加到list集合的最后一个元素，所以它按照预期最后处理。

    var items = new ArrayList<Item>(...);
    items.add(MUST_BE_PROCESSED_LAST);
    for (var item : items) ...
现在假设为了删除重复的项目，程序员修改此代码以使用HashSet而不是ArrayList：

    var items = new HashSet<Item>(...);
    items.add(MUST_BE_PROCESSED_LAST);
    for (var item : items) ...
这段代码现在有个bug，因为Set没有定义迭代顺序。不过，程序员可能会立即修复这个bug，因为items变量的使用与其声明相邻。

现在假设这段代码是一个大方法的一部分，相应地items具有很大的变量范围：

    var items = new HashSet<Item>(...);
    
    // ... 100 lines of code ...
    
    items.add(MUST_BE_PROCESSED_LAST);
    for (var item : items) ...
使ArrayList更改为HashSet的影响不再明显，因为使用items的代码与声明items的代码离得很远。这意味着上面所说bug似乎可以存活
更长时间。

如果items已经明确声明为List<String>，更改初始化程序还需要将类型更改为Set<String>。这可能会提示程序员检查方法的其余部分
是否存在受此类更改影响的代码。(问题来了，他也可能不会检查)。如果使用var会消除这类影响，不过可能会增加在此类代码中
引入错误的风险。

这似乎是反对使用var的论据，但实际上并非如此。使用var的初始示例非常精细。仅当变量的范围很大时才会出现此问题。应该更改代码
以减少局部变量的范围，然后用var声明它们，而不是简单地避免在这些情况下使用var。

#### G3. 当初始化程序为读者提供足够的信息时，请考虑使用var
局部变量通常在构造函数中进行初始化。正在构造的类的名称通常与左侧显示声明的类型重复。如果类型名称很长，就可以使用var提供
简洁而不会丢失信息：

    // 原始写法：
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    // 改进写法
    var outputStream = new ByteArrayOutputStream();
在初始化程序是方法调用的情况下，使用var也是合理的，例如静态工厂方法，而不是构造函数，并且其名称包含足够的类型信息：

    //原始写法
    BufferedReader reader = Files.newBufferedReader(...);
    List<String> stringList = List.of("a", "b", "c");
    // 改进写法
    var reader = Files.newBufferedReader(...);
    var stringList = List.of("a", "b", "c"); 
在这些情况下，方法的名称强烈暗示其特定的返回类型，然后用于推断变量的类型。

#### G4. 使用var局部变量分解链式或嵌套表达式
考虑使用字符串集合并查找最常出现的字符串的代码，可能如下所示：
    
    return strings.stream()
                   .collect(groupingBy(s -> s, counting()))
                   .entrySet()
                   .stream()
                   .max(Map.Entry.comparingByValue())
                   .map(Map.Entry::getKey);
这段代码是正确的，但它可能令人困惑，因为它看起来像是一个单一的流管道。事实上，它是一个短暂的流，接着是第一个流结果生成
的第二个流，然后是第二个流的可选结果映射后的流。表达此代码的最易读的方式是两个或三个语句；第一组实体放入一个Map,然后减少
这个Map,然后从结果中提取出这个Key，如下所示：
    
    Map<String, Long> freqMap = strings.stream()
                                       .collect(groupingBy(s -> s, counting()));
    Optional<Map.Entry<String, Long>> maxEntryOpt = freqMap.entrySet()
                                                           .stream()
                                                           .max(Map.Entry.comparingByValue());
    return maxEntryOpt.map(Map.Entry::getKey);
但编写者可能会拒绝这样做，因为编写中间变量的类型似乎太过于繁琐，相反他们篡改了控制流程。使用var允许我们更自然地表达代码
，而无需付出显式声明中间变量类型的高代价：

    var freqMap = strings.stream()
                         .collect(groupingBy(s -> s, counting()));
    var maxEntryOpt = freqMap.entrySet()
                             .stream()
                             .max(Map.Entry.comparingByValue());
    return maxEntryOpt.map(Map.Entry::getKey);
有些人可能更倾向于第一个片段及其单个长的链式调用。但是，在某些条件下，最好分解长的方法链。对这些情况使用var是一种可行的
方法，而在第二个片段中使用中间变量的完整声明会是一个不好的选择。 和许多其他情况一样，正确使用var可能会涉及拿掉一些东西
（显示类型）和加入一些东西（更好的变量名称，更好的代码结构）。         
 
 #### G5. 不用过分担心"使用接口编程" 中局部变量的使用问题
 Java编程中常见的习惯用法是构造具体类型的实例，但要将其分配给接口类型的变量。这将代码绑定到抽象上而不是具体实现上，为
 代码以后的维护保留了灵活性。
    
    //原始写法, list类型为接口List类型
    List<String> list = new ArrayList<>()
如果使用var,可以推断出list具体的实现类型ArrayList而不是接口类型List   
 
    // 推断出list的类型是 ArrayList<String>.
    var list = new ArrayList<String>(); 
这里要再次重申一次，var只能用于局部变量。它不能用于推断字段类型，方法参数类型和方法返回类型。"使用接口编程"的原则在这些
情况下仍然和以往一样重要。

主要问题是使用该变量的代码可以形成对具体实现的依赖性。如果变量的初始化程序以后要改变，这可能导致其推断类型发生变化，在
使用该变量的后续代码中产生异常或bug。

如果，如指南G2中所建议的那样，局部变量的范围很小，可能影响后续代码的具体实现的"漏洞"是有限的。如果变量仅用于几行之外的
代码，应该很容易避免这些问题或者缓解这些出现的问题。

在这种特殊情况下，ArrayList只包含一些不在List上的方法，如ensureCapacity()和trimToSize()。这些方法不会影响到List,所以
调用他们不会影响程序的正确性。这进一步降低了推断类型作为具体实现类型而不是接口类型的影响。 

#### G6. 使用带有<>和泛型方法的var时候要小心
var和<>功能允许您在可以从已存在的信息派生时，省略具体的类型信息。你能在同一个变量声明中使用它们吗？

考虑一下以下代码：

    PriorityQueue<Item> itemQueue = new PriorityQueue<Item>();
这段代码可以使用var或<>重写，并且不会丢失类型信息:

    // 正确：两个变量都可以声明为PriorityQueue<Item>类型
    PriorityQueue<Item> itemQueue = new PriorityQueue<>();
    var itemQueue = new PriorityQueue<Item>();    
同时使用var和<>是合法的，但推断类型将会改变：

    // 危险: 推断类型变成了 PriorityQueue<Object>
    var itemQueue = new PriorityQueue<>();
从上面的推断结果来看，<>可以使用目标类型（通常在声明的左侧）或构造函数作为<>里的参数类型。如果两者都不存在，它会追溯到
最宽泛的适用类型，通常是Object。这通常不是我们预期的结果。    
  