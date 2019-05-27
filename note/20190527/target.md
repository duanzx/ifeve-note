### **JEP 286: 本地变量类型推断**
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
### **摘要**
 增强Java语言以使用初始值设定项将类型推断扩展为局部变量的声明。
### **目标**
 我们通过减少与编写Java代码相关的范式来寻求改善开发者的使用体验，同时保持Java对静态类型安全的承诺,
 允许开发人员忽略一些往往没有必要的本地变量类型的显式声明。这种特性将会被允许，举个例子，
可以使用以下方式声明变量：
```$xslt
    var list = new ArrayList<String>();  // 指定list变量为ArrayList<String>类型
    var stream = list.stream();          // 指定stream变量为Stream<String>类型
```
这种处理方式将局限于可以初始化值的本地变量、增强的for循环中的索引、以及传统for循环内声明的变量；它不适用于
方法形式、构造函数形式、方法返回类型、字段、catch形式或其他类型的变量声明。
### 成功的标准





