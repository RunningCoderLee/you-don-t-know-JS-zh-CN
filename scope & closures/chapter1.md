# 你不知道的JS：作用域和闭包(You Don't Know JS: Scope & Closures) --(罗尧)
# 什么是作用域？（Chapter 1: What is Scope?）

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program *state*.    
几乎所有编程语言的最基本的功能之一是将值存储在变量中，然后检索或修改这些值。事实上，存储值和从变量中取出值的能力赋予了程序*状态(state)*。

Without such a concept, a program could perform some tasks, but they would be extremely limited and not terribly interesting.     
没有这样的概念，程序可以执行一些任务，但是它们将是非常有限的，并不会非常有趣。

But the inclusion of variables into our program begets the most interesting questions we will now address: where do those variables *live*? In other words, where are they stored? And, most importantly, how does our program find them when it needs them?    
但将变量包含在我们的程序中产生了我们现在要解决的最有趣的问题：这些变量*住*在哪里？ 换句话说，他们在哪里存储？ 最重要的是，我们的程序在需要它们时如何找到它们？

These questions speak to the need for a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules: *Scope*.     
这些问题说明，需要一个明确定义的规则，用于在某个位置存储变量，并在以后找到这些变量。我们将这套规则称为：*作用域（Scope）*。

But, where and how do these *Scope* rules get set?   
但是，这些*作用域*规则在哪里及如何设置的呢？

## 编译原理（Compiler Theory）

It may be self-evident, or it may be surprising, depending on your level of interaction with various languages, but despite the fact that JavaScript falls under the general category of "dynamic" or "interpreted" languages, it is in fact a compiled language. It is *not* compiled well in advance, as are many traditionally-compiled languages, nor are the results of compilation portable among various distributed systems.     
或许你早就知道，或者你是从未听说，这取决于你的知识水平，尽管JavaScript属于“动态”或“解释”语言的一般类别，但它实际上是一门编译语言。和许多传统编译的语言不同，它*不是*提前编译的，编译结果也不能在各种分布式系统间移植。

But, nevertheless, the JavaScript engine performs many of the same steps, albeit in more sophisticated ways than we may commonly be aware, of any traditional language-compiler.       
但是，尽管如此，JavaScript引擎和传统的编译语言执行许多相同的步骤，有些甚至比我们想的的更加复杂。

In traditional compiled-language process, a chunk of source code, your program, will undergo typically three steps *before* it is executed, roughly called "compilation":      
在传统的编译语言过程中，一段源代码，你的程序，将在执行*之前*经历三个步骤，大致称为“编译”：

1. **Tokenizing/Lexing:** breaking up a string of characters into meaningful (to the language) chunks, called tokens. For instance, consider the program: `var a = 2;`. This program would likely be broken up into the following tokens: `var`, `a`, `=`, `2`, and `;`. Whitespace may or may not be persisted as a token, depending on whether it's meaningful or not.1.       
**分词/词法分析（Tokenizing/Lexing）:** 将字符串分解成有意义的（对于语言）块，称为词法单元（tokens）。例如，考虑程序：`var a = 2;`。这个程序可能被分解成下面的标记：`var`，`a`，`=`，`2`和`;`。空白是否被当作词法单元，这取决于它是否有意义。

    **Note:** The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a *stateless* or *stateful* way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, *that* would be **lexing**.    
    **注意:** 分词和词法分析之间的区别是微妙的，主要区分方法是对词法单元的识别是通过*有状态*还是*无状态*方式进行的。简单地说，如果词法单元生成器调用有状态的解析规则来判断`a`是否应被视为一个独立的词法单元还是另一个词法单元的一部分，这就是*词法分析*。

2. **Parsing:** taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).     
**解析（Parsing）:** 解析将获取词法单元流（数组）并将其转换成嵌套元素表示程序语法结构的树，这棵树被称为“AST”(<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).

    The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is `a`), and another child called `AssignmentExpression` which itself has a child called `NumericLiteral` (whose value is `2`).`var a = 2;`的抽象语法树可能从名为`VariableDeclaration`的顶级节点开始，下面一个称为`Identifier`（其值为`a`）的子节点和另一个称为`AssignmentExpression`的子节点，`AssignmentExpression`有一个名为NumericLiteral的子节点（值为2）。

3. **Code-Generation:** the process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it's targeting, etc.     
**代码生成（Code-Generation）：** 获取AST并将其转换为可执行代码的过程被称为代码生成。根据不同的语言，目标平台等而有所不同。
 
    So, rather than get mired in details, we'll just handwave and say that there's a way to take our above described AST for `var a = 2;` and turn it into a set of machine instructions to actually *create* a variable called `a` (including reserving memory, etc.), and then store a value into `a`.     
    抛开具体细节，简单来说有一种方法将`var a = 2;`的我们上面提到的AST变成一组机器指令去*创建*一个称为`a`的变量（包括分配内存等），然后将值存储到`a`中。
    
    **Note:** The details of how the engine manages system resources are deeper than we will dig, so we'll just take it for granted that the engine is able to create and store variables as needed.       
    **注意:** 关于引擎管理系统资源的细节超出了讨论范围，我们只要知道引擎能够根据需要创建和存储变量就可以了。

The JavaScript engine is vastly more complex than *just* those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing redundant elements, etc.      
JavaScript引擎比那些*只有*三个步骤的语言编译器复杂得多。例如，在解析和代码生成的过程中，肯定有步骤来优化执行的性能，包括折叠冗余元素等。

So, I'm painting only with broad strokes here. But I think you'll see shortly why *these* details we *do* cover, even at a high level, are relevant.       
所以，我在这里只做宏观、简单地介绍。但我想你会很快看到为什么我们会覆盖到这么高难度的内容，这与我么要讨论的事有所关联。

For one thing, JavaScript engines don't get the luxury (like other language compilers) of having plenty of time to optimize, because JavaScript compilation doesn't happen in a build step ahead of time, as with other languages.    
首先，JavaScript引擎不会像其他语言编译器那样花费大量时间来优化，因为JavaScript编译不会像其他语言一样在提前的构建步骤中发生。

For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like JITs, which lazy compile and even hot re-compile, etc.) which are well beyond the "scope" of our discussion here.      
对于JavaScript，执行前的bianyi编译大多数情况下只用几微秒（甚至更少），为了确保最佳性能，JS引擎使用各种各样的技巧（如JIT，延迟编译甚至实施重编译等），这些都超出了我们在这里讨论的“作用域”范围。

Let's just say, for simplicity's sake, that any snippet of JavaScript has to be compiled before (usually *right* before!) it's executed. So, the JS compiler will take the program `var a = 2;` and compile it *first*, and then be ready to execute it, usually right away.      
简单来说，任何JavaScript代码片段必须在执行之前编译（通常*就是在*执行之前！）。所以，JS编译器*首先*将对程序`var a = 2;`进行编译，然后准备执行它，通常是立刻执行。

## 理解作用域（Understanding Scope）

The way we will approach learning about scope is to think of the process in terms of a conversation. But, *who* is having the conversation?    
我们学习范围的方法是将这个过程想象成几个人在对话。那么，是*谁*在谈话？

| 单词 | 音标 | 释义 |
| ---- | ---- | ---- |
|concept|['kɒnsept]|n. 观念，概念|
|academic|[ækə'demɪk]|adj. 学术的；理论的；学院的 n.大学生，大学教师；学者|
|tokenizer||n. 分词器；编译器|
|invoke|[ɪn'vəʊk]|vt. 调用；祈求；引起；恳求|
|occurs||v. 重现（occur的第三人称单数）|

### The Cast --(张静)

Let's meet the cast of characters that interact to process the program `var a = 2;`, so we understand their conversations that we'll listen in on shortly:

1. *Engine*: responsible for start-to-finish compilation and execution of our JavaScript program.

2. *Compiler*: one of *Engine*'s friends; handles all the dirty work of parsing and code-generation (see previous section).

3. *Scope*: another friend of *Engine*; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code.

For you to *fully understand* how JavaScript works, you need to begin to *think* like *Engine* (and friends) think, ask the questions they ask, and answer those questions the same.

### Back & Forth

When you see the program `var a = 2;`, you most likely think of that as one statement. But that's not how our new friend *Engine* sees it. In fact, *Engine* sees two distinct statements, one which *Compiler* will handle during compilation, and one which *Engine* will handle during execution.

So, let's break down how *Engine* and friends will approach the program `var a = 2;`.

The first thing *Compiler* will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree. But when *Compiler* gets to code-generation, it will treat this program somewhat differently than perhaps assumed.

A reasonable assumption would be that *Compiler* will produce code that could be summed up by this pseudo-code: "Allocate memory for a variable, label it `a`, then stick the value `2` into that variable." Unfortunately, that's not quite accurate.

*Compiler* will instead proceed as:

1. Encountering `var a`, *Compiler* asks *Scope* to see if a variable `a` already exists for that particular scope collection. If so, *Compiler* ignores this declaration and moves on. Otherwise, *Compiler* asks *Scope* to declare a new variable called `a` for that scope collection.

2. *Compiler* then produces code for *Engine* to later execute, to handle the `a = 2` assignment. The code *Engine* runs will first ask *Scope* if there is a variable called `a` accessible in the current scope collection. If so, *Engine* uses that variable. If not, *Engine* looks *elsewhere* (see nested *Scope* section below).

If *Engine* eventually finds a variable, it assigns the value `2` to it. If not, *Engine* will raise its hand and yell out an error!

To summarize: two distinct actions are taken for a variable assignment: First, *Compiler* declares a variable (if not previously declared in the current scope), and second, when executing, *Engine* looks up the variable in *Scope* and assigns to it, if found.

### Compiler Speak

We need a little bit more compiler terminology to proceed further with understanding.

When *Engine* executes the code that *Compiler* produced for step (2), it has to look-up the variable `a` to see if it has been declared, and this look-up is consulting *Scope*. But the type of look-up *Engine* performs affects the outcome of the look-up.

In our case, it is said that *Engine* would be performing an "LHS" look-up for the variable `a`. The other type of look-up is called "RHS".

I bet you can guess what the "L" and "R" mean. These terms stand for "Left-hand Side" and "Right-hand Side".

Side... of what? **Of an assignment operation.**

In other words, an LHS look-up is done when a variable appears on the left-hand side of an assignment operation, and an RHS look-up is done when a variable appears on the right-hand side of an assignment operation.

Actually, let's be a little more precise. An RHS look-up is indistinguishable, for our purposes, from simply a look-up of the value of some variable, whereas the LHS look-up is trying to find the variable container itself, so that it can assign. In this way, RHS doesn't *really* mean "right-hand side of an assignment" per se, it just, more accurately, means "not left-hand side".

Being slightly glib for a moment, you could also think "RHS" instead means "retrieve his/her source (value)", implying that RHS means "go get the value of...".

Let's dig into that deeper.

When I say: --(翠翠)                     
思考以下代码: --(翠翠)

```js
console.log( a );
```

The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we're looking-up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`.                                    
`a`的引用是一个RHS引用，因为这里没有任何值分配给`a`.相反，我们查找得到`a`的值，这样值就可以传递给`console.log(...)`.                       

By contrast:             
相比之下：                       

```js
a = 2;
```

The reference to `a` here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.   
这里`a`的引用则是一个LHS引用，因为我们实际上并不关心当前的值是什么，我们仅仅想找到变量作为一个目标关于`=2`赋值操作.                                   

**Note:** LHS and RHS meaning "left/right-hand side of an assignment" doesn't necessarily literally mean "left/right side of the `=` assignment operator". There are several other ways that assignments happen, and so it's better to conceptually think about it as: "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".                       
**注意:** LHS和RHS的意思是"赋值操作的左边/右边"，不一定就是字面上的意思"`=`赋值操作的左边/右边"。发生赋值操作还有其他几种方式，所以最好是在概念上理解它:"谁是赋值操作的目标(LHS)"和"谁是赋值操作的来源(RHS)".                             

Consider this program, which has both LHS and RHS references:           
思考以下程序，LHS和RHS都有引用:                    

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

The last line that invokes `foo(..)` as a function call requires an RHS reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me." Moreover, `(..)` means the value of `foo` should be executed, so it'd better actually be a function!                        
最后一行调用`foo(..)`函数需要对`foo`做一个RHS引用,意味着,"去查找`foo`的值,并把它给我."然而,`(..)`意味着`foo`的值应该被执行,所以它最好是一个函数！                     

There's a subtle but important assignment here. **Did you spot it?**                        
这里有一个容易忽略但是又很重要的细节在赋值操作中. **你发现它了吗？**                     

You may have missed the implied `a = 2` in this code snippet. It happens when the value `2` is passed as an argument to the `foo(..)` function, in which case the `2` value is **assigned** to the parameter `a`. To (implicitly) assign to parameter `a`, an LHS look-up is performed.                             
你也许忽略掉了隐式的`a=2`在这段代码片段中。当值`2`作为参数传递给`foo(..)`函数时,`2`这个值就会**分配**给参数`a`.为了(隐式的)分配给参数`a`，一个LHS查找就会执行.                              

There's also an RHS reference for the value of `a`, and that resulting value is passed to `console.log(..)`. `console.log(..)` needs a reference to execute. It's an RHS look-up for the `console` object, then a property-resolution occurs to see if it has a method called `log`.                              
这里值`a`也有RHS引用,并且结果值指向`console.log(..)`.`console.log(..)`需要一个引用来执行.这是一个RHS查询关于`console`对象,然后查询得到的值中是否有一个叫`log`的方法.                       

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value `2` (by way of variable `a`'s RHS look-up) into `log(..)`. Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which (perhaps called `arg1`) has an LHS reference look-up, before assigning `2` to it.                             
最后，我们可以概念化理解LHS/RHS交换传递值`2`(通过变量`a`的RHS查询方式)在`log(..)`中。`log(..)`的内部本地实现，我们可以假设它有参数，在把`2`分配给它之前，它的第一个参数(也许被叫做`arg1`)做了一个LHS引用查找.          

**Note:** You might be tempted to conceptualize the function declaration `function foo(a) {...` as a normal variable declaration and assignment, such as `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.                        
**注意:** 你可能有意概念化的将函数声明`function foo(a) {...`作为一个标准的变量声明和赋值,如`var foo`和`foo = function(a){...`.这样做,将会有意认为这个函数声明执行了LHS查询.                

However, the subtle but important difference is that *Compiler* handles both the declaration and the value definition during code-generation, such that when *Engine* is executing code, there's no processing necessary to "assign" a function value to `foo`. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.                       
然而,容易忽略但很重要的区别在于 *编译器(Compiler)* 操纵着声明和值的定义在代码产生中,如当 *引擎(Engine)* 执行着代码时,这里没有进程必须将一个函数值分配给`foo`.因此,这里我们讨论认为将一个函数声明作为LHS查询赋值的方式不是真正的合适                        

### Engine/Scope Conversation  --(张雪)

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ...

### Quiz

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

2. Identify all the RHS look-ups (there are 4!).

**Note:** See the chapter review for the quiz answers!

## Nested Scope  --(张润)

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

## 嵌套作用域
我们所说的 *作用域*是在根据特定名称查找某个变量时所要遵循的一些规则.然而,我们通常所要考虑的不仅仅是一个*作用域*这么简单.

如下：

Consider:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

`b`右侧的参数引用不能被`foo`函数内部解析,但是它能够被其所属的`作用域`解析(在这个例子中,是全局).

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

因此,重新回顾 *Engine* 和 *Scope* 之间的会话,我们将能窃听到如下会话:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***Engine***: "Hey, `foo`中的 *作用域*,你听说过`b`么?它的右边有一个参数."

> ***Scope***: "Nope, never heard of it. Go fish."

> ***Scope***: "没有, 从来没听说过. 走开."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***Engine***: "Hey, `foo`外部的*作用域*, oh是一个全局的 *作用域*, ok cool. 听说过`b`吗? 它的右边有一个变量."

> ***Scope***: "Yep, 当然. 这有儿有你要的."




The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

遍历嵌套的*作用域*的规则很简单: *Engine* 从当前*作用域*开始执行,寻找变量,如果没找到,再从下一级开始找,如此循环.当找到最外层的全局作用域时,无论是否找到该变量,都会停止查找.

### Building on Metaphors

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

为了使这个嵌套*作用域*的过程可视化,我希望你想象一座高层建筑.

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

这个建筑代表着我们程序的嵌套`作用域`的规则集.建筑的第一层代表着当前执行程序的`作用域`,无论你处于什么位置.该建筑的最高层都是全局`作用域`.

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

你可以通过查找本楼层作用域来解析等号左边和右边的引用,但是如果在本楼层作用域中找不到对应的变量引用,就需要乘坐电梯到下一楼层寻找,以此类推.当你到达了顶层(即全局作用域),同样开始查找,但是没有找到,无疑你应该到此为止停止继续找下去了.



## Errors

Why does it matter whether we call it LHS or RHS?

为什么我们如此强调LHS或者RHS

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

因为当一个变量还没有被声明(即在任何`作用域`中都无法被解析)的时候,这两种类型的查找行为是全然不同的.

如下:

Consider:

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );   // ‘b’ is not defined
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.  --(李欣)
第一次对`b`进行RHS查询时，是无法找到的。也就是说`b`是一个`未声明`的变量，因为在作用域中无法找到它

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.
如果RHS查询在各级嵌套的作用域中都无法找到变量，那么*引擎*会抛出一个`ReferenceError`错误。 `ReferenceError`是非常重要的一种错误类型。

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*.
相比之下，如果*引擎*在执行LHS查询时，如果直到最顶端（全局*作用域*）依然未能找到，并且程序是运行在“严格模式”下，那么在*全局作用域*中会创建一个具有该名称的变量，并将其返回给*引擎*

*"No, there wasn't one before, but I was helpful and created one for you."*
*”不，这个变量并不存在，但是我很乐于助人的帮你创建了一个“*

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.
”严格模式“是在ES5中被加入的，它与普通/宽松/懒惰模式有很多不同，其中一个是在严格模式中禁止自动或隐式创建全局变量。在这种情况下，LHS查询失败后不会自动创建一个全局变量并返回，此时*引擎*会抛出一个同RHS查询失败时类似的`ReferenceError`错误

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.
现在，如果一个变量在RHS查询时被找到，但此时你想要对这个变量的值做一些不合理的操作，比如对一个非函数类型的值执行函数调用，或者引用`null`或`undefined`，这样的操作是无效的并且*引擎*会抛出另一种叫做`TypeError`的错误

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.
`ReferenceError`与*作用域*判别失败相关，`TypeError`则表示作用域判断成功，但是对结果的操作是非法或不合理的。

## Review (TL;DR)
## 回顾

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.
作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。如果查找的目的是为了对变量进行赋值，那么就会使用LHS（left-hand-side）查询，如果目的是获取变量的值，就使用RHS（right-hand-side）查询。

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.
赋值操作会导致LHS查询， `=`运算符或调用函数时的参数传递都会导致关联*作用域*的赋值操作。

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:
JavaScript*引擎*在代码执行前会先进行编译，在这个过程中，它会将类似`var a = 2`这样的声明语句分解成2个独立的步骤：

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.
1. 首先，在所在*作用域*通过`var a`声明变量，这个过程会在最开始执行，也就是代码执行前的进行。

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.
2. 接着，`a = 2`会查找变量（LHS查询）并且在找到后对其进行赋值。

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.
LHS和RHS查询都是在当前执行*作用域*开始，如果有需要（也就是说，没有找到所需的变量），就会继续向上级作用域继续查找，每次上升一级作用域（1层），最终抵达全局作用域（顶层），无论找到或者没找到都会停止。

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).
执行失败的RHS引用会抛出一个`ReferenceError`错误。执行失败的LHS引用会导致自动隐式的创建一个全局变量（非“严格模式”），或抛出`ReferenceError`错误（”严格模式“）

### Quiz Answers
### 测验

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).
1. 找出所有的LHS查询（共有3处）

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).
2. 找出所有的RHS查询（共有4处）

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)


## 单词本

| 单词  | 音标   | 释义           |
| ---- | ----  | -------------  |
| LHS  |       | Left Hand Side |
| retrieve | 英 [rɪ'triːv]  美 [rɪ'triv] |vt. [计] 检索；恢复；重新得到vi. 找回猎物n. [计] 检索；恢复，取回|
| By contrast || 相比之下，与之相比 |
| literally | 英 ['lɪt(ə)rəlɪ]  美 ['lɪtərəli] | adv. 照字面地；逐字地；不夸张地；正确地；简直 |
| execute | 英 ['eksɪkjuːt]  美 ['ɛksɪkjut] | vt. 实行；执行；处死 |
| implementation | 英 [ɪmplɪmen'teɪʃ(ə)n] | n. [计] 实现；履行；安装启用 |
| assume | 英 [ə'sjuːm]  美 [ə'sum] | vi. 假定；设想；承担；采取vt. 僭取；篡夺；夺取；擅用；侵占|
| By contrast |  | 相比之下 |
| In that case |  | 在那种情况下 |
| whereas | ['wɛr'æz] | conj. 然而；鉴于；反之 |
| Unfulfilled | [,ʌnfʊl'fɪld] | adj. 未得到满足的；没有成就感的 |


## 疑难杂句
`It's an RHS look-up for the ``console` `object, then a property-resolution occurs to see if it has a method called` `log`.

`it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here`

