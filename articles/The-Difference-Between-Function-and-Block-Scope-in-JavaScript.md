

>- 原文地址：[The Difference Between Function and Block Scope in JavaScript](https://medium.com/@josephcardillo/the-difference-between-function-and-block-scope-in-javascript-4296b2322abe)
>- 原文作者：[Joseph Cardillo](https://medium.com/@josephcardillo)
>- 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
>- 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md](https://github.com/dawn-teams/translate/blob/master/articles/The-Difference-Between-Function-and-Block-Scope-in-JavaScript.md)
>- 译者：[眠云](https://github.com/JeromeYangtao)
>- 校对者：

---

# The Difference Between Function and Block Scope in JavaScript

Back to the basics with the var, let and const variables



![](https://s1.ax1x.com/2018/09/22/iunxXD.jpg)



When I mentioned to someone recently that I’ve been wanting to get back to the basics with JavaScript, they recommended [Wes Bos](https://medium.com/@wesbos)’s “ES6 For Everyone” course. Wes is a great teacher and a clear communicator with a gift for distilling complex topics.



I started working through it today to help me nail down the basics, and begin my journey deeper into the chasms of JavaScript.



### The difference between var, let and const variables

There are three ways to declare variables in JS:



```js
var width = 100;
let height = 200;
const key = 'abc123';
```



### var

What is unique about `var`? It can be reassigned and updated. For example:



```js
// Define the variable:
var width = 100;

// Call the variable:
width;
// It returns:
100
// Reassign the variable and call it again:
width = 200;

width;
// Returns:
200
```

`var` variables are ‘function scope.’ What does this mean? It means they are only available *inside* the function they’re created in, or if not created inside a function, they are ‘globally scoped.’



If `var` is defined inside a function and I subsequently try to call it outside the function, it won’t work.



For example:



```js
function setWidth(){
    var width = 100;
    console.log(width);
}
width;
// Returns:
Uncaught ReferenceError: width is not defined
```



Again, because `var` is defined within the function, it is not available outside the function.



### Understanding scope



```js
Example: 
var age = 100;
if (age > 12){
  var dogYears = age * 7;
  console.log(`You are ${dogYears} dog years old!`);
}
```



In the example above, the console returns:



```js
100
You are 700 dog years old!
```



But if I call `dogYears` in the console in returns:



```js
dogYears;
// returns:
700
```



So what happened? Why is `dogYears` available at the global scope?



Because `var` variables are function scoped — and because `var dogYears` is *not* defined within a function in the above example — `var` is instead defined at the ‘window,’ or global scope.



In other words, `var` is not limited to the curly brackets. It is the function which defines the scope.



#### Benefits of using let and const

What are the benefits of using `let` and `const`? Rather than being scoped to the *function* they are scoped to the *block*.



What is the block? A block is a set of opening and closing curly brackets.



So in the example above, if we change `var dogYears` to `let dogYears`, then call `dogYears` in the console, it returns `Uncaught ReferenceError: dogYears is not defined`.



```js
var age = 100;
if (age > 12){
  let dogYears = age * 7;
  console.log(`You are ${dogYears} dog years old!`);
 }
```



We could also substitute `const` for `let` in the example above.



### Recap



```
var is function scope.
let and const are block scope.

Function scope is within the function.
Block scope is within curly brackets.

```





