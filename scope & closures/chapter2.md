# 你不知道的JS：作用域和闭包（You Don't Know JS: Scope & Closures） --（罗尧）
# 第二章：词法作用域（Chapter 2: Lexical Scope）

In Chapter 1, we defined "scope" as the set of rules that govern how the *Engine* can look up a variable by its identifier name and find it, either in the current *Scope*, or in any of the *Nested Scopes* it's contained within.  
在第1章中，我们将“作用域”定义为一组规则，它们管理*引擎*如何在当前*作用域*中或在包含的任何*嵌套作用域*中通过其标识符名称查找变量。

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of programming languages. It's called **Lexical Scope**, and we will examine it in-depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc.) is called **Dynamic Scope**.  
作用域工作原理有两个主要模型。第一个是迄今为止最常见的，被绝大多数编程语言使用。它被称为**词法作用域（Lexical Scope）**，我们将深入研究它。 另一种模型仍然由一些语言（如Bash脚本，Perl中的一些模式等）使用被称为**动态作用域（Dynamic Scope）**。

Dynamic Scope is covered in Appendix A. I mention it here only to provide a contrast with Lexical Scope, which is the scope model that JavaScript employs.   
在附录A中会介绍动态作用域。我在这里提到它只是为了提供与词法作用域的对比，这是JavaScript采用的作用域模型。

## 词法阶段（Lex-time）

As we discussed in Chapter 1, the first traditional phase of a standard language compiler is called lexing (aka, tokenizing). If you recall, the lexing process examines a string of source code characters and assigns semantic meaning to the tokens as a result of some stateful parsing.   
正如我们在第1章中讨论的，标准语言编译器的第一个传统阶段称为词法化（又叫单词化）。如果你记得，词法化过程检查一串源代码字符，并为某些有状态的解析的过程赋予单词语义。

It is this concept which provides the foundation to understand what lexical scope is and where the name comes from.     正是这个概念提供了理解词法作用域是什么以及名字来自哪里的基础。 

To define it somewhat circularly, lexical scope is scope that is defined at lexing time. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code.   
简单来说，词法作用域是在词法分析阶段定义的作用域。换句话说，词法作用域基于变量和块作用域写在哪里，因此在词法分析器处理你的代码的时候不会改变（大部分是这样）。

**Note:** We will see in a little bit there are some ways to cheat lexical scope, thereby modifying it after the lexer has passed by, but these are frowned upon. It is considered best practice to treat lexical scope as, in fact, lexical-only, and thus entirely author-time in nature.     
**注意：**我们可以用一些方法来欺骗词法作用域，从而在词法分析器通过之后对其进行修改，但这些都不推荐。事实上，让词法作用域根据词法关系保持书写时的自然关系不变，是一个非常好的最佳实践。

Let's consider this block of code:      
让我们来思考下面一段代码   

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

There are three nested scopes inherent in this code example. It may be helpful to think about these scopes as bubbles inside of each other.  此代码示例中有三个嵌套作用域。 将这些作用域看作彼此内部的气泡可能会有所帮助。

<img src="/images/chapter2/fig2.png" width="500">

**Bubble 1** encompasses the global scope, and has just one identifier in it: `foo`. --(张静)
***气泡1*包含了“包含着整个全局作用域，其中只有一个标识符：foo。

**Bubble 2** encompasses the scope of `foo`, which includes the three identifiers: `a`, `bar` and `b`.
**气泡2**包含着`foo`所创建的作用域，其中有三个标识符：`a`、`bar`h和`b`。

**Bubble 3** encompasses the scope of `bar`, and it includes just one identifier: `c`.
**气泡3**包含着`bar`所创建的作用域，其中只有一个标识符：`c`。

Scope bubbles are defined by where the blocks of scope are written, which one is nested inside the other, etc. In the next chapter, we'll discuss different units of scope, but for now, let's just assume that each function creates a new bubble of scope.
作用域气泡由其对应的作用域块代码写在哪里决定，它们是逐级包含的。下一章会讨论不同类型的作用域，但现在，只要假设每一个函数都会创建一个新的作用域气泡就好了。

The bubble for `bar` is entirely contained within the bubble for `foo`, because (and only because) that's where we chose to define the function `bar`.
`bar`的气泡被完全包含在`foo`所创建的气泡中，唯一的原因是那里就是我们希望定义函数`bar`的位置。

Notice that these nested bubbles are strictly nested. We're not talking about Venn diagrams where the bubbles can cross boundaries. In other words, no bubble for some function can simultaneously exist (partially) inside two other outer scope bubbles, just as no function can partially be inside each of two parent functions.
注意，这里所说的气泡是严格包含的。我们并不是在讨论（Venn diagrams）文氏图1这种可以跨越边界的气泡。换句话说，没有任何函数的气泡可以（部分地）同时出现在两个外部作用域的气泡中，就如同没有任何函数可以部分地同时出现在两个父级函数中一样。

### 查找（Look-ups）

The structure and relative placement of these scope bubbles fully explains to the *Engine* all the places it needs to look to find an identifier.
这些作用域气泡的结构和位置关系给引擎提供了足够的用于查找标识符的位置的信息。

In the above code snippet, the *Engine* executes the `console.log(..)` statement and goes looking for the three referenced variables `a`, `b`, and `c`. It first starts with the innermost scope bubble, the scope of the `bar(..)` function. It won't find `a` there, so it goes up one level, out to the next nearest scope bubble, the scope of `foo(..)`. It finds `a` there, and so it uses that `a`. Same thing for `b`. But `c`, it does find inside of `bar(..)`.
在上一个代码片段中，引擎执行`console.log(..)`声明，并查找`a`、`b`和`c`三个变量的引用。它首先从最内部的作用域，也就是`bar(..)`函数的作用域气泡开始查找。引擎将在这找不到`a`，因此会去上一级，到所嵌套的`foo(..)`的作用域中继续查找。在这里找到了`a`，因此引擎使用了这个引用。对`b`来讲也是一样的。而对`c`来说，引擎在`bar(..)`中就找到了它。

Had there been a `c` both inside of `bar(..)` and inside of `foo(..)`, the `console.log(..)` statement would have found and used the one in `bar(..)`, never getting to the one in `foo(..)`.
如果a、c都存在于bar(..)和foo(..)的内部，`console.log(..)`就可以直接使用`bar(..)`中的变量，而不用到外面的`foo(..)`中查找。

**Scope look-up stops once it finds the first match**. The same identifier name can be specified at multiple layers of nested scope, which is called "shadowing" (the inner identifier "shadows" the outer identifier). Regardless of shadowing, scope look-up always starts at the innermost scope being executed at the time, and works its way outward/upward until the first match, and stops.
**作用域查找会在找到第一个匹配的标识符时停止**。在多层的嵌套作用域中可以定义同名的标识符，这叫作“遮蔽效应（shadowing）”（内部的标识符“遮蔽”了外部的标识符）。抛开遮蔽效应，作用域查找始终从运行时所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见第一个匹配的标识符为止。 

**Note:** Global variables are also automatically properties of the global object (`window` in browsers, etc.), so it *is* possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object.
**笔记:** 全局变量会自动成为全局对象（比如浏览器中的window对象）的属性，因此可以不直接通过全局对象的词法名称，而是间接地通过对全局对象属性的引用来对其进行访问。

```js
window.a
```

This technique gives access to a global variable which would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed.
通过这种技术可以访问那些被同名变量所遮蔽的全局变量。但非全局的变量如果被遮蔽了，无论如何都无法被访问到。

No matter *where* a function is invoked from, or even *how* it is invoked, its lexical scope is **only** defined by where the function was declared.
无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。

The lexical scope look-up process *only* applies to first-class identifiers, such as the `a`, `b`, and `c`. If you had a reference to `foo.bar.baz` in a piece of code, the lexical scope look-up would apply to finding the `foo` identifier, but once it locates that variable, object property-access rules take over to resolve the `bar` and `baz` properties, respectively.
词法作用域查找只会查找一级标识符，比如a、b和c。如果代码中引用了｀foo.bar.baz｀，词法作用域查找只会试图查找｀foo｀标识符，找到这个变量后，对象属性访问规则会分别接管对｀bar｀和｀baz｀属性的访问。

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
JS里有一些其他的方法，有着和`eval(..)`. `setTimeout(..)` 以及 `setInterval(..)` 相似的作用，这些方法接受一个字符串作为第一个参数，并将其作为动态生成的函数代码执行。这是一种古老的方法，现在已经不推荐使用了。

The `new Function(..)` function constructor similarly takes a string of code in its **last** argument to turn into a dynamically-generated function (the first argument(s), if any, are the named parameters for the new function). This function-constructor syntax is slightly safer than `eval(..)`, but it should still be avoided in your code.
函数构造器`new Function(..)`也有类似的的作用，它接受一个字符串作为 **最后** 一个参数并将其转换成动态生成的函数代码（如果构造器有第一个参数，第一个参数会被当成新函数的形参），这种函数构造语法比`eval(..)`安全一点，但也应尽量避免使用。

The use-cases for dynamically generating code inside your program are incredibly rare, as the performance degradations are almost never worth the capability.
你很少会在程序里用到动态生成代码，而且动态生成代码会带来性能下降，不值得使用。

### `with`

The other frowned-upon (and now deprecated!) feature in JavaScript which cheats lexical scope is the `with` keyword. There are multiple valid ways that `with` can be explained, but I will choose here to explain it from the perspective of how it interacts with and affects lexical scope.
另一个不推荐并且已经弃用JS特性`with`,`with`能欺骗词法作用域。`with`的用法可以从很多种角度解释，这里我会从它如何与词法作用域交互以及对词法作用域的影响来解释。

`with` is typically explained as a short-hand for making multiple property references against an object *without* repeating the object reference itself each time.
`with` 常常被用作快速引用一个对象中的多个属性，这样可以避免每次对对象的重复引用。

For example:
例如：

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// 这样写很枯燥
obj.a = 2;
obj.b = 3;
obj.c = 4;

// 这样写很便捷
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

-- （张润）
However, there's much more going on here than just a convenient short-hand for object property access. Consider:

无论如何,这都代表着更多的意义,而不单单是为了方便快速查找对象属性访问权

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


在这段代码实例中,两个对象o1和o2被创建了.其中一个有a属性,而另一个没有.`foo(..)`这个函数将`obj`这个对象的引用作为参数传递,同时调用了`with（obj）`这个函数的引用.在`with`这个块作用域内,我们看似只是对`a`这个变量做了一个普通的词法引用,实际上是一个`LHS`引用,将2赋值给它.


When we pass in `o1`, the `a = 2` assignment finds the property `o1.a` and assigns it the value `2`, as reflected in the subsequent `console.log(o1.a)` statement. However, when we pass in `o2`, since it does not have an `a` property, no such property is created, and `o2.a` remains `undefined`.


当我们传递`o1`作为参数时,`a=2`这个操作找到`o1.a` ,同时将2赋值给o1.a,如同`console.log(01.a)`所体现的一样.然而,当我们传递`o2`这个参数过去时,因为`o2`中没有`a`这个属性,也没有这样的属性会被创建，则`o2.a`就是`undefined`了.


But then we note a peculiar side-effect, the fact that a global variable `a` was created by the `a = 2` assignment. How can this be?

但是我们注意到一个奇怪的副作用, `a = 2`这个操作产生了一个全局变量`a`,这是为什么呢?

The `with` statement takes an object, one which has zero or more properties, and **treats that object as if *it* is a wholly separate lexical scope**, and thus the object's properties are treated as lexically defined identifiers in that "scope".

`with`可以将一个没有或者有很多属性的对象**隔离成一个完全分离的词法作用域**,因此在这个作用域中的对象的属性也被认为是该作用域中的词法标识符

**Note:** Even though a `with` block treats an object like a lexical scope, a normal `var` declaration inside that `with` block will not be scoped to that `with` block, but instead the containing function scope.

**Note:**即使一个`with`块将一个对象当做是一个词法作用域,而一个正常用`var`声明的内部变量将不会被包含在`with`块中,而是包含在调用`with`所在的函数作用域中.

While the `eval(..)` function can modify existing lexical scope if it takes a string of code with one or more declarations in it, the `with` statement actually creates a **whole new lexical scope** out of thin air, from the object you pass to it.

当有一些代码声明在函数内部时,`eval(..)`函数就能够修改现存的词法作用域,而实际情况下`with`声明则根据你传进来的对象凭空创建了**一个全新的词法作用域**


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
如果代码中大量使用` eval(..)`或`with`，那么运行起来一定会变得非常慢。无论*引擎*多聪明，**试图将这些悲观情况的副作用限制在最小范围内，也无法避免如果没有这些优化，代码会运行得更慢这个事实。**

## Review (TL;DR)
## 回顾

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.
词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。变异的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.
JavaScript中有两个机制可以“欺骗”此法作用域：`eval(..)`和`with`。前者可以对一段包含一个或多个声明的“代码”字符串进行演算，并借此来修改已经存在的词法作用域（在运行时）。后者本质上是通过将一个对象的引用当做作用域来处理，将对象的属性*当做*作用域中的"标识符"来处理，从而创建了一个新的词法作用域（同样是在运行时）。

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**
这两个机制的副作用是*引擎*无法在编译时对作用域查找进行优化，因为*引擎*只能谨慎地认为这样的优化是无效的。使用这其中任何一个机制都将导致代码运行变慢。**不用使用它们**

## 单词本

| 单词 | 音标 | 释义 |
| ---- | ---- | ---- |
| deprecated | ['deprɪkeɪt] | 不赞成；祈免；贬低 |
| lexical | ['leksɪk(ə)l] | 词汇的 |
| innermost | ['ɪnəməʊst] | adj. 内心的；最里面的，最深处的；秘密的 |
| simultaneously | [,sɪml'teɪnɪəslɪ] | adv. 同时地 |
| diagrams | ['daɪə,græm] | n. 图表；[数] 图解（diagram的复数）；略图 v. 用图解释；作…的图解（diagram的三单形式） |
| encompass | [ɪn'kʌmpəs; en-] | vt. 包含；包围，环绕；完成 |
| entirely | [ɪn'taɪɚli] | adv. 完全地，彻底地 |
| treat | [trit] | vt. 治疗；对待；探讨；视为vi. 探讨；请客；协商n. 请客；款待|
| previous | ['privɪəs] |adj. 以前的；早先的；过早的adv. 在先；在…以前 |
| dynamically | [daɪ'næmɪkli] | adv. 动态地；充满活力地；不断变化地 |
| interpret | [ɪn'tɝprɪt] | vt. 说明；口译vi. 解释；翻译 |
| essentially | [ɪ'sɛnʃəli] | adv. 本质上；本来 |
| indirectly | [,ɪndə'rɛktli] | adv. 间接地；不诚实；迂回地 |
| lexical |  | 词汇的 |
| peculiar | | 奇异的 |
| lexically defined identifiers | | 词法标识符 |
| statement | | 声明 |
| out of thin air | | 凭空 |

## 疑难杂句
* It is considered best practice to treat lexical scope as, in fact, lexical-only, and thus entirely author-time in nature.  
  - 官方翻译：事实上，让词法作用域根据词法关系保持书写时的自然关系不变，是一个非常好的最佳实践。 
  - 罗尧：最好的做法是将词汇作为事实上的词法作用域来处理。   
