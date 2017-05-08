# 你不知道的js：作用域和闭包（You Don't Know JS: Scope & Closures） --(罗尧)

# 第五章：作用域闭包（Chapter 5: Scope Closure）

We arrive at this point with hopefully a very healthy, solid understanding of how scope works.   
接下来要讲的这些需要你对作用域有非常深刻详细的了解。

We turn our attention to an incredibly important, but persistently elusive, almost mythological, part of the language: closure. If you have followed our discussion of lexical scope thus far, the payoff is that closure is going to be, largely, anticlimactic, almost self-obvious. There's a man behind the wizard's curtain, and we're about to see him. No, his name is not Crockford!   
我们把注意力转移到这门语言非常重要的，难以掌控的，几乎*神话*的一个概念上：**闭包**。如果你已经已经学习过了之前的词法作用域的讨论，那么闭包对于你来说就是不言自明的，就是显而易见的。*魔术师的幕后藏有一个人，我们即将看到他*。不，他的名字不是Crockford！

## 启示（Enlightenment）

For those who are somewhat experienced in JavaScript, but have perhaps never fully grasped the concept of closures, *understanding closure* can seem like a special nirvana that one must strive and sacrifice to attain.    
对于那些对JavaScript有一点使用经验但是有不完全理解闭包的人来说，*理解闭包*看起来就像是一次特殊的重生，但是需要达到这点要做出努力和牺牲。

I recall years back when I had a firm grasp on JavaScript, but had no idea what closure was. The hint that there was *this other side* to the language, one which promised even more capability than I already possessed, teased and taunted me. I remember reading through the source code of early frameworks trying to understand how it actually worked. I remember the first time something of the "module pattern" began to emerge in my mind. I remember the *a-ha!* moments quite vividly.   
前几年，我对JavaScript有一个牢固的掌握，但不知道什么是闭包。总以为这门语言有其隐晦的另一面，可以帮我成长为能力更强的人，现实打脸。我记得阅读了大量的早期框架的源代码，试图了解闭包的实际工作原理。我记得第一次在我的脑海里出现了“模块模式（module pattern）”的一些东西时那份激动。

What I didn't know back then, what took me years to understand, and what I hope to impart to you presently, is this secret: **closure is all around you in JavaScript, you just have to recognize and embrace it.** Closures are not a special opt-in tool that you must learn new syntax and patterns for. No, closures are not even a weapon that you must learn to wield and master as Luke trained in The Force.   
当时我不理解的并花了多年的时间才能明白，希望现在能传达给你，这是就是秘诀：**闭包在JavaScript里面无处不在，你只需要认识并拥抱它。**闭包不是一个你必须学习新的语法和模式才能使用的工具。不，闭包甚至不是必须接受如Luke一样的原力训练才能使用和掌握的武装。

Closures happen as a result of writing code that relies on lexical scope. They just happen. You do not even really have to intentionally create closures to take advantage of them. Closures are created and used for you all over your code. What you are *missing* is the proper mental context to recognize, embrace, and leverage closures for your own will.   
闭包是基于词法作用域的写代码而自然产生的的。他们只是发生。你甚至不需要有意创造闭包以此来理解它们。闭包自然地被创建并作用于你的所有代码。你所*缺少*的是适当的心理背景，以自己的意志来理解，拥抱和使用闭包。

The enlightenment moment should be: **oh, closures are already occurring all over my code, I can finally *see* them now.** Understanding closures is like when Neo sees the Matrix for the first time.    
启蒙的关键点应该是：**哦，关闭已经在我的代码中发生了，我终于可以看到他们了**。了解闭包就像当Neo第一次看到Matrix时一样。

## 实质问题（Nitty Gritty）

OK, enough hyperbole and shameless movie references.   
好了，足够的夸张和浮夸的电影比喻已经够多了。

Here's a down-n-dirty definition of what you need to know to understand and recognize closures:   
下面就是直接了当的定义帮助你理解掌握闭包：

> Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.
> 闭包是函数能够记住和访问其词法作用域，即使该函数在其词法作用域之外执行。

Let's jump into some code to illustrate that definition.
让我们看一些代码来说明这个定义。

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

This code should look familiar from our discussions of Nested Scope. Function `bar()` has *access* to the variable `a` in the outer enclosing scope because of lexical scope look-up rules (in this case, it's an RHS reference look-up).    
这段代码和我们之前讨论的嵌套作用域很相似。由于词法作用域查找规则（在这种情况下，它是RHS参考查找），函数`bar()`可以*访问*外部封闭范围中的变量`a`。

Is this "closure"?
这就是“闭包”？

Well, technically... *perhaps*. But by our what-you-need-to-know definition above... *not exactly*. I think the most accurate way to explain `bar()` referencing `a` is via lexical scope look-up rules, and those rules are *only* (an important!) **part** of what closure is.  
从技术上来说...*或许是*。但是按照上面我们需要掌握的的定义来说...*不完全是*。我认为解释`bar()`引用`a`最准确的说法是通过词法范围查找规则，而这些规则* 只是* （但是是非常重要的）闭包的一部分。

From a purely academic perspective, what is said of the above snippet is that the function `bar()` has a *closure* over the scope of `bar()` (and indeed, even over the rest of the scopes it has access to, such as the global scope in our case). Put slightly differently, it's said that `bar()` closes over the scope of `foo()`. Why? Because `bar()` appears nested inside of `foo()`. Plain and simple.   
从纯粹的学术角度来说，上面的代码片断说明的是，函数`bar()`在`foo()`的作用域内有一个闭包（实际上，甚至在其可以访问的其他作用域内，例如在全局作用域）。也可以说`bar()`被封闭在`foo()`的作用域。为什么？答案简单明了，因为`bar()`嵌套在`foo()`中。

But, closure defined in this way is not directly *observable*, nor do we see closure *exercised* in that snippet. We clearly see lexical scope, but closure remains sort of a mysterious shifting shadow behind the code.   
但是，以这种方式定义的闭包不是*直接可见*的，我们也不会看到在该代码段中如何执行了闭包。我们只能看到词法作用域，但闭包仍然藏在代码背后的一个神秘的阴影里。

Let us then consider code which brings closure into full light:   
为了更透彻地观察闭包，让我们来看下面一段代码：

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- wow，朋友，你看到闭包了吧.
```

The function `bar()` has lexical scope access to the inner scope of `foo()`. But then, we take `bar()`, the function itself, and pass it *as* a value. In this case, we `return` the function object itself that `bar` references.   
函数`bar()`具有对`foo()`的内部作用域的词法作用域访问。但是，我们把`bar()`函数本身作为一个值传递。在这个例子中，我们`return``bar`函数对象本身。

After we execute `foo()`, we assign the value it returned (our inner `bar()` function) to a variable called `baz`, and then we actually invoke `baz()`, which of course is invoking our inner function `bar()`, just by a different identifier reference.  
在我们执行`foo()`之后，我们将它返回的值（我们的内部`bar()`函数）赋给一个名为`baz`的变量，然后我们实际上调用了`baz()`，当然这是调用我们的内部函数`bar()`不同的标识符引用。

`bar()` is executed, for sure. But in this case, it's executed *outside* of its declared lexical scope.   
`bar()` 被执行。但在这个例子中，它在其声明的词法作用域外执行。

After `foo()` executed, normally we would expect that the entirety of the inner scope of `foo()` would go away, because we know that the *Engine* employs a *Garbage Collector* that comes along and frees up memory once it's no longer in use. Since it would appear that the contents of `foo()` are no longer in use, it would seem natural that they should be considered *gone*.    
在执行`foo()`之后，通常我们会期望`foo()`的内部作用域全部都消失，因为我们知道引擎使用了一个*垃圾收集器*，一旦它不再使用就释放内存。由于看起来`foo()`的内容不再被使用，所以很自然地应该被*回收*。

But the "magic" of closures does not let this happen. That inner scope is in fact *still* "in use", and thus does not go away. Who's using it? **The function `bar()` itself**.  
但是“神奇的”的是闭包会阻止这事的发生。内部作用域实际上*仍然*“还在被使用”，因此不会消失。那谁又在使用这个作用域？答案是**`bar()`函数自己**

By virtue of where it was declared, `bar()` has a lexical scope closure over that inner scope of `foo()`, which keeps that scope alive for `bar()` to reference at any later time.      
由于它被声明在那个地方，`bar()`在`foo()`的内部范围内有一个词法作用域闭包，它使该作用域一直存货以便之后的`bar()`引用。


**`bar()` still has a reference to that scope, and that reference is called closure.**   
** `bar()`仍然具有对该作用域的引用，该引用称为闭包。** 

So, a few microseconds later, when the variable `baz` is invoked (invoking the inner function we initially labeled `bar`), it duly has *access* to author-time lexical scope, so it can access the variable `a` just as we'd expect.   
所以，几微秒后，当调用变量`baz`（调用内部函数，我们最初标记为bar）时，它就可以访问定义时的词法作用域，所以它可以像我们所期望的那样访问变量`a`。

The function is being invoked well outside of its author-time lexical scope. **Closure** lets the function continue to access the lexical scope it was defined in at author-time.  
该函数在其定义词汇作用域之外的地方被调用。闭包允许函数继续访问定义时的词法作用域。

Of course, any of the various ways that functions can be *passed around* as values, and indeed invoked in other locations, are all examples of observing/exercising closure.   
当然，无论使用哪种方法对函数类型的值进行*传递*，并且在其他位置被调用，这些可以看得到闭包。

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // look ma, I saw closure!
}
```

-- (张静)
We pass the inner function `baz` over to `bar`, and call that inner function (labeled `fn` now), and when we do, its closure over the inner scope of `foo()` is observed, by accessing `a`.                                                      
我们把内部function`baz` 传给 `bar`，当调用这个内部函数（现在叫`fn`）时，`foo()`的内部作用域能被观察到，通过变量`a`。

These passings-around of functions can be indirect, too.                                  
传递函数也可以是间接的。

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // assign `baz` to global variable（分配`baz`给全局变量）
}

function bar() {
	fn(); // look ma, I saw closure!（看，这就是闭包）
}

foo();

bar(); // 2
```

Whatever facility we use to *transport* an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute it, that closure will be exercised.                              
无论通过何种方式将内部函数传递到它的词法作用域外，它都会保持对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。

## 现在我懂了(Now I Can See)

The previous code snippets are somewhat academic and artificially constructed to illustrate *using closure*. But I promised you something more than just a cool new toy. I promised that closure was something all around you in your existing code. Let us now *see* that truth.                                                  
之前的代码段有点学术，并且为了举例说明如何使用闭包人为的修改了结构。但是我保证闭包不仅仅是一个好玩的玩具。闭包存在你之前写过的很多代码段中。让我们现在来看看这个事实。

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

We take an inner function (named `timer`) and pass it to `setTimeout(..)`. But `timer` has a scope closure over the scope of `wait(..)`, indeed keeping and using a reference to the variable `message`.                                                   
我们将内部函数`timer`传给`setTimeout(..)`。但是`timer`有一个覆盖`wait(..)`作用域的闭包，因此保有对变量`message`的引用。

A thousand milliseconds after we have executed `wait(..)`, and its inner scope should otherwise be long gone, that inner function `timer` still has closure over that scope.                                                                        
`wait(..)`执行1000毫秒后，它的内部作用域并不会消失，timer函数依然保有`wait(..)`作用域的闭包。

Deep down in the guts of the *Engine*, the built-in utility `setTimeout(..)` has reference to some parameter, probably called `fn` or `func` or something like that. *Engine* goes to invoke that function, which is invoking our inner `timer` function, and the lexical scope reference is still intact.                                    
深入到*引擎*的内部原理，内置的函数`setTimeout(..)`持有对一些参数的引用，可能叫`fn` 或 `func`或者一些类似的名字。*引擎*会调用这个函数，这里就是`timer`这个函数，词法作用域在这个过程中仍然完整。

**Closure.**                                                                  
**闭包。**

Or, if you're of the jQuery persuasion (or any JS framework, for that matter):                                            
或者，如果你很熟悉jQuery(或者任何JS框架)：

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

I am not sure what kind of code you write, but I regularly write code which is responsible for controlling an entire global drone army of closure bots, so this is totally realistic!                                                               
我不知道你写的哪种代码，但是我写的代码负责控制由闭包机器人组成的整个全球无人机大军，这是完全可以实现的。

(Some) joking aside, essentially *whenever* and *wherever* you treat functions (which access their own respective lexical scopes) as first-class values and pass them around, you are likely to see those functions exercising closure. Be that timers, event handlers, Ajax requests, cross-window messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a *callback function*, get ready to sling some closure around!                         
除开玩笑，本质上无论何时何地，如果将函数（访问它们各自的词法作用域）当作第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包！

**Note:** Chapter 3 introduced the IIFE pattern. While it is often said that IIFE (alone) is an example of observed closure, I would somewhat disagree, by our definition above.                                                                
**注意:** 第三章介绍了IIFE模式。通常认为IIFE是闭包的一个例子，我不同意。

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

This code "works", but it's not strictly an observation of closure. Why? Because the function (which we named "IIFE" here) is not executed outside its lexical scope. It's still invoked right there in the same scope as it was declared (the enclosing/global scope that also holds `a`). `a` is found via normal lexical scope look-up, not really via closure.                                             
这段代码可以运行，但严格来讲它并不是闭包。为什么？因为函数（这里是IIFE）在它的词法作用域外面没有执行。它在定义时所在的作用域中调用（而外部作用域，也就是全局作用域也持有`a`）。`a`是通过普通的词法作用域查找不是闭包被发现的。

While closure might technically be happening at declaration time, it is *not* strictly observable, and so, as they say, *it's a tree falling in the forest with no one around to hear it.*                                                         
尽管从技术上讲，闭包可能发生在定义时，但不明显，所以说，既非风动，亦非幡动，仁者心动耳。

Though an IIFE is not *itself* an example of closure, it absolutely creates scope, and it's one of the most common tools we use to create scope which can be closed over. So IIFEs are indeed heavily related to closure, even if not exercising closure themselves.                                                            
尽管IIFE本身不是一个闭包的例子，但它明显创造了一个作用域，并且也是最常用来创建作用域的工具。因此IIFE确实与闭包息息相关，即使本身并不会真的使用闭包。

Put this book down right now, dear reader. I have a task for you. Go open up some of your recent JavaScript code. Look for your functions-as-values and identify where you are already using closure and maybe didn't even know it before.                                                                                  
亲爱的读者，现在放下书，我有个任务要给你。打开你最近写得js代码，找到其中函数类型的值，并指出哪里使用了闭包，即使你之前可能并不知道它是闭包。

I'll wait.                                     
等着你。

Now... you see!                           
现在，你懂了吧！

## Loops + Closure  --（翠翠）
## 循环和闭包 -- (翠翠)

The most common canonical example used to illustrate closure involves the humble for-loop.                    
最常见的典型例子用来说明闭包就是for循环.     

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Note:** Linters often complain when you put functions inside of loops, because the mistakes of not understanding closure are **so common among developers**. We explain how to do so properly here, leveraging the full power of closure. But that subtlety is often lost on linters and they will complain regardless, assuming you don't *actually* know what you're doing.           
**注意:** 由于 **大多数的开发者** 不是很理解闭包，所以当循环内部包含函数定义时，代码格式检查器经常发出警告。在这里我们会解释如何正确的使用闭包并发挥它的全部威力。但是代码格式检查器没有那么灵敏，它会假设你并不 **真正** 了解你在做什么，所以无论如何他们会发出警告（Linters译成代码格式检查器，complain译成警告）              
The spirit of this code snippet is that we would normally *expect* for the behavior to be that the numbers "1", "2", .. "5" would be printed out, one at a time, one per second, respectively.          
正常情况，我们对这段代码片段行为的 **期望** 是输出数字“1”，“2”，..“5”，分别每秒一次，每次一个。             

In fact, if you run this code, you get "6" printed out 5 times, at the one-second intervals.                
但实际上,如果你运行那段代码，你会得到每隔一秒钟的频率输出5次“6”。

**Huh?**                
**这是为什么**                 

Firstly, let's explain where `6` comes from. The terminating condition of the loop is when `i` is *not* `<=5`. The first time that's the case is when `i` is 6. So, the output is reflecting the final value of the `i` after the loop terminates.                   
首先，我们解释`6`是从哪里来的。这个循环的结束条件是当`i`*不再*`<=5`。条件第一次成立是当`i`等于6时。所以，输出显示的是在循环结束后最终`i`的值。             

This actually seems obvious on second glance. The timeout function callbacks are all running well after the completion of the loop. In fact, as timers go, even if it was `setTimeout(.., 0)` on each iteration, all those function callbacks would still run strictly after the completion of the loop, and thus print `6` each time.                  
仔细想一下又好像显而易见。延迟函数的回调会在循环完成时才执行。实际上，当定时器运行时，即使`setTimeout(..,0)`在每一次迭代，所有这些函数回调仍然严格运行在循环结束后，因此每一次都输出`6`.                  

But there's a deeper question at play here. What's *missing* from our code to actually have it behave as we semantically have implied?                 
但是这里引入了一个更深入问题。我们的代码有什么*缺陷*导致它的行为与我们的语义暗示不一样？                     

What's missing is that we are trying to *imply* that each iteration of the loop "captures" its own copy of `i`, at the time of the iteration. But, the way scope works, all 5 of those functions, though they are defined separately in each loop iteration, all **are closed over the same shared global scope**, which has, in fact, only one `i` in it.                   
*缺陷*是我们试图*假设* 每个循环迭代在迭代时都会*捕获*自己，然后复制一个`i`。但是，根据作用域的工作原理，尽管这5个函数被分别的定义每个循环迭代中，但是它们 **都被封闭在一个共享的全局作用域中**，事实上，只有一个`i`        

Put that way, *of course* all functions share a reference to the same `i`. Something about the loop structure tends to confuse us into thinking there's something else more sophisticated at work. There is not. There's no difference than if each of the 5 timeout callbacks were just declared one right after the other, with no loop at all.               
这样，*当然* 所有的函数共享引用一个`i`.一些关于循环结构往往会混淆我们去认为还有其他更复杂的机制。然而并没有。如果将延迟函数的回调重复定义5次，完全不使用循环，那它同这段代码没有什么不同。      

OK, so, back to our burning question. What's missing? We need more ~~cowbell~~ closured scope. Specifically, we need a new closured scope for each iteration of the loop.                 
好的，回到我们的正题、什么是缺陷？我们需要更多的闭包作用域。特别的，在每一个循环迭代中我们需要一个新的闭包作用域                     

We learned in Chapter 3 that the IIFE creates scope by declaring a function and immediately executing it.                     
在第3章我们知道IIFE通过声明一个函数并且立即执行它来创建作用域。                

Let's try:             
让我们试一下：          
```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Does that work? Try it. Again, I'll wait.          
这样能行吗？试一下，我等着你。          

I'll end the suspense for you. **Nope.** But why? We now obviously have more lexical scope. Each timeout function callback is indeed closing over its own per-iteration scope created respectively by each IIFE.                     
我不卖关子了。**这样不行** 但是为什么？我们现在显然的有更多的词法作用域。每个延迟函数回调都会将会IIFE在每次迭代中创建的作用域封闭起来。              

It's not enough to have a scope to close over **if that scope is empty**. Look closely. Our IIFE is just an empty do-nothing scope. It needs *something* in it to be useful to us.               
**如果作用域是空的** 那么将它们仅仅封闭是不够的。仔细看一下，IIFE只是个什么都没有的空作用域。它需要*一些实质内容*才能让我们使用。                

It needs its own variable, with a copy of the `i` value at each iteration.           
它需要有自己的变量，在每次迭代中复制`i`的值。         

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Eureka! It works!**          
**有了，它能正常工作了***         

A slight variation some prefer is:                     
可以改造成像下面这样
```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

Of course, since these IIFEs are just functions, we can pass in `i`, and we can call it `j` if we prefer, or we can even call it `i` again. Either way, the code works now.                
当然，这些IIFE仅仅是函数，如果我们喜欢我们可以传进去`i`然后我们调用它时作为`j`，或者我们任然可以再次把它叫做`i`，无论哪种方法，这个代码现在都能执行。                   

The use of an IIFE inside each iteration created a new scope for each iteration, which gave our timeout function callbacks the opportunity to close over a new scope for each iteration, one which had a variable with the right per-iteration value in it for us to access.                      
在迭代中使用IIFE会给每个迭代创建一个新的作用域，让我们的延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代中都会含有一个正确值的变量让我们访问。                

Problem solved!          
问题解决了

### Block Scoping Revisited              
 ### 重返块级作用域            

Look carefully at our analysis of the previous solution. We used an IIFE to create new scope per-iteration. In other words, we actually *needed* a per-iteration **block scope**. Chapter 3 showed us the `let` declaration, which hijacks a block and declares a variable right there in the block.              
仔细思考我们对前面解决方案的分析。我们使用IIFE在每次迭代中创建一个新的作用域。换句话说，每次迭代我们都*需要*一个**块级作用域**。在第3章介绍了`let`的声明，可以用来劫持块级作用域，并且在这个作用中声明一个变量。              

**It essentially turns a block into a scope that we can close over.** So, the following awesome code "just works":                     
**它本质上是将一个块变成一个可以关闭的作用域** 因此下面看起来很酷的代码就可以“正常运行”             

```js
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

--(张雪)
*But, that's not all!* (in my best Bob Barker voice). There's a special behavior defined for `let` declarations used in the head of a for-loop. This behavior says that the variable will be declared not just once for the loop, **but each iteration**. And, it will, helpfully, be initialized at each subsequent iteration with the value from the end of the previous iteration.

*不仅如此*（我用Bob Barker声音说道）。在for循环顶部使用`let`定义循环变量会产生一个特殊的行为：变量不会只被声明一次，而是每次循环的时候都会被声明，并且会将上一次循环末尾的值作为初始值。

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

How cool is that? Block scoping and closure working hand-in-hand, solving all the world's problems. I don't know about you, but that makes me a happy JavaScripter.

很酷吧？会级作用域和闭包一起解决了世界级难题，我不知道这对你来说怎么样，但这让我变成了一个更快乐的JS工程师。

## 模块（Modules）

There are other code patterns which leverage the power of closure but which do not on the surface appear to be about callbacks. Let's examine the most powerful of them: *the module*.
一些其他的代码模式也使用到了闭包，但不想回调函数那样明显。让我们看看其中有用的一个：*模块*

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

As this code stands right now, there's no observable closure going on. We simply have some private data variables `something` and `another`, and a couple of inner functions `doSomething()` and `doAnother()`, which both have lexical scope (and thus closure!) over the inner scope of `foo()`.

以上代码中并没有闭包。我们只是简单地放入了私有变量`something` 和 `another`，以及内部函数`doSomething()` 和 `doAnother()`，他们在相同的词法作用域内（也可以看成是闭包），即在`foo()`的作用域内部。

But now consider:

考虑如下代码：

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

This is the pattern in JavaScript we call *module*. The most common way of implementing the module pattern is often called "Revealing Module", and it's the variation we present here.

这种模式在JS中叫做 *模块*，使用模块最常见的方法称作“暴露模块”，就像上面呈现的。

Let's examine some things about this code.

让我仔细看看这段代码。

Firstly, `CoolModule()` is just a function, but it *has to be invoked* for there to be a module instance created. Without the execution of the outer function, the creation of the inner scope and the closures would not occur.

首先，`CoolModule()`只是一个函数，但它必须被执行，因为模块必须要有一个实例。如果函数没有执行，其创建的内部作用域也不会存在。

Secondly, the `CoolModule()` function returns an object, denoted by the object-literal syntax `{ key: value, ... }`. The object we return has references on it to our inner functions, but *not* to our inner data variables. We keep those hidden and private. It's appropriate to think of this object return value as essentially a **public API for our module**.

其次，`CoolModule()`函数返回了一个对象，使用了对象的字面量语法表示`{ key: value, ... }`。我们返回的对象保持了对内部函数的引用，但没有对内部变量的引用。我们隐藏了这些变量。而被返回的对象自然地被当成 **模块的公共API**

This object return value is ultimately assigned to the outer variable `foo`, and then we can access those property methods on the API, like `foo.doSomething()`.
被返回的对象赋值给了我们的外部变量`foo`，我们可以访问它的方法属性。比如`foo.doSomething()`。

**Note:** It is not required that we return an actual object (literal) from our module. We could just return back an inner function directly. jQuery is actually a good example of this. The `jQuery` and `$` identifiers are the public API for the jQuery "module", but they are, themselves, just a function (which can itself have properties, since all functions are objects).

**注意：** 模块并不一定要返回一个对象，我们也可以直接返回一个内部函数。jQuery就是一个例子。`jQuery` 和 `$` 都是jQuery模块的公共API，但其本身只是一个函数（函数也有属性，毕竟所有函数都是对象）.

The `doSomething()` and `doAnother()` functions have closure over the inner scope of the module "instance" (arrived at by actually invoking `CoolModule()`). When we transport those functions outside of the lexical scope, by way of property references on the object we return, we have now set up a condition by which closure can be observed and exercised.

`doSomething()` 和 `doAnother()`是模块实例的闭包（在`CoolModule()`执行时创建）。当我们将这些函数转移到其词法作用域外，我们创建了可以被外部实践和观察的闭包。


To state it more simply, there are two "requirements" for the module pattern to be exercised:

简单来说，模块模式有两条实践要求：

1. There must be an outer enclosing function, and it must be invoked at least once (each time creates a new module instance).

1. 必须有一个外部包裹的函数，而且必须至少被执行一次（每次创建一个新实例）

2. The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope, and can access and/or modify that private state.

2. 包裹函数必须至少返回一个内部函数，这样这个内部函数就是一个保持了私有作用域的闭包，而且这个函数也能够访问或修改内部状态。

An object with a function property on it alone is not *really* a module. An object which is returned from a function invocation which only has data properties on it and no closured functions is not *really* a module, in the observable sense.

一个单独的含有方法属性的对象并不一定是模块，一个被函数返回的只有数据属性的对象也不是模块。

The code snippet above shows a standalone module creator called `CoolModule()` which can be invoked any number of times, each time creating a new module instance. A slight variation on this pattern is when you only care to have one instance, a "singleton" of sorts:

这段代码说明了如何用`CoolModule()`创建一个独立的模块，这个函数可以被执行很多次，每次都会创建一个模块实例。这种模式的一个变体：单例，此时只用创建一个实例。

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

--（张润）
Here, we turned our module function into an IIFE (see Chapter 3), and we *immediately* invoked it and assigned its return value directly to our single module instance identifier `foo`.

Modules are just functions, so they can receive parameters:

现在把我们的函数变成一个立即执行函数(参照第三章),现在我们立即调用它,并且直接将返回值赋给我们的单例模块实例标识符`foo`

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Another slight but powerful variation on the module pattern is to name the object you are returning as your public API:

模块机制另一个简单而强大的用法是命名你要返回的作为公共API的对象.

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

By retaining an inner reference to the public API object inside your module instance, you can modify that module instance **from the inside**, including adding and removing methods, properties, *and* changing their values.

通过在实例对象里保留一个内部的公共API对象的引用,你可以通过内部方法修改模块实例,包括添加和移除方法,属性和改变他们的值.

### Modern Modules
### 模块机制

Various module dependency loaders/managers essentially wrap up this pattern of module definition into a friendly API. Rather than examine any one particular library, let me present a *very simple* proof of concept **for illustration purposes (only)**:

很多模块依赖加载器/管理器　其主要原因是通过包裹这种模块模式将其定义进一个有好的API里.而不是研究某个特殊的类库,为了说明我的用意,让我`简单的`介绍一些概念．

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

The key part of this code is `modules[name] = impl.apply(impl, deps)`. This is invoking the definition wrapper function for a module (passing in any dependencies), and storing the return value, the module's API, into an internal list of modules tracked by name.

And here's how I might use it to define some modules:

这段代码的主要部分是`modules[name] = impl.apply(impl, deps)`,主要作用是定义模块(可以传入任何依赖的模块)中的包装函数,同时将返回值即模块的API存储到一个通过名字来索引的模块内部的列表.


```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

--（李欣）
Both the "foo" and "bar" modules are defined with a function that returns a public API. "foo" even receives the instance of "bar" as a dependency parameter, and can use it accordingly.

Spend some time examining these code snippets to fully understand the power of closures put to use for our own good purposes. The key take-away is that there's not really any particular "magic" to module managers. They fulfill both characteristics of the module pattern I listed above: invoking a function definition wrapper, and keeping its return value as the API for that module.

In other words, modules are just modules, even if you put a friendly wrapper tool on top of them.

### Future Modules

ES6 adds first-class syntax support for the concept of modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both import other modules or specific API members, as well export their own public API members.

**Note:** Function-based modules aren't a statically recognized pattern (something the compiler knows about), so their API semantics aren't considered until run-time. That is, you can actually modify a module's API during the run-time (see earlier `publicAPI` discussion).

By contrast, ES6 Module APIs are static (the APIs don't change at run-time). Since the compiler knows *that*, it can (and does!) check during (file loading and) compilation that a reference to a member of an imported module's API *actually exists*. If the API reference doesn't exist, the compiler throws an "early" error at compile-time, rather than waiting for traditional dynamic run-time resolution (and errors, if any).

ES6 modules **do not** have an "inline" format, they must be defined in separate files (one per module). The browsers/engines have a default "module loader" (which is overridable, but that's well-beyond our discussion here) which synchronously loads a module file when it's imported.

Consider:

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Note:** Separate files **"foo.js"** and **"bar.js"** would need to be created, with the contents as shown in the first two snippets, respectively. Then, your program would load/import those modules to use them, as shown in the third snippet.

`import` imports one or more members from a module's API into the current scope, each to a bound variable (`hello` in our case). `module` imports an entire module API to a bound variable (`foo`, `bar` in our case). `export` exports an identifier (variable, function) to the public API for the current module. These operators can be used as many times in a module's definition as is necessary.

The contents inside the *module file* are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier.

## Review (TL;DR)

Closure seems to the un-enlightened like a mystical world set apart inside of JavaScript which only the few bravest souls can reach. But it's actually just a standard and almost obvious fact of how we write code in a lexically scoped environment, where functions are values and can be passed around at will.

**Closure is when a function can remember and access its lexical scope even when it's invoked outside its lexical scope.**

Closures can trip us up, for instance with loops, if we're not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like *modules* in their various forms.

Modules require two key characteristics: 1) an outer wrapping function being invoked, to create the enclosing scope 2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper.

Now we can see closures all around our existing code, and we have the ability to recognize and leverage them to our own benefit!


## 单词本
| 单词 | 音标 | 释义 |
|------|------|-----|
| closure | ['kloʒɚ] | n. 关闭；终止，结束  vt. 使终止 |
| essentially | [ɪ'sɛnʃəli] | adv. 本质上；本来 |
| regularly | [['reɡjələli]] | adv. 定期地；有规律地；整齐地；匀称地 |
| invoke | [ɪn'vok] | vt. 调用；祈求；引起；恳求 |
| utility | [ju'tɪləti] | n. 实用；效用；公共设施；功用 adj. 实用的；通用的；有多种用途的 |
| illustrate | ['ɪləstret] | vt. 阐明，举例说明；图解 vi. 举例 |
| Linters ||代码格式检查器|
| properly | ['prɑpɚli] | adv. 适当地；正确地；恰当地 |
| subtlety | ['sʌtlti] |n. 微妙；敏锐；精明（灵敏）|
|terminate | ['tɝmɪnet] | vt. 使终止；使结束；解雇vi. 结束，终止；结果adj. 结束的|
| captures | ['kæptʃɚ] | vt. 俘获；夺得；捕捉，拍摄,录制n. 捕获；战利品，俘虏 |
| separately | ['sɛprətli] | adv. 分别地；分离地；个别地 |
| sophisticated | [sə'fɪstɪketɪd | adj. 复杂的；精致的；久经世故的；富有经验的 |
|instance identifier| |实例标识符|
｜module pattern｜ ｜模块模式｜
｜examine｜ ｜研究｜
｜wrapper function｜ ｜包装函数｜

## 疑难杂句
* I regularly write code which is responsible for controlling an entire global drone army of closure bots, so this is totally realistic!                          
我写的代码负责控制由闭包机器人组成的整个全球无人机大军，这是完全可以实现的。

* it's a tree falling in the forest with no one around to hear it.                                                      
既非风动，亦非幡动，仁者心动耳。比喻客观存在和观察认知之间的关系。