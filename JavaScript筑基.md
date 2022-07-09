# 参考资料

《JavaScript高级程序设计》（第四版）

《JavaScript忍者秘籍》（第二版）

《你不知道的JavaScript》

《JavaScript语言精髓与编程实战》

《JavaScript权威指南》

[JavaScript深入系列-冴羽](https://github.com/mqyqingfeng/Blog)
[深入理解JavaScript系列-汤姆大叔](https://www.cnblogs.com/TomXu/archive/2011/12/15/2288411.html)



## 第一章 JS执行原理

### 执行上下文

#### 初始化全局对象

执行 JavaScript 代码的第一步，就是初始化全局对象

最新262规范定义的全局对象

[ECMAScript® 2022 Language Specification (ecma-international.org)](https://262.ecma-international.org/13.0/#sec-global-object)

> The global object:
>
> - is created before control enters any [execution context](https://262.ecma-international.org/13.0/#sec-execution-contexts).
> - does not have a [[Construct]] internal method; it cannot be used as a [constructor](https://262.ecma-international.org/13.0/#constructor) with the `new` operator.
> - does not have a [[Call]] internal method; it cannot be invoked as a function.
> - has a [[Prototype]] internal slot whose value is [host-defined](https://262.ecma-international.org/13.0/#host-defined).
> - may have [host-defined](https://262.ecma-international.org/13.0/#host-defined) properties in addition to the properties defined in this specification. This may include a property whose value is the global object itself.



翻译一下重点, 大概是

-  控制器 `control` 在进入全局上下文 `execution context` 之前 `global object` 会被创建；可以理解为js引擎在执行代码之前，会在堆内存中创建一个**全局对象：global object**
- 里面会包含宿主环境 `host-defined` 的属性 `properties` ；比如在浏览器中会有 `window` 属性指向自己

- 其中第四点

  > A host environment is a particular choice of definition for all [host-defined](https://262.ecma-international.org/13.0/#host-defined) facilities. A [host environment](https://262.ecma-international.org/13.0/#host-environment) typically includes objects or functions which allow obtaining input and providing output as [host-defined](https://262.ecma-international.org/13.0/#host-defined) properties of the [global object](https://262.ecma-international.org/13.0/#sec-global-object).

  在宿主环境中会内置一些可以调用的对象（原型）或者函数可以通过 `global object` 被使用；例如Date、Array、Number、setTimeout等



#### 执行上下文

前面提到进入执行上下文之前会创建一个全局对象，那么执行上下文是什么？



[ECMAScript® 2022 Language Specification (ecma-international.org)](https://262.ecma-international.org/13.0/#sec-execution-contexts)

> An execution context is purely a specification mechanism and need not correspond to any particular artefact of an ECMAScript implementation. It is impossible for ECMAScript code to directly access or observe an execution context.
>
> 执行上下文只是一种规范机制。其不需要对应于代码的某种实现，也不能通过代码直接访问或者观察

执行上下文 `execution context` 是ECMA-262标准里的一个抽象概念，用于同可执行代码 `executable code` 概念进行区分。
在概念中，可执行代码是执行上下文的一部分。



这玩意儿有什么作用呢？

>An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation. At any point in time, there is at most one execution context per [agent](https://262.ecma-international.org/13.0/#agent) that is actually executing code. This is known as the [agent](https://262.ecma-international.org/13.0/#agent)'s running execution context.
>
>执行上下文是一个规范设备，用于跟踪代码的运行时评估，任一时间点只会代理一个执行上下文。

在JavaScript中，函数执行的基础单元是函数。且一个函数可以调用第二个函数，第二个又可以调用第三个函数，以此类推。

因为JavaScript是单线程的，当完成函数调用时，程序需要回到函数调用的位置，所以需要通过跟踪代码的执行上下文来确定位置。而跟踪执行上下文的方式，是使用执行上下文栈。

一旦发生函数调用，当前的执行上下文必须停止执行，并创建新的函数执行上下文来执行函数。当函数执行完成后，将函数执行上下文销毁，并重新回到发生调用时的执行上下文中。所以需要跟踪执行上下文——正在执行的上下文以及正在等待的上下文。



最重要的是，执行上下文包含**作用域链** `scope chain`、**变量环境**`VariableEnvironment `、**this**、**可执行代码**。在后续的文章中会将这些内容展开。



说人话就是当前变量或函数的执行上下文，决定了他们可以访问到哪些数据，以及可以进行哪些操作。而且当前变量或函数都只能由一个执行上下文限制。



#### 创建执行上下文

这个时候问题又来了，这个执行上下文它是什么时候被创建的？



>  A new execution context is created whenever control is transferred from the executable code associated with the currently [running execution context](https://262.ecma-international.org/13.0/#running-execution-context) to executable code that is not associated with that execution context.

当 JavaScript 代码执行一段**可执行代码** `executable code` 时，会创建对应的**执行上下文** `execution context`



**三种类型：**

- **全局执行上下文（ GEC）**：不在任何函数中的代码都位于全局执行上下文中，只有一个，当JavaScript程序开始执行时就已经创建了全局上下文。浏览器中的全局对象就是 window 对象，在非严格模式下，`this` 指向这个全局对象。
- **函数执行上下文（ FEC）**：**只有调用**函数时，才会为该函数创建一个新的执行上下文，可以存在无数个，每当一个新的执行上下文被创建，它都会按照特定的顺序执行一系列步骤。
- **Eval 函数执行上下文（eval代码）**： 指的是运行在 `eval` 函数中的代码，很少使用。





#### 执行上下文栈

既然每遇到一个函数（全局也可以算是一个函数）都会创建一个执行上下文，那么如何管理这些上下文？



js 引擎内部有一个执行上下文栈 `execution context stack` （ESC），它是用于执行代码的调用栈

>The execution context stack is used to track execution contexts. The [running execution context](https://262.ecma-international.org/13.0/#running-execution-context) is always the top element of this stack. A new execution context is created whenever control is transferred from the executable code associated with the currently [running execution context](https://262.ecma-international.org/13.0/#running-execution-context) to executable code that is not associated with that execution context. The newly created execution context is pushed onto the stack and becomes the [running execution context](https://262.ecma-international.org/13.0/#running-execution-context).
>
>当前正在运行的执行上下文总是在执行上下文栈的栈顶，每当一个可执行代码（函数）被创建时就会创建一个新的执行上下文，新创建的执行上下文就会被压入栈中，并且成为**正在运行的执行上下文**

简单来说就是有个栈来管理执行上下文，每创建一个执行上下文（无论类型），都会被压入栈中。正在运行的执行上下文始终处于栈顶，当函数执行完毕，就会被弹出栈（回收）。

因此可以得出一个简单的结论，由于全局执行上下文是第一个执行的，因此它始终在栈底，当全局代码执行完毕，才会被弹出销毁



#### 进入执行上下文











#### 生命周期

前面讲了执行上下文怎么创建，执行，在最后讲一下回收的原理

执行上下文又包括三个生命周期阶段：**创建阶段 --> 执行阶段 --> 回收阶段**

上下文在其所有代码都执行完毕后会被销毁，包括定义在它上面的所有变量和函数（全局上下文在应用程序退出前才会被销毁，比如关闭网页或退出浏览器）。

而函数在执行完之后，这个函数执行上下文对应的 VO(AO) 是否销毁，要看这个 AO 中的数据是否被其它作用域引用（闭包的自由变量）



#### 相关调试

虽然执行上下文栈是JavaScript内部概 念，但仍然可以通过JavaScript调试器中查看，在JavaScript调试器中可 以看到对应的调用栈`call back`。

![](https://github.com/4may-mcx/images/blob/main/js/ecstack_1.png)