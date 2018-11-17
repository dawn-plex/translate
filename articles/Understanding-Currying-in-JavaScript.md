> * 原文地址：[Understanding Currying in JavaScript](https://blog.bitsrc.io/understanding-currying-in-javascript-ceb2188c339)
> * 原文作者：[Chidume Nnamdi](https://blog.bitsrc.io/@kurtwanger40?source=post_header_lockup)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md](https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md)
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：[也树](https://github.com/xdlrt)、[靖鑫](https://github.com/luckyjing)

---

# 理解JavaScript的柯里化

函数式编程是一种将函数作为参数传递并返回没有副作用函数（修改参数的状态）的编程风格。

许多计算机语言都采用了这种编程风格。在这些语言中，JavaScript、Haskell、Clojure、Erlang 和 Scala 是最流行的几种。

由于这种风格具有传递和返回函数的能力，它带来了许多概念：

* 纯函数
* 柯里化
* 高阶函数

我们接下来要谈到的概念就是这其中的**柯里化**。

在这篇文章📄中，我们会看到柯里化如何工作以及它是如何被软件开发者运用到实践中的。

**提示**：除了复制粘贴，你可以使用 [Bit](https://bitsrc.io/) 把可复用的 JavaScript 功能转换为组件，这样可以快速地和你的团队在项目之间共享。

## 什么是柯里化？

柯里化其实是函数编程的一个过程，在这个过程中我们能把一个带有多个参数的函数转换成一系列的嵌套函数。它返回一个新函数，这个新函数期望传入下一个参数。

它不断地返回新函数（像我们之前讲的，这个新函数期望当前的参数），直到所有的参数都被使用。参数会一直保持 `alive` （通过闭包），当柯里化函数链中最后一个函数被返回和调用的时候，它们会用于执行。

> 柯里化是一个把具有较多 arity 的函数转换成具有较少 arity 函数的过程 -- Kristina Brainwave

**注意**：上面的术语 arity ，指的是函数的参数数量。举个例子，

```
function fn(a, b)
    //...
}
function _fn(a, b, c) {
    //...
}
```

函数`fn`接受两个参数（2-arity函数），`_fn`接受3个参数（3-arity函数）

所以，柯里化把一个多参数函数转换为一系列只带**单个参数**的函数。

让我们来看一个简单的示例：

```
function multiply(a, b, c) {
    return a * b * c;
}
```

这个函数接受3个数字，将数字相乘并返回结果。

```
multiply(1,2,3); // 6
```

你看，我们如何调用这个具有完整参数的乘法函数。让我们创建一个 `curried` 版本，然后看看在一系列的调用中我们如何调用相同的函数（并且得到相同的结果）：

```
function multiply(a) {
    return (b) => {
        return (c) => {
            return a * b * c
        }
    }
}
log(multiply(1)(2)(3)) // 6
```

我们已经将 `multiply(1,2,3)` 函数调用转换为多个 `multiply(1)(2)(3)` 的多个函数调用。

一个独立的函数已经被转换为一系列函数。为了得到`1`, `2` 和 `3`三个数字想成的结果，这些参数一个接一个传递，每个数字都预先传递给下一个函数以便在内部调用。

我们可以拆分 `multiply(1)(2)(3)` 以便更好的理解它：

```
const mul1 = multiply(1);
const mul2 = mul1(2);
const result = mul2(3);
log(result); // 6
```

让我们依次调用他们。我们传递了`1`给`multiply`函数：

```
let mul1 = multiply(1);
```

它返回这个函数：

```
return (b) => {
        return (c) => {
            return a * b * c
        }
    }
```

现在，`mul1`持有上面这个函数定义，它接受一个参数`b`。

我们调用`mul1`函数，传递`2`：

```
let mul2 = mul1(2);
```

num1会返回第三个参数：

```
return (c) => {
  return a * b * c
}
```

返回的参数现在存储在变量`mul2 `。

`mul2`会变成：

```
mul2 = (c) => {
    return a * b * c
}
```

当传递参数`3`给函数`mul2`并调用它，

```
const result = mul2(3);
```

它和之前传递进来的参数：`a = 1`, `b = 2`做了计算，返回了`6`。

```
log(result); // 6
```

作为嵌套函数，`mul2`可以访问外部函数的变量作用域。

这就是`mul2`能够使用在已经退出的函数中定义的变量做加法运算的原因。尽管这些函数很早就返回了，并且从内存进行了垃圾回收，但是它们的变量仍然保持 `alive`。

你会看到，三个数字一个接一个地应用于函数调用，并且每次都返回一个新函数，直到所有数字都被应用。

让我们看另一个示例：

```
function volume(l,w,h) {
    return l * w * h;
}
const aCylinder = volume(100,20,90) // 180000
```

我们有一个函数`volume`来计算任何一个固体形状的体积。

被柯里化的版本将接受一个参数并且返回一个函数，这个新函数依然会接受一个参数并且返回一个新函数。这个过程会一直持续，直到最后一个参数到达并且返回最后一个函数，最后返回的函数会使用之前接受的参数和最后一个参数进行乘法运算。

```
function volume(l) {
    return (w) => {
        return (h) => {
            return l * w * h
        }
    }
}
const aCylinder = volume(100)(20)(90) // 180000
```

像我们在函数`multiply`一样，最后一个函数只接受参数`h`，但是会使用其他变量来做运算，这些变量的包含作用于早已经返回。由于**闭包**的原因，它们仍然可以工作。

柯里化背后的想法是，接受一个函数并且得到一个函数，这个函数返回**专用**的函数。

## 数学中的柯里化
我比较喜欢数学插图👉[Wikipedia](https://en.m.wikipedia.org/wiki/Currying)，它进一步演示了柯里化的概念。让我们看看我们自己的示例。

假设我们有一个方程式：

```
f(x,y) = x^2 + y = z
```

这里有两个变量 x 和 y 。如果这两个变量被赋值，`x=3`、`y=4`，最后得到 `z` 的值。

:如果我们在方法`f(z,y)`中，给`y` 赋值`4`，给`x`赋值`3`，

```
f(x,y) = f(3,4) = x^2 + y = 3^2 + 4 = 13 = z
```

我们会的到结果，`13`。

我们可以柯里化`f(x,y)`，分离成一系列函数：

```
h = x^2 + y = f(x,y)
hy(x) = x^2 + y = hx(y) = x^2 + y
[hx => w.r.t x] and [hy => w.r.t y]

```
**注意**：**hx**，**x**是 **h** 的下标；**hy**，**y** 是 **h** 的下标。

如果我们在方程式 `hx(y) = x^2 + y` 中设置 `x=3`，它会返回一个新的方程式，这个方程式有一个变量`y`：

```
h3(y) = 3^2 + y = 9 + y
Note: h3 is h subscript 3
```

它和下面是一样的：

```
h3(y) = h(3)(y) = f(3,y) = 3^2 + y = 9 + y
```

这个值并没有被求出来，它返回了一个新的方程式`9 + y`，这个方程式接受另一个变量, `y`。

接下来，我们设置`y=4`：

```
h3(4) = h(3)(4) = f(3,4) = 9 + 4 = 13
```

`y`是这条链中的最后一个变量，加法操作会对它和依然存在的之前的变量`x = 3`做运算并得出结果，`13`。

基本上，我们柯里化这个方程式，将`f(x,y) = 3^2 + y`划分成了一个方程组：

```
3^2 + y -> 9 + y
f(3,y) = h3(y) = 3^2 + y = 9 + y
f(3,y) = 9 + y
f(3,4) = h3(4) = 9 + 4 = 13
```

在最后得到结果之前。

Wow！！这是一些数学问题，如果你觉得不够清晰😕。可以在[Wikipedia](https://en.m.wikipedia.org/wiki/Currying)查看📖完整的细节。 

## 柯里化和部分函数应用

现在，有些人可能开始认为，被柯里化的函数所具有的嵌套函数数量取决于它所依赖的参数个数。是的，这是决定它成为**柯里化**的原因。

我设计了一个被柯里化的求体积的函数：

```
function volume(l) {
    return (w, h) => {
        return l * w * h
    }
}
```

我们可以如下调用L：

```
const hCy = volume(70);
hCy(203,142);
hCy(220,122);
hCy(120,123);
```

或者

```
volume(70)(90,30);
volume(70)(390,320);
volume(70)(940,340);
```

我们定义了一个用于专门计算任何长度的圆柱体体积（`l`）的函数，`70`。

它有3个参数和2个嵌套函数。不像我们之前的版本，有3个参数和3个嵌套函数。

这不是一个柯里化的版本。我们只是做了体积计算函数的部分应用。

柯里化和部分应用是相似的，但是它们是不同的概念。

部分应用将一个函数转换为另一个较小的函数。

```
function acidityRatio(x, y, z) {
    return performOp(x,y,z)
}
|
V
function acidityRatio(x) {
    return (y,z) => {
        return performOp(x,y,z)
    }
}
```

**注意**：我故意忽略了`performOp`函数的实现。在这里，它不是必要的。你只需要知道柯里化和部分应用背后的概念。

这是 acidityRatio 函数的部分应用。这里面不涉及到柯里化。acidityRatio被部分应用化，它期望接受比原始函数更少的参数。

让它变成柯里化，会是这样：

```
function acidityRatio(x) {
    return (y) = > {
        return (z) = > {
            return performOp(x,y,z)
        }
    }
}
```

柯里化根据函数的参数数量创建嵌套函数。每个函数接受一个参数。如果没有参数，那就不是柯里化。

> 柯里化在具有两个参数以上的函数工作 -  [Wikipedia](https://en.m.wikipedia.org/wiki/Currying)

> 柯里化将一个函数转换为一系列只接受单个参数的函数。、

这里有一个柯里化和部分应用相同的例子。假设我们有一个函数：

```
function div(x,y) {
    return x/y;
}
```

如果我们部分应用化这个函数。会得到：

```
function div(x) {
    return (y) => {
        return x/y;
    }
}
```

而且，柯里化会得出相同的结果：

```
function div(x) {
    return (y) => {
        return x/y;
    }
}
```

尽管柯里化和部分应用得出了相同的结果，但是它们是两个完全不同的概念。

像我们之前说的，柯里化和部分应用是相似的，但是实际上定义却不同。它们之间的相同点就是依赖闭包。

## 柯里化有用吗?

当然，只要你想，柯里化就信手拈来：


**1、编写小模块的代码，可以更轻松的重用和配置，就行 npm 做的那样：**

举个例子，你有一个商店🏠，你想给你的顾客 10% 的折扣：

```
function discount(price, discount) {
    return price * discount
}
```

当一个有价值的客户买了一件$500的商品，你会给他：

```
const price = discount(500,0.10); // $50 
// $500  - $50 = $450
```

你会发现从长远来看，我们每天都自己计算10%的折扣。

```
const price = discount(1500,0.10); // $150
// $1,500 - $150 = $1,350
const price = discount(2000,0.10); // $200
// $2,000 - $200 = $1,800
const price = discount(50,0.10); // $5
// $50 - $5 = $45
const price = discount(5000,0.10); // $500
// $5,000 - $500 = $4,500
const price = discount(300,0.10); // $30
// $300 - $30 = $270
```

我们可以柯里化这个折扣函数，这样就不需要每天都添加0.10这个折扣值：

```
function discount(discount) {
    return (price) => {
        return price * discount;
    }
}
const tenPercentDiscount = discount(0.1);
```

现在，我们可以只用你有价值的客户购买的商品价格来进行计算了：

```
tenPercentDiscount(500); // $50
// $500 - $50 = $450
```

再一次，发生了这样的情况，有一些有价值的客户比另一些有价值的客户更重要 -- 我们叫他们超级价值客户。并且我们想给超级价值客户20%的折扣。

我们使用被柯里化的折扣函数：

```
const twentyPercentDiscount = discount(0.2);
```
我们为超级价值客户设置了一个新函数，这个新函数调用了接受折扣值为0.2的柯里化函数。

返回的函数`twentyPercentDiscount`将被用于计算超级价值客户的折扣：

```
twentyPercentDiscount(500); // 100
// $500 - $100 = $400
twentyPercentDiscount(5000); // 1000
// $5,000 - $1,000 = $4,000
twentyPercentDiscount(1000000); // 200000
// $1,000,000 - $200,000 = $600,000
```

**2、避免频繁调用具有相同参数的函数：**

举个例子，我们有一个函数来计算圆柱体的体积：

```
function volume(l, w, h) {
    return l * w * h;
}
```

碰巧，你的仓库所有的圆柱体高度都是 100m。你会发现你会重复调用接受高度为 100 的参数的函数：

```
volume(200,30,100) // 2003000l
volume(32,45,100); //144000l
volume(2322,232,100) // 53870400l
```

为了解决这个问题，需要柯里化这个计算体积的函数（像我们之前做的一样）：

```
function volume(h) {
    return (w) => {
        return (l) => {
            return l * w * h
        }
    }
}
```

我们可以定义一个特定的函数，这个函数用于计算特定的圆柱体高度：

```
const hCylinderHeight = volume(100);
hCylinderHeight(200)(30); // 600,000l
hCylinderHeight(2322)(232); // 53,870,400l
```

## 通用的柯里化函数

让我们开发一个函数，它能接受任何函数并返回一个柯里化版本的函数。

为了做到这一点，我们需要这个（尽管你自己使用的方法和我的不同）：

```
function curry(fn, ...args) {
    return (..._arg) => {
        return fn(...args, ..._arg);
    }
}
```

我们在这里做了什么呢？我们的柯里化函数接受一个我们希望柯里化的函数（fn），还有一系列的参数（...args）。扩展运算符是用来收集`fn`后面的参数到`...args`中。

接下来，我们返回一个函数，这个函数同样将剩余的参数收集为`..._args`。这个函数将`...args`传入原始函数`fn`并调用它，通过使用扩展运算符将`..._args`也作为参数传入，然后，得到的值会返回给用户。

现在我们可以使用我们自己的`curry`函数来创造专用的函数了。

Let’s use our `curry` function to create a more specific function (one that calculates the volume of 100m(length) cylinders) of the volume function:让我们使用自己的`curry`函数来创建更多的专用函数（其中一个就是专门用来计算高度为100m的圆柱体体积的方法）

```
function volume(l,h,w) {
    return l * h * w
}
const hCy = curry(volume,100);
hCy(200,900); // 18000000l
hCy(70,60); // 420000l
```

## 总结
闭包使柯里化在JavaScript中得以实现。它保持着已经执行过的函数的状态，使我们能够创建工厂函数 - 一种我们能够添加特定参数的函数。

要想将你的头脑充满着柯里化、闭包和函数式编程是非常困难的。但我向你保证，花时间并且在日常应用，你会掌握它的诀窍并看到价值😘。


## 参考
👉[Currying — Wikipedia](https://en.m.wikipedia.org/wiki/Currying)
👉[Partial Application Function — Wikipedia](https://en.m.wikipedia.org/wiki/Partial_application)







