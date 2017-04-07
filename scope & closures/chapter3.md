# 你不知道的JS：作用域和闭包（You Don't Know JS: Scope & Closures） --(罗尧)
# 第三章：函数作用域和块作用域（Chapter 3: Function vs. Block Scope）

As we explored in Chapter 2, scope consists of a series of "bubbles" that each act as a container or bucket, in which identifiers (variables, functions) are declared. These bubbles nest neatly inside each other, and this nesting is defined at author-time.  
正如我们在第2章中所探讨的，作用域由一系列“气泡”组成，每个气泡用作容器或桶，在其中声明标识符（变量，函数）。这些气泡整齐地嵌套在彼此内部，并且这种嵌套在创建时就被定义。

But what exactly makes a new bubble? Is it only the function? Can other structures in JavaScript create bubbles of scope?   
但是到底是什么创建了一个新的泡沫？就是函数创建的吗？JavaScript中的其他结构可以创建作用域气泡吗？

## 函数中的作用域（Scope From Functions）

The most common answer to those questions is that JavaScript has function-based scope. That is, each function you declare creates a bubble for itself, but no other structures create their own scope bubbles. As we'll see in just a little bit, this is not quite true.  
这些问题最常见的答案是JavaScript具有基于函数的作用域。也就是说，你声明的每个函数为自己创建一个气泡，没有其他结构创建自己的作用域气泡。这是错误的，我们会在稍后解释。

But first, let's explore function scope and its implications.    
首先让我们探讨函数作用域及其影响。

Consider this code:  
思考以下的代码：

```js
function foo(a) {
	var b = 2;

	// some code

	function bar() {
		// ...
	}

	// more code

	var c = 3;
}
```

In this snippet, the scope bubble for `foo(..)` includes identifiers `a`, `b`, `c` and `bar`. **It doesn't matter** *where* in the scope a declaration appears, the variable or function belongs to the containing scope bubble, regardless. We'll explore how exactly *that* works in the next chapter.   
在这个代码片段中，`foo(..)`的作用域气泡包括标识符`a`，`b`，`c`和`bar`。不管在作用域中的哪里声明的函数和变量都属于这个作用域气泡。我们将在下一章中探讨具体的工作原理。


`bar(..)` has its own scope bubble. So does the global scope, which has just one identifier attached to it: `foo`.   
`bar(..)`有自己的作用域气泡。全局作用域也是这样，它只有一个标识符：`foo`。

Because `a`, `b`, `c`, and `bar` all belong to the scope bubble of `foo(..)`, they are not accessible outside of `foo(..)`. That is, the following code would all result in `ReferenceError` errors, as the identifiers are not available to the global scope:   
因为`a`，`b`，`c`和`bar`都属于`foo(..)`的作用域气泡，它们不能在`foo(..)`外面访问。也就是说，以下代码将导致`ReferenceError`错误，全局作用域不能访问这些标识符：

```js
bar(); // 失败

console.log( a, b, c ); // 3个全失败
```

However, all these identifiers (`a`, `b`, `c`, `foo`, and `bar`) are accessible *inside* of `foo(..)`, and indeed also available inside of `bar(..)` (assuming there are no shadow identifier declarations inside `bar(..)`).    
但是，所有这些标识符（`a`，`b`，`c`，`foo`和`bar`）都可以访问`foo(..)`的*内部*被访问，同样也能在 `bar(..)`的内部被访问（这里假设在`bar(..)`内部没有同名的标识符）。

Function scope encourages the idea that all variables belong to the function, and can be used and reused throughout the entirety of the function (and indeed, accessible even to nested scopes). This design approach can be quite useful, and certainly can make full use of the "dynamic" nature of JavaScript variables to take on values of different types as needed.   
函数作用域的含义是指，属于函数的全部变量都在整个函数中使用和重用（甚至可以在嵌套的作用域中被访问）。这种设计方法非常有用，并且可以充分利用JavaScript变量的“动态”性质，根据需要接受不同类型的值。

On the other hand, if you don't take careful precautions, variables existing across the entirety of a scope can lead to some unexpected pitfalls.     
另一方方面，如果你谨慎处理那些在整个作用域范围内存被访问的变量可能会导致一些意想不到的问题。


## 隐藏内部实现（Hiding In Plain Scope）

The traditional way of thinking about functions is that you declare a function, and then add code inside it. But the inverse thinking is equally powerful and useful: take any arbitrary section of code you've written, and wrap a function declaration around it, which in effect "hides" the code.   
传统的函数思维方式是声明一个函数，然后在其中添加代码。但是逆向思维同样是强大和有用的：在你编写的任意代码段周围包装一个函数声明会“隐藏”代码。

The practical result is to create a scope bubble around the code in question, which means that any declarations (variable or function) in that code will now be tied to the scope of the new wrapping function, rather than the previously enclosing scope. In other words, you can "hide" variables and functions by enclosing them in the scope of a function.     
实际上是在代码周围创建了一个作用域气泡，意味着在该代码中任何的声明（变量或函数）都会绑定到新的包装函数的作用域，而不是先前所在的作用域。换句话说，你可以通过将变量和函数包含在函数的作用域内来“隐藏”变量和函数。

Why would "hiding" variables and functions be a useful technique?   
为什么“隐藏”变量和函数是一个有用的技术？

There's a variety of reasons motivating this scope-based hiding. They tend to arise from the software design principle "Principle of Least Privilege" [^note-leastprivilege], also sometimes called "Least Authority" or "Least Exposure". This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and "hide" everything else.   
提倡这种基于作用域的隐藏的原因有很多。它们大都引申自程序件设计原则中的“最低权限原则”[^note-leastprivilege]，有时也称为“最少权限”或“最少曝光”。这个原则声明，在程序设计中，例如某个模块或对象的API，你应该只暴露的必要的内容，并将其余部分“隐藏”起来。

This principle extends to the choice of which scope to contain variables and functions. If all variables and functions were in the global scope, they would of course be accessible to any nested scope. But this would violate the "Least..." principle in that you are (likely) exposing many variables or functions which you should otherwise keep private, as proper use of the code would discourage access to those variables/functions.     
这个原则可以延伸为如何选择作用域来包含变量和函数。如果所有变量和函数都在全局作用域内，它们当然可以被任何嵌套的作用域访问。但是这会违反“最小特权”原则，因为你（可能）暴露了你应该保持私有的许多变量或函数，正确的代码应该可以阻止对这些变量和函数的访问。

For example:  
例如：

```js
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

In this snippet, the `b` variable and the `doSomethingElse(..)` function are likely "private" details of how `doSomething(..)` does its job. Giving the enclosing scope "access" to `b` and `doSomethingElse(..)` is not only unnecessary but also possibly "dangerous", in that they may be used in unexpected ways, intentionally or not, and this may violate pre-condition assumptions of `doSomething(..)`.   
在这个代码片段中，变量`b`和函数`doSomethingElse(..)`可能是`doSomething(..)`内部具体实现的“私有”的内容。给外面的作用域“访问”`b`和`doSomethingElse(..)`的权限不仅是不必要的，而且可能是“危险的”，因为它们可能以意想不到的方式被有意或无意地使用，从而导致超出了`doSomething(..)`的适用条件。

A more "proper" design would hide these private details inside the scope of `doSomething(..)`, such as:     
一个更合理的设计应该是将这些私有的细节隐藏在`doSomething(..)`内部，例如：

```js
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

Now, `b` and `doSomethingElse(..)` are not accessible to any outside influence, instead controlled only by `doSomething(..)`. The functionality and end-result has not been affected, but the design keeps private details private, which is usually considered better software.    
现在，`b`和`doSomethingElse（..）`都不能从外部访问，而只能由`doSomething（..）`控制。功能和最终结果没有受到影响，但设计上保持了私有实现细节，这通常被认为是更好的程序。

### 避免冲突（Collision Avoidance）   --（张静）

Another benefit of "hiding" variables and functions inside a scope is to avoid unintended collision between two different identifiers with the same name but different intended usages. Collision results often in unexpected overwriting of values.                                                                                
"隐藏"作用域内的变量和函数的另一个好处，就是避免两个不同标识符之间同名字但不同用途的无意识的冲突。冲突经常导致变量的值被意外覆盖。

For example:                                                                             
例如：                                                                   


```js
function foo() {
	function bar(a) {
		i = 3; // changing the `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

The `i = 3` assignment inside of `bar(..)` overwrites, unexpectedly, the `i` that was declared in `foo(..)` at the for-loop. In this case, it will result in an infinite loop, because `i` is set to a fixed value of `3` and that will forever remain `< 10`.                                               
很不幸，`bar(..)`内部的赋值表达式`i = 3`，覆盖了在`foo(..)`循环中声明的`i`。在这个例子中将导致无限循环，因为`i`被设置成固定值`3`，永远满足小于`< 10`这个条件。

The assignment inside `bar(..)` needs to declare a local variable to use, regardless of what identifier name is chosen. `var i = 3;` would fix the problem (and would create the previously mentioned "shadowed variable" declaration for `i`). An *additional*, not alternate, option is to pick another identifier name entirely, such as `var j = 3;`. But your software design may naturally call for the same identifier name, so utilizing scope to "hide" your inner declaration is your best/only option in that case.                                                                                     
`bar(..)`表达式的内部需要声明一个本地变量来使用，采用什么名字都行。`var i = 3;`就可以解决这个问题(同时也为`i`声明一个前面提到过的“遮蔽变量”)。另外一种方法，就是采用一个完全不同的标识符命名，例如`var j = 3;`。但是软件设计可能要求使用同样的标识符名称，因此在这种情况下使用作用域来“隐藏”内部声明是最佳选择。

#### 全局命名空间 (Global "Namespaces")

A particularly strong example of (likely) variable collision occurs in the global scope. Multiple libraries loaded into your program can quite easily collide with each other if they don't properly hide their internal/private functions and variables.                                                           
变量冲突的一个典型例子存在于全局作用域中。当你的程序中加载多个库时，如果他们没有适当地将内部函数和变量隐藏起来，就会很容易引起冲突。

Such libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope. This object is then used as a "namespace" for that library, where all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers themselves.                                                       
这些库会在全局作用域中创建一个名字够特别的变量声明，通常是一个对象。这个对象被用作库的命名空间，所有需要暴露给外界的功能都会成为这个对象的属性，而不是将自己的标识符暴露在顶级的词法作用域中。

For example:
例如：

```js
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### 模块管理（Module Management）

Another option for collision avoidance is the more modern "module" approach, using any of various dependency managers. Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager's various mechanisms.                                                                          
另外一个避免冲突的办法和现代的模块机制很像，就是使用任意一个依赖管理器。使用这些工具，没有库再需要添加任何标识符到全局作用域中，而是通过依赖管理器的机制将库的标识符显式的导入到另一个特定的作用域中。

It should be observed that these tools do not possess "magic" functionality that is exempt from lexical scoping rules. They simply use the rules of scoping as explained here to enforce that no identifiers are injected into any shared scope, and are instead kept in private, non-collision-susceptible scopes, which prevents any accidental scope collisions.                                               
很明显，这些工具没有能免除词法作用域规则的“魔力”。他们仅仅使用作用域的规则强制所有标识符都不能注入到共享作用域中，而是保持私有，无冲突的作用域中，这样可以有效的规避所有的意外冲突。

As such, you can code defensively and achieve the same results as the dependency managers do without actually needing to use them, if you so choose. See the Chapter 5 for more information about the module pattern.                                                                                           
因此，只要你愿意，你可以不使用依赖管理工具而实现相同的结果，关于模块模式详见第五章。

## 函数作用域（Functions As Scopes）

We've seen that we can take any snippet of code and wrap a function around it, and that effectively "hides" any enclosed variable or function declarations from the outside scope inside that function's inner scope.                                                                                                 
我们明白我们能在任意代码段外部添加包装函数，有效的隐藏任何内部变量和函数声明，外部作用域无法访问包装函数内部的任何内容。

For example:     
例如：

```js
var a = 2;

function foo() { // <-- insert this   添加这一行

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

While this technique "works", it is not necessarily very ideal. There are a few problems it introduces. The first is that we have to declare a named-function `foo()`, which means that the identifier name `foo` itself "pollutes" the enclosing scope (global, in this case). We also have to explicitly call the function by name (`foo()`) so that the wrapped code actually executes.                                                                                                    
虽然这种技术可以解决一些问题，但是它并不理想。还会产生一些问题。首先是我们必须声明一个具名函数（named-function）`foo()`，意味着这个标识名称`foo`本身污染了所在的作用域(在这个例子中是全局)。我们也必须显式的通过函数名调用这个函数才能运行其中的代码。

It would be more ideal if the function didn't need a name (or, rather, the name didn't pollute the enclosing scope), and if the function could automatically be executed.                                                                
如果这个函数不需要名字(或者名字至少能不污染所在作用域)，并且能自动运行，这将更理想。

Fortunately, JavaScript offers a solution to both problems.                                                   
幸好，JavaScript提供了一种能同时解决这两个问题的方法。

```js
var a = 2;

(function foo(){ // <-- insert this  添加这行

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

Let's break down what's happening here.                                                                      
让我们来看一下这里发生了什么。

First, notice that the wrapping function statement starts with `(function...` as opposed to just `function...`. While this may seem like a minor detail, it's actually a major change. Instead of treating the function as a standard declaration, the function is treated as a function-expression.                                                                                                 
首先，注意这个包装函数声明是以`(function...`开头的，而不仅仅是`function...`。虽然这看上去是个次要的细节，但事实上是一个重要的改变。函数会被当成函数表达式而不是一个标准的函数声明来处理。

**Note:** The easiest way to distinguish declaration vs. expression is the position of the word "function" in the statement (not just a line, but a distinct statement). If "function" is the very first thing in the statement, then it's a function declaration. Otherwise, it's a function expression.                                                                                           
**注意:** 区分函数表达式和函数声明最简单的办法是看声明中"function"出现的位置（不仅仅是一行代码中，而是在整个声明中的位置）。如果"function"是在声明中的第一个词，则是一个函数声明。否则，它是一个函数表达式。

The key difference we can observe here between a function declaration and a function expression relates to where its name is bound as an identifier.                                                      
函数表达式和函数声明之间最重要的不同就是他们的名字作为一个标识符被绑定在哪里。

Compare the previous two snippets. In the first snippet, the name `foo` is bound in the enclosing scope, and we call it directly with `foo()`. In the second snippet, the name `foo` is not bound in the enclosing scope, but instead is bound only inside of its own function.                                                                                        
比较之前两个代码段。第一个中，`foo`绑定在整个作用域中，可以直接通过`foo()`来调用它。第二个中，`foo`不是绑定在整个作用域中，而仅仅是绑定在自身的函数中。

In other words, `(function foo(){ .. })` as an expression means the identifier `foo` is found *only* in the scope where the `..` indicates, not in the outer scope. Hiding the name `foo` inside itself means it does not pollute the enclosing scope unnecessarily.                                                                   
换句话说，`(function foo(){ .. })`作为一个表达式意味着标识符`foo`只能在`..`所代表的位置中被访问，外部则不行。`foo`被隐藏在自身中意味着不会非必要地污染外部作用域(the enclosing scope)。

### Anonymous vs. Named   --（翠翠）         
### 匿名和命名

You are probably most familiar with function expressions as callback parameters, such as:                    
也许你最熟悉的函数表达式就是回调参数的形式,例如:                
```js
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

This is called an "anonymous function expression", because `function()...` has no name identifier on it. Function expressions can be anonymous, but function declarations cannot omit the name -- that would be illegal JS grammar.                         
这叫做"匿名函数表达式",因为函数`function()...`没有名称标识符.函数表达式可以是匿名的,但是函数声明式不能省略名字,那是非法的JS语法.                                

Anonymous function expressions are quick and easy to type, and many libraries and tools tend to encourage this idiomatic style of code. However, they have several draw-backs to consider:              
匿名函数表达式书写起来简单快捷,许多库和工具倾向于鼓励使用这种惯用的代码风格.但是,他们也有几个缺点要考虑:                                

1. Anonymous functions have no useful name to display in stack traces, which can make debugging more difficult.                                
1. 在堆栈跟踪中匿名函数没有有用的名字去显示,这使得调试更困难.                                

2. Without a name, if the function needs to refer to itself, for recursion, etc., the **deprecated** `arguments.callee` reference is unfortunately required. Another example of needing to self-reference is when an event handler function wants to unbind itself after it fires.                             
2. 函数没有名字,如果这个函数需要引用它自己时只能使用已经过期的`argument.callee`引用,如递归等.函数需要引用自身的另一个例子是当一个事件触发后函数想解绑它自己.                             

3. Anonymous functions omit a name that is often helpful in providing more readable/understandable code. A descriptive name helps self-document the code in question.                     
3. 匿名函数省略名字有助于提高代码的可读/可理解性.一个描述性的名字有助于理解代码的含义.                           

**Inline function expressions** are powerful and useful -- the question of anonymous vs. named doesn't detract from that. Providing a name for your function expression quite effectively addresses all these draw-backs, but has no tangible downsides. The best practice is to always name your function expressions:                         
**行内函数表达式** 非常强大且有用,匿名和命名的问题并不会影响它.给你的函数表达式提供一个名字可以非常有用的解决所有的这些问题,但是没有切实的缺点.最佳实践方式就是始终命名你的函数表达式:                

```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
	console.log( "I waited 1 second!" );
}, 1000 );
```

### Invoking Function Expressions Immediately                   
### 立即执行函数表达式                         

```js
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

Now that we have a function as an expression by virtue of wrapping it in a `( )` pair, we can execute that function by adding another `()` on the end, like `(function foo(){ .. })()`. The first enclosing `( )` pair makes the function an expression, and the second `()` executes the function.                       
现在我们有一个函数表达式并由一对`()`包含,我们可以执行该函数通过在其末尾添加另一个`()`,像`(function foo(){ .. })()`.第一个封闭的一对`()`使得函数成为一个表达式,第二个`()`则表示执行该函数.                

This pattern is so common, a few years ago the community agreed on a term for it: **IIFE**, which stands for **I**mmediately **I**nvoked **F**unction **E** xpression.               
这种模式是很常见的,几年前社区达成了一个术语:**IIFE**,代表立即执行函数表达式(Invoking Function Expressions Immediately)                   

Of course, IIFE's don't need names, necessarily -- the most common form of IIFE is to use an anonymous function expression. While certainly less common, naming an IIFE has all the aforementioned benefits over anonymous function expressions, so it's a good practice to adopt.                         
当然,IIFE不是必须有函数名的,相反,最常见的IIFE是使用匿名函数表达式.而更常见的,命名一个IIFE比前面提到的匿名函数表达式更有好处,因此这也是一个好的采用实践方式.                            

```js
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

There's a slight variation on the traditional IIFE form, which some prefer: `(function(){ .. }())`. Look closely to see the difference. In the first form, the function expression is wrapped in `( )`, and then the invoking `()` pair is on the outside right after it. In the second form, the invoking `()` pair is moved to the inside of the outer `( )` wrapping pair.                     
对于传统的IIFE这里有一些轻微变化,一些更喜欢这样的`(function(){..}())`.仔细的看下不同.在第一种形式中,函数表达式被`()`包含着,然后在后面用一对`()`来调用.在第二种形式中,调用的一对`()`被移动到外部的`()`括号内包含着.               

These two forms are identical in functionality. **It's purely a stylistic choice which you prefer.**            
两种形式都有着同样的功能.**哪种风格的选择看你自己喜好**                            

Another variation on IIFE's which is quite common is to use the fact that they are, in fact, just function calls, and pass in argument(s).                     
IIFE的另一种不同是非常常见的使用方法当作函数调用，并传递参数.              

For instance:          
例如:            

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

We pass in the `window` object reference, but we name the parameter `global`, so that we have a clear stylistic delineation for global vs. non-global references. Of course, you can pass in anything from an enclosing scope you want, and you can name the parameter(s) anything that suits you. This is mostly just stylistic choice.                       
我们传递`window`这个对象引用,但是我们把这个参数命名为`global`,因此在代码风格上对全局对象的引用变得比引用一个没有全局字样的变量更加清晰.当然,你可以从封闭的作用域传递任何你想要的,你可以任意命名参数只有是适合你的.这仅仅是风格的选择.                         

Another application of this pattern addresses the (minor niche) concern that the default `undefined` identifier might have its value incorrectly overwritten, causing unexpected results. By naming a parameter `undefined`, but not passing any value for that argument, we can guarantee that the `undefined` identifier is in fact the undefined value in a block of code:                 
另一种应用模式(次要的场景)是解决`undefined`标识符的默认值可能错误的被覆盖,导致意想不到的结果.通过命名一个参数`undefind`,但是不对参数传递任何值,我们可以确保`undefind`标识符的值是undefined在代码片段中                

```js
undefined = true; // setting a land-mine for other code! avoid! //  为其他代码设置了一个坑,不允许    

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```

Still another variation of the IIFE inverts the order of things, where the function to execute is given second, *after* the invocation and parameters to pass to it. This pattern is used in the UMD (Universal Module Definition) project. Some people find it a little cleaner to understand, though it is slightly more verbose.                         
IIFE还有一种变化的用途是倒置代码的运行顺序,将需要运行的函数放在第二位,在IIFE执行之后当作参数传递进去.这个模式被用于UMD(Universal Module Definition)项目中.一些人觉得它更容易理解,尽管它稍微冗长.                         

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

The `def` function expression is defined in the second-half of the snippet, and then passed as a parameter (also called `def`) to the `IIFE` function defined in the first half of the snippet. Finally, the parameter `def` (the function) is invoked, passing `window` in as the `global` parameter.                         
`def`函数表达式被定义在代码片段的第二部分,然后作为一个参数(也叫`def`)传递`IIFE`函数定义在代码片段的第一部分.最后,参数`def`(`def`函数)被执行,传递`window`作为`global`参数.                  

## 块级作用域（Blocks As Scopes）  --（张雪）

While functions are the most common unit of scope, and certainly the most wide-spread of the design approaches in the majority of JS in circulation, other units of scope are possible, and the usage of these other scope units can lead to even better, cleaner to maintain code.		
虽说函数是JS中最常见而且被广泛使用的作用域单元，但也有一些其他形式的作用域单元可以让你的代码更好维护，更整洁。

Many languages other than JavaScript support Block Scope, and so developers from those languages are accustomed to the mindset, whereas those who've primarily only worked in JavaScript may find the concept slightly foreign.		
除了JavaScript之外的很多语言都支持块级作用域，所以从别的语言转到JS的开发者们对这个概念非常熟悉，但那些只用Javascript工作的开发者们可能会觉得这个概念有些陌生。

But even if you've never written a single line of code in block-scoped fashion, you are still probably familiar with this extremely common idiom in JavaScript:		
就算你从没有用写过一句块级作用域风格的代码，你也很可能熟悉下面这个JS中很常见的例子：

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

We declare the variable `i` directly inside the for-loop head, most likely because our *intent* is to use `i` only within the context of that for-loop, and essentially ignore the fact that the variable actually scopes itself to the enclosing scope (function or global).
我们在for循环的顶部直接声明了变量 `i` ，因为我们想在for循环体中使用 `i` ，但忽略了变量本身是属于一个闭合的作用域的（函数或是全局作用域）。		

That's what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another example:		
这也正是块级作用域存在的意义，就近声明变量，离变量被使用的地方越近越好。另一个例子：

```js
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

We are using a `bar` variable only in the context of the if-statement, so it makes a kind of sense that we would declare it inside the if-block. However, where we declare variables is not relevant when using `var`, because they will always belong to the enclosing scope. This snippet is essentially "fake" block-scoping, for stylistic reasons, and relying on self-enforcement not to accidentally use `bar` in another place in that scope.		
我们只在if语句的执行环境内部使用了变量`bar`，所以我们在if语句块内部声明这个变量是比较合理的。尽管我们声明变量的位置和我们何时使用变量并没有关系，因为他们必须总是在同一个作用域内，这个代码片段从风格上看是一个“假的”块级作用域，我们只能强制不在其他的地方使用变量`bar`。

Block scope is a tool to extend the earlier "Principle of Least ~~Privilege~~ Exposure" [^note-leastprivilege] from hiding information in functions to hiding information in blocks of our code.		
块级作用域是“最小特权原则”的一种实现，把在函数内隐藏信息的方法扩展到了在代码区块中隐藏信息。

Consider the for-loop example again:		
考虑下面的for循环例子：

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

Why pollute the entire scope of a function with the `i` variable that is only going to be (or only *should be*, at least) used for the for-loop?		
为什么要用本该在for循环内部使用的`i`污染全局作用域呢？

But more importantly, developers may prefer to *check* themselves against accidentally (re)using variables outside of their intended purpose, such as being issued an error about an unknown variable if you try to use it in the wrong place. Block-scoping (if it were possible) for the `i` variable would make `i` available only for the for-loop, causing an error if `i` is accessed elsewhere in the function. This helps ensure variables are not re-used in confusing or hard-to-maintain ways.		
更重要的是，开发者可能需要要检查这些变量，以免在作用范围外被意外使用。块级作用域（如果有效的话）会使得变量`i`值在for循环内有效，在别处访问`i`时会报错。这样就确保了变量不会被重复使用而导致的维护困难。

But, the sad reality is that, on the surface, JavaScript has no facility for block scope.		
而且，悲剧的是，表面上看，JS没有块级作用域的功能。

That is, until you dig a little further.		
除非你再深入研究一下。

### `with`

We learned about `with` in Chapter 2. While it is a frowned upon construct, it *is* an example of (a form of) block scope, in that the scope that is created from the object only exists for the lifetime of that `with` statement, and not in the enclosing scope.		
我们再第二章学到了`with`。它是一种难以理解的结构，一种块级作用域的形式，它从某个对象建立的块级作用域只会在`with`内部存在。

### `try/catch`

It's a *very* little known fact that JavaScript in ES3 specified the variable declaration in the `catch` clause of a `try/catch` to be block-scoped to the `catch` block.
这是ES3中鲜为人知的一个知识点：在`try/catch`的`catch`中声明的变量是属于`catch`块级作用域的。		

For instance:		
例如：

```js
try {
	undefined(); // 用非法操作来产生一个异常
}
catch (err) {
	console.log( err ); // 捕获该异常
}

console.log( err ); // ReferenceError: `err` not found 外部无法访问`err`
```

As you can see, `err` exists only in the `catch` clause, and throws an error when you try to reference it elsewhere.
正如你所看到的，`err`值存在于`catch`的语句中，你在别处引用就会抛出错误。		

**Note:** While this behavior has been specified and true of practically all standard JS environments (except perhaps old IE), many linters seem to still complain if you have two or more `catch` clauses in the same scope which each declare their error variable with the same identifier name. This is not actually a re-definition, since the variables are safely block-scoped, but the linters still seem to, annoyingly, complain about this fact.		
**注意:** 虽然该特性已经成为标准且适用于所有的JS运行环境（可能除了某些老版本的IE），当在同一个作用域内的两个或多个`catch`语句中声明名称相同的变量时，许多静态语法检查器依然会报错。实际上这并没有重新声明变量，因为变量可以在其块级作用域中安全使用，但静态语法检查器依然会报错。

To avoid these unnecessary warnings, some devs will name their `catch` variables `err1`, `err2`, etc. Other devs will simply turn off the linting check for duplicate variable names.		
为了避免不必要的警告，一些开发者将`catch`中的变量命名成`err1`, `err2`等等。其他开发者可能会简单地吧静态检查器中检查重名变量的功能给关了。

The block-scoping nature of `catch` may seem like a useless academic fact, but see Appendix B for more information on just how useful it might be.		
`catch`的块级作用域特性看起来好像偏学术，用处不大，但附录B中有这方面很多有用的信息。

### `let`  --（张润）

Thus far, we've seen that JavaScript only has some strange niche behaviors which expose block scope functionality. If that were all we had, and *it was* for many, many years, then block scoping would not be terribly useful to the JavaScript developer.

截止到目前为止,我们知道JavaScript仅仅在暴露块作用域的功能方面有一些奇怪的行为。如果这是我们知道的全部,那么块作用域就不会成为多年来JavaScript开发者最常用的开发机制了.

Fortunately, ES6 changes that, and introduces a new keyword `let` which sits alongside `var` as another way to declare variables.

不幸的是,ES6改变了这些,它引进了一个新的单词`let`,提供作为`var`之外的另一种变量的声明方式.

The `let` keyword attaches the variable declaration to the scope of whatever block (commonly a `{ .. }` pair) it's contained in. In other words, `let` implicitly hijacks any block's scope for its variable declaration.

`let`关键字可以将变量绑定到任何一个包含它的作用域中(通常来说是一个{..}内部).换句话说,`let`为声明的变量隐式的劫持了一个块作用域.

```js
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

Using `let` to attach a variable to an existing block is somewhat implicit. It can confuse you if you're not paying close attention to which blocks have variables scoped to them, and are in the habit of moving blocks around, wrapping them in other blocks, etc., as you develop and evolve code.

使用`let`将变量绑定到一个已经存在的作用域中的过程是一个隐式的操作.在开发和修改代码中,如果你没有注意到哪个作用域包含这个变量,或者习惯性的移动这些作用域,将这些作用域包含在另一个作用域中,

Creating explicit blocks for block-scoping can address some of these concerns, making it more obvious where variables are attached and not. Usually, explicit code is preferable over implicit or subtle code. This explicit block-scoping style is easy to achieve, and fits more naturally with how block-scoping works in other languages:

为一个块创建一个显示的作用域能够解决这个问题,可以显示的展示变量所属的作用域.通常来说,显示的代码优于隐式的或者一些不够清晰的代码.这种隐式的块作用域风格很容易书写,而且也更适用于其他语言中的块作用域的实现.

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

We can create an arbitrary block for `let` to bind to by simply including a `{ .. }` pair anywhere a statement is valid grammar. In this case, we've made an explicit block *inside* the if-statement, which may be easier as a whole block to move around later in refactoring, without affecting the position and semantics of the enclosing if-statement.

只要声明的语法是有效的,我们可以用一个{..}为`let`绑定任意一个块.在这个例子中,我们在if内部显示的声明了一个块,如果需要重构,移动整个块都可以被很方便的移动,而不影响if声明范围内的块所在的位置和语法.

**Note:** For another way to express explicit block scopes, see Appendix B.

**注意:**另外一种显示块作用域的表达请参照附录B.

In Chapter 4, we will address hoisting, which talks about declarations being taken as existing for the entire scope in which they occur.

在第四章,我们会讨论提升这个概念,即其声明将会存在其所发生的整个作用域内.

However, declarations made with `let` will *not* hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.

然而,`let`的声明*不会*提升至其所存在的整个作用域块中.除非提前声明,否则声明不会显示的"存在"于块中.

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

#### Garbage Collection

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory. We'll briefly illustrate here, but the closure mechanism is explained in detail in Chapter 5.

另一方面,块作用域对于内存回收机制相关的闭包和垃圾回收有关.我们会在这里简要的说明一下,但是闭包机制的具体内容是在第五章.

Consider:

```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

The `click` function click handler callback doesn't *need* the `someReallyBigData` variable at all. That means, theoretically, after `process(..)` runs, the big memory-heavy data structure could be garbage collected. However, it's quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the `click` function has a closure over the entire scope.   
`click`这个函数的处理器的回调函数根本不*需要*`someReallyBigData`.这意味着,理论上来说,`process(..)`运行过后,大量占用空间的数据结构就可以被垃圾回收了.然而,尽管`click`函数已经在其作用域中形成了一个闭包,JS引擎很可能(也可能运行环境决定)将会继续履行职责.

Block-scoping can address this concern, making it clearer to the engine that it does not need to keep `someReallyBigData` around:   
块作用域可以为你解决这些顾虑,使你的机器清楚的知道无需再保留`someReallyBigData`这样的数据:

```js
function process(data) {
	// do something interesting
}

// anything declared inside this block can go away after!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.
在你的代码工具箱中为变量显示的声明块或者将变量绑定到本地都是一种很有效的方法.

#### `let` Loops  --（李欣）
#### `let` 循环

A particular case where `let` shines is in the for-loop case as we discussed previously.   
一个`let`可以发挥优势的典型例子就是之前讨论的for循环

```js
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

Not only does `let` in the for-loop header bind the `i` to the for-loop body, but in fact, it **re-binds it** to each *iteration* of the loop, making sure to re-assign it the value from the end of the previous loop iteration.   
for循环头部的`let`不仅将`i`绑定到了for循环体中，事实上，它将其**重新绑定**到了每一个*迭代*中，确保使用上一个循环迭代结束时的值重新进行赋值。

Here's another way of illustrating the per-iteration binding behavior that occurs:   
这里是另一种用来说明每次迭代时进行重新绑定的行为

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

The reason why this per-iteration binding is interesting will become clear in Chapter 5 when we discuss closures.   
每个迭代进行重新绑定的原因很有趣，我们会在第5章讨论闭包的时候进行说明。

Because `let` declarations attach to arbitrary blocks rather than to the enclosing function's scope (or global), there can be gotchas where existing code has a hidden reliance on function-scoped `var` declarations, and replacing the `var` with `let` may require additional care when refactoring code.   
由于`let`声明附属于一个新的作用域而不是当前的函数作用域（也不是全局作用域），当代码中存在对于函数作用域中var声明的隐式依赖时，就会有很多隐藏的陷阱，如果用`let`来替代var
则需要在代码重构的过程中付出额外的精力。

Consider:   
考虑以下代码

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}
```

This code is fairly easily re-factored as:   
这段代码可以简单的重构成下面的形式

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

But, be careful of such changes when using block-scoped variables:   
但是在使用块级作用域的变量时需要注意以下变化：

```js
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <-- don't forget `bar` when moving!
		console.log( baz );
	}
}
```

See Appendix B for an alternate (more explicit) style of block-scoping which may provide easier 
to maintain/refactor code that's more robust to these scenarios.   
参考附录，其中介绍了另外一种块作用域形式，可以用更健壮的方式实现目的，并且写出的代码更易维护和重构。

### `const`

In addition to `let`, ES6 introduces `const`, which also creates a block-scoped variable, but 
whose value is fixed (constant). Any attempt to change that value at a later time results in an 
error.   
除了`let`以外，ES6还引入了`const`，通欧阳可以用来创建作用域变量，但其值是固定的（常量）。之后任何试图修改值的操作都会引起错误。

```js
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // block-scoped to the containing `if`

	a = 3; // just fine!
	b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Review (TL;DR)
## 回顾（TL；DR）

Functions are the most common unit of scope in JavaScript. Variables and functions that are 
declared inside another function are essentially "hidden" from any of the enclosing "scopes", 
which is an intentional design principle of good software.   
函数是JavaScript中最常见的作用域单元。本质上，声明在一个函数内部的变量或函数会在所处的作用域中"隐藏"起来，这是有意为之的良好的软件设计原则

But functions are by no means the only unit of scope. Block-scope refers to the idea that 
variables and functions can belong to an arbitrary block (generally, any `{ .. }` pair) of code, 
rather than only to the enclosing function.   
但函数不是唯一的作用域单元。块作用域指的是变量和函数不仅可以属于所处的作用域，也可以属于某个代码块（一般指`{ .. }`内部）

Starting with ES3, the `try/catch` structure has block-scope in the `catch` clause.   
从ES3开始，`try/catch`结构在`catch`分句中具有块作用域。

In ES6, the `let` keyword (a cousin to the `var` keyword) is introduced to allow declarations of 
variables in any arbitrary block of code. `if (..) { let a = 2; }` will declare a variable `a` 
that essentially hijacks the scope of the `if`'s `{ .. }` block and attaches itself there.   
在ES6中引入了`let`关键字（`var`关键字的表亲），用来在任意代码块中声明变量。`if (..) { let a = 2; }`会声明一个劫持了`if`的 `{ .. 
}`块的变量`a`，并且将其添加到这个块中。

Though some seem to believe so, block scope should not be taken as an outright replacement of 
`var` function scope. Both functionalities co-exist, and developers can and should use both 
function-scope and block-scope techniques where respectively appropriate to produce better, more 
readable/maintainable code.   
有些人认为块作用域不应该完全作为函数作用域的替代方案。两种功能应该同时存在，开发者可以并且也应该根据需要选择使用何种作用域，创造可读的，可维护的优良代码。

[^note-leastprivilege]: [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege)

## 单词本

| 单词 | 音标 | 释义 |
| ---- | ---- | ---- |
| mindset | ['maɪn(d).set] | 观念、心态 |
| linter |  ['lɪntə] | 静态语法检查器、棉绒剔除机 |
| omit | [ə'mɪt] | vt. 省略；遗漏；删除；疏忽 |
| illegal | [ɪ'ligl] | adj.[法]非法的；违法的；违反规则的n.非法移民，非法劳工 |
| idiomatic | [,ɪdɪə'mætɪk] | adj. 惯用的；符合语言习惯的；通顺的 |
| slightly | ['slaɪtli] | adv. 些微地，轻微地；纤细地 |
| collision | [kə'lɪʒən] | n.（意见，看法）的抵触，冲突 |
| avoidance | [ə'vɔɪdəns] | n. 逃避；废止；职位空缺 |
| minor | ['maɪnɚ] | adj. 未成年的；次要的；较小的；小调的；二流的 n. 未成年人；小调；副修科目 vi. 副修 |
| injected | | 充血的 注入的 |
| exempt | [ɪg'zɛmpt] | vt. 免除；豁免  adj. 被免除的；被豁免的  n. 免税者；被免除义务者 |
| as such | | 同样地；本身；就其本身而论 |
| utilizing | [ˈjuːtɪˌlaɪzɪŋ] | v. 利用（utilize的ing形式） |
| illustrating | ['ɪləstret] | vt. 阐明，举例说明；图解 vi. 举例 |
| arbitrary | [ˈɑːrbətreri] | adj. [数] 任意的；武断的；专制的 |
| gotchas | | int. 明白了（等于got you） |
| fairly | ['fɛrli] | adv. 相当地；公平地；简直 |
| Appendix | [ə'pɛndɪks] | n. 附录；阑尾；附加物 |


## 疑难杂句

  * Anonymous function expressions are quick and easy to type
  - 官译:匿名函数表达式书写起来简单快捷
  - 匿名函数表达式是快速且简单的类型(李翠翠)
* if the function needs to refer to itself, for recursion, etc., the **deprecated** `arguments.callee` reference is unfortunately required
  - 官译:如果这个函数需要引用它自己时,**不赞成** `arguments.callee`引用是不幸的需求,如递归等
  - 如果这个函数需要引用它自己时只能使用已经过期的`argument.callee`引用,如递归等(李翠翠)
* so that we have a clear stylistic delineation for global vs. non-global references
  - 官译:因此在代码风格上对全局对象的引用变得比引用一个没有全局字样的变量更加清晰
  - 以便我们有一个清楚的风格去描述全局对象的引用比没有全局对象的引用(李翠翠)
* Still another variation of the IIFE inverts the order of things, where the function to execute is given second, *after* the invocation and parameters to pass to it.
  - 官译:IIFE还有一种变化的用途是倒置代码的运行顺序,将需要运行的函数放在第二位,在IIFE执行之后当作参数传递进去.
  - IIFE还有一个变化是使事物命令颠倒,函数执行放在第二个,在调用后才传递参数（李翠翠）