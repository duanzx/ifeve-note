JEP 286: Local-Variable Type Inference
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
Summary

Enhance the Java Language to extend type inference to declarations of local variables with initializers.
Goals

We seek to improve the developer experience by reducing the ceremony associated with writing Java code, while maintaining Java's commitment to static type safety, by allowing developers to elide the often-unnecessary manifest declaration of local variable types. This feature would allow, for example, declarations such as:

var list = new ArrayList<String>();  // infers ArrayList<String>
var stream = list.stream();          // infers Stream<String>

This treatment would be restricted to local variables with initializers, indexes in the enhanced for-loop, and locals declared in a traditional for-loop; it would not be available for method formals, constructor formals, method return types, fields, catch formals, or any other kind of variable declaration.
Success Criteria

Quantitatively, we want that a substantial percentage of local variable declarations in real codebases can be converted using this feature, inferring an appropriate type.

Qualitatively, we want that the limitations of local variable type inference, and the motivations for these limitations, be accessible to a typical user. (This is, of course, impossible to achieve in general; not only will we not be able to infer reasonable types for all local variables, but some users imagine type inference to be a form of mind reading, rather than an algorithm for constraint solving, in which case no explanation will seem sensible.) But we seek to draw the lines in such a way that it can be made clear why a particular construct is over the line -- and in such a way that compiler diagnostics can effectively connect it to complexity in the user's code, rather than an arbitrary restriction in the language.
Motivation

Developers frequently complain about the degree of boilerplate coding required in Java. Manifest type declarations for locals are often perceived to be unnecessary or even in the way; given good variable naming, it is often perfectly clear what is going on.

The need to provide a manifest type for every variable also accidentally encourages developers to use overly complex expressions; with a lower-ceremony declaration syntax, there is less disincentive to break complex chained or nested expressions into simpler ones.

Nearly all other popular statically typed "curly-brace" languages, both on the JVM and off, already support some form of local-variable type inference: C++ (auto), C# (var), Scala (var/val), Go (declaration with :=). Java is nearly the only popular statically typed language that has not embraced local-variable type inference; at this point, this should no longer be a controversial feature.

The scope of type inference was significantly broadened in Java SE 8, including expanded inference for nested and chained generic method calls, and inference for lambda formals. This made it far easier to build APIs designed for call chaining, and such APIs (such as Streams) have been quite popular, showing that developers are already comfortable having intermediate types inferred. In a call chain like:

int maxWeight = blocks.stream()
                      .filter(b -> b.getColor() == BLUE)
                      .mapToInt(Block::getWeight)
                      .max();

no one is bothered (or even notices) that the intermediate types Stream<Block> and IntStream, as well as the type of the lambda formal b, do not appear explicitly in the source code.

Local variable type inference allows a similar effect in less tightly structured APIs; many uses of local variables are essentially chains, and benefit equally from inference, such as:

var path = Paths.get(fileName);
var bytes = Files.readAllBytes(path);

Description

For local variable declarations with initializers, enhanced for-loop indexes, and index variables declared in traditional for loops, allow the reserved type name var to be accepted in place of manifest types:

var list = new ArrayList<String>(); // infers ArrayList<String>
var stream = list.stream();         // infers Stream<String>

The identifier var is not a keyword; instead it is a reserved type name. This means that code that uses var as a variable, method, or package name will not be affected; code that uses var as a class or interface name will be affected (but these names are rare in practice, since they violate usual naming conventions).

Forms of local variable declarations that lack initializers, declare multiple variables, have extra array dimension brackets, or reference the variable being initialized are not allowed. Rejecting locals without initializers narrows the scope of the feature, avoiding "action at a distance" inference errors, and only excludes a small portion of locals in typical programs.

The inference process, substantially, just gives the variable the type of its initializer expression. Some subtleties:

    The initializer has no target type (because we haven't inferred it yet). Poly expressions that require such a type, like lambdas, method references, and array initializers, will trigger an error.
    If the initializer has the null type, an error occurs—like a variable without an initializer, this variable is probably intended to be initialized later, and we don't know what type will be wanted.
    Capture variables, and types with nested capture variables, are projected to supertypes that do not mention capture variables. This mapping replaces capture variables with their upper bounds and replaces type arguments mentioning capture variables with bounded wildcards (and then recurs). This preserves the traditionally limited scope of capture variables, which are only considered within a single statement.
    Other than the above exceptions, non-denotable types, including anonymous class types and intersection types, may be inferred. Compilers and tools need to account for this possibility.

Applicability and impact

Scanning the OpenJDK code base for local variable declarations, we found that 13% cannot be written using var, since there is no initializer, the initializer has the null type, or (rarely) the initializer requires a target type. Among the remaining local variable declarations:

    94% have an initializer with the exact type present in the source code (63% of cases with parameterized types)
    5% have an initializer with some sharper denotable type (29% of cases with parameterized types)
    1% have an initializer with a type that mentions a capture variable (7% of cases with parameterized types)
    <1% have an initializer with an anonymous class type or intersection type (same for cases with parameterized types)

Alternatives

We could continue to require manifest declaration of local variable types.

Rather than supporting var, we could limit support to uses of diamond in variable declarations; this would address a subset of the cases addressed by var.

The design described above incorporates several decisions about scope, syntax, and non-denoteable types; alternatives for those choices which were also considered are documented here.
Scope Choices

There are several other ways we could have scoped this feature. We considered restricting the feature to effectively final locals (val). However, we backed off from this position because:

    The majority (more than 75% in both JDK and broader corpus) of local variables with initializers were already effectively immutable anyway, meaning that any "nudge" away from mutability that this feature could have provided would have been limited;

    Capturability by lambdas/inner classes already provides a significant push towards effectively final locals;

    In a code block with (say) 7 effectively final locals and 2 mutable ones, the types required for the mutable ones would be visually jarring, undermining much of the benefit of the feature.

On the other hand, we could have expanded this feature to include the local equivalent of "blank" finals (i.e., not requiring an initializer, instead relying on definite assignment analysis.) We chose the restriction to "variables with initializers only" because it covers a significant fraction of the candidates while maintaining the simplicity of the feature and reducing "action at a distance" errors.

Similarly, we also could have taken all assignments into account when inferring the type, rather than just the initializer; while this would have further increased the percentage of locals that could exploit this feature, it would also increase the risk of "action at a distance" errors.
Syntax Choices

There was a diversity of opinions on syntax. The two main degrees of freedom here are what keywords to use (var, auto, etc), and whether to have a separate new form for immutable locals (val, let). We considered the following syntactic options:

    var x = expr only (like C#)
    var, plus val for immutable locals (like Scala, Kotlin)
    var, plus let for immutable locals (like Swift)
    auto x = expr (like C++)
    const x = expr (already a reserved word)
    final x = expr (already a reserved word)
    let x = expr
    def x = expr (like Groovy)
    x := expr (like Go)

After gathering substantial input, var was clearly preferred over the Groovy, C++, or Go approaches. There was a substantial diversity of opinion over a second syntactic form for immutable locals (val, let); this would be a tradeoff of additional ceremony for additional capture of design intent. In the end, we chose to support only var. Some details on the rationale can be found here.
Non-denotable types

Sometimes the type of the initializer is a non-denotable type, such as a capture variable type, intersection type, or anonymous class type. In such cases, we have a choice of whether to i) infer the type, ii) reject the expression, or iii) infer a denotable supertype.

Compilers (and attentive programmers!) must already be comfortable reasoning about non-denoteable types. However, their use as the types of local variables will significantly increase their exposure, revealing compiler/specification bugs and forcing programmers to confront them more frequently. Pedagogically, it's nice to have a simple syntactic transformation between explicitly-typed and implicitly-typed declarations.

That said, it's unhelpfully pedantic to simply reject initializers with non-denotable types (often to the surprise of the programmer, such as in the declaration var c = getClass()). And a mapping to a supertype can be unexpected and lossy.

These considerations led us to different answers:

    A null-typed variable is practically useless, and there is no good alternative for an inferred type, so we reject these.
    Allowing capture variables to flow into subsequent statements adds new expressiveness to the language, but that's not a goal of this feature. Instead, the proposed projection operation is one we need to use anyway to address various bugs in the type system (see, e.g., JDK-8016196), and it's reasonable to apply it here.
    Intersection types are especially difficult to map to a supertype—they're not ordered, so one element of the intersection is not inherently "better" than the others. The stable choice for a supertype is the lub of all the elements, but that will often be Object or something equally unhelpful. So we allow them.
    Anonymous class types cannot be named, but they're easily understood—they're just classes. Allowing variables to have anonymous class types introduces a useful shorthand for declaring a singleton instance of a local class. We allow them.

Risks and Assumptions

Risk: Because Java already does significant type inference on the RHS (lambda formals, generic method type arguments, diamond), there is a risk that attempting to use var on the LHS of such an expression will fail, and possibly with difficult-to-read error messages.

We've mitigated this by using simplified error messages when the LHS is inferred.

Examples:

Main.java:81: error: cannot infer type for local
variable x
        var x;
            ^
  (cannot use 'val' on variable without initializer)

Main.java:82: error: cannot infer type for local
variable f
        var f = () -> { };
            ^
  (lambda expression needs an explicit target-type) 

Main.java:83: error: cannot infer type for local
variable g
        var g = null;
            ^
  (variable initializer is 'null')

Main.java:84: error: cannot infer type for local
variable c
        var c = l();
            ^
  (inferred type is non denotable)

Main.java:195: error: cannot infer type for local variable m
        var m = this::l;
            ^
  (method reference needs an explicit target-type)

Main.java:199: error: cannot infer type for local variable k
        var k = { 1 , 2 };
            ^
  (array initializer needs an explicit target-type)

Risk: source incompatibilities (someone may have used var as a type name.)

Mitigated with reserved type names; names like var do not conform to the naming conventions for types, and therefore are unlikely to be used as types. The name var is commonly used as an identifier; we continue to allow this.

Risk: reduced readability, surprises when refactoring.

Like any other language feature, local variable type inference can be used to write both clear and unclear code; ultimately the responsibility for writing clear code lies with the user. See the style guidelines for using var, and the Frequently Asked Questions.
