# 你不知道的JS（You Don't Know JS: Up & Going）  --(罗尧)
# 第三章：进入YDKJS（Chapter 3: Into YDKJS）


What is this series all about? Put simply, it's about taking seriously the task of learning *all parts of JavaScript*, not just some subset of the language that someone called "the good parts," and not just whatever minimal amount you need to get your job done at work.    
这个系列是什么？ 简单地说，它将帮你深入学习*JavaScript*的所有部分，而不只是语言中被称为“好的部分”的那些内容，你需要学习不止一点。

Serious developers in other languages expect to put in the effort to learn most or all of the language(s) they primarily write in, but JS developers seem to stand out from the crowd in the sense of typically not learning very much of the language. This is not a good thing, and it's not something we should continue to allow to be the norm.    
其他语言开发者期望读者努力学习他们主要写的大多数或所有的语言，但是JS开发人员似乎与众不同，通常不会让你学习很多语言。这不是一件好事，也不应该继续被允许成为规范。

The *You Don't Know JS* (*YDKJS*) series stands in stark contrast to the typical approaches to learning JS, and is unlike almost any other JS books you will read. It challenges you to go beyond your comfort zone and to ask the deeper "why" questions for every single behavior you encounter. Are you up for that challenge?   
*你不知道JS*（*YDKJS*）系列与典型的学习JS的教程形成鲜明对比，并且几乎与你读的任何其他JS书不同。 它可能会让你感到不舒服，对你遇到的每一个单一的行为探索更深层次的原因。你准备好迎接挑战了吗？

I'm going to use this final chapter to briefly summarize what to expect from the rest of the books in the series, and how to most effectively go about building a foundation of JS learning on top of *YDKJS*.    
我会在最后一章简要总结一下这系列中其他书的目标，以及如何最有效系统地从*YDKJS*学习JS基础。

## 作用域和闭包（Scope & Closures）  --(罗尧)

Perhaps one of the most fundamental things you'll need to quickly come to terms with is how scoping of variables really works in JavaScript. It's not enough to have anecdotal fuzzy *beliefs* about scope.   
或许你需要快速地去理解的一个基本的内容就是变量的作用域在JavaScript中是如何工作的。对于作用域只是了解大概是不够的。

The *Scope & Closures* title starts by debunking the common misconception that JS is an "interpreted language" and therefore not compiled. Nope.       
*作用域和闭包*标题一开始就揭露了一个常见的误解——JS是一门“解释型语言”，因此不编译。然而事实并非如此。

The JS engine compiles your code right before (and sometimes during!) execution. So we use some deeper understanding of the compiler's approach to our code to understand how it finds and deals with variable and function declarations. Along the way, we see the typical metaphor for JS variable scope management, "Hoisting."      
JS引擎在执行之前（有时是在执行中！）编译您的代码。因此，我们可以从编译器的角度进行更深入的理解，以了解它如何查找、处理变量及函数声明。在这过程中我们会看到关于JS变量作用域管理的典型比喻。

This critical understanding of "lexical scope" is what we then base our exploration of closure on for the last chapter of the book. Closure is perhaps the single most important concept in all of JS, but if you haven't first grasped firmly how scope works, closure will likely remain beyond your grasp.         
对“词法作用域”的及其重要的理解将在本书最后一章进行探讨。闭包可能是所有JS中最重要的一个概念，但如果你没有先深层次地理解闭包是如何工作的，闭包可能会超出你的掌握。

One important application of closure is the module pattern, as we briefly introduced in this book in Chapter 2. The module pattern is perhaps the most prevalent code organization pattern in all of JavaScript; deep understanding of it should be one of your highest priorities.          
闭包的一个重要应用是模块模式，我们会在这本书的第2章中简要介绍。模块模式可能是所有JavaScript中最流行的代码组织模式; 对它的深刻理解应该是你的最高优先事项之一。

## this和对象原型（this & Object Prototypes）  --(张静)

Perhaps one of the most widespread and persistent mistruths about JavaScript is that the `this` keyword refers to the function it appears in. Terribly mistaken.
也许JavaScript中最常见最持久的错误，就是'this'关键词引用它出现的function。大错特错。（refers to：引用）

The `this` keyword is dynamically bound based on how the function in question is executed, and it turns out there are four simple rules to understand and fully determine `this` binding.
the function in question：所讨论的function
'this'关键词基于所讨论的function是如何执行的而动态绑定，结果有四条简单的规则来理解和确定‘this’绑定。

Closely related to the `this` keyword is the object prototype mechanism, which is a look-up chain for properties, similar to how lexical scope variables are found. But wrapped up in the prototypes is the other huge miscue about JS: the idea of emulating (fake) classes and (so-called "prototypal") inheritance.
与‘this’关键词密切相关的是对象原型机制，这是一条属性查找链，类似于怎样去查找词汇作用域变量。但是在原型中包装是JS的另一巨大失误：模仿类和继承的想法。

Unfortunately, the desire to bring class and inheritance design pattern thinking to JavaScript is just about the worst thing you could try to do, because while the syntax may trick you into thinking there's something like classes present, in fact the prototype mechanism is fundamentally opposite in its behavior.
不幸的，将类和继承这种设计模式引入js的想法可能是你尝试过的最糟糕的事情，因为这个语法会诱使你认为真的存在类一样的事物，事实上，原型机制从根本上与这种行为相反。(trick into:诱使)

What's at issue is whether it's better to ignore the mismatch and pretend that what you're implementing is "inheritance," or whether it's more appropriate to learn and embrace how the object prototype system actually works. The latter is more appropriately named "behavior delegation."
问题是，假设你正在用继承，却忽略这种不匹配更好，还是更适合学习了解对象原型系统是怎样工作的。后者更适合命名为“行为委托”。

This is more than syntactic preference. Delegation is an entirely different, and more powerful, design pattern, one that replaces the need to design with classes and inheritance. But these assertions will absolutely fly in the face of nearly every other blog post, book, and conference talk on the subject for the entirety of JavaScript's lifetime.
这不仅仅是语法偏好。委托是一个完全不同的，更强大的设计模式，它代替了用类和继承进行设计的需要。但是这些断言一定不会考虑到那些在JavaScript整个生命周期中讨论这个主题的几乎所有的博客、书籍和会议（fly in the face of ： 悍然不顾）

The claims I make regarding delegation versus inheritance come not from a dislike of the language and its syntax, but from the desire to see the true capability of the language properly leveraged and the endless confusion and frustration wiped away.
我对代理与继承的声明并不是因为我对这种语言和语法的不喜欢，而是来自于一个期望，期望看到语言的真正能力被适当地利用以及无穷的混乱和沮丧被消除。

But the case I make regarding prototypes and delegation is a much more involved one than what I will indulge here. If you're ready to reconsider everything you think you know about JavaScript "classes" and "inheritance," I offer you the chance to "take the red pill" (*Matrix* 1999) and check out Chapters 4-6 of the *this & Object Prototypes* title of this series.
但是我做的这个原型和代表的案例是我想参与的更多的一个。如果你准备重新思考关于你对js中类和继承的认识，我给你提供“选择红色药丸”的机会，(源自黑客帝国：矩阵，1999)，请查阅本系列标题为“this和对象原型”的第四到六章。

## Types & Grammar  --(翠翠)

The third title in this series primarily focuses on tackling yet another highly controversial topic: type coercion. Perhaps no topic causes more frustration with JS developers than when you talk about the confusions surrounding implicit coercion.

By far, the conventional wisdom is that implicit coercion is a "bad part" of the language and should be avoided at all costs. In fact, some have gone so far as to call it a "flaw" in the design of the language. Indeed, there are tools whose entire job is to do nothing but scan your code and complain if you're doing anything even remotely like coercion.

But is coercion really so confusing, so bad, so treacherous, that your code is doomed from the start if you use it?

I say no. After having built up an understanding of how types and values really work in Chapters 1-3, Chapter 4 takes on this debate and fully explains how coercion works, in all its nooks and crevices. We see just what parts of coercion really are surprising and what parts actually make complete sense if given the time to learn.

But I'm not merely suggesting that coercion is sensible and learnable, I'm asserting that coercion is an incredibly useful and totally underestimated tool that *you should be using in your code.* I'm saying that coercion, when used properly, not only works, but makes your code better. All the naysayers and doubters will surely scoff at such a position, but I believe it's one of the main keys to upping your JS game.

Do you want to just keep following what the crowd says, or are you willing to set all the assumptions aside and look at coercion with a fresh perspective? The *Types & Grammar* title of this series will coerce your thinking.

## Async & Performance  --(张雪)

The first three titles of this series focus on the core mechanics of the language, but the fourth title branches out slightly to cover patterns on top of the language mechanics for managing asynchronous programming. Asynchrony is not only critical to the performance of our applications, it's increasingly becoming *the* critical factor in writability and maintainability.

The book starts first by clearing up a lot of terminology and concept confusion around things like "async," "parallel," and "concurrent," and explains in depth how such things do and do not apply to JS.

Then we move into examining callbacks as the primary method of enabling asynchrony. But it's here that we quickly see that the callback alone is hopelessly insufficient for the modern demands of asynchronous programming. We identify two major deficiencies of callbacks-only coding: *Inversion of Control* (IoC) trust loss and lack of linear reason-ability.

To address these two major deficiencies, ES6 introduces two new mechanisms (and indeed, patterns): promises and generators.

Promises are a time-independent wrapper around a "future value," which lets you reason about and compose them regardless of if the value is ready or not yet. Moreover, they effectively solve the IoC trust issues by routing callbacks through a trustable and composable promise mechanism.

Generators introduce a new mode of execution for JS functions, whereby the generator can be paused at `yield` points and be resumed asynchronously later. The pause-and-resume capability enables synchronous, sequential looking code in the generator to be processed asynchronously behind the scenes. By doing so, we address the non-linear, non-local-jump confusions of callbacks and thereby make our asynchronous code sync-looking so as to be more reason-able.

But it's the combination of promises and generators that "yields" our most effective asynchronous coding pattern to date in JavaScript. In fact, much of the future sophistication of asynchrony coming in ES7 and later will certainly be built on this foundation. To be serious about programming effectively in an async world, you're going to need to get really comfortable with combining promises and generators.

If promises and generators are about expressing patterns that let our programs run more concurrently and thus get more processing accomplished in a shorter period, JS has many other facets of performance optimization worth exploring.

Chapter 5 delves into topics like program parallelism with Web Workers and data parallelism with SIMD, as well as low-level optimization techniques like ASM.js. Chapter 6 takes a look at performance optimization from the perspective of proper benchmarking techniques, including what kinds of performance to worry about and what to ignore.

Writing JavaScript effectively means writing code that can break the constraint barriers of being run dynamically in a wide range of browsers and other environments. It requires a lot of intricate and detailed planning and effort on our parts to take a program from "it works" to "it works well."

The *Async & Performance* title is designed to give you all the tools and skills you need to write reasonable and performant JavaScript code.

## ES6 & Beyond  --(张润)

No matter how much you feel you've mastered JavaScript to this point, the truth is that JavaScript is never going to stop evolving, and moreover, the rate of evolution is increasing rapidly. The spirit of to embrace that we'll never fully *know* every part of JS, because as soon as you master it all, there's going to be new stuff coming down the line that you'll need to learn.
无论你感觉你已经多大程度上掌握了JavaScript,而事实却是JavaScript从未停止发展的脚步，而且，发展的速度正在逐步增快。心理上必须承认，我们可能永远也不会完全*了解*JS中的每一个部分,因为一旦你掌握了所有JS知识,又会有新的落于纸上的知识需要你掌握。

This title is dedicated to both the short- and mid-term visions of where the language is headed, not just the *known* stuff like ES6 but the *likely* stuff beyond.
这个标题致力于提出对当下最流行的语言的短期和中期的愿景,不仅仅是为了让你*了解*类似于ES6这样的知识,而是要提出一些超前的*有可能*出现的知识。

While all the titles of this series embrace the state of JavaScript at the time of this writing, which is mid-way through ES6 adoption, the primary focus in the series has been more on ES5. Now, we want to turn our attention to ES6, ES7, and ...
尽管本系列的所有标题在编写时都说明了JavaScript的发展现状,即正是ES6被采用的中期,但主旨还是集中在ES5更多一些.当下,我们期望你将目光转移到ES6,ES7上等等

Since ES6 is nearly complete at the time of this writing, *ES6 & Beyond* starts by dividing up the concrete stuff from the ES6 landscape into several key categories, including new syntax, new data structures (collections), and new processing capabilities and APIs. We cover each of these new ES6 features, in varying levels of detail, including reviewing details that are touched on in other books of this series.
既然在写这本书时,ES6已经接近尾声,*ES6 & 更高版本* 首先将ES6中的具体知识分为几个主要的领域,包括新的语法,新的数据结构(集合),和新的执行性能和APIs.本书在很多细节方面涵盖了一些新的ES6功能,还参考了其他书上本系列的一些细节知识.

Some exciting ES6 things to look forward to reading about: destructuring, default parameter values, symbols, concise methods, computed properties, arrow functions, block scoping, promises, generators, iterators, modules, proxies, weakmaps, and much, much more! Phew, ES6 packs quite a punch!
一些令人兴奋的ES6功能期待被阅读: 解构,默认参数值,符号,简明的方法,计算属性,箭头函数,块范围,promises函数,生成器,迭代器,模块,代理,weakmaps,还有很多很多！ES6是如此的强有力!

The first part of the book is a roadmap for all the stuff you need to learn to get ready for the new and improved JavaScript you'll be writing and exploring over the next couple of years.
这本书的第一部分是一个路线图,里面包含你将会用到的,在未来几年不断发展的,即你需要学习的新的和优化后的JavaScript.

The latter part of the book turns attention to briefly glance at things that we can likely expect to see in the near future of JavaScript. The most important realization here is that post-ES6, JS is likely going to evolve feature by feature rather than version by version, which means we can expect to see these near-future things coming much sooner than you might imagine.
本书后面的一部分主要致力于简单浏览一下我们将来所期望看到的JavaScript.最需要我们认识到的是,ES6之后,JS很可能以功能演进代替版本演进,这意味着近未来这些演进可能比你想象中的更快到来.

The future for JavaScript is bright. Isn't it time we start learning it!?
未来JavaScript值得期待.此时正是我们开始学习它的大好时机！


## 回顾(Review)  --(李欣)

The *YDKJS* series is dedicated to the proposition that all JS developers can and should learn all of the parts of this great language. No person's opinion, no framework's assumptions, and no project's deadline should be the excuse for why you never learn and deeply understand JavaScript.
*YDKJS*系列致力于让所有的JS开发者们可以并且应该去学习这门伟大语言的所有部分。个人意见，框架假设和项目截止日期都不应该成为你不学习和深入了解JavaScript的借口。


We take each important area of focus in the language and dedicate a short but very dense book to fully explore all the parts of it that you perhaps thought you knew but probably didn't fully.
我们把语言中所关注的每一个重点领域提取出来并且制作成一本简短但是浓缩了很多精华的书，在这些书里全面的阐述了那些你或许只知其然不知其所以然的内容


"You Don't Know JS" isn't a criticism or an insult. It's a realization that all of us, myself included, must come to terms with. Learning JavaScript isn't an end goal but a process. We don't know JavaScript, yet. But we will!
"你不知道的JS"并不是一种批评或侮辱。这是一种认知，我们所有人，包括我自己，必须接受它。学习JavaScript是一个过程而不是一个最终目标。我们不了解JavaScript，但是我们终将会了解并完全掌握它。


## 单词本

| 单词 | 音标 | 释义 |
| ---- | ---- | ---- |
| encounter | [ɛnˈkaʊntɚ] | vt.遭遇;不期而遇;对抗n.相遇，碰见;遭遇战;对决，冲突vi.碰见，尤指不期而遇|
| foundation | [faʊnˈdeʃən] | n.基础;地基;粉底;基金（会）|
| fundamental | [ˌfʌndəˈmɛntl]| adj.基础的，基本的，根本的，重要的，原始的，主要的，十分重大的;[物]基频的，基谐波的;[乐]基音的|
| come to terms with || 与…达成协议;忍受，对待 |
|anecdotal|[ˌænɪkˈdoʊtl]| adj.轶事的;趣闻的;多轶事的;含轶事的 |
| fuzzy | [ˈfʌzi] | adj.模糊的;绒毛般的，毛茸茸的;糊涂的，含糊不清的 |
| beliefs | [bɪ'li:fs] | n.信，信任（belief的复数形式）|
| lexical | [ˈlɛksɪkəl] | adj.词汇的;具词典性质的，词典的 |
| widespread | [ˈwʌɪdsprɛd] | adj. 普遍的，广泛的；分布广的 |
| persistent | [pə'sɪst(ə)nt] | adj. 固执的，坚持的；持久稳固的 |
| dynamically | [dai'næmikəli] | adv. 动态地；充满活力地；不断变化地 |
| bound | [baʊnd] | adj. 有义务的；必定的；受约束的；装有封面的  vt. 束缚；使跳跃  n. 范围；跳跃  vi. 限制；弹起 |
| executed | ['eksɪ,kjʊtɪd］ | adj. 已执行的；已生效的 |
| mechanism | ['mek(ə)nɪz(ə)m] | n. 机制；原理，途径；进程；机械装置；技巧 |
| lexical | ['leksɪk(ə)l] | adj. 词汇的；[语] 词典的；词典编纂的 |
| miscue | [mɪs'kjuː] | n. 撞歪；失误  vi. 撞歪；错过提示 |
| emulate | ['emjʊleɪt] | vt. 仿真；模仿；尽力赶上；同…竞争  n. 仿真；仿效 |
| syntax | ['sɪntæks] | n. 语法；句法；有秩序的排列 |
| fundamentally | [fʌndə'mentəlɪ] | adv. 根本地，从根本上；基础地 |
| embrace | [ɪm'breɪs; em-] | vt. 拥抱；信奉，皈依；包含  vi. 拥抱  n. 拥抱 |
| delegation | [delɪ'geɪʃ(ə)n] | n. 代表团；授权；委托 |
| assertions | [ə'sə:ʃən] | n. 断言（assertion的复数）；认定 |
| versus | ['vɜːsəs] | prep. 对；与...相对；对抗 |
| leverage | ['liːv(ə)rɪdʒ; 'lev(ə)rɪdʒ] | n. 手段，影响力；杠杆作用；杠杆效率  v. 利用；举债经营 |
| indulge | [ɪn'dʌldʒ] | vt. 满足；纵容；使高兴；使沉迷于…  vi. 沉溺；满足；放任 |
| dedicated | ['dɛdə'ketɪd] | adj. 专用的；专注的；献身的 |
| proposition | [,prɑpə'zɪʃən] | n. [数] 命题；提议；主题；议题 |
| assumption | [ə'sʌmpʃən] | n. 假定；设想；担任；采取 |
| dense | [dɛns] | adj. 稠密的；浓厚的；愚钝的 |
| criticism | ['krɪtə'sɪzəm] | n. 批评；考证；苛求 |
| insult | [ɪn'sʌlt] | vt. 侮辱；辱骂；损害  n. 侮辱；凌辱；无礼 |

