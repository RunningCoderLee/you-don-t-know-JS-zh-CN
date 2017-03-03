# You Don't Know JS: Scope & Closures --（罗尧）
# Chapter 2: Lexical Scope

In Chapter 1, we defined "scope" as the set of rules that govern how the *Engine* can look up a variable by its identifier name and find it, either in the current *Scope*, or in any of the *Nested Scopes* it's contained within.

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of programming languages. It's called **Lexical Scope**, and we will examine it in-depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc.) is called **Dynamic Scope**.

Dynamic Scope is covered in Appendix A. I mention it here only to provide a contrast with Lexical Scope, which is the scope model that JavaScript employs.

## Lex-time

As we discussed in Chapter 1, the first traditional phase of a standard language compiler is called lexing (aka, tokenizing). If you recall, the lexing process examines a string of source code characters and assigns semantic meaning to the tokens as a result of some stateful parsing.

It is this concept which provides the foundation to understand what lexical scope is and where the name comes from.

To define it somewhat circularly, lexical scope is scope that is defined at lexing time. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code.

**Note:** We will see in a little bit there are some ways to cheat lexical scope, thereby modifying it after the lexer has passed by, but these are frowned upon. It is considered best practice to treat lexical scope as, in fact, lexical-only, and thus entirely author-time in nature.

Let's consider this block of code:

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

There are three nested scopes inherent in this code example. It may be helpful to think about these scopes as bubbles inside of each other.

<img src="fig2.png" width="500">

**Bubble 1** encompasses the global scope, and has just one identifier in it: `foo`. --(张静)

**Bubble 2** encompasses the scope of `foo`, which includes the three identifiers: `a`, `bar` and `b`.

**Bubble 3** encompasses the scope of `bar`, and it includes just one identifier: `c`.

Scope bubbles are defined by where the blocks of scope are written, which one is nested inside the other, etc. In the next chapter, we'll discuss different units of scope, but for now, let's just assume that each function creates a new bubble of scope.

The bubble for `bar` is entirely contained within the bubble for `foo`, because (and only because) that's where we chose to define the function `bar`.

Notice that these nested bubbles are strictly nested. We're not talking about Venn diagrams where the bubbles can cross boundaries. In other words, no bubble for some function can simultaneously exist (partially) inside two other outer scope bubbles, just as no function can partially be inside each of two parent functions.

### Look-ups

The structure and relative placement of these scope bubbles fully explains to the *Engine* all the places it needs to look to find an identifier.

In the above code snippet, the *Engine* executes the `console.log(..)` statement and goes looking for the three referenced variables `a`, `b`, and `c`. It first starts with the innermost scope bubble, the scope of the `bar(..)` function. It won't find `a` there, so it goes up one level, out to the next nearest scope bubble, the scope of `foo(..)`. It finds `a` there, and so it uses that `a`. Same thing for `b`. But `c`, it does find inside of `bar(..)`.

Had there been a `c` both inside of `bar(..)` and inside of `foo(..)`, the `console.log(..)` statement would have found and used the one in `bar(..)`, never getting to the one in `foo(..)`.

**Scope look-up stops once it finds the first match**. The same identifier name can be specified at multiple layers of nested scope, which is called "shadowing" (the inner identifier "shadows" the outer identifier). Regardless of shadowing, scope look-up always starts at the innermost scope being executed at the time, and works its way outward/upward until the first match, and stops.

**Note:** Global variables are also automatically properties of the global object (`window` in browsers, etc.), so it *is* possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object.

```js
window.a
```

This technique gives access to a global variable which would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed.

No matter *where* a function is invoked from, or even *how* it is invoked, its lexical scope is **only** defined by where the function was declared.

The lexical scope look-up process *only* applies to first-class identifiers, such as the `a`, `b`, and `c`. If you had a reference to `foo.bar.baz` in a piece of code, the lexical scope look-up would apply to finding the `foo` identifier, but once it locates that variable, object property-access rules take over to resolve the `bar` and `baz` properties, respectively.

## Cheating Lexical  --（翠翠）            
## 欺骗的词法 --（翠翠）                         

If lexical scope is defined only by where a function is declared, which is entirely an author-time decision, how could there possibly be a way to "modify" (aka, cheat) lexical scope at run-time?                        
如果词法作用域被定义在一个仅仅是函数声明的地方,这完全地是一个创作时的决定,怎么样才可能有一种方法在运行的时候"修改"(即,欺骗)词法作用域？                      

JavaScript has two such mechanisms. Both of them are equally frowned-upon in the wider community as bad practices to use in your code. But the typical arguments against them are often missing the most important point: **cheating lexical scope leads to poorer performance.**                    
JavaScript种有两种这样的机制.在广泛的社区觉得这两种机制使用在你的代码中都是一些糟糕的练习.但是典型的反对他们常常会忽略最重要的一点: **欺骗的词法作用域导致性能降低.**                     

Before I explain the performance issue, though, let's look at how these two mechanisms work.                      
但在我解释性能问题之前,让我们先看这两种机制是如何工作的.                           

### `eval`

The `eval(..)` function in JavaScript takes a string as an argument, and treats the contents of the string as if it had actually been authored code at that point in the program. In other words, you can programmatically generate code inside of your authored code, and run the generated code as if it had been there at author time.                                 
在JavaAcript中`eval(...)`函数可以接受一个字符串作为参数,并且字符串的内容被视为好像完全的授权在程序的代码那个位置.换句话说,你可以在编写代码里面以编程方式生成代码,并且运行生成的代码好像在已经才那里.              

Evaluating `eval(..)` (pun intended) in that light, it should be clear how `eval(..)` allows you to modify the lexical scope environment by cheating and pretending that author-time (aka, lexical) code was there all along.                            
这样去领悟`eval(..)`(一语双关)就很简单了,`eval(..)`是怎样通过欺骗和伪装创作时(即,词法)代码允许你修改词法作用域环境,这就很清楚了.                   

On subsequent lines of code after an `eval(..)` has executed, the *Engine* will not "know" or "care" that the previous code in question was dynamically interpreted and thus modified the lexical scope environment. The *Engine* will simply perform its lexical scope look-ups as it always does.                          
在执行`eval(..)`之后一行的代码,*引擎* 并不"知道"或者"在乎"先前的代码是动态的解释这个问题因此修改了词法作用域环境.*引擎* 总是只会执行它的词法作用域查找.                 

Consider the following code:              
思考下面代码:                 

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1, 3
```

The string `"var b = 3;"` is treated, at the point of the `eval(..)` call, as code that was there all along. Because that code happens to declare a new variable `b`, it modifies the existing lexical scope of `foo(..)`. In fact, as mentioned above, this code actually creates variable `b` inside of `foo(..)` that shadows the `b` that was declared in the outer (global) scope.                         
在`eval(..)`调用的位置，字符串`"var b = 3;"`作为代码一直在那里执行.因为代码发生了声明一个新的变量`b`,它修改了`foo(..)`已经存在的词法作用域.事实上,正如上面提到一样,这段代码实际上在`foo(..)`之内创建变量`b`,遮蔽外部（全局）作用域已经声明了的变量`b`.                                

When the `console.log(..)` call occurs, it finds both `a` and `b` in the scope of `foo(..)`, and never finds the outer `b`. Thus, we print out "1, 3" instead of "1, 2" as would have normally been the case.                                
当`console.log(..)`被调用时,在`foo(..)`作用域同时找到`a`和`b`,却从不会找到外部的`b`.因此,我们输出"1,3"而不是通常情况下的"1,2"。                        

**Note:** In this example, for simplicity's sake, the string of "code" we pass in was a fixed literal. But it could easily have been programmatically created by adding characters together based on your program's logic. `eval(..)` is usually used to execute dynamically created code, as dynamically evaluating essentially static code from a string literal would provide no real benefit to just authoring the code directly.                      
**注意:** 在这个例子中,为简单起见,我们传递的字符串"代码"是固定不变的.但是它很容易的以编程方式创建通过添加字符连在一起基于你程序的逻辑.`eval(..)`通常用来执行动态的创建代码,动态的执行本质上一个字符串文字静态代码,直接编写代码不会提供真正的好处.                    

By default, if a string of code that `eval(..)` executes contains one or more declarations (either variables or functions), this action modifies the existing lexical scope in which the `eval(..)` resides. Technically, `eval(..)` can be invoked "indirectly", through various tricks (beyond our discussion here), which causes it to instead execute in the context of the global scope, thus modifying it. But in either case, `eval(..)` can at runtime modify an author-time lexical scope.                          
默认情况下,如果一个字符串的代码`eval(..)`执行包含一个或多个声明（无论是变量还是函数）,就会触发修改已经存在的`eval(..)`所在的词法作用域.从技术上讲,`eval(..)`可以间接的被调用，通过各种技巧（这里超出了我们的讨论范围）,导致它在上下文的全局作用域中被修改.但在任何情况下,`eval(..)`能在运行时修改一个创作时的词法作用域.                               

**Note:** `eval(..)` when used in a strict-mode program operates in its own lexical scope, which means declarations made inside of the `eval()` do not actually modify the enclosing scope.                          
**注意:** 在严格模式程序中`eval(..)`使用运行时有自己的词法作用域,这就意味着`eval()`内部的声明不能实际的修改其封闭的作用域.                  

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

-- （张雪）
There are other facilities in JavaScript which amount to a very similar effect to `eval(..)`. `setTimeout(..)` and `setInterval(..)` *can* take a string for their respective first argument, the contents of which are `eval`uated as the code of a dynamically-generated function. This is old, legacy behavior and long-since deprecated. Don't do it!

The `new Function(..)` function constructor similarly takes a string of code in its **last** argument to turn into a dynamically-generated function (the first argument(s), if any, are the named parameters for the new function). This function-constructor syntax is slightly safer than `eval(..)`, but it should still be avoided in your code.

The use-cases for dynamically generating code inside your program are incredibly rare, as the performance degradations are almost never worth the capability.

### `with`

The other frowned-upon (and now deprecated!) feature in JavaScript which cheats lexical scope is the `with` keyword. There are multiple valid ways that `with` can be explained, but I will choose here to explain it from the perspective of how it interacts with and affects lexical scope.

`with` is typically explained as a short-hand for making multiple property references against an object *without* repeating the object reference itself each time.

For example:

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

-- （张润）
However, there's much more going on here than just a convenient short-hand for object property access. Consider:

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

In this code example, two objects `o1` and `o2` are created. One has an `a` property, and the other does not. The `foo(..)` function takes an object reference `obj` as an argument, and calls `with (obj) { .. }` on the reference. Inside the `with` block, we make what appears to be a normal lexical reference to a variable `a`, an LHS reference in fact (see Chapter 1), to assign to it the value of `2`.

When we pass in `o1`, the `a = 2` assignment finds the property `o1.a` and assigns it the value `2`, as reflected in the subsequent `console.log(o1.a)` statement. However, when we pass in `o2`, since it does not have an `a` property, no such property is created, and `o2.a` remains `undefined`.

But then we note a peculiar side-effect, the fact that a global variable `a` was created by the `a = 2` assignment. How can this be?

The `with` statement takes an object, one which has zero or more properties, and **treats that object as if *it* is a wholly separate lexical scope**, and thus the object's properties are treated as lexically defined identifiers in that "scope".

**Note:** Even though a `with` block treats an object like a lexical scope, a normal `var` declaration inside that `with` block will not be scoped to that `with` block, but instead the containing function scope.

While the `eval(..)` function can modify existing lexical scope if it takes a string of code with one or more declarations in it, the `with` statement actually creates a **whole new lexical scope** out of thin air, from the object you pass to it.


-- （李欣）
Understood in this way, the "scope" declared by the `with` statement when we passed in `o1` was `o1`, and that "scope" had an "identifier" in it which corresponds to the `o1.a` property. But when we used `o2` as the "scope", it had no such `a` "identifier" in it, and so the normal rules of LHS identifier look-up (see Chapter 1) occurred.
可以这样理解，当我们传递`o1`时，`with`语句所声明的作用域是`o1`，这个作用域中含有一个通`o1.a`属性一致的“标识符”。但当我们将`o2`作为“作用域”时，其中并没有`a`“标识符”，因此进行了正常的LHS标识符查询（参见第一章）

Neither the "scope" of `o2`, nor the scope of `foo(..)`, nor the global scope even, has an `a` identifier to be found, so when `a = 2` is executed, it results in the automatic-global being created (since we're in non-strict mode).
不论是o2的作用域，还是`foo(..)`的作用域，亦或者全局作用域中都没有找到标识符`a`，因此当`a=2`执行时，自动创建了一个全局变量（因为是非严格模式）

It is a strange sort of mind-bending thought to see `with` turning, at runtime, an object and its properties into a "scope" *with* "identifiers". But that is the clearest explanation I can give for the results we see.
*with*这种将对象及其属性放进一个`作用域`并同时分配“标识符”的行为很让人费解。但为了说明我们所看到的现象，这是我能给出的最直白的解释了。

**Note:** In addition to being a bad idea to use, both `eval(..)` and `with` are affected (restricted) by Strict Mode. `with` is outright disallowed, whereas various forms of indirect or unsafe `eval(..)` are disallowed while retaining the core functionality.
**注意：**另外一个不推荐使用`eval(..)`和`with`的原因是会被严格模式影响（限制）。`with`被完全禁止，而在保留核心功能的前提下，间接或非安全地使用`eval(..)`也被禁止了。

### Performance
### 性能

Both `eval(..)` and `with` cheat the otherwise author-time defined lexical scope by modifying or creating new lexical scope at runtime.
`eval(..)`和`with`会在运行时修改或创建新的作用域，从此来欺骗其他在书写时定义的词法作用域。

So, what's the big deal, you ask? If they offer more sophisticated functionality and coding flexibility, aren't these *good* features? **No.**
你可能会问，那又怎样呢？如果他们能实现更复杂的功能，并且代码更具有扩展性，难道不是非常好的功能吗？答案是否定的。

The JavaScript *Engine* has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre-determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution.
JavaScript*引擎*会在编译阶段进行大量的性能优化。其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。

But if the *Engine* finds an `eval(..)` or `with` in the code, it essentially has to *assume* that all its awareness of identifier location may be invalid, because it cannot know at lexing time exactly what code you may pass to `eval(..)` to modify the lexical scope, or the contents of the object you may pass to `with` to create a new lexical scope to be consulted.
但如果*引擎*在代码中发现`eval(..)`或`with，`，它只能简单地*假设*关于标识符位置的判断都是无效的，因为无法在此法分析阶段明确知道`eval(..) `会接收到什么代码，这些代码会如何对作用域进行修改，也无法知道传递给`with`用来创建新词法作用域的对象的内容到底是什么。

In other words, in the pessimistic sense, most of those optimizations it *would* make are pointless if `eval(..)` or `with` are present, so it simply doesn't perform the optimizations *at all*.
最悲观的情况是如果出现了`eval(..)`或`with`，所有的优化可能都是无意义的，因此最简单的做法就是*完全*不做任何优化。

Your code will almost certainly tend to run slower simply by the fact that you include an `eval(..)` or `with` anywhere in the code. No matter how smart the *Engine* may be about trying to limit the side-effects of these pessimistic assumptions, **there's no getting around the fact that without the optimizations, code runs slower.**
如果代码中大量使用` eval(..)`或`with`，那么运行起来一定会变得非常慢。无论*引擎*多聪明，识图将这些悲观情况的副作用限制在最小范围内，也无法避免如果没有这些优化，代码会运行得更慢这个事实。

## Review (TL;DR)
## 回顾

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.
词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。变异的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.
JavaScript中有两个机制可以“欺骗”此法作用域：eval(..)和with。前者可以对一段

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**

词 法 作 用 域 意 味 着 作 用 域 是 由 书 写 代 码 时 函 数 声 明 的 位 置 来 决 定 的。 编 译 的 词 法 分 析 阶 段 基 本 能 够 知 道 全 部 标 识 符 在 哪 里 以 及 是 如 何 声 明 的， 从 而 能 够 预 测 在 执 行 过 程 中 如 何 对 它 们 进 行 查 找。 JavaScript 中 有 两 个 机 制 可 以“ 欺 骗” 词 法 作 用 域： eval(..) 和 with。 前 者 可 以 对 一 段 包 含 一 个 或 多 个 声 明 的“ 代 码” 字 符 串 进 行 演 算， 并 借 此 来 修 改 已 经 存 在 的 词 法 作 用 域（ 在 运 行 时）。 后 者 本 质 上 是 通 过 将 一 个 对 象 的 引 用 当 作 作 用 域 来 处 理， 将 对 象 的 属 性 当 作 作 用 域 中 的 标 识 符 来 处 理， 从 而 创 建 了 一 个 新 的 词 法 作 用 域（ 同 样 是 在 运 行 时）。 这 两 个 机 制 的 副 作 用 是 引 擎 无 法 在 编 译 时 对 作 用 域 查 找 进 行 优 化， 因 为 引 擎 只 能 谨 慎 地 认 为 这 样 的 优 化 是 无 效 的。 使 用 这 其 中 任 何 一 个 机 制 都 将 导 致 代 码 运 行 变 慢。 不 要 使用他们


## 单词
| 单词 | 音标 | 词意 |
|------|------|------|
| entirely | [ɪn'taɪɚli] | adv. 完全地，彻底地 |
| treat | [trit] | vt. 治疗；对待；探讨；视为vi. 探讨；请客；协商n. 请客；款待|
| previous | ['privɪəs] |adj. 以前的；早先的；过早的adv. 在先；在…以前 |
| dynamically | [daɪ'næmɪkli] | adv. 动态地；充满活力地；不断变化地 |
| interpret | [ɪn'tɝprɪt] | vt. 说明；口译vi. 解释；翻译 |
| essentially | [ɪ'sɛnʃəli] | adv. 本质上；本来 |
| indirectly | [,ɪndə'rɛktli] | adv. 间接地；不诚实；迂回地 |
