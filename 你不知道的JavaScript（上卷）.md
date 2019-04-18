# 你不知道的JavaScript（上卷）

## 第一部分  作用域和闭包

### 第1章  作用域是什么

#### 1.1编译原理

尽管通常将JavaScript归类为“动态”或“解释执行”语言，但事实上它是一门编译语言。

但与传统的编译语言不同，它不是提前编译的，编译结果也不能在分布式系统中进行移植。

尽管如此，JavaScript引擎进行编译的步骤和传统的编译语言非常相似，在某些环节可能比预想的要复杂。

在传统编译语言的流程中，程序中的一段源代码在执行之前会经历三个步骤，统称为“编译”。

- 分词/词法分析
- 解析/语法分析
- 代码生成

比起那些编译过程只有三个步骤的语言的编译器，JavaScript引擎要复杂得多。例如，在语法分析和代码生成阶段有特定的步骤来对运行性能进行优化，包括对冗余元素进行优化等。

JavaScript引擎不会有大量的（想其他语言编译器那么多的）时间用来进行优化，因为与其他语言不同，JavaScript的编译过程不是发生在构建之前的。

对于JavaScript来说，大部分情况下编译发生在代码执行前的几微妙（甚至更短！）的时间内。在我们所要讨论的作用域背后，JavaScript引擎用尽了各种办法（比如JIT，可以延迟编译甚至实施重编译）来保证性能最佳。

#### 1.2  理解作用域

##### 1.2.1  演员表

- 引擎

  从头到尾负责整个JavaScript程序的编译及执行过程。

- 编译器

  引擎的好朋友之一，负责语法分析及代码生成等脏活累活。

- 作用域

  引擎的另一位好朋友，负责收集并维护由所有生命的标识符（变量）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

##### 1.2.2  对话

当你看见var a = 2;这段程序时，很可能认为这是一句声明。但我们的新朋友引擎却不这么看。事实上，引擎认为这里有两个完全不同的声明，一个由编译器在编译时处理，另一个则由引擎在运行时处理。

下面我们将 var a = 2;分解，看看引擎和它的朋友么是如何协同工作的。

编译器首先会将这段程序分解成词法单元，然后将词法单元解析成一个树结构。但是当编译器开始进行代码生成时，它对这段程序的处理方式会和预期的有所不同。

可以合理地假设编译器所产生的代码能够用下面的伪代码进行概括：“为一个变量分配内存，将其命名为a，然后将值2保存进这个变量。”然而，这并不完全正确。

事实上编译器会进行如下处理。

1.遇到 var a，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的集合中。如果是，编译器会忽略声明，继续进行编译；否则它会要求作用域在当前作用域的集合中声明一个新的变量，并命名为a。

2.接下来编译器会为引擎生成运行时所需的代码，这些代码被用来处理 a = 2 这个赋值操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫做a的变量。如果是，引擎就会使用这个变量；如果否，引擎会继续查找该变量。

如果引擎最终找到了a变量，就会将2赋值给它。否则引擎就会举手示意并抛出一个异常！

##### 1.2.3  编译器有话说

编译器在编译过程的第二步中生成了代码，引擎执行它时，会通过查找变量a来判断它是否已声明过。查找的过程由作用域进行协助，但是引擎执行怎样的查找，会影响最终的查找结果。

LHS和RHS的含义是“赋值操作的左侧或右侧”并不一定意味着就是“=赋值操作符的左侧或右侧“。赋值操作还有其他几种形式，因此在概念上最好将其理解为”赋值操作的目标是谁（LHS)“以及”谁是赋值操作的源头（RHS）“。

#### 1.3  作用域嵌套

我们说过，作用域是根据名称查找变量的一套规则。实际情况中，通常需要同时估计几个作用域。

当一个块或函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。因此，在当前作用域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量，或抵达最外层的作用域（也就是全局作用域）为止。

#### 1.4  异常

为什么区分LHS和RHS是一件重要的事情？

因为在变量还没有声明（在任何作用域中都无法找到该变量）的情况下，这两种查询的行为是不一样的。

如果RHS查询在所有嵌套的作用域中追寻不到所需的变量，引擎就会抛出ReferenceError异常。值得注意的是，ReferenceError是非常重要的异常类型。

相较之下，当引擎执行LHS查询时，如果在顶层（全局作用域）中无法找到目标变量，全局作用域中就会创建一个具有该名称的变量，并将其返还给引擎，前提是程序运行在非“严格模式”下。

### 第2章  词法作用域

#### 2.1  词法阶段

第1章介绍过，大部分标准语言编译器的第一个工作阶段叫做词法化（也叫单词化）。回忆一下，词法化的过程会对源代码中的字符进行检查，如果是有状态的解析过程，还会赋予单词语义。

简单地说，词法作用域就是定义在词法阶段的作用域。换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变（大部分情况下是这样的）。

#### 2.2  欺骗词法

### 第3章  函数作用域和块作用域

#### 3.1  函数中的作用域

对于前面提出的问题，最常见的答案是JavaScript具有基于函数的作用域，意味着每声明一个函数都会为其自身创建一个气泡，而其他结构都不会创建作用域气泡。但事实上这并不完全正确。

函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上在嵌套的作用域中也可以使用）。这种设计方案是非常有用的，能充分利用JavaScript变量可以根据需要改变值类型的“动态”特性。

但与此同时，如果不细心处理那些可以在整个作用域范围内被访问的变量，可能会带来意想不到的问题。

#### 3.2  隐藏内部实现

对函数的传统认知就是先声明一个函数，然后再向里面添加代码。但反过来也可可以带来一些启示：从缩写的代码中挑选出一个任意的片段，然后用函数声明对它就行包装，实际上就是把这些代码“隐藏“起来了。

实际的结果就是在这个代码片段的周围创建了一个作用域气泡，也就是说这段代码中的任何声明（变量或函数）都将绑定在这个新创建的包装函数的作用域中，而不是先前所在的作用域中。换句话说，可以把变量和函数包裹在一个函数的作用域中，然后用这个作用域来“隐藏”它们。

为什么“隐藏”变量和函数是一个有用的技术？

- 最小特权原则
- 规避冲突

#### 3.3  函数作用域

区分函数声明和表达式最简单的方法是看function关键字出现在声明中的位置（不仅仅是一行代码，而是整个声明中的位置）。如果function是声明中的第一个词，否则就是一个函数表达式。

##### 3.3.1  匿名和具名

匿名函数表达式，function()...没有名称标识符。

匿名函数表达式书写起来简单快捷，很多库和工具也倾向使用这种风格的代码。但是它也有几个缺点需要考虑。

1. 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。
2. 如果没有函数名，当函数需要引用自身时只能使用已经过期的argument.callee引用，比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身。
3. 匿名函数省略了对于代码可读性/可理解性很重要的函数名。一个描述性的名称可以让代码不言自明。

##### 3.3.2  立即执行函数表达式

```js
var a = 2;
(function foo(){
    var a = 3;
    console.log(a);//3
})()
console.log(a);//2
```

由于函数被包含在一对（）括号内部，因此成为了一个表达式，通过在末尾加上另外一个（）可以立即执行这个函数，比如（function  foo（）{...}）（）。第一个（）将函数变成表达式，第二个（）执行了这个函数。

这种模式很常见，几年前社区给它规定了一个属于：IIFE，代表立即执行函数表达式；

两种形式，第一种形式中函数表达式被包含在（）中，然后在后面用另一个（）括号来调用。第二种形式中用来调用的（）括号被移进了用来包装的（）括号中。

IIFE的另一个非常普遍的进阶用法是把他们当做函数调用并传递参数进去。

#### 3.4  块作用域

块作用域是一个用来对之前的最小授权原则进行扩展的工具，将代码从在函数中隐藏信息扩展为在块中隐藏信息。

##### 3.4.1  with

##### 3.4.2  try/catch

try/catch的catch分局会创建一个块作用域，其中声明的变量仅在catch内部有效。

```js
try {
    undifined();//执行一个非法操作来强制制造一个异常
}catch(err){
    console.log(err);//能够正常执行
}
console.log(err);//ReferenceError:err nor found
```

正如你所看到的，err仅存在catch分居内部，当试图从别处引用它是会抛出错误。

##### 3.4.3  let

let关键字可以将变量绑定到所在的任意作用域中（通常是{ .. }内部）。换句话说，let为其声明的变量隐式了所在的块作用域。

通常来讲，显式地代码优于隐式或一些精巧但不清晰的代码。显式地块作用域风格非常容易书写，并且和其他语言中块作用域的工作原理一致：

```js
var foo = true;
if(foo){
    {//<--显式地块
        let bar = foo * 2;
        bar = something(bar);
        console.log(bar);
    }
}
console.log(bar);//ReferenceError
```

只要声明是有效的，在声明中的任意位置都可以使用{ .. }括号来为let创建一个用于绑定的块。在这个例子中，我们在if声明内部显式地创建了一个块，如果需要对其进行重构，整个块都可以被方便地移动而不会对外部if声明的位置和语义产生任何影响。

在第4章，我们会讨论提升，提升是指声明会被视为存在于其所出现的作用域的整个范围内。但是使用let进行的声明不会在块作用域中进行提升。声明的代码被运行之前，声明并不“存在”。

```js
{
    console.log(bar);//ReferenceError!
    let bar = 2;
}
```

1. 垃圾收集

   另一个块作用域非常有用的原因和闭包及回收内存垃圾的回收机制相关。这里简要说明一下，而内部的实现原理，也就是闭包的机制会在第5章详细解释。

   考虑以下代码：

   ```js
   function process(data){
       // 在这里做点有趣的事情
   }
   var someReallyBigData = { .. };
   process( someReallyBigData );
   var btn = document.getElementById( "my_button" );
   btn.addEventListener('click',function click(evt){
       console.log('button clicked');
   },/*capturingPhase=*/false);
   ```

   click函数的点击回调并不需要someReallyBigData变量。理论上这意味着当process( .. )执行后，在内存中占用大量空间的数据结构就可以被垃圾回收了。但是，由于click函数形成了一个覆盖整个作用域的闭包，JavaScript引擎极有可能依然保存着这个结构（取决于具体实现）。

   块作用域可以打消这种顾虑，可以让引擎清除地知道没必要继续保存someReallyBigData了：

   ```js
   function process(data){
       // 在这里做点有趣的事情
   }
   // 在这个块定义的内容可以销毁了！
   {
       let someReallyBigData = { .. };
       process( someReallyBigData );
   }
   var btn = document.getElementByUd( "my_button" );
   btn.addEventListener("click",function click(evt){
       btn.addEventListener("click",function click(evt)
           console.log("button clicked");
   },/*capturingPhase=*/false);
   ```

   为变量显式声明块作用域，并对变量进行本地绑定是非常有用的工具，可以把它添加到你的代码工具箱中了。

2. let循环

   一个let可以发挥优势的典型例子就是之前讨论的for循环。

   ```js
   for(let i = 0; i < 10; i++){
       console.log( i );
   }
   console.log( i );//ReferenceError
   ```

   for循环头部的let不仅将i绑定到了for循环的块中，事实上它将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。

   下面通过另一种方式来说明每次迭代时进行重新绑定的行为：

   ```js
   {
       let j;
       for(j = 0;j < o; j++){
           let i = j;//每次迭代重新绑定！
           console.log( i );
       }
   }
   ```

   每个迭代进行重新绑定的原因非常有趣，我们会在第5章讨论闭包时进行说明。

   由于let声明附属于一个新的作用域而不是当前的函数作用域（也不属于全局作用域），当代码中存在对于函数作用域中var声明的隐式依赖时，就会有很多隐藏的陷阱，如果用let来替代var则需要在代码重构的过程中付出额外的精力。

   考虑以下代码：

   ```js
   var foo = true,baz = 10;
   if(foo){
       var bar = 3;
       if(baz > bar){
           console.log(baz);
       }
       //...
   }
   ```

   这段代码可以简单地被重构成下面的同等形式：

   ```js
   var foo = true,baz = 10;
   if(foo){
       var bar = 3;
       // ...
   }
   if(baz > bar){
       console.log(baz);
   }
   ```

   但是在使用块级作用域的变量时需要注意以下变化：

   ```js
   var foo = true,baz = 10;
   if(foo){
       let bar = 3;
       if(baz > bar){// <-- 移动代码时不要忘了bar！
           console.log( baz );
       }
   }
   ```

##### 3.4.4  const

除了let以外，ES6还引入了const,同样可以用来创建块作用域变量，但其值是固定的（常量）。之后任何视图修改至的操作都会引起错误。

### 第4章  提升

任何声明在某个作用域内的变量，都将附属于这个作用域。

#### 4.1  先有鸡还是先有蛋

#### 4.2  编译器再度来袭

为了搞明白这个问题，我们需要回复一下第1章中关于编译器的内容。回忆一下，引擎会在解释JavaScript代码之前首先对其进行编译。编译阶段中的一部分工作就是找到所有的声明，并用合适的作用域将它们关联起来。第2章中展示了这个机制，也正是词法作用域的核心内容。

因此，正确的思考思路是，包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。

#### 4.3  函数优先

函数声明和变量声明都会被提升。但是一个值得注意的细节（这个细节可以出现在多个“重复“声明的代码中）是函数会首先被提升，然后才是变量。

一个普通块内部的函数声明通常会被提升到所在作用域的顶部，这个过程不会像下面的代码暗示的那样可以被条件判断所控制：

```js
foo();//"b"
var a = true;
if(a){
    function foo(){console.log("a");}
}else{
    function foo(){console.log("b");}
}
```

### 第5章  作用域闭包

#### 5.1  启示

#### 5.2  实质问题

当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行的。

```js
function foo(){
    var a = 2;
    function bar(){
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz();//2---朋友，这就是闭包的效果。
```

函数bar()的词法作用域能够访问foo()的内部作用域。然后我们将bar()函数本身当做一个值类型进行传递。在这个例子中，我们将bar所引用的函数对象本身当做返回值。

在foo()执行后，通常会期待foo()的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不再使用的内存空间。由于看上去foo()的内容不会再被使用，所以很自然地会考虑对其进行回收。

而闭包的“神奇”之处正式可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回收。谁在使用这个内部作用域？原来是bar()本身在使用。

bar()依然持有对该作用域的引用，而这个引用就叫做闭包。

当然，无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。

```js
function foo(){
    var a = 2;
    function baz(){
        console.log(a);//2
    }
    bar(baz);
}
function bar(fn){
    fn();
}
```

把内部函数baz传递给bar，当调用这个函数时（现在叫做fn），它涵盖的foo()内部作用域的闭包就可以观察到了，因为它能够访问a。

传递函数当然也可以是间接的。

```js
var fn;
function foo(){
    var a = 2;
    function baz(){
        console.log(a);
    }
    fn = baz;
}
functin bar(){
    fn();
}
foo();
bar();//2
```

无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。

#### 5.4  循环和闭包

要说明闭包，for循环是最常见的例子。

```js
for(var i = 1; i <= 5; i++){
    setTimeout(function timer(){
        console.log( i );
    }, i*1000);
}
```

正常情况下，我们对这段代码行为的预期是分别输出数字1-5，每秒一次，每次一个。但实际上，这段代码在运行时会以每秒一次的频率输出五次6。

延迟函数的回调会在循环结束时才执行。事实上，当定时器运行时即使每个迭代中执行的setTimeout(.., 0)，所有的回调函数依然是循环结束后才会被执行，因此会每次输出一个6出来。

这里引申出一个更深入的问题，代码中到底有什么缺陷导致它的行为同语义所暗示的不一致呢？

缺陷是我们试图假设循环中的每个迭代在运行时都会给自己“捕获”一个i的副本。但是根据作用域的工作原理，实际情况是尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个i。

我们需要更多的闭包作用域，特别是在循环的过程中每个迭代都需要一个闭包作用域。

第3章介绍过，IIFE会通过声明并立即执行一个函数来创建作用域。

```js
for(var i = 0; i <= 5; i++){
    (function(){
        var j = i;
        setTimeout(function timer(){
            console.log(j);
        },j*1000) 
    })()
}
```

可以对这段代码进行一些改进：

```js
for(var i = 1; i <= 5; i++){
    (function(j){
        setTimeout(function timer(){
            console.log(j);
        })
    })(i);
}
```

- 重返块作用域

  仔细思考我们对前面的解决方案的分析。我们使用IIFE在每次迭代时都创建一个新的作用域。换句话说，每次迭代我们都需要一个块作用域。第3章介绍了let声明，可以用来劫持块作用域，并且在这个块作用域中声明一个变量。

  本质上这是将一个块转换成一个可以被关闭的作用域。

  for循环头部的let声明还会有一个特殊的行为。这个行为之处变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

  ```js
  for(let i = 1; i <= 5; i++){
      setTimeout(function timer(){
          console.log(i);
      },i*1000);
  }
  ```

#### 5.5  模块

还有其他的代码模式利用闭包的强大威力，但从表面上看，它们似乎与回调无关。下面一起来研究其中最强大的一个：模块。

```js
fnction foo(){
    var something = "cool";
    var another = [1,2,3];
    function doSomething(){
        console.log(something);
    }
    function doAnother(){
        console.log(another.join( " ! " ));
    }
}
```

正如在这段代码中所看到的，这里并没有明显的闭包，只有两个私有数据变量something和another，以及doSomething()和doAnother()两个内部函数，它们的词法作用域（而这就是闭包）也就是foo()的内部作用域。

接下来考虑以下代码：

```js
function CoolModule(){
    var something = "cool";
    var another = [1,2,3];
    function doSomething(){
        console.log( something );
    }
    function doAnother(){
        console.log(another.join("!"));
    }
    return{
        doSomething:doSomething,
        doAnother:doAnother
    };
}
var foo = CoolModule();
foo.doSomething(); // cool
foo.aoAnother(); // 1!2!3
```

这个模式在JavaScript中被称为模块。最常见的实现模块模式的方法通常被称为模块暴露，这里展示的是其变体。

首先，CoolModule()只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。

其次，CoolModule()返回一个用对象字面量语法{key : value, ...}来表示的对象。这个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量时隐藏且私有的状态。可以将这个对象类型的返回值看做本质上是模块的公共API。

模块模式需要具备两个必要条件。

1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

一个具有函数属性的对象本身并不是真正的模块。从方便观察的角度看，一个从函数调用所返回的，只有数据属性而没有闭包函数的对象并不是真正的模块。

上一个示例代码中有一个叫做CoolModule()的独立的模块创建器，可以被调用任意多次，每次调用都会创建一个新的模块实例。当只需要一个实例时，可以对这个模式进行简单的改进来实现单例模式：

```js
var foo = (function CoolModule(){
    var something = "cool";
    var another = [1,2,3];
    function doSomething(){
        console.log( something );
    }
    function doAnother(){
        console.log( another.join("!") );
    }
    return{
        doSomething: doSomething,
        doAnother: doAnother
    }
})();
foo.doSomething();//cool
foo.doAnother();//1!2!3
```

我们将模块函数转换成了IIFE，立即调用这个函数并将返回值直接赋值给单例的模块实例标识符foo。

模块也是普通的函数，因此可以接受参数：

```js
function CoolModule(id){
    function identify(){
        console.log(id);
    }
    return {
        identify: identify
    };
}
var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );
foo1.identify();//"foo 1"
foo2.identify();//"foo 2"
```

模块模式另一个简单但强大的变化用法是，命名将要作为公共API返回的对象：

```js
var foo = (function CoolModule(id){
    function change(){
        // 修改公共API
        publicAPI.identify = identify2;
    }
    function identify1(){
        console.log(id);
    }
    function identify2(){
        console.log(id.toUpperCase());
    }
    var publicAPI = {
        change: change,
        identify: identify1
    };
    return publicAPI;
})("foo module");
foo.identify();//foo module
foo.change();
foo.identify();//FOO MODULE
```

通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对弄快实例进行修改，包括添加或删除方法和属性，以及修改他们的值。

##### 5.5.1  现代的模块机制

大多数模块依赖加载器/管理器本质上都是将这种模块定义封装进一个友好的API。这里并不会研究某个具体的库，为了宏观了解我会简单地介绍一些核心概念：

```js
var MyModules = (function Manager(){
    var modules = {};
    function define(name,deps,impl){
        for(var i = 0; i < deps; i++){
            deps[i] = module[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }
    function get(name){
        return modules[name];
    }
    return {
        define: define,
        get: get
    };
})();
```

这段代码的核心是module[name] = impl.apply(impl, deps)。为了模块的定义引入了包装函数（可以传入任何依赖），并且将返回值，也就是模块的API，储存在一个根据名字来管理的模块列表中。

下面展示了如何使用它来定义模块：

```js
MyModules.define("bar",[],function(){
    function hello(who){
        return "let me introduce:" + who;
    }
    return{
        hello:hello
    };
});
MyModules.define("foo",["bar"],function(bar){
    var hungry = "hippo";
    function awesome(){
        console.log( bar.hello(hungry).toUpperCase());
    }
    return {
        awesome: awesome
    };
});
var bar = MyModules.get("bar");
var foo = MyModules.get("foo");
console.log(
	bar.hello("hippo")
);// Let me introduce:hippo
foo.awesome();// LET ME INTRODUCE: HIPPO
```

“foo”和“bar”模块都是通过一个返回公共API的函数来定义的。“foo”甚至接受“bar”的示例作为依赖参数，并能相应地使用它。

##### 5.5.2  未来的模块机制

ES6中为模块增加了一级语法支持。但通过模块系统进行加载时，ES6会将文件当做独立的模块来处理。每个模块都可以导入其他模块或特定的API成员，同样也可以导出自己的API成员。

基于函数的模块并不是一个能被稳定识别的模式（编译器无法识别），它们的API语义只有在运行时才会被考虑进来。因此可以在运行时修改一个模块的API。

相比之下，ES6模块API更加稳定（API不会在运行时改变）。由于编辑器知道这一点，因此可以在编译期对导入模块的API成员的引用是否真实存在。如果API引用不存在，编译器会在运行时抛出一个或多个“早起”错误，而不会像往常一样在运行期采用动态的解决方案。

ES6的模块没有“行内”格式，必须被定义在独立的文件中（一个文件一个模块）。浏览器或引擎有一个默认的“模块加载器”可以在导入模块时异步地加载块文件。

考虑以下代码：

bar.js

```js
function hello(who){
    return "Let me introduce:" + who;
}
export hello;
```

foo.js

```js
// 仅从“bar”模块导入hello()
import hello frol "bar";
var hungry = "hippo";
function awesome(){
    console.log(
    	hello( hungry ).toUpperCase()
    );
}
export awesome;
```

baz.js

```js
// 导入完成的“foo”和“bar”模块
module foo from "foo";
module bar from "bar";
console.log(
	bar.hello("rhino")
);// Let me introduce: rhino
foo.awesome();//LET ME INTRODUCE: HIPPO
```

import可以将一个模块中的一个或多个API导入到当前作用域中，并分别绑定在一个变量上（在我们的例子里是hello）。module会将整个模块的API导入并绑定到一个变量上（在我们的例子里是foo和bar）。export会将当前模块的一个标识符（变量、函数）导出为公共API。这些操作可以在模块定义中根据需要使用任意多次。

## 第二部分  this和原型

### 第1章  关于this

#### 1.1  为什么要用this

```js
function identify(){
    return this.name.toUpperCase();
}
function speak(){
    var greeting = "Hello, I'm " + identify.call(this);
    console.log(greeting);
}
var me = {
    name : "Kyle"
};
var you = {
    name: "Reader"
}
identify.call(me);//KYLE
identify.call(you);//READER

speak.call(me);//Hello,我是KYLE
speak.call(you);//Hello,我是READER
```

这段代码可以在不同的上下文对象（me和you）中重复使用函数identify()和speak()，不用针对每个对象编写不同版本的函数。

如果不使用this，那就需要给identify()和speak()显式传入一个上下文对象。

```js
function identify(context){
    return context.name.toUpperCase();
}
function speak(context){
    var greeting = "Hello, I'm" + identify(context);
    console.log(greeting);
}
identify( you );//READER
speak( me );//hello,我是KYLE
```

然而，this提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将API设计得更加简洁并且易于复用。

随着你的使用模式越来越复杂，显式传递上下文对象会让代码变得越来越混乱，使用this则不会这样。当我们介绍对象和原型时，你就会明白函数可以自动引用合适的上下文对象有多重要。

#### 1.2  误解

##### 1.2.1  指向自身

人们很容易把this理解成指向函数自身，这个推断从英语的语法角度来说是说得通的。

那么为什么需要从函数内部引用函数自身呢？常见的原因是递归（从函数内部调用这个函数）或者写一个在第一次被调用后自己解除绑定的事件处理器。

我们想要记录一下函数foo被调用的次数，思考一下下面的代码：

```js
function foo(num){
    console.log("foo" + num);
    // 记录foo被调用的次数
    this.count++;
}
foo.count = 0;
var i;
for(i = 0; i < 10; i++){
    if(i > 5){
        foo(i);
    }
}
// foo : 6
// foo : 7
// foo : 8
// foo : 9

//foo被调用了多少次？
console.log(foo.count);// 0 --- WTF?
```

console.log语句产生了4条输出，证明foo(..)确实被调用了4次，但是foo.count仍然是0。显然从字面意思来理解this是错误的。

执行foo.count = 0时，的确向函数对象foo添加了一个属性count。但是函数内部代码this.count中的this并不是指向那个函数对象，所以虽然属性名相同，根对象却并不相同，困惑随之产生。

如果要从函数对象内部引用它自身，那只是用this是不够的。一般来说你需要通过一个指向函数对象的词法标识符（变量）来引用它。

思考下面这两个函数：

```js
function foo(){
    foo.count = 4;//foo指向它自身
}
setTimeount(function(){
    // 匿名函数无法指向自身
},10)
```

第一个函数被称为具名函数，在它内部可以使用foo来引用自身。

但是在第二个例子中，传入setTimeout(..)的回调函数没有名称标识符，因此无法从函数内部引用自身。

强制this指向foo函数对象：

```js
function foo(num){
    console.log("foo" + num);
    
    //记录foo被调用的次数
    //注意，在当前的调用方式下，this确实指向foo
    this.count++;
}
foo.count = 0;
var i;
for(i = 0; i < 10; i++){
    if(i > 5){
        // 使用call(..)可以确保this指向函数对象foo本身
        foo.call(foo,i);
    }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// foo被调用了多少次？
console.log( foo.count ); // 4
```

##### 1.2.2  它的作用域

需要明确的是，this在任何情况下都不指向函数的词法作用域。在JavaScript内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过JavaScript代码访问，它存在于JavaScript引擎内部。

思考一下下面的代码，它试图跨越边界，使用this来隐式引用函数的词法作用域：

```js
function foo(){
    var a = 2;
    this.bar();
}
function bar(){
    console.log(this.a);
}
foo();//ReferenceError: a is not defined
```

首先，这段代码试图通过this.bar()来引用bar()函数。这是绝对不可能成功的，我们之后会解释原因。调用bar()最自然得方法是省略前面的this，直接使用词法引用表示符。

此外，编写这段代码的开发者还试图使用this联通foo()和bar()的词法作用域，从而让bar()可以访问foo()作用域里的变量a。这是不可能实现的，你不能使用this来引用一个词法作用域内部的东西。

#### 1.3  this到底是什么

之前我们说过this是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this和绑定的函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this就是记录的其中一个属性，会在函数执行的过程中用到。

### 第2章  this全面解析

#### 2.1  调用位置

在理解this的绑定过程之前，首先要理解调用位置：调用位置就是函数在代码中被调用的位置（而不是声明的位置）。只有仔细分析调用位置才能回答这个问题：这个this到底引用的是什么？

通常来说，寻找调用位置就是寻找“函数被调用的位置”，但是做起来并没有这么简单，因为某些变成模式可能会隐藏真正的调用位置。

最重要的是要分析调用栈（就是为了到达当前执行位置所调用的所有函数）。我们关心的调用位置就在当前正在执行的函数中的前一个调用中。

下面我们来看看到底什么是调用栈和调用位置：

```js
function baz(){
    // 当前调用栈是：baz
    // 因此，当前调用位置是全局作用域
    console.log("baz");
    bar();// <-- bar的调用位置
}
function bar(){
    // 当前调用栈是 baz -> bar
    // 因此，当前调用位置在baz中
    console.log("bar");
    foo(); // <-- foo的调用位置
}
function foo(){
    // 当前调用栈是baz -> bar -> foo
    // 因此，当前调用位置在bar中
    console.log("foo");
}
baz(); // <-- baz的调用位置
```

注意我们是如何（从调用栈中）分析出真正的调用位置的，因为它决定了this的绑定。

#### 2.2  绑定规则

你必须找到调用位置，然后判断需要应用下面四条规则中的哪一条。我们首先会分别解释这四条规则，然后解释多条规则都可用时它们的优先级如何排列。

##### 2.2.1  默认绑定

首先要介绍的是最常用的函数调用类型：独立函数调用。可以把这条规则看做是无法应用其他规则时的默认规则。

思考一下下面的代码：

```js
function foo(){
    console.log(this.a);
}
var a = 2;
foo(); // 2
```

在本例中，函数调用时应用了this的默认绑定，因此this指向全局对象，当调用foo()时，this.a被解析成了全局变量a。

那么我们怎么知道这里应用了默认绑定呢？可以通过分析调用位置来看看foo()是如何调用的。在代码中，foo()是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则。

如果使用严格模式，那么全局对象将无法使用默认绑定，因此this会绑定到undefined：

```js
function foo(){
    "use strict";
    console.log( this.a );
}
var a = 2;
foo(); // TypeEroor: this is undefined
```

这里有一个微妙但非常重要的细节，虽然this的绑定规则完全取决于调用位置，但是只有foo()运行在非严格模式下，默认绑定才能绑定到全局对象；严格模式下与foo()的调用位置无关：

```js
function foo(){
    console.log(this.a);
}
var a = 2;
(function(){
    "use strict";
    foo();// 2
})();
```

##### 2.2.2  隐式绑定

另一条需要考虑的规则时调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，不过这种说法可能会造成一些误导。

思考下面的代码：

```js
function foo(){
    console.log(this.a);
}
var obj = {
    a:2,
    foo:foo
};
obj.foo(); // 2
```

首先需要注意的是foo()的声明方式，及其之后是如何被当做引用属性添加到obj中的。但是无论是直接在obj中定义还是先定义再添加为引用属性，这个函数严格来说都不属于obj对象。

然而，调用位置会使用obj上下文来引用函数，因此你可以说函数被调用时obj对象“拥有”或者“包含”它。

无论你如何称呼这个模式，当foo()被调用时，它的落脚点确实指向obj对象。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。因为调用foo()是this被绑定到obj，因此this.a和obj.a是一样的。

对象属性引用链中只有最顶层或最后一层会影响调用位置。举例来说：

```js
function foo(){
    console.log(this.a);
}
var obj2 = {
    a:42,
    foo:foo
};
var obj1 = {
    a:2,
    obj2:obj2
};
obj1.obj2.foo();//42
```

- 隐式丢失

  一个最常见的this绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把this绑定到全局对象或者undefined上，取决于是否是严格模式。

  思考下面的代码：

  ```js
  function foo(){
      console.log(this.a);
  }
  var obj = {
      a:2,
      foo:foo
  }
  var bar = obj.foo; // 函数别名
  var a = "oops, global"; // a是全局对象的属性
  bar();// "oops, global"
  ```

  虽然bar是obj.foo的一个引用，但是实际上，它引用的是foo函数本身，因此此时的bar()其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

  一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时：

  ```js
  function foo(){
      console.log( this.a );
  }
  function doFoo(fn){
      // fn其实引用的是foo
      fn(); // <-- 调用位置！
  }
  var obj = {
      a:2,
      foo:foo
  };
  var a = "oops,global"; // a是全局对象的属性
  doFoo(obj.foo);// "oops,global"
  ```

  参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一个例子一样。

  如果把函数传入语言内置的函数而不是传入你自己声明的函数，会发生什么呢？结果是一样的，没有区别：

  ```js
  function foo(){
      console.log(this.a);
  }
  var obj = {
      a:2,
      foo:foo
  };
  var a = "oops,global"; // a是全局对象的属性
  setTimeout( obj.foo, 100);// "oops,global"
  ```

  JavaScript环境中的内置setTimeout()函数实现和下面的伪代码类似：

  ```js
  function setTimeount(fn，delay){
      // 等待delay毫秒
      fn(); // <-- 调用位置！
  }
  ```

##### 2.2.3  显式绑定

如果我们不想在对象内部包含函数引用，而想在某个对象上强制调用函数，该怎么做呢？

可以用函数的call( .. )和apply( .. )方法。严格来说，JavaScript的宿主环境有时会提供一些非常特殊的函数，它们并没有这两个方法。但是这样的函数非常罕见，JavaScript提供的绝大多数函数以及你自己常见的所有函数都可以使用call( .. )和apply( .. )方法。

它们的第一个参数是一个对象，它们会把这个对象绑定到this，接着在调用函数时指定这个this。因此你可以直接指定this的绑定对象，因此我们称之为显式绑定。

思考下面的代码：

```js
function foo(){
    console.log(this.a);
}
var obj = {
    a:2
};
foo.call(obj);//2
```

通过foo.call(..)，我们可以在调用foo时强制把它的this绑定到obj上。

如果你传入一个原始值（字符串类型、布尔类型或者数字类型）来当做this的绑定对象，这个原始值会被转换成它的对象形式（也就是 new String(..)、new Boolean(..)或者new Number(..)）。这通常被称为“装箱”。

可惜，显示绑定仍然无法解决我们之前提出的丢失绑定问题。

1. 硬绑定

   但是显示绑定的一个变种可以解决这个问题。

   思考下面的代码：

   ```js
   function foo(){
       console.log(this.a);
   }
   var obj = {
       a:2
   };
   var bar = function(){
       foo.call(obj);
   };
   bar(); // 2
   setTimeout(bar, 100); // 2
   // 硬绑定的bar不可能再修改它的this
   bar.call(window); // 2
   ```

   我们创建了函数bar()，并在它的内部手动调用了foo.call(obj)，因此强制把foo的this绑定到了obj。无论之后如何调用函数bar，它总会手动在obj上调用foo。这种绑定是一种显示的强制绑定，因此我们称之为硬绑定。

   硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接受到的所有值：

   ```js
   function foo(something){
       console.log(this.a, something);
       return this.a + something;
   }
   var obj = {
       a:2
   };
   var bar = function(){
       return foo.apply( obj.arguments );
   }
   var b = bar( 3 ); // 2 3
   console.log( b ); // 5
   ```

   另一种使用方法是创建一个i可以重复使用的辅助函数：

   ```js
   function foo(something){
       console.log(this.a,something);
       return this.a + something;
   }
   // 简单的辅助绑定函数
   function bind(fn,obj){
       return function(){
           return fn.apply(obj,arguments);
       }
   }
   var obj = {
       a:2
   };
   var bar = bind(foo,obj);
   var b =bar(3);// 2 3
   console.log(b);// 5
   ```

   由于硬绑定是一种非常常用的模式，所以在ES5中提供了内置的方法Function.prototyps.bind，它的用法如下：

   ```js
   function foo(something){
       console.log(this.a,something);
       return this.a + something;
   }
   var obj = {
       a:2
   };
   var bar = foo.bind(obj);
   var b = bar(3);// 2 3
   console.log(b);// 5
   ```

   bind( .. )会返回一个硬编码的新函数，它会把参数设置为this的上下文并调用原始函数。

2. API调用的“上下文”

   第三方库的许多函数，以及JavaScript语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为“上下文”（context），其作用和bind( .. )一样，确保你的回调函数使用指定的this。

   举例来说：

   ```js
   function foo(el){
       console.log(el, this.id);
   }
   var obj = {
       id: "awesome"
   };
   
   // 调用foo( .. )时把this绑定到obj
   [1,2,3].forEach( foo, obj );
   // 1 awesome 2awesome  3awesome
   ```

   这些函数实际上就是通过call( .. )或者apply( .. )实现了显式绑定，这样你可以少写一些代码。

##### 2.2.4  new绑定

这是第四条也是最后一条this的绑定规则，在讲解它之前我们首先需要澄清一个非常常见的关于JavaScript中函数和对象的误解。

在传统的面向类的语言中，“构造函数”是类中的一些特殊方法，使用new初始化类时会调用类中的构造函数。通常的形式是这样的：

```js
something = new MyClass( .. );
```

JavaScript也有一个new操作符，使用方法看起来也和那些面向类的语言一样，绝大多数开发者都认为JavaScript中new的机制也和那些语言一样。然而，JavaScript中new的机制实际上和面向类的语言完全不同。

首先我们重新定义一下JavaScript中的“构造函数”。在JavaScript中，构造函数只是一些使用new操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。

举例来说，思考一下Number( .. )作为构造函数时的行为，ES5.1中这样描述它：

- 当Number在new表达式中被调用时，它是一个构造函数：它会初始化新创建的对象。

所以，包括内置对象函数在内的所有函数都可以用new来调用，这种函数调用被称为构造函数调用。这里有一个重要但是非常细微的区别：实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

我们现在关心的是第1步、第3步、第4部，所以暂时跳过第2步，第5章会详细介绍它。

思考下面的代码：

```js
function foo(a){
    this.a = a;
}
var bar = new foo(2);
console.log(bar.a); // 2
```

使用new来调用foo(..)时，我们会构造一个新对象并把它绑定到foo(..)调用中的this上。new是最后一种可以影响函数调用时this绑定行为的方法，我们称之为new绑定。

#### 2.3  优先级

现在我们已经了解了函数调用中this绑定的四条规则，你需要做的就是找到函数的调用位置并判断应当应用哪条规则。但是，如果某个调用位置可以应用多条规则怎么办？为了解决这个问题就必须给这些规则设定优先级，这就是我们接下来要介绍的内容。

毫无疑问，默认绑定的优先级是四条负责中最低的，我们可以先不考虑它。

隐式绑定和显式绑定哪个优先级更高？我们来测试一下：

```js
function foo(){
    console.log(this.a);
}
var obj1 = {
    a:2,
    foo:foo
};
var obj2 = {
    a:3,
    foo:foo
}
obj1.foo();// 2
obj2.foo();// 3

obj1.foo.call( obj2 );// 3
obj2.foo.call( obj1 );// 2
```

可以看到，显式绑定优先级更高，也就是说在判断是应当先考虑是否可以应用显式绑定。

现在我们需要搞清楚new绑定和隐式绑定的优先级谁高谁低：

```js
function foo(something){
    this.a = something;
}
var obj1 = {
    foo:foo
};
var obj2 = {};

obj1.foo(2);
console.log(obj1.a);// 2

obj1.foo.call(obj2,3);
console.log(obj2.a);// 3

var bar = new obj1.foo(4);
console.log(obj1.a);// 2
console.log(bar.a);// 4
```

可以看到new绑定比隐式绑定优先级高。但是new绑定和显式绑定谁的优先级更高呢？

```js
function foo(something){
    this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a);// 2

var baz = new bar(3);
console.log(obj1.a);// 2
console.log(baz.a);// 3
```

bar被硬绑定到obj1上，但是new bar(3)并没有像我们预计的那样把obj1.a修改为3相反，new修改了硬绑定（到obj1的）调用bar( .. )中的this。因为使用了new绑定，我们得到了一个名字为baz的新对象，并且baz.a的值是3。

再来看看我们之前介绍的“裸”辅助函数bind：

```js
function bind(fn,obj){
    return function(){
        fn.apply(obj,arguments);
    };
}
```

非常令人惊讶，因为看起来在辅助函数中new操作符的调用无法修改this绑定，但是在刚才的代码中new确实修改了this绑定。

实际上，ES5中内置的Function.prototype.bind(..)更加复杂。下面是MDN提供的一种bind(..)实现。

```js
if(!Function.prototype.bind){
    Function.prototype.bind = function(oThis){
        if(typeof this !== "function"){
            // 与ECMAScript5最接近的
            // 内部IsCallable函数
            throw new TypeError(
            	"Function.protype.bind - what id trying"+
                "to be bound is not callable"
            );
        }
        
        var aArgs = Array.prototype.slice.call(arguments, 1),
            fToBind = this,
            fNOP = function(){},
            fBound = function(){
                return fToBind.apply(
                	(
                    	this instanceof FNOP &&
                        oThis ? this : oThis
                    ),
                    aArgs.concat(
                    	Array.prototype.slice.call(arguments)
                    );
                )
            }
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
        
        return fBound;
    }
}
```

这种bind(..)是一种ployfill代码（ployfill就是我们常说的刮墙用的腻子，ployfill代码主要用于就浏览器的兼容，比如说在旧的浏览器中并没有内置bind函数，因此可以使用playfill代码在旧浏览器中实现新的功能），对于new使用的硬绑定函数来说，这段ployfill代码和ES5内置的bind(..)函数并不完全相同（后面会介绍为什么要在new中使用硬绑定函数）。由于ployfill并不是内置函数，所以无法创建一个不包含.prototype的函数，因此会具有一些副作用。如果你要在new中使用硬绑定函数并且依赖ployfill代码的话，一定要非常小心。

下面是new修改this的相关代码：

```js
this instanceof FNOP && oThis ? this : oThis
// ... 以及
fNOP.prototype = this.prototype
fBound.prototype = new fNOP();
```

我们并不会详细解释这段代码做了什么（这非常复杂并且不再我们的讨论范围之内），不过简单来说，这段代码会判断硬绑定函数是否是被new调用，如果是的话就会使用新创建的this替换硬绑定的this。

那么，为什么要在new中使用硬绑定函数呢？直接使用普通函数不是更简单吗？

之所以要在new中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用new进行初始化时就可以只传入其余的参数。bind(..)的功能之一就是可以把除了第一个参数（第一个参数用于绑定this）之外的其他参数都传给下层的函数（这种技术称为“部分应用”，是“柯里化”的一种）。举例来说：

```js
function foo(p1,p2){
    this.val = p1 + p2;
}
// 之所以使用null是因为在本例中我们并不关心硬绑定的this是什么
// 反正使用new时this会被修改
var bar = foo.bind( null, "p1");

var baz = new bar("p2");

baz.val; // p1p2
```

- 判断this

  现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照下面的顺序来进行判断：、

  1. 函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。

     ```js
     var bar = new foo();
     ```

  2. 函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this绑定的是指定的对象。

     ```js
     var bar = foo.call(obj2);
     ```

  3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。

     ```js
     var bar = obj1.foo();
     ```

  4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

     ```js
     var bar = foo();
     ```

就是这样。对于正常的函数调用来说，理解了这些知识你就可以明白this的绑定原理了。不过。。。凡事总有例外。

#### 2.4  绑定例外

在某些场景下this的绑定行为会出乎意料，你认为应当应用其他绑定规则时，实际上应用的可能是默认绑定规则。

##### 2.4.1  被忽略的this

如果你把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则：

```js
function foo(){
    console.log(this.a);
}
var a = 2;
foo.call(null); // 2
```

那么什么情况下你会传入null呢？

一种非常常见的做法是使用apply(..)来“展开”一个数组，并当做参数传入一个函数。类似的，bind(..)可以对参数进行柯里化（预先设置一些参数），这种方法有时非常有用：

```js
function foo(a,b){
    console.log("a:"+a+",b:"+b);
}
// 把数组“展开”成参数
foo.apply(null, [2,3]);//a:2,b:3
// 使用bind(..)进行柯里化
var bar= foo.bind(null，2）;
bar(3);//a:2,b:3                 
```

这两种方法都需要传入一个参数当做this的绑定对象。如果函数并不关心this的话，你仍然需要传入一个占位置，这时null可能是一个不错的选择，就像代码所示的那样。

然而，总是使用null来忽略this绑定可能产生一些副作用。如果某个函数确实使用了this（比如第三方库中的一个函数），那默认绑定规则会把this绑定到全局对象（在浏览器中这个对象是window），这将导致不可预计的后果（比如修改全局对象）。

显而易见，这种方式可能会导致许多难以分析和追踪的bug。

- 更安全的this

  一种“更安全”的做法是传入一个特殊的对象，把this绑定到这个对象不会对你的程序产生任何副作用。就像网络（以及军队）一样，我们可以创建一个“DMZ”对象，它就是一个空的非委托的对象。

  如果我们在忽略this绑定时总是传入一个DMZ对象，那就什么都不用担心了，因为任何对于this的使用都会限制在这个空对象中，不会对全局对象产生任何影响。

  无论你叫它什么，在JavaScript中创建一个空对象最简单的方法就是Object.create(null)。Object.create(null)和{}很像，但是并不会创建object.prototype这个委托，所以它比{}“更空”：

  ```js
  function foo(a,b){
      console.log("a:"+a+",b:"+b);
  }
  // 我们的DMZ空对象
  var N = object.create(null);
  
  // 把数组展开成参数
  foo.apply(N,[2,3]);// a:2,b:3
  
  // 使用bind(..)进行柯里化
  var bar = foo.bind(N,2);
  bar(3);// a:2,b:3
  ```

##### 2.4.2  间接引用

另一个需要注意的是，你有可能创建一个函数的“间接引用”，在这种情况下，调用这个函数会应用默认绑定规则。

间接引用最容易在赋值是发生：

```js
function foo(){
    console.log(this.a);
}
var a = 2;
var o = {a:3,foo:foo};
var p = {a:4};

o.foo(); //3
(p.foo = o.foo)(); //2
```

赋值表达式p.foo = o.foo的返回值是目标函数的引用，因此调用位置是foo()而不是p.foo()或o.foo()。根据我们之前说过的，这里会应用默认绑定。

注意：对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this会被绑定到undefined，否则this会被绑定到全局对象。

##### 2.4.3  软绑定

之前我们已经看到过，硬绑定这种方式可以把this强制绑定到指定的对象（除了使用new时），防止函数调用应用默认绑定规则。问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法只用隐式绑定或者显式绑定来修改this。

如果可以给默认绑定指定一个全局对象或者undefined以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显式绑定修改this的能力。

可以通过一种被称为软绑定的方法来实现我们想要的效果：

```js
if(!Function.prototype.softBind){
    Function.prototype.sofeBind = function(obj){
        var fn = this;
        // 捕获所有curried参数
        var curried = [].slice.call(arguments,1);
        var bound = function(){
            return fn.apply(
            	(!this || this === (window || global))?
                   obj : this
                curried.concat.apply(curried,arguments)
            );
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    }
}
```

除了软绑定之外，softBind(..)的其他原理和ES5内置的bind(..)类似。它会对指定的函数进行封装，首先检查调用时的this，如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this，否则不会修改this。此外，这段代码还支持可选的柯里化。

下面我们看看softBind是否实现了软绑定功能：

```js
function foo(){
    console.log("name: " + this.name);
}
var obj = {name:"obj"},
    obj2 = {name:"obj2"},
    obj3 = {name:"obj3"};
var fooOBJ = foo.softBind(obj);

fooOBJ(); // name: obj
obj2.foo = foo.softBind(obj);
obj2.foo();// name: obj2 <-------看！！

fooOBJ.call(obj3);// name: obj3 <--------看！！

setTimeout(obj2.foo,10);
// name: obj <--------应用了软绑定

```

可以看到，软绑定版本foo()可以手动将this绑定到obj2或者obj3上，但如果应用默认绑定，则会将this绑定到obj。

#### 2.5  this词法

我们之前介绍的四条规则已经可以包含所有正常的函数。但是ES6中介绍了一种无法使用这些规则的特殊函数类型：箭头函数。

箭头函数不使用this的四种标准规则，而是根据外层（函数或者全局）作用域来决定this。

我们来看看箭头函数的词法作用域：

```js
function foo(){
    // 返回一个箭头函数
    return (a) => {
        // this继承自foo()
        console.log(this.a);
    };
}
var obj1 = {
    a:2
};
var obj2 = {
    a:3
};
var bar = foo.call(obj1);
bar.call(obj2);// 2,不是3！
```

foo()内部创建的箭头函数会捕获调用时foo()的this。由于foo()的this绑定到obj，bar（引用箭头函数）的this也会绑定到obj1，箭头函数的绑定无法被修改（new也不行！）

箭头函数最常用于回调函数中，例如事件处理器或者定时器：

```js
function foo(){
    setTimeout(() => {
        //这里的this在词法上继承自foo()
        console.log(this.a);
    },100)
}
var obj = {
    a:2
};
foo.call(obj);// 2
```

箭头函数可以像bind(..)一样确保函数的this被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的this机制。实际上，在ES6之前我们就已经在使用一种几乎和箭头函数完全一样的模式。

```js
function foo(){
    var self = this;// lexical capture of this
    setTimeout(function(){
        console.log(self.a);
    },100);
}
var obj = {
    a:2
};
foo.call(obj); // 2
```

虽然self = this和箭头函数看起来都可以取代bind(..)，但是本质上来说，它们想替代的是this机制。

如果你经常编写this风格的代码，但是绝大部分时候都会使用self = this或者箭头函数来否定this机制，那你或许应当：

1. 只使用词法作用域并完全抛弃错误this风格的代码；
2. 完全采用this风格，在必要时使用bind(..)，尽量避免使用self = this和箭头函数。

当然，包含这两种代码风格的程序可以正常运行，但是在同一个函数或者同一个程序中混合使用这两种风格通常会使代码更难维护，并且可能也会更难编写。

### 第3章  对象

#### 3.1  语法

对象可以通过两种形式定义：声明（文字）形式和构造形式。

对象的文字语法大概是这样：

```js
var myObj = {
    key:value
    // ...
};
```

构造形式大概是这样：

```js
var myObj = new Object();
myObj.key = value;
```

构造形式和文字形式生成的对象是一样的。唯一的区别是，在文字声明中你可以添加多个键/值对，但是在构造形式中你必须逐个添加属性，

#### 3.2  类型

对象是JavaScript的基础。在JavaScript中一种有六种主要类型（术语是“语言类型”）：

- string
- number
- boolean
- null
- undefined
- object

注意，简单基本类型（string、boolean、number、null和undefined）本身并不是对象。null有时会被当做一种对象类型，但是这其实只是语言本身的一个bug，即对null执行typeof null时会返回字符串“object”。实际上，null本身是基本类型。

- 内置对象

  JavaScript中还有一些对象子类型，通常被称为内置对象。有些内置对象的名字看起来和简单基础类型一样，不过实际上它们的关系更复杂，我们稍后会详细介绍

  - String
  - Number
  - Boolean
  - Object
  - Function
  - Array
  - Date
  - RegExp
  - Error

  这些内置对象从表现形式来说很像其他语言中的类型（type）或者类（class),比如Java的String类。

  但是在JavaScript中，他们实际上只是一些内置函数。这些内置函数可以当做构造函数来使用，从而可以构造一个对应子类型的新对象。举例来说：

  ```js
  var strPrimitive = "I am a string";
  typeof srtPrimitive; // "string"
  strPrimitive instanceof String; // false
  
  var strObject = new String("I am a string");
  typeof strObject; // "object"
  strObject instanceof String; // true
  
  // 检查sub-type对象
  Object.prototype.toString.call(strObject);//[object String]
  ```

#### 3.3  内容

之前我们提到过，对象的内容是由一些存储在特定命名位置的（任意类型的）值组成的，我们称之为属性。

需要强调的一点是，当我们说“内容”时，似乎在暗示这些值实际上被存储在对象内部，但是这只是它的表现形式。在引擎内部，这些值得存储方式是多种多样的，一般并不会存在对象容器内部。存储在对象容器内部的是这些属性的名称，它们就像指针（从技术角度来说就是引用）一样，指向这些值真正的存储位置。

##### 3.3.1  可计算属性名

如果你需要通过表达式来计算属性名，那么我们刚刚讲到的myObject[..]这种属性访问语法就可以派上用场了，如可以使用myObject[prefix + name]。但是使用文字形式来声明对象时这样做是不行的。

##### 3.3.2  属性和方法

如果访问的对象属性是一个函数，有些开发者喜欢使用不一样的叫法以作区分。由于函数很容易被认为是属于某个对象，在其他语言中，属于对象（也被称为“类”）的函数通常被称为“方法”，因此把“属性访问”说成是“方法访问”也就不奇怪了。

有意思的是，JavaScript的语法规范也做出了同样的区分。

从技术角度来说，函数永远不会“属于”一个对象，所以把对象内部引用的函数称为“方法”似乎有点不妥。

确实，有些函数具有this引用，有时候这些this确实会指向调用位置的对象引用。但是这种用法从本质上来说并没有把一个函数变成一个“方法”，因此this是在运行时根据调用位置动态绑定的，所以函数和对象的关系最多也只能说是间接关系。

无论返回值是什么类型，每次访问对象的属性就是属性访问。如果属性访问返回的是一个函数，那它并不是一个“方法”。属性访问返回的函数和其他函数没有任何区别（除了可能发生的隐式绑定this，就像我们刚才提到的）。

举例来说：

```js
function foo(){
    console.log("foo");
}
var someFoo = foo; // 对foo的变量引用
var myObject = {
    someFoo = foo
};
foo; // function foo(){..}
someFoo; // function foo(){..}
myObject.someFoo; // function foo(){..}
```

someFoo和myObject.someFoo只是对于同一个函数的不用引用，并不能说明这个函数是特别的或者“属于”某个对象。如果foo()定义时在内部有一个this引用，那这两个函数引用的唯一区别就是myObject.someFoo中的this会被隐式绑定到一个对象。无论哪种引用形式都不能称之为“方法”。

##### 3.3.3  数组

数组也支持[]访问形式，不过就像我们之前提到过的，数组有一套更加结构化的值存储机制（不过仍然不限制值得类型）。数组期望的是数值下标，也就是说值存储的位置（通常被称为索引）是整数。

##### 3.3.4  复制对象

思考一下这个对象：

```js
function anotherFunction(){ /*..*/ }
var anotherObject = {
    c: true
};
var anotherArray = [];
var myObject = {
    a:2,
    b:anotherObject, // 引用，不是复本！
    c:anotherArray, // 另一个引用！
    d:anotherFunction
};
anotherArray.push(anotherObject,myObject);
```

如何准确地表示myObject的复制呢？

首先，我们应该判断它浅复制还是深复制。对于浅拷贝来说，复制出的新对象a中的值会复制就对象中a的值，也就是2，但是新对象中b、c、d三个属性其实只是三个引用，它们和旧对象中b、c、d引用的对象是一样的。对于深复制来说，除了复制myObject以外还会复制anotherObject和anotherArray。这时问题就来了，anotherArray引用了anotherObject和myObject，所以又需要复制myObject，这样就会由于循环引用导致死循环。

##### 3.3.5  属性描述符

在ES5之前，JavaScript语言本身并没有提供可以直接检测属性特性的方法，比如判断属性是否是只读。

但是从ES5开始，所有的属性都具备了属性描述符。

思考下面的代码：

```js
var myObject = {
    a:2
};
Object.getOwnPropertyDescriptor(myObject,"a");
// {
//    value:2,
//    writable:true,
//    enumerable: true,
//    configurable:true 
// }
```

在创建普通属性时属性描述符会使用默认值，我们也可以使用Object.defineProperty(..)来添加一个新属性或者修改一个已有属性（如果它是configurable）并对特性进行设置。

举例来说：

```js
var myObject = {};
Object.defineProperty(myObject,"a",{
    value:2,
    writable:true,
    configurable:true,
    enumerable:true
});
myObject.a; // 2
```

##### 3.3.6  不变性

有时候你会希望属性或者对象是不可改变的，在ES5中可以通过很多种方法来实现。

很重要的一点是，所有的方法创建的都是浅不变形，也就是说，它们只会影响目标对象和它的直接属性。如果目标对象引用了其他对象（数组、对象、函数等），其他对象的内容不受影响，仍然是可变的：

```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push(4);
myImmutableObject.foo; // [1,2,3,4]
```

假设代码中的myImmutableObject已经被创建而且是不可变的，但是为了保护它的内容myImmutableObject.foo，你还需要使用下面的方法让foo也不可变。

1. 对象常量

   结合writable:false和configurable:false就可以创建一个真正的常量属性（不可修改、重定义或者删除）：

   ```js
   var myObject = {};
   Object.defineProperty(myObject, "FAVORITE_NUMBER",{
       value:42,
       writeable:false,
       configurable:false
   })
   ```

2. 禁止扩展

   如果你想禁止一个对象添加新属性并保留已有属性，可以使用Object.preventExtensions(..);

   ```js
   var myObject = {
       a:2
   };
   Object.preventExtensions(myObject);
   
   myObject.b = 3;
   myObject.b; // undefined
   ```

   在非严格模式下，创建属性b会静默失败。在严格模式下，将会抛出TypeError错误。

3. 密封

   Object.seal(..)会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用Object.preventExtensions(..)并把所有现有属性标记为configurable:false。

   所以，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（虽然可以修改属性的值）。

4. 冻结

   Object.freeze(..)会创建一个冻结对象，这个方法实际上会在一个现有对象上调用Object.seal(..)并把所有“数据访问”属性记为writable:false，这样就无法修改它们的值。

##### 3.3.7  [[Get]]

属性访问在实现时有一个微妙却非常重要的细节，思考下面的代码：

```js
var myObject = {
    a:2
};
myObject.a; // 2
```

myObject.a是一次属性访问，但是这条语句并不仅仅是在myObject中查找名字为a的属性，虽然看起来好像是这样。

在语言规范中，myObject.a在myObject上实际上是实现了[[Get]]操作（有点像函数调用）。对象默认的内置[[Get]]操作首先在对象中查找是否有名称相同的属性，如果找到就返回这个属性的值。

然而，如果没有找到名称相同的属性，按照[[Get]]算法的定义会执行另外一种非常重要的行为。我们会在第5章中介绍这个行为（其实就是遍历可能存在的[[Prototype]]链，也就是原型链）。

如果无论如何都没有找到名称相同的属性，那[[Get]]操作会返回值undefined。

注意，这种方法和访问变量时是不一样的。如果你引用了一个当前词法作用域中不存在的变量，并不会像对象属性一样返回undefined，而是会抛出一个ReferenceError异常：

```js
var myObject = {
    a:undefined
};
myObject.a; // undefined
myObject.b; // undefined
```

从返回值的角度来说，这两个引用没有区别---它们都返回了undefined。然而，尽管乍看着下没什么区别，实际上底层[[Get]]操作对myObject.b进行了更复杂的处理。

由于仅根据返回值无法判断出到底变量的值为undefined还是变量不存在，所以[[Get]]操作返回了undefined。不过稍后我们会介绍如何区分这两种情况。

##### 3.3.8  [[Put]]

[[Put]]被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性（这个是最重要的因素）。

如果已经存在这个属性，[[Put]]算法大致会检查下面这些内容。

1. 属性是否是访问描述符？如果是并且存在setter就调用seeter。
2. 属性的数据描述符中writable是否为false？如果是，在非严格模式下静默失败，在严格模式下抛出TypeError异常。
3. 如果都不是，将该值设置为属性的值。

如果对象中不存在这个属性，[[Put]]操作会更加复杂。

##### 3.3.9  Getter和Setter

对象默认的[[Put]]和[[Get]]操作分别可以控制属性值的设置和获取。

在ES5中可以使用getter和setter部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。getter是一个隐藏函数，会在获取属性值时调用。setter也是一个隐藏函数，会在设置属性值时调用。

当你给一个属性定义getter、setter或者两者都有时，这个属性会被定义为“访问描述符”（和“数据描述符”相对）。对于访问描述符来说，JavaScript会忽略他们的value和writable特性，取而代之的是关心set和get（还有configurable和enumerable）特性。

思考下面的代码：

```js
var myObject = {
    // 给a定义一个getter
    get a(){
        return 2;
    }
}
Object.defineProperty(
	myObject, // 目标对象
    "b",      // 属性名
    {    // 描述符
        // 给b设置一个getter
        get:function(){return this.a * 2},
        
        // 确保b会出现在对象的属性列表中
        enumerable:true
        
    }
);

myObject.a; // 2
myObject.b; // 4
```

不管是对象文字语法中的get a(){..}，还是defineProperty(..)中的显式定义，二者都会在对象中创建一个不包含值得属性，对于这个属性的访问会自动调用一个隐藏函数，它的返回值会被当做属性访问的返回值：

```js
var myObject = {
    // 给a定义了一个getter
    get a(){
        return 2;
    }
}
myObject.a = 3;
myObject.a; // 2
```

由于我们只定义了a的getter，所以对a的值进行设置时set操作会忽略赋值操作，不会抛出错误。而且即便有合法的setter，由于我们自定的getter只会返回2，所以set操作是没有意义的。

为了让属性更合理，还应当定义setter，和你期望的一样，setter会覆盖单个属性默认的[[Put]]