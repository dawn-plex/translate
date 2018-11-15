> * 原文地址：[A tour of JavaScript timers on the web](https://nolanlawson.com/2018/09/01/a-tour-of-javascript-timers-on-the-web/)
> * 原文作者：[Nolan Lawson](https://nolanlawson.com/about/)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md](https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md)
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：[也树](https://github.com/xdlrt)、[靖鑫](https://github.com/luckyjing)、[眠云](https://github.com/JeromeYangtao)

---

# Understanding Currying in JavaScript

Functional programming is a style of programming that attempts to pass functions as arguments(callbacks) and return functions without side-effects(changes to the program’s state).

So many languages adopted this programming style. JavaScript, Haskell, Clojure, Erlang, and Scala are the most popular among them.

And with its ability to pass and return functions, it brought so many concepts:

* Pure Functions
* Currying
* Higher-Order functions

And one of the concepts we are going to look at here is **Currying**.

In this article📄, we will see how currying works and how it will be useful in our work as software developers.

**Tip**: instead of copy-pasting reusable JS functionalities you can turn them into components with [Bit](https://bitsrc.io/), and quickly share them across projects with your team.

## What is Currying?

Currying is a process in functional programming in which we can transform a function with multiple arguments into a sequence of nesting functions. It returns a new function that expects the next argument inline.

It keeps returning a new function (that expects the current argument, like we said earlier) until all the arguments are exhausted. The arguments are kept `"alive"`(via closure) and all are used in execution when the final function in the currying chain is returned and executed.

> Currying is the process of turning a function with multiple arity into a function with less arity — Kristina Brainwave

*Note*: The term arity, refers to the number of arguments a function takes. For example,

```
function fn(a, b) {
    //...
}
function _fn(a, b, c) {
    //...
}
```

function `fn` takes two arguments (2-arity function) and `_fn` takes three arguments (3-arity function).

So, currying transforms a function with multiple arguments into a sequence/series of functions each taking a **single argument**.

Let’s look at a simple example:

```
function multiply(a, b, c) {
    return a * b * c;
}
```

This function takes three numbers, multiplies the numbers and returns the result.

```
multiply(1,2,3); // 6
```
See, how we called the multiply function with the arguments in full. Let’s create a `curried` version of the function and see how we would call the same function (and get the same result) in a series of calls:

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

We have turned the `multiply(1,2,3)` function call to `multiply(1)(2)(3)` multiple function calls.

One single function has been turned to a series of functions. To get the result of multiplication of the three numbers `1`, `2` and `3`, the numbers are passed one after the other, each number prefilling the next function inline for invocation.

We could separate this `multiply(1)(2)(3)` to understand it better:

```
const mul1 = multiply(1);
const mul2 = mul1(2);
const result = mul2(3);
log(result); // 6
```

Let’s take it one after the other. We passed `1` to the `multiply` function:

```
let mul1 = multiply(1);
```

It returns the function:

```
return (b) => {
        return (c) => {
            return a * b * c
        }
    }
```

Now, `mul1` holds the above function definition which takes an argument `b`.

We called the `mul1` function, passing in `2`:

```
let mul2 = mul1(2);
```

The mul1 will return the third function:

```
return (c) => {
  return a * b * c
}
```

The returned function is now stored in `mul2` variable.

In essence, `mul2` will be:

```
mul2 = (c) => {
    return a * b * c
}
```

When `mul2` is called with `3` as the parameter,

```
const result = mul2(3);
```

it does the calculation with the previously passed in parameters: `a = 1`, `b = 2` and returns `6`.

```
log(result); // 6
```

Being a nested function, `mul2` has access to the variable scope of the outer functions, `multiply` and `mul1`.

This is how `mul2` could perform the multiplication operation with variables defined in the already `exit`-ed functions. Though the functions have long since returned and `garbage collected` from memory, yet its variables are somehow still kept `"alive"`.

You see that the three numbers were applied one at a time to the function, and at each time, a new function is returned until all the numbers are exhausted.

Let’s look at another example:

```
function volume(l,w,h) {
    return l * w * h;
}
const aCylinder = volume(100,20,90) // 180000l
```

We have a function `volume` that calculates the volume of any solid shape.

The curried version will accept one argument and return a function, which also will accept one argument and return a function. This will loop/continue until the last argument is reached and the last function is returned, which will perform the multiplication operation with the previous arguments and the last argument.

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

Like what we had in the `multiply` function, the last function only accepts `h` but will perform the operation with other variables whose enclosing function scope has long since returned. It works nonetheless because of ***Closure***.

The idea behind currying is to take a function and derive a function that returns **specialized** function(s).

## Currying in Mathematics
I kinda liked the mathematical illustration 👉[Wikipedia](https://en.m.wikipedia.org/wiki/Currying) gave to demonstrate further the concept of currying. Let’s look at it here with our own example.

If we have an equation:

```
f(x,y) = x^2 + y = z
```

There are two variables x and y. If the two variables were given as `x=3` and `y=4`, find the value of `z`.

If we substitute `y` for `4` and `x` for `3` in `f(x,y)`:

```
f(x,y) = f(3,4) = x^2 + y = 3^2 + 4 = 13 = z
```

We get the result, `13`.

We can curry `f(x,y)` to provide the variables in a series of functions:

```
h = x^2 + y = f(x,y)
hy(x) = x^2 + y = hx(y) = x^2 + y
[hx => w.r.t x] and [hy => w.r.t y]

```
**Note**: **hx** is **h** subscript **x** and hy is **h** subscript y. w.r.t is **with respect to**.

If we fix `x=3`in equation `hx(y) = x^2 + y` , it will return a new equation that have `y` as the variable:

```
h3(y) = 3^2 + y = 9 + y
Note: h3 is h subscript 3
```

It is the same as:

```
h3(y) = h(3)(y) = f(3,y) = 3^2 + y = 9 + y
```

The value hasn’t been resolved, it returned a new equation `9 + y` expecting another variable, `y`.

Next, we pass in `y=4`:

```
h3(4) = h(3)(4) = f(3,4) = 9 + 4 = 13
```

`y` being last in the variable chain, The addition op is performed with the previous variable `x = 3` still retained and a value is resolved, `13`.

So basically, we curried the equation `f(x,y) = 3^2 + y` to a sequence of equations:

```
3^2 + y -> 9 + y
f(3,y) = h3(y) = 3^2 + y = 9 + y
f(3,y) = 9 + y
f(3,4) = h3(4) = 9 + 4 = 13
```

before finally getting the result.

Wow!! That’s some math, if you find this not clear enough 😕. You can read📖 the full details on 👉[Wikipedia](https://en.m.wikipedia.org/wiki/Currying).

## Currying and Partial Function Application

Now, some might begin to think that the number of nested functions a curried function has depends on the number of arguments it receives. Yes, that makes it a **curry**.

I can design the curried function of volume to be this:

```
function volume(l) {
    return (w, h) => {
        return l * w * h
    }
}
```

So it can be called like this:

```
const hCy = volume(70);
hCy(203,142);
hCy(220,122);
hCy(120,123);
```

or

```
volume(70)(90,30);
volume(70)(390,320);
volume(70)(940,340);
```

We just defined a specialized function that calculates a volume of any cylinder of length (`l`), `70`.

It expects `3` arguments and has `2` nested functions, unlike our previous version that expects `3` arguments and has `3`nesting functions.

This version isn’t a curry. We just did a partial application of the volume function.

Currying and Partial Application are related, but they are of different concepts.

Partial application transforms a function into another function with smaller arity.

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

**Note**: I purposely left out the implementation of the `performOp` function. Here, it isn't necessary. All you have to know is the concept behind currying and partial application.

This is the partial application of the acidityRatio function. This is no currying involved here. The acidityRatio function was partially applied to receive less arity, to expect `less argument` than its original function.

To make it be currying, it would be like this:

```
function acidityRatio(x) {
    return (y) = > {
        return (z) = > {
            return performOp(x,y,z)
        }
    }
}
```

Currying creates nesting functions according to the number of the arguments of the function. Each function receives an argument. If there is no argument there is no currying.

> Currying works for functions with more than two arguments — [Wikipedia](https://en.m.wikipedia.org/wiki/Currying)

> Currying transforms a function into a sequence of functions each taking a single argument of the function.

There might be a case whereby currying and partial application kinda meet👬 each other. Let’s say we have a function:

```
function div(x,y) {
    return x/y;
}
```

If we partially apply it. We will get:

```
function div(x) {
    return (y) => {
        return x/y;
    }
}
```

Also, currying will give us the same result:

```
function div(x) {
    return (y) => {
        return x/y;
    }
}
```

Though currying and partial function gave the same result, they are two different entities.

Like we said earlier, currying and partial application are related, but not actually the same by design. the common thing between them is that they depend on closure to work.

## Is Currying Useful?

Of course, currying comes in handy when you want to:

**1. Write little code modules that can be reused and configured with ease, much like what we do with npm:**

For example, you own a store🏠 and you want to give 10%💵 discount to your fav customers:

```
function discount(price, discount) {
    return price * discount
}
```

When a fav customer buys a good worth of $500, you give him:

```
const price = discount(500,0.10); // $50 
// $500  - $50 = $450
```

You see that in the long run, we would find ourselves calculating discount with 10% on a daily basis.

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

We can curry the discount function, so we don’t always add the 0.10 discount:

```
function discount(discount) {
    return (price) => {
        return price * discount;
    }
}
const tenPercentDiscount = discount(0.1);
```

Now, we can now calculate only with price of the goods bought by your fav customers:

```
tenPercentDiscount(500); // $50
// $500 - $50 = $450
```

Again, it happens that, some fav customers are more important than some fav customers- let’s call them super-fav customers. And we want to give 20% discount to our super-fav customers.

We use our curried discount function:

```
const twentyPercentDiscount = discount(0.2);
```

We setup a new function for our super-fav customers by calling the curry function discount with a `0.2` value , that is `20%`.

The returned function `twentyPercentDiscount` will be used to calculate discounts for our super-fav customers:

```
twentyPercentDiscount(500); // 100
// $500 - $100 = $400
twentyPercentDiscount(5000); // 1000
// $5,000 - $1,000 = $4,000
twentyPercentDiscount(1000000); // 200000
// $1,000,000 - $200,000 = $600,000
```

**2. Avoid frequently calling a function with the same argument:**

For example, we have a function to calculate the volume of a cylinder:

```
function volume(l, w, h) {
    return l * w * h;
}
```

It happens that all the cylinders in your warehouse🏠 are of height 100m. You will see that you will repeatedly call this function with `h` as `100`:

```
volume(200,30,100) // 2003000l
volume(32,45,100); //144000l
volume(2322,232,100) // 53870400l
```

To resolve this, you curry the volume function(like we did earlier):

```
function volume(h) {
    return (w) => {
        return (l) => {
            return l * w * h
        }
    }
}
```

We can define a specific function for a particular cylinder height:

```
const hCylinderHeight = volume(100);
hCylinderHeight(200)(30); // 600,000l
hCylinderHeight(2322)(232); // 53,870,400l
```

## General Curry Function
Let’s develop a function that takes any function and returns a curried version of the function.

To do that we will have this(though you own approach could be different from mine):

```
function curry(fn, ...args) {
    return (..._arg) => {
        return fn(...args, ..._arg);
    }
}
```

What did we do here? Our curry function accepts a function (fn) that we want to curry and a variable number of parameters(…args). The rest operator is used to gather the number of parameters after `fn` into ...args.

Next, we return a function that also collects the rest of the parameters as …_args. This function invokes the original function fn passing in `...args` and `..._args` through the use of the spread operator as parameters, then, the value is returned to the user.

We can now use our own `curry` function to create specific functions.

Let’s use our `curry` function to create a more specific function (one that calculates the volume of 100m(length) cylinders) of the volume function:

```
function volume(l,h,w) {
    return l * h * w
}
const hCy = curry(volume,100);
hCy(200,900); // 18000000l
hCy(70,60); // 420000l
```

## Conclusion
Closure makes currying possible in JavaScript. It’s ability to retain the state of functions already executed, gives us the ability to create factory🏭 functions — functions that can add a specific value to their argument.

It is quite tricky to wrap your head around currying, closures and functional programming. But I assure you with time⌚ and constant practice🏪, you will start to get the hang of it and see how worthwhile it is 😘.

If you have any questions regarding this post or anything I should add, correct or remove, feel free to comment📝, email📮 or DM me💬. Thanks !!!😀

## References
👉[Currying — Wikipedia](https://en.m.wikipedia.org/wiki/Currying)
👉[Partial Application Function — Wikipedia](https://en.m.wikipedia.org/wiki/Partial_application)








