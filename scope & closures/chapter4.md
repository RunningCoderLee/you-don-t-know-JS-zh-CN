# 你不知道的js：作用域和闭包（You Don't Know JS: Scope & Closures） --(罗尧)
# 提升（Chapter 4: Hoisting）

By now, you should be fairly comfortable with the idea of scope, and how variables are attached to different levels of scope depending on where and how they are declared. Both function scope and block scope behave by the same rules in this regard: any variable declared within a scope is attached to that scope.   
到目前为止，您应该比较了解作用域，以及变量如何被附加到不同级别的作用域，这取决于它们在哪里和如何声明。在这方面，函数作用域和块作用域都有相同的规则：在作用域内声明的任何变量都附加到该作用域。

But there's a subtle detail of how scope attachment works with declarations that appear in various locations within a scope, and that detail is what we will examine here.    
但是，作用域同其中的变量声明出现的位置有一个微妙的联系，这些细节将在这里进行研究。

## 先有鸡还是先有蛋（Chicken Or The Egg?）

There's a temptation to think that all of the code you see in a JavaScript program is interpreted line-by-line, top-down in order, as the program executes. While that is substantially true, there's one part of that assumption which can lead to incorrect thinking about your program.   
有一种诱惑，认为在程序执行时，您在JavaScript程序中看到的所有代码按顺序执行，自上而下依次执行。虽然这是基本的事实，但这一假设有一部分可能会导致错误的理解。

Consider this code:   
思考下面的代码：

```js
a = 2;

var a;

console.log( a );
```

What do you expect to be printed in the `console.log(..)` statement?   
你期望`console.log(..)`打印出什么？

Many developers would expect `undefined`, since the `var a` statement comes after the `a = 2`, and it would seem natural to assume that the variable is re-defined, and thus assigned the default `undefined`. However, the output will be `2`.   
许多开发人员会觉得是`undefined`，因为`var a`语句在`a = 2`之后，假设变量被重新定义，因此分配了默认的undefined是很自然的。 但是，输出将为2。

Consider another piece of code:   
考虑另一段代码：

```js
console.log( a );

var a = 2;
```

You might be tempted to assume that, since the previous snippet exhibited some less-than-top-down looking behavior, perhaps in this snippet, `2` will also be printed. Others may think that since the `a` variable is used before it is declared, this must result in a `ReferenceError` being thrown.   
你可能会想到，由于之前的代码片段显示出一些不符合自上而下执行的行为，也许在这段代码中，`2`也将被打印出来。 其他人可能会认为，因为`a`变量在被声明之前被使用，所以必须导致引用一个`ReferenceError`错误。

Unfortunately, both guesses are incorrect. `undefined` is the output.   
不幸的是，两种猜测都是错误的，会打印出`undefined`。

**So, what's going on here?** It would appear we have a chicken-and-the-egg question. Which comes first, the declaration ("egg"), or the assignment ("chicken")?     
**所以，这里发生了什么?** 就像是先有鸡还是蛋的问题。哪个在前面，声明（“蛋”）还是赋值（“鸡”）？

## 编译器再度来袭(The Compiler Strikes Again)  --(张静)

To answer this question, we need to refer back to Chapter 1, and our discussion of compilers. Recall that the *Engine* actually will compile your JavaScript code before it interprets it. Part of the compilation phase was to find and associate all declarations with their appropriate scopes. Chapter 2 showed us that this is the heart of Lexical Scope.                                                   
为了回答这个问题，我们需要回顾第一章关于编译器的讨论。回忆一下，引擎通常在解释你的代码之前先对其进行编译。编译阶段的一部分工作就是找到所有的声明，并用合适的作用域把它们关联起来。第二章展示了这个，也是词法作用域的核心。

So, the best way to think about things is that all declarations, both variables and functions, are processed first, before any part of your code is executed.                                                                            
所以，思考所有声明的最好方式是，变量和函数都在任何代码被执行前首先被处理。

When you see `var a = 2;`, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: `var a;` and `a = 2;`. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left **in place** for the execution phase.                                                                          
当你看见`var a = 2;`，你可能会认为它是一个声明。但是javascript通常认为这是两个声明：`var a;` 和 `a = 2;`。第一个在编译阶段进行。第二个赋值声明会被留在原地等待执行阶段。

Our first snippet then should be thought of as being handled like this:                        
我们的第一个代码段会以如下形式被思考：

```js
var a;
```
```js
a = 2;

console.log( a );
```

...where the first part is the compilation and the second part is the execution.                                                                            
第一部分是编译，第二部分是执行。

Similarly, our second snippet is actually processed as:                                  
同样的，我们的第二个代码段通常如下进行：

```js
var a;
```
```js
console.log( a );

a = 2;
```

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are "moved" from where they appear in the flow of the code to the top of the code. This gives rise to the name "Hoisting".                                                                           
所以，照这个思考的方式，打个比方，这个过程就好像变量和函数声明从他们在代码中出现的位置被“移动”到最上方。这叫“变量提升”。

In other words, **the egg (declaration) comes before the chicken (assignment)**.                                                                                      
换句话说，先有蛋(声明)后有鸡(赋值)。

**Note:** Only the declarations themselves are hoisted, while any assignments or other executable logic are left *in place*. If hoisting were to re-arrange the executable logic of our code, that could wreak havoc.                                
**注意:** 只有声明本身会被提升，而赋值或者其他可运行的逻辑被留在原地。如果提升重新改变了代码执行的顺序，会造成非常严重的后果。

```js
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```
-- (翠翠)
The function `foo`'s declaration (which in this case *includes* the implied value of it as an actual function) is hoisted, such that the call on the first line is able to execute.                         
函数`foo`的声明被提升了（这个例子还包括实际函数中的隐含值）,因此第一行的函数调用是可以执行的。                   

It's also important to note that hoisting is **per-scope**. So while our previous snippets were simplified in that they only included global scope, the `foo(..)` function we are now examining itself exhibits that `var a` is hoisted to the top of `foo(..)` (not, obviously, to the top of the program). So the program can perhaps be more accurately interpreted like this:                    
同样重要的要注意的是**每个作用域**的提升.尽管我们之前的代码片段被简化了，因为它们只包含全局作用域，我们现在说的`foo(..)`函数它自己也会在内部将变量`var a`提升到函数`foo(..)`的顶部（并不是程序的顶部）.因此程序也许可以更准确的解释为这样:                     

```js
function foo() {
	var a;

	console.log( a ); // undefined

	a = 2;
}

foo();
```

Function declarations are hoisted, as we just saw. But function expressions are not.                  
正如我们看到的，函数声明会被提升，但是函数表达式不会被提升.                       

```js
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
	// ...
};
```

The variable identifier `foo` is hoisted and attached to the enclosing scope (global) of this program, so `foo()` doesn't fail as a `ReferenceError`. But `foo` has no value yet (as it would if it had been a true function declaration instead of expression). So, `foo()` is attempting to invoke the `undefined` value, which is a `TypeError` illegal operation.                       
在这个程序中变量标识符`foo`被提升并且被附加到所在的作用域（全局作用域）,所以`foo()`不会抛出`ReferenceError`异常.但是`foo`依旧没有值(如果它是一个真的函数声明而不是函数表达式，那它就会被赋值).因此,`foo()`试图调用`undefined`值时,这是一个`TypeError`非法操作。                       

Also recall that even though it's a named function expression, the name identifier is not available in the enclosing scope:                               
同时也要记住即使它是一个命名函数表达式,名称标识符在赋值之前在其作用域中也不能使用:                               

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

--(张雪)
This snippet is more accurately interpreted (with hoisting) as:
有了变量提升的感念，就能更加准确地理解下面的代码：

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```

## 函数在上（Functions First）

Both function declarations and variable declarations are hoisted. But a subtle detail (that *can* show up in code with multiple "duplicate" declarations) is that functions are hoisted first, and then variables.
函数声明和变量声明都有提升的作用，但有一个细微的区别（在代码中有重复定义时会显示出来）：函数先提升，然后才是变量。

Consider:

```js
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

--(张润)
`1` is printed instead of `2`! This snippet is interpreted by the *Engine* as:

输出`1`而不是`2`!这段代码片段被机器解释为一下几行:

```js
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

Notice that `var foo` was the duplicate (and thus ignored) declaration, even though it came before the `function foo()...` declaration, because function declarations are hoisted before normal variables.

我们注意到`var foo`实在重复的声明(因此被忽略了),但是它不会被提升到了`function foo()`声明的前面,因为函数的声明会在普通变量的声明之前被提前提升.

While multiple/duplicate `var` declarations are effectively ignored, subsequent function declarations *do* override previous ones.

尽管重复的声明`var`被有效的忽略了,然而后来的函数声明又会覆盖以前的函数声明.

```js
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

While this all may sound like nothing more than interesting academic trivia, it highlights the fact that duplicate definitions in the same scope are a really bad idea and will often lead to confusing results.

尽管这些听起来都像是学术派的语言,却强调了一个事实就是在一个作用域中重复的定义绝对不是一个好事情,它可能会导致一些意想不到的结果.

Function declarations that appear inside of normal blocks typically hoist to the enclosing scope, rather than being conditional as this code implies:

函数的声明会显示的被提升至其所在的作用域的顶部,即便出现以下这种情况,它也不会受到任何影响.

```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

--(李欣)
However, it's important to note that this behavior is not reliable and is subject to change in future versions of JavaScript, so it's probably best to avoid declaring functions in blocks.   
可是，请注意这个行为并不可靠并且在未来的JavaScript版本中会更改它，因此最好避免在块中声明函数

## Review (TL;DR)
## 回顾

We can be tempted to look at `var a = 2;` as one statement, but the JavaScript *Engine* does not see it that way. It sees `var a` and `a = 2` as two separate statements, the first one a compiler-phase task, and the second one an execution-phase task.   
我们可以尝试把`var a = 2`看成一条语句，但是JavaScript*引擎*并不这么认为。它会看成`var a`和`a = 2`2条语句，第一条是编译阶段的任务，第二条是执行阶段的任务。

What this leads to is that all declarations in a scope, regardless of where they appear, are processed *first* before the code itself is executed. You can visualize this as declarations (variables and functions) being "moved" to the top of their respective scopes, which we call "hoisting".   
这导致的结果是，范围内的所有声明（无论它们在哪里出现）都会在执行代码本身之前被*优先*处理。你可以将声明（变量和函数）看做被“移动”到其各自作用域的顶部，我们称之为“变量提升”。


Declarations themselves are hoisted, but assignments, even assignments of function expressions, are *not* hoisted.   
它们各自的声明被提升，但是赋值语句并没有，即使是函数表达式的赋值也*不会*被提升。

Be careful about duplicate declarations, especially mixed between normal var declarations and function declarations -- peril awaits if you do!   
要注意重复声明，特别是混合了正常的var声明和函数声明之间 - 如果你这样做，会很危险。

## 单词本

| 单词 | 音标 | 释义 |
|------|------|-----|
| hoisting | ['hɔistiŋ] | n. 提升；起重 v. 升高；举起（hoist的ing形式） |
| metaphorically | [,metə'fɔrikəli] | adv. 隐喻地；用比喻 |
| interprets | 解释 作口译 |

